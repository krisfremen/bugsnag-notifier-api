Bugsnag Notifier API
-----------

The Bugsnag Notifier API is used to notify Bugsnag of an error or exception in an application. [Official libraries](https://github.com/organizations/bugsnag) are available
in several languages.

If there is no plugin available in the language you are using, then why not write one yourself? We will gladly feature your plugin within Bugsnag when someone
creates an appropriate project.

Notification Methods
--------------------

Bugsnag uses a simple JSON based API to notify the service of an error. Simply POST the JSON to [http://notify.bugsnag.com](http://notify.bugsnag.com) and Bugsnag will 
process the error.

A plugin can notify Bugsnag of an error at [http://notify.bugsnag.com](http://notify.bugsnag.com) using http. The plugin should also be capable of notifying Bugsnag using SSL
at [https://notify.bugsnag.com](https://notify.bugsnag.com).

JSON Payload
--------------------

Here is the standard JSON payload for a notice to Bugsnag that an error has occurred in an application.

```javascript
{
    // The API Key associated with the project. Informs Bugsnag which project has generated this error.
    apiKey: "c9d60ae4c7e70c4b6c4ebd3e8056d2b8",

    // This object describes the notifier itself. These properties are used within Bugsnag to track error 
    // rates from a notifier.
    notifier: {
        
        // The notifiers name. Used internally within Bugsnag to track error rates from notifiers.
        name: "Bugsnag Ruby",

        // The notifiers current version. Used internally within Bugsnag to track error rates from notifiers.
        version: "1.0.11",

        // The URL associated with the notifier.
        url: "https://github.com/bugsnag/bugsnag-ruby"
    },

    // An array of events that have occurred and Bugsnag should be notified of. A notifier can choose to group 
    // notices into an array to minimize network traffic, or can notify Bugsnag each time an event occurs. Each 
    // individual event within the array is subject to the projects throttling controls, so it is possible that not
    // all of the events included in the array will be processed successfully, if some of the events cause
    // the project to hit its rate limit.
    events: [{
        
        // A unique identifier for a user affected by this event. This could be any distinct identifier
        // that makes sense for your application/platform. (optional, default none)
        userId: "snmaynard",

        // The version number of the application which generated the error. (optional, default none)
        appVersion: "1.1.3",

        // The release stage that this error occurred in. This can be any
        // value, but "production" will be highlighted differently in
        // bugsnag in the future, so please use "production" appropriately.
        releaseStage: "production",

        // A string representing what was happening in the application at the time of the error. 
        // This string could be used for grouping purposes, depending on the event itself.
        // Usually this would represent the controller and action in a server based project. 
        // It could represent the screen that the user was interacting with in a client side project.
        // For example,
        //   * On Ruby on Rails the context could be controller#action
        //   * In Android, the context could be the top most Activity.
        //   * In iOS, the context could be the name of the top most UIViewController.
        context: "auth/session#create",

        // An array of exceptions that occurred during this event. Most of the time there will 
        // only be one exception, but some languages support wrapping exceptions
        // in other exceptions, which can pollute the stacktrace. The exceptions should be 
        // unwrapped and put into the array one at a time. The first exception raised
        // should be first in this array.
        exceptions: [{
    
            // The class of error that occurred. This field is used to group the errors together 
            // so should not contain any contextual information that would prevent correct grouping. 
            // This would ordinarily be the Exception name when dealing with an exception.
            errorClass: "NoMethodError",
    
            // The error message associated with the error. Usually this will contain some information
            // about this specific instance of the error and is not used to group the errors (optional, default none).
            errorMessage: "Unable to connect to database.",
    
            // An array of stackframe objects. Tells Bugsnag what was happening within the application at
            // the time of the error and is shown to the developer, as well
            // as being used to group the error.
            stacktrace: [{
                
                // The file that this stack frame was executing.
                file: "controllers/auth/session_controller.rb",
        
                // The line of the file that this frame of the stack was in.
                lineNumber: 1234,
        
                // The method that this particular stack frame is within.
                method: "create",
        
                // If the stack frame is in the users project set this to true (optional, default false).
                // It is useful for developers to be able to see which lines of a stacktrace are within their own
                // application, and which are within third party libraries. This boolean field allows Bugsnag 
                // to display this information in the stacktrace as well as use the
                // information to help group errors better.
                inProject: true
            }]
        }],

        // A hash containing any further data you wish to accompany the payload. 
        // This ordinarily will contain multiple hashes, with each hash included being
        // displayed as a tab within the event details on the Bugsnag website (optional).
        metaData: {
            
            // This will displayed as the first tab after the stacktrace on the Bugsnag website.
            someData: {

                // A key value pair that will be displayed in the first tab
                key: "value",

                // This is shown as a section within the first tab
                setOfKeys: {
                    key: "value",
                    key2: "value"
                }
            },
            
            // This would be the second tab on the Bugsnag website.
            someMoreData: {
                ...
            }
        }
    }]
}
```

Bugsnag Libraries
---------------------------
When writing a library for other Bugsnag users, it is important to try and get a consistent interface across various platforms, so that developers get used
to the Bugsnag interface.

### Configuration
On startup, a Bugsnag notifier should request the following configuration values from the developer.

- **API Key** - The apiKey for the project.
- **Release Stage** - The release stage for the current deployment. Most platforms have a sensible way of obtaining this, and this should be used if possible.
- **Notify Release Stages** - A list of stages that notifications should succeed for. If the current release stage is not in this list, any notify should be blocked by the client.
This should default to notifying for the "production" release stage only.
- **Auto Notify** - If this is true, the plugin should notify Bugsnag of any uncaught exceptions. This should default to true.
- **Use SSL** - If this is true, the plugin should notify Bugsnag using the SSL endpoint. This should default to false.
- **Endpoint** - This should default to notify.bugsnag.com and informs the plugin where to post notifications to.

When running a Bugsnag notifier plugin should provide the following run time properties.

- **User ID** - The current user using the application. This should use a sensible default. Some platforms, especially web platforms, provide a mechanism
for ascertaining who the current user is, and if this is available the plugin should use it.
- **Extra Data** - Any extra data that will be sent as meta data with the request. Plugins should configure sensible defaults for meta data this based on 
their own platform. Users may provide a lambda function or equivalent to supplement this metadata when an error occurs.
- **Context** - Set the context that is currently active in the application. This should default to a sensible approximation if the platform allows.

Response Codes
---------------------------
If the payload could not be processed properly, Bugsnag will respond with a response code that indicates what the error was. It will also
include a JSON payload containing an error string. The response codes can be any of,

- **400 Bad Request** - The payload failed syntax validation and was not processed.
- **401 Unauthorized** - Indicates that the API Key was incorrect.
- **413 Request Too Large** - Indicates that the payload was too large to be processed.
- **429 Too Many Requests** - Indicates that the payload was not processed due to rate limiting.