# Expo-Managed-Workflow
We have an Expo managed workflow app and we're using AppsFlyer to bridge our app to ads network like Meta and Google. We're using react-native-fbsdk-next to integrate Meta SDK, Firebase for any Google Services and react-native-appsflyer for AppsFlyer. THE ISSUE: We are struggling to get data from our ads. Meta: We have data only for Android on AppsFlyer (installation & purchase - Perfect) and nothing on Meta, but we have NOTHING for iOS, both on Meta and AppsFlyer. Google: We can't track the app installs and the user session on Google Ads and Google Analytics, nothing in AppsFlyers as well. MORE INFORMATION: - We are requesting tracking permission using the Transparency API - We're initializing Facebook SDK on App start - We're initializing AppsFlyer SDK on App start We are sending af_subscribe event with (af_revenue, af_currency, af_subscription_id, subscription_type) parameters when a user subscribe to our services We do not use the validateAndLogInAppPurchase for now as we think it should be working with regular events (right?) We've been in contact with AppsFlyer and Meta Support, they confirm that the setups are correct, and still we can't get it working... Do you have any idea where the issue might be coming from? Just a quick note, we've been using AppsFlyer specifically because we couldn't track subscriptions in our ads and some people recommended us to use such a MMP to help with SKAN, data event tracking etc... If you have any recommandation to make it work without AppsFlyer, we're down for it. We just want to be able to track any event from our app in ads analytics. We also made several test both on Meta Event Tester and AppsFlyers Tester, and everything is working properly, the SDK are installed, the events are received, both for organic and non-organic on AppsFlyer, but for some reason the issue is on the attribution level Appreciate your help.
------------
It seems like you’re dealing with a complex issue related to attribution, especially when integrating AppsFlyer, Meta (Facebook SDK), and Google in an Expo-managed workflow. Given that everything is working fine for Android but there are issues with iOS tracking (on both AppsFlyer and Meta), the problem likely lies in a few key areas, such as configuration, iOS-specific setups, and possibly App Tracking Transparency (ATT) handling for iOS.

Here’s a step-by-step breakdown of what might be going wrong and how to troubleshoot the issue:
1. Review iOS Specific Issues:

Since Android works fine but iOS is facing issues, it's likely that App Tracking Transparency (ATT) permissions are affecting the attribution process for iOS. With iOS 14.5 and later, App Tracking Transparency (ATT) became mandatory for tracking users across apps and websites. If ATT is not properly handled, tracking events won’t be sent to Meta or AppsFlyer.
1.1. Ensure ATT Permissions Are Requested Correctly

First, make sure you are requesting ATT permissions before initializing any SDKs (including Facebook and AppsFlyer). If you don't handle this correctly, it could prevent tracking across Meta and AppsFlyer.

Here’s how you can request ATT permissions using react-native-permissions:

    Install the required package:

npm install react-native-permissions

Request permission in your App.js or wherever your app starts:

    import { request, PERMISSIONS, RESULTS } from 'react-native-permissions';

    const requestTrackingPermission = async () => {
      try {
        const status = await request(PERMISSIONS.IOS.APP_TRACKING_TRANSPARENCY);
        if (status === RESULTS.GRANTED) {
          console.log('Tracking permission granted');
        } else {
          console.log('Tracking permission denied');
        }
      } catch (error) {
        console.error('Error requesting permission', error);
      }
    };

    // Call this function on app start or when needed
    requestTrackingPermission();

    Make sure to request this permission before initializing the Facebook SDK and AppsFlyer SDK.

1.2. Facebook SDK (Meta) Configuration

Since you're using react-native-fbsdk-next, ensure that you have the correct iOS setup.

    Check Facebook app id and app secret in your iOS configuration. This should be set up properly in both Info.plist and your Facebook Developer Console.

    Ensure you’re using App Events properly. Double-check that events are sent to Facebook after the ATT permission is granted.

In App.js, initialize Facebook SDK like this:

import { Settings } from 'react-native-fbsdk-next';

// Make sure you initialize the Facebook SDK before tracking events
Settings.initializeSDK();

