## Contents
- [Introduction](#introduction)
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
- [Troubleshooting](#trouble-shooting)
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
  
  - If you add AdSupport.framework, SDK collects [IFA(Identifier For Advertisers)](https://developer.apple.com/library/ios/documentation/AdSupport/Reference/ASIdentifierManager_Ref/ASIdentifierManager.html#jumpTo_3) value to distinguish the user's device. We use this value to provide the cross-promotion campaign with install and action tracking.
  - If you do not add AdSupport.framework, SDK uses [IFV(Identifier For Vendor)](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDevice_Class/Reference/UIDevice.html#jumpTo_7) value to distinguish user's device. In this case, you can't use any cross promotion feature. Also, as IFV's policy, your user may be recognized as a new user after re-installing app.

  If you'd like to add AdSupport.framework or remove the framework from existing xcode project with our SDK, please refer to the [IFV Only Option](#ifv-only-option) section to migrate your users.

3. Add -ObjC to Other Linker Flag on your target's build setting.

  <img src="https://adfresca.zendesk.com/attachments/token/rny0s0zm3modful/?name=2Untitled.png" width="600" />

4. In Info.plst, set 'aps-environment' value as 'production'. It is necessary to use a push notification feature.

  <img src="https://adfresca.zendesk.com/attachments/token/bd7oz41zoh5zjs4/?name=Screen+Shot+2013-02-07+at+5.22.50+PM.png" width="600" />

  Also, set your own URL Scheme value. the example below shows how to set URL Scheme with "myapp" value. It will be used in the cross promotion feature.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

AD fresca SDK has been successfully installed without any build error. If you have a 'Duplicate Symbol' error, please refer to the [Troubleshooting](#trouble-shooting) section.

### Start Session

Now, start to put some simple SDK codes in your app. You first need to call startSession() method with your API Key. To get your API Key, go to our [Dashboard](https://admin.adfresca.com) and then click 'Settings - API Keys' button in your app's 'Overview' page.

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

With the in-app messaging feature, you can deliver a message to your in-app users in real time. Simply put 'loadAd' and 'showAd' methods where you want to deliver a message. The type of message can be an interstitial image, text, and iframe webpage. The message is only shown when your user matches the in-app messaging campaign's target logics. We will discuss more details of the in-app messaging's dynamic targeting features in the [Dynamic Targeting](#dynamic-targeting) section.

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application {
  AdFrescaView *fresca = [AdFrescaView sharedAdView]; 
  [fresca loadAd]; 
  [fresca showAd]; 
} 
```

When you first call in-app messaging methods, you will see the test message below. If you tap on the image, it will redirect to the product page of the app on the app store. You will hide this test message by chagning the test mode configuration later.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

You can also deliver your push messages anytime you want. Follow the steps below to configure the push notification settings in your app.

1. Upload your APNS Certificate file (.p12) to our Dashboard
  - You can export your .cer file to .p12 file using Keychain. Please refer [iOS Push Notification Certificate Guide](https://adfresca.zendesk.com/entries/21714780) to generate .p12 and upload to [Dashboard](https://admin.adfresca.com)

2. Check your provisioning
  - AD fresca only supports APNS production environment. So, you should build your app with App Store or Ad Hoc Provisioning file to enable production mode

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

AD fresca supports a test mode feature. With the test mode feature, you can deliver your test message to only registred test devices. 

To register your test device to our dashboard, you need to know your test device ID from our SDK. SDK provodies two ways to show test device ID.
 
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
  - It is useful when testers are working remotely. 
  - printTestDeviceId property must be set to false when you distribute your app on the store. 

  ```objective-c
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  fresca.printTestDeviceId = YES;
  [fresca loadAd];
  [fresca showAd];
  ```

After you have your test device ID, you have to register it to [Dashboard](https://admin.adfresca.com). You can register your device in the 'Test Device' menu.

* * *

## IAP & Reward

### In-App Purchase Tracking (Beta)

_**(In-App-Purchase Tracking feature is only available in v1.4.0-beta)**_

With In-App-Purchase Tracking , you can analyze all the purchases of your users, and use it for targeting specific user segment to display your campaigns. (targeting feature is coming soon)

There are two types of purchases you can track with our SDK.

1. **Actual Item Purchase Tracking:**  the purchases made by real money. For example, user purchased 'USD 1.99' to get 'Gold 100' cash item.
2. **Virtual Item Purchase Tracking:** the purchases made by virtual money. For example, user purchased 'Gold 10' to get 'Rocket Launcher' item 

You don't need to write down any item list manually. All the Items tracked by SDK are automatically added to our dashboard. To see the list of item, go to 'Overview > Settings > In-App Items' page in our dashboard.

Let's get started to implement SDK codes with examples below. 

#### Actual Item Tracking

In iOS, the purchase of 'Actual Item' is made with Apple's Storekit framework. When your user purchased the item successfully, simply create AFPurchase object and use logPurchase() method.

```objective-c
- (void)completeTransaction:(SKPaymentTransaction *)transaction
{
  // productInfo is NSMutableDictionary object to store fetched SKProduct object with its productIdentifier as a key.
  SKProduct *product = [productInfo objectForKey:transaction.payment.productIdentifier];

  NSString *itemId = transaction.payment.productIdentifier;
  NSString *currencyCode = [product.priceLocale objectForKey:NSLocaleCurrencyCode];
  NSNumber *price = product.price;
  NSDate *transactionDate = transaction.transactionDate;
  NSData *transactionReceiptData = transaction.transactionReceipt;

  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeActualItem
                                                    itemId:itemId
                                              currencyCode:currencyCode
                                                     price:[price doubleValue]
                                              purchaseDate:transactionDate
                                    transactionReceiptData:transactionReceiptData];

  [[AdFrescaView shardAdView] logPurchase:purchase];
}
```

For more details of AFPurchase object with the actual item , check the table below.

Method | Description
------------ | ------------- | ------------
itemId(string) | Set the unique identifier of your item. This value is may not be different per the os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service distinguish each item by this value.
currencyCode(string) | Set the current code of IOS 4217 standard. You may use SKProduct's's value or manually set the value form your server.
price(double) | Set the item price. you may use SKProduct's value or manually set the value form your server.
purchaseDate(date) | Set the date of purchase. You may use SKPaymentTransaction.transactionDate value. If you set nil value, it will be automatically recorded by our SDK and server. Please don't use local time of user's device.
transactionReceiptData(nsdata) | Set the receipt property of SKPaymentTransaction object. We will use it to verify the receipt in the future.

#### Virtual Item Tracking

When users purchased your virtual item in the app, you can also create AFPurchase object and call logPurchase() method.

```objective-c
- (void)didPurchaseVirtualItem {
  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeVirtualItem
                                                    itemId:@"gun_001"
                                              currencyCode:@"gold"
                                                     price:100
                                              purchaseDate:nil
                                    transactionReceiptData:nil]; 

  [[AdFrescaView shardAdView] logPurchase:purchase];
}
```

For more details of AFPurchase object with the virtual item, check the table below.. You don't need to set transactionReceiptData property in the virtual item tracking.

Method | Description
------------ | ------------- | ------------
itemId(string) | Set the unique identifier of your item. This value is may not be different per the os platform or app store. We recommend that you make this value unique for all platforms and stores. Our service distinguish each item by this value.
currencyCode(string) | Set the item's virtual currency code. (ex: 'gold', 'gas')
price(double) | Set the item price. You may get this value form your server. (ex: 100 of gold)
purchaseDate(date) | Set the date of purchase. If you set nil value, it will be automatically recorded by our SDK and server. Please don't use local time of user's device.
transactionReceiptData(nsdata) | Set nil for AFPurchaseTypeVirtualItem

#### IAP Trouble Shooting

After you call logPurchase() method, the purchase data is updated to our dashboard in real-time. You can see the list of updated item in 'Overview > Settings > In-App Items' menu.

If you can't see any data in our dashboard, your AFPurchase object may be invalid. To check it, you can implement  AFPurchaseDelegate and call log logPurchase(purchase, delegate) method. 

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFPurchaseDelegate> {
  ...
}

// AppDelegate.m
- (void)didPurchaseVirtualItem {
  AFPurchase *purchase = [AFPurchase buildPurhcaseWithType:AFPurchaseTypeVirtualItem
                                               itemId:@"gun_001"
                                              currencyCode:@"gold"
                                                     price:100
                                              purchaseDate:nil
                                    transactionReceiptData:nil];
  [[AdFrescaView shardAdView] logPurchase:purchase, self];
}

- (void)purchase:(AFPurchase *)purchase didFailToLogWithException:(AdFrescaException *)exception {
  NSLog(@"AFPurchase didFailToLogWithException :: purchase = %@, exception = %@", [purchase JSONRepresentation], [exception description]);
}
```

* * *

### In-App Purchase Count

With In-App Purchase Count feature, you can set user's total number of in-app purchases with actual currency to use targeting features.  

You will set 'numberOfInAppPurchases' property after your app is launched, and also set the property after user purchased the item. You may get the number value from your server.

```java
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions 
{
  ......
  AdFrescaView *fresca = [AdFrescaView sharedAdView];  
  fresca.numberOfInAppPurchases = user.numberOfInAppPurchases; 
  ......
}

- (void)userDidPurchase:(int)numberOfInAppPurchases 
{
  currentUser.numberOfInAppPurchases = numberOfInAppPurchases;

  AdFrescaView *fresca = [AdFrescaView sharedAdView];   
  fresca.numberOfInAppPurchases = user.numberOfInAppPurchases; 
}  
```

* * *

### Give Reward

When you set 'Reward Item' section of the announcement campaign or 'Inventive item' section of the incentivized CPI & CPA campaign, you should implement this 'reward item' code to give an reward item to your users.

Implementing reward item codes, you can check if your user has any reward to receive, and then will be noticed with an reward item info.

To implement codes, we use two codes below:

- checkRewardItems method: this method is to check if any item is available to receive. we recommend to put this code when app becomes active. 
- AFRewardItemDelegate implemenation: when the reward condition is completed with current user, itemRewarded event is automatically called with AFRewardItem object from our SDK. you can give an item to the user with AFRewardItem object.

```objective-c
// AppDelegate.h

@interface AppDelegate : UIResponder <UIApplicationDelegate, AFRewardItemDelegate> {
  ...
}


// AppDelegate.m

- (void)applicationDidBecomeActive:(UIApplication *)application 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setRewardDelegate:self];
  [fresca checkRewardItems];
}

- (void)itemRewarded:(AFRewardItem *)item 
{
  NSString *logMessage = [NSString stringWithFormat:@"You got the reward item! (%@)", item.name];
  NSLog(@"%@", logMessage);
  
  // Using uniqueValue property, you can give an item to users.  
  [self sendItemToUser:item.uniqueValue];
}
```

You will implement your own 'sendItemToUser' method. This method may send the current user info and item's uniqueValue to your server. Then server gives the item to the user.

itemRewarded event is called when each type of campaign's reward condition is completed.

- Announcement Campaign: the event is called when your user see the campaign contents
- Incentivized CPI Campaign: the event is called when SDK checks Advertising App's install
- Incentivized CPA Campaign: the event is called after SDK checks Advertising App's install and the user called the targeted marketing event in Advertising App
 
If your users have any network disconnection or loss in theirs device, our SDK stored the reward data in the app's local storage, and then re-check in the next app session. So, we guarantee users will always get a reward from our SDK.

(getAvailableRewardItems is deprecated, but we still support this method for backward comparability)

* * *
