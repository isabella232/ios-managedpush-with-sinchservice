#Introducing SinchService

At Sinch, we've been fortunate to not only provide an intuitive API, but also contribute to the open source community around it. Part of that initiative is the SinchService for iOS. With it, we've abstracted a lot of the heavy lifting and common functionality away for you so you can get started with Sinch as fast as possible.

In this tutorial, we'll take a look at the SinchService and talk about some best practices and how to use it.

##Getting Started
To get started with this tutorial, be sure to download the starter project [here](https://www.sinch.com/downloads/#downloads-ios). The SDK comes packaged with a few sample projects, but the one we are interested in is called "SinchCallingPushWithService". Note that if you already have a project using the Sinch SDK, you can use that as well. 

####CocoaPods
Next, install the SinchService via [CocoaPods](https://cocoapods.org/). If you are unfamiliar with CocoaPods, don't worry. You can get everything set up by following these steps:

- **If you've never installed Cocoapods**, open terminal and first update RubyGems to ensure you have the latest version using this command: "sudo gem update --system". 
- Next, install Cocoapods: "sudo gem install cocoapods".
- Now, navigate to your project's folder. If you are using the demo app and didn't move it from the downloads folder, you can copy this command verbatim:"cd downloads/Sinch/samples".
- We've included a pod file for you already, so just run "pod install" and you'll be all set.

You can also add more libraries to the pod file as well. Once you've navigated to where the pod file is, use this command: "open -a Xcode Podfile".

####Push Notification Certificates
No matter if you are using the sample app or one of your own, you'll need to have push notifications enabled. If you haven't done this before, you'll want to follow the guide created by Apple [here](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/ConfiguringPushNotifications/ConfiguringPushNotifications.html).

Once you've generated the certificates, upload them to your Sinch dashboard for the application you're working with. All three push notifications certificate types are supported:

![Certificates](ssKeys.png)

At this point, you're all set to start using Sinch and push notifications. Let's take a look at some of the things SinchService can do for us.

##SinchService - Why use it?
At its core, SinchService acts as a supplement to the Sinch API. If you are familiar with our API already, it specifically bolsters both SinchRTC and SINClient.

For example, the SINClient is the main entry point to interact with our API. When your app leverages several of our features, you may have a class that conforms to several protocols. 

Say your app is using our API to enable calling and push notifications. If you implement your client on the AppDelegate as we have in several code samples, your implementation file may look something like this:

    @interface AppDelegate () <SINClientDelegate, SINCallClientDelegate, SINManagedPushDelegate>

While that certainly works, SinchService makes this more concise. There are essentially three main responsibilities here, and right now they are going to be spread across the AppDelegate. In addition, to properly support push notifications an instance of SINManagedPush is necessary as well:

    @property (nonatomic, readwrite, strong) id<SINManagedPush> push;

SinchService can take care of a lot of this setup for you.

## Initializing SinchService
Getting SinchService up and running is easy. It's initializer looks very similar to the SINClient but differs in that we'll want to do some configuration upfront. Once we've set our configuration options, we'll call the initializer to get our instance of SinchService ready to go:

    id config = [[SinchService configWithApplicationKey:@"<YOUR_APP_KEY>" applicationSecret:@"<YOUR_APP_SECRET>"                                 environmentHost:@"sandbox.sinch.com"]        pushNotificationsWithEnvironment:SINAPSEnvironmentAutomatic];
    
    id<SINService> sinch = [SinchService serviceWithConfig:config];


At this point, you've already offloaded the work SINClient would normally do over to the SinchService instance. Plus, using the SINAPSEnvironmentAuotmatic macro, we don't have to worry about accidentally supplying the incorrect device token to the Apple Push Notification Service Environment. It'll detect whether the environment is debug or production and take the right action automatically.

##Sinch Service Delegate
The SinchService instance should have a delegate assigned to it. One helpful delegate method to implement is `-(void)service:(id<SINService>)service didFailWithError:(NSError *)error`. This method will be fired if something goes wrong when getting the SinchService going or communicating with the API.

Once you set its delegate, you are able to do all the things you can with the SINClient object. If we refer to our example before, you'll see there is one less protocol to conform to:

    @interface AppDelegate () <SINServiceDelegate, SINCallClientDelegate>

This makes your code easier to follow and more concise. You no longer need the SINManagedPush protocol or the SINManagedPush object. SinchService houses all of these together.

##Calling, Push Notifications, and More
One of the great things about SinchService is that it can handle any of Sinch's core functions. In iOS, that means making and receiving calls, sending and receiving push notifications, messaging, and more. You still use the same protocols you are used to if you've worked with our API before.

If you haven't though, don't worry. I'll provide a brief overview of those functions in action using SinchService. Let's take a look at making calls to start things off.

####Making Calls
Two important things you'll want to know about when dealing with calls using Sinch are the **SINCall** object and **SINCallDelegate**. In addition, there are two delegate methods you can implement in the `SINCallClientDelegate` protocol to know when a call finished or a notification was received for one.

First, we'll talk about SINCall. SINCall is responsible for controlling the flow of the calls. With it, you can do the following actions:

- Answer calls
- Decline calls
- Hang up a call

The methods for doing these actions are named appropriately (i.e. answer and dismiss). Typically, you'll use the `dismiss` method to either hang up a call or decline an incoming call. 

Updating your user interface makes the call session feel natural and understandable to the user. We have some helpful delegate methods that can be implemented to take action during the lifetime of a call.

For example, the user will need to know when a call has started, ended, or is ringing. These delegate methods will let you know when any of these actions has occurred:

- `(void)callDidProgress:(id<SINCall>)call`
- `(void)callDidEstablish:(id<SINCall>)call`
- `(void)callDidEnd:(id<SINCall>)call`

You could even create a simple timer so that the user will know how long the call has progressed. Using the `-(void)callDidEstablish:(id<SINCall>)call` function, simply start the timer and update the user interface. We have a working sample of this in the demo app that be found in the `CallViewController`.

To see a hands on example of handling calls with Sinch, be sure to take a look at our SinchCalling project. It can be found inside the samples folder that's included in the Sinch API download. Calling is a core feature of our API and getting started is as easy as calling `callUserWithId:`!

####Push Notifications
The SINManagedPush object works to simplify the process of working with push notifications. To work with notifications, you'll first need to acquire a push notification token and then register it with the Sinch client. Using Sinch Service, we can do all of that.

Keep in mind that we'll want to create an instance of SINManagedPush as early as possible in the application lifecycle. Normally, that code would look something like this  inside `application:DidFinishLaunchingWithOptions`:

    self.push = [Sinch managedPushWithAPSEnvironment:SINAPSEnvironmentAutomatic];
    self.push.delegate = self;
    [self.push setDesiredPushTypeAutomatically];
    [self.push registerUserNotificationSettings];

The SINManagedPush does quite a bit of work for us. It simplifies the iOS SDK API differences between iOS 7 and iOS 8 as well as differences for regular remote push notifications and VoIP push notifications. So, four lines of code is not much of a burden.

Even so, SinchService makes this even easier. When you initialize SinchService, we saw earlier that you do so with an initial configuration. To refresh your memory, it looked like this:

    id config = [[SinchService configWithApplicationKey:@"<YOUR_APP_KEY>" applicationSecret:@"<YOUR_APP_SECRET>"                                 environmentHost:@"sandbox.sinch.com"]        pushNotificationsWithEnvironment:SINAPSEnvironmentAutomatic];

Using SinchService, it has has taken care of all of the SINManagedPush work for us already. All that we need to do is give it the appropriate environment. Now, you can interact with SINManagedPush easily since it lives as a property on your SinchService instance. You can manage incoming notifications, receiving a token, and more using SinchService.

Remember, SinchService and SINManagedPush can handle remote, local, and VoiP notifications.


##Wrapping Up
Hopefully you'll get some good use out of SInchService. It's been tuned to make your experience with our API in iOS as seamless as possible. It abstracts a lot of what SINClient does and makes working with SINManagedPush easier as well. We hope you continue to build amazing apps with Sinch and feel free to reach out to us if you have any questions.