const trackFacebookEvent = () => {
  // Example event
  logEvent('af_subscribe', {
    af_revenue: '20.99',
    af_currency: 'USD',
    af_subscription_id: 'subscription_123',
    subscription_type: 'premium',
  });
};

    Ensure that the correct SDK version for iOS is being used. Sometimes, older versions may not fully support iOS 14.5 and later ATT requirements.

2. AppsFlyer SDK for iOS:
2.1. Initialization Check

Verify that you are initializing the AppsFlyer SDK properly on iOS. If the AppsFlyer SDK is initialized after you request tracking permissions, the SDK won’t be able to track attribution events.

In Expo managed workflow, you should initialize the AppsFlyer SDK in the appropriate lifecycle event, like componentDidMount() or after ensuring ATT permissions are granted.

For Expo Managed Workflow, initialize AppsFlyer SDK like this:

    Install react-native-appsflyer:

npm install react-native-appsflyer

Initialize AppsFlyer SDK (ensure it’s after the ATT permission request):

    import AppsFlyer from 'react-native-appsflyer';

    const initAppsFlyer = () => {
      const options = {
        devKey: 'YOUR_APPSFLYER_DEV_KEY',
        isDebug: true,
        appId: 'YOUR_APP_ID',  // Only for iOS
      };

      AppsFlyer.initSdk(options, (result) => {
        console.log('AppsFlyer SDK Initialized', result);
      }, (error) => {
        console.error('AppsFlyer initialization failed', error);
      });
    };

    // Ensure this is initialized after ATT permission is granted
    initAppsFlyer();

2.2. Event Tracking

You’re sending the af_subscribe event with the right parameters, but ensure you’re calling AppsFlyer’s logEvent API after initialization:

const trackSubscriptionEvent = () => {
  AppsFlyer.logEvent('af_subscribe', {
    af_revenue: '20.99',
    af_currency: 'USD',
    af_subscription_id: 'subscription_123',
    subscription_type: 'premium',
  });
};

This should be triggered after the user subscribes to your services.
2.3. Check iOS Build Configurations

In your iOS build, check that:

    Info.plist has proper entries for AppsFlyer and Facebook.
    Tracking is enabled for both Meta and AppsFlyer (this might require changes to your AppDelegate.m file, especially for manual linking).

3. Debugging
3.1. Meta and AppsFlyer Test Events

Since you’ve already used the Meta Event Tester and AppsFlyer Tester, confirm that the test events are appearing correctly for both iOS and Android.

    For Meta, check your Facebook Events Manager and ensure events are being logged. Sometimes, events may be delayed or might not show up immediately.

    For AppsFlyer, make sure that iOS installs are correctly attributed. If the install events are not showing up on AppsFlyer, it could be a permission issue or an issue with your dev key configuration.

3.2. Check iOS Console Logs

For better debugging:

    Run the app on an iOS simulator or device.
    Check Xcode console logs to see if any errors or warnings are related to AppsFlyer or Facebook SDK.

Use Xcode's Console or debugging tools to verify that events are being sent to both AppsFlyer and Meta. Look for messages like AppsFlyer - Event sent or Facebook SDK - Event logged.
3.3. Double Check Data Sharing Permissions

For iOS 14.5+:

    Make sure your app’s privacy policies are up-to-date.
    The user must opt-in via ATT to allow tracking. If the user denies permission, tracking won’t work, and data won’t be passed to AppsFlyer or Meta.

4. Alternative to AppsFlyer:

If you're looking to track without AppsFlyer, and especially for SKAdNetwork tracking (required for iOS on Apple’s ecosystem), consider using Adjust, Branch, or Firebase Analytics. These can help track attribution and provide richer insights into user behavior, all without relying on an MMP like AppsFlyer.

Firebase Analytics can be used to track user sessions, installs, and conversions, and it also integrates well with Google Ads and Meta. Firebase offers a free and powerful tool for tracking in both iOS and Android.
Final Thoughts:

    ATT Permissions are crucial on iOS. Ensure you’re requesting it before initializing the SDKs.
    Meta and AppsFlyer Setup: Make sure all SDK configurations, especially on iOS, are correctly initialized.
    Test Environment: Ensure that everything is properly set up for testing in both Meta’s and AppsFlyer’s testers.
    SDK Initialization: Initialize Facebook SDK and AppsFlyer SDK after obtaining user consent.
