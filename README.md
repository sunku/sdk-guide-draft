## Contents
- [Basic Integration](#basic-integration)
  - [Installation](#installation)
  - [Start Session](#start-session)
  - [Sign In & Sign Out](#sign-in--sign-out)
  - [In-App Messaging](#in-app-messaging)
  - [Push Messaging](#push-messaging)
  - [Test Device Registration](#test-device-registration)
  - [Test Mode](#test-mode)
- [IAP, Reward and Sales Promotion](#iap-reward-and-sales-promotion)
  - [In-App Purchase Tracking](#in-app-purchase-tracking)
  - [Give Reward](#give-reward)
  - [Sales Promotion](#sales-promotion)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Stickiness Custom Parameter](#stickiness-custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [Custom Banner (Android Only)](#custom-banner)
  - [AFShowListener](#afshowlistener)
  - [Timeout Interval](#timeout-interval)
- [Reference](#reference)
  - [Deep Link](#deep-link)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [Google Referrer Tracking](#google-referrer-tracking)
  - [Proguard Configuration](#proguard-configuration)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

...

### Sign In & Sign Out

You can track a user’s sign in or sign out actions using these Sign In and Sign Out functions. Nudge will use a string passed using signIn or signOut method as a user identifier and track users with multiple devices, which makes statistics more accurate and campaigns will recognize users, not devices so they will run more effectively. (Users will no longer claim the same rewards multiple times by using different devices.)

You need to pass a user identifier (string) to **signIn()** method when a user signs in to your server (including auto sign-in). You also need to put **signOut()** method when a user signs out.

```java
public void onSignIn {
  AdFresca.getInstance(currentActivity).signIn("user_id");
}

public void onSignOut {
  AdFresca.getInstance(currentActivity).signOut();
}
```

Nudge also supports 'guest sign in' with signInAsGuest() method.

```java
public void onGuestSignIn {
  AdFresca.getInstance(currentActivity).signInAsGuest("guest_user_id");
}
```

You can check a user’s current sign-in status by calling **getSignedUserId()** method. This method which returns an user identifier used in last sign in, and device identifier after the user signed out. Please use this method to test your codes.

* * *

## Dynamic Targeting

### Custom Parameter

Custom Parameter is a user attribute used to classify users for marketing purpose. You can use any custom values (e.g. user level, stage, and play count) to define a user segment and monitor it in real time. You can achieve better campaign performance when targeting users with higher accuracy. (Nudge SDK automatically collects default values such as device id, language, country, app version, and others so you don’t need to define those values as custom parameters.)

Nudge SDK provides two tracking methods by types of custom parameters. 

- To track the current status of a user
  - It is used to track the current value of specific user attributes
  - ex: level, current stage, facebook sign-in flag
  - SDK Code: Use **setCustomParameterValue** method to pass the current status (Integer, Boolean type) to SDK.

- To track specific event count
  - It is used to track count for a specific event.
  - ex: play count, a number of gacha count
  - SDK Code: Use **incrCustomParameterValue** method to pass increased value (Integer) to SDK after an event occurred.

First, you need to define ‘Unique Key’ string value to define a custom parameter. (e.g. "level", "facebook_flag", "play_count") Then write the tracking codes when an user launches your app or signs in to your server.

```java
public void onCreate() {
  AdFresca fresca = AdFresca.getInstance(this);     
  fresca.setCustomParameterValue("level", User.level);
  fresca.setCustomParameterValue("facebook_flag", User.hasFacebookAccount);
  fresca.startSession();
}
```

Then you need to put tracking codes whenever its value changes.

```java
public void onUserLevelChanged(int level) {  
  AdFresca fresca = AdFresca.getInstance(this);     
  fresca.setCustomParameterValue("level", level);
}

public void onGameFinished {
  AdFresca fresca = AdFresca.getInstance(this);     
  fresca.incrCustomParameterValue("play_count", 1);
}
```

If you successfully writes codes and set custom parameters, you will see a list of custom parameters you added on [Dashboard](https://admin.adfresca.com). 1) Select an App 2) In 'Overview' menu, click 'Settings - Custom Parameters' button.

<img src="https://s3-ap-northeast-1.amazonaws.com/file.adfresca.com/guide/sdk/custom_parameter_index.png">

In order to activate a custom parameter, you need to set ‘Name’.. (You can activate custom parameters up to 20.) Nudge only stores data of activated custom parameters and use them for targeting.

#### Stickiness Custom Parameter

Stickiness custom parameter is a special custom parameter to measure a user’s stickiness. For example, if you set ‘play count’ as stickiness custom parameter in a stage-based game, You can define user segments with filters like ‘Today’s play count, ‘Play count in a week’, and ‘Average play count in a week’. Stickiness custom parameter will help you to classify user groups by their loyalty and to monitor their activities in real time. 

You must use **incrCustomParameterValue* method for stickiness custom parameters. If you want to use stickiness custom parameters, please send an email to support@nudge.do after you activate your custom parameter in your dashboard.

* * *


.....
