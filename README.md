## Contents
- [Basic Integration](#basic-integration)
    - [Installation](#installation)
    - [Start Session](#start-session)
    - [In-App Messaging](#in-app-messaging)
    - [Push Messaging](#push-messaging)
    - [Test Device Registration](#test-device-registration)
- [IAP & Reward](#iap--reward)
  - [In-App Purchase Tracking (Beta)](#in-app-purchase-tracking-beta)
  - [In-App-Purchase Count](#in-app-purchase-count)
  - [Give Reward](#give-reward)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [AdFrescaViewDelegate](#adfrescaviewdelegate) 
  - [Timeout Interval](#timeout-interval) 
- [Reference](#reference)
  - [Custom URL Schema](#custom-url-schema)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [IFV Only Option](#ifv-only-option)
- [Trouble Shooting](#trouble-shooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

 Download SDK on the following link:

[iOS SDK Download](http://file.adfresca.com/distribution/sdk-for-iOS.zip) (v1.3.5)

[iOS SDK with IAP Tracking BETA Download](https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/distribution/sdk-for-iOS-iap-beta.zip) (v1.4.0-beta1)

To add SDK into your Xcode project, please follow the instructions below:

1. Drag & Drop AdFresca folder into the framework folder on your Xcode project.

  <img src="https://adfresca.zendesk.com/attachments/token/4uzya7c9rw4twus/?name=Screen+Shot+2013-03-27+at+8.22.04+PM.png" width="600" />

2. Add System Configuration.framework and AdSupport.framework, StoreKit.framework into your target if these frameworks are not added yet.
  
  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />
  
  - If you add AdSupport.framework, SDK collects [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) value to distnigush user's device. We use this value to provide cross-promotion campaign with install and action traciing.
  - If you do not AdSupport.framework, SDK uses [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) value to distnigush user's device. In this case, you can't use any cross promotion feature. Also, as IFV's policy, your user may be recognized as a new user after re-installing app.

  If you'd like to add AdSupport.framework or remove the framework from exsisting xcode project with our SDK, please refer [IFV Only Option](#ifv-only-option) section to migrate your users.

3. Add -ObjC to Other Linker Flag on your target's build setting.

  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />

4. In Info.plst, set 'aps-environment' value as 'production'. It is necessary to use a push notification feature.

  <img src="https://adfresca.zendesk.com/attachments/token/bd7oz41zoh5zjs4/?name=Screen+Shot+2013-02-07+at+5.22.50+PM.png" width="600" />

  Also, set your own URL Scheme value. the example below shows how to set URL Scheme with "myapp" value. It will be used in cross promotion feature.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

AD fresca SDK has been successfully installed without any build error. If you have 'Duplicate Symbol' error, please refer '[Trouble Shooting](#trouble-shooting)' section.

### Start Session

Now, start to put some simple SDK codes in your app. You firstly need to call startSession() method with your API Key. To get your API Key, go to our [Dashboard](https://admin.adfresca.com) and then click 'Settings - API Keys' button in your app's 'Overview' page.

startSession() will start to detect when user starts app and resumes from the background.

```objective-c
// AppDelegate.m
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  ....
} 
```

### In-App Messaging

With in-app messaging feature, you can deliver your message to your in-app users in real time. Simply put 'loadAd' and 'showAd' methods where you want to deliver message. The type of message can be an interstitial image, text, and iframe webpage. The message is only shown when your user matches in-app messaging campaign's target logics. We will discuss more details of in-app messaging's dynamic targeting features in [Dynamic Targeting](#dynamic-targeting) section.

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView sharedAdView]; 
  [fresca loadAd]; 
  [fresca showAd]; 
} 
```

When you firstly call in-app messaging methods, you will see the test message as below. If you tap on the image, it will redirect to the product page of the app on the app store. You will hide this test message by chagning test mode configuration later.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

You can also deliver your push messages anytime you want. Follow the steps below to configure push notificaiton settings in your app.

1. Upload your APNS Certificate file (.p12) to our Dashboard
  - You can export your .cer file to .p12 file using Keychain. Please refer [iOS Push Notification Certificate Guide](https://adfresca.zendesk.com/entries/21714780) to generate .p12 and upload to [Dashboard](https://admin.adfresca.com)

2. Check your provisioning
  - AD fresca only support production environment of APNS. So, you should build your app with App Store or Ad Hoc Provisioning file to enable production mode

3. Add some codes to AppDelegate 
  ```objective-c
  #import <AdFresca/AdFrescaView.h>

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ....
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];   // Push Notification 기능을 이용할 경우 등록.      
  } 

  - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    // Register user's push device token to our SDK
    [AdFrescaView registerDeviceToken:deviceToken];
  }

  - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    /// Check a push notification is form AD fresca. Also, ignore a notification received when app is already running 
    if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
      [AdFrescaView handlePushNotification:userInfo];
    }  
  } 
  ```

### Test Device Registration

AD fresca supports a test mode feature. With a test mode feature, you can deliver your test message to only registred test devices. 

To register your test device to our dashboard, you need to know your test device ID from our SDK. SDK provides two ways to show test device ID.
 
1. Using testDeviceId Property
  - After connecting your device with Xcode, you can simply print out test device ID with a logger.

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  NSLog(@"AD fresca Test Device ID = %@", fresca.testDeviceId); 
  [fresca loadAd];
  [fresca showAd];
```

2. Displaying test device ID on your app screen using printTestDeviceId property
  - When you are not able to connect tester's device in your office, you have to set printTestDeviceId to true, and then let them install this app build. They can see their own test device ID on the app screen. 
  - It is useful when testers are working on the remote. 
  - printTestDeviceId property must be set to false when you distribute your app on the store. 

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  fresca.printTestDeviceId = YES;
  [fresca loadAd];
  [fresca showAd];
  ```

After you have your test device ID, you have to register it to [Dashboard](https://admin.adfresca.com). Please refer to [Test Device & Test Mode Management Guide](https://adfresca.zendesk.com/entries/21921047)

* * *
