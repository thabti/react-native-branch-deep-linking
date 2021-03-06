# Branch Metrics React Native SDK Reference

[![build status](https://img.shields.io/travis/BranchMetrics/react-native-branch-deep-linking.svg?style=flat)](https://travis-ci.org/BranchMetrics/react-native-branch-deep-linking)
[![npm version](https://img.shields.io/npm/v/react-native-branch.svg?style=flat)](https://www.npmjs.com/package/react-native-branch)
[![npm downloads](https://img.shields.io/npm/dm/react-native-branch.svg?style=flat)](https://www.npmjs.com/package/react-native-branch)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://raw.githubusercontent.com/BranchMetrics/react-native-branch-deep-linking/master/LICENSE)

This is a repository of our open source React Native SDK. The information presented here serves as a reference manual for the SDK. See the table of contents below for a complete list of the content featured in this document.

### Draft

Please note that this is a draft of the new reference manual. Some updates will follow before the
production release of 2.0.0.

___

## React Native Reference

1. External resources
  + [Full integration guide](https://dev.branch.io/getting-started/sdk-integration-guide/guide/react/)
  + [Change log](https://github.com/BranchMetrics/react-native-branch-deep-linking/blob/master/ChangeLog.md)
  + [Support portal](http://support.branch.io)

2. Getting started
  + [Installation using react-native link](#pure-react-native-app-using-react-native-link)
  + [Installation in a native iOS app using the React pod](#native-ios-app-using-the-react-pod)
  + [Updating from an earlier SDK version](#updating-from-an-earlier-version)
  + [Register for Branch key](#register-your-app)
  + [Project setup](#setup)

3. Branch general methods
  + [Register a subscriber](#register-a-subscriber)
  + [Unregister a subscriber](#unregister-a-subscriber)
  + [Adjust cached link TTL](#adjust-cached-link-ttl)
  + [Retrieve latest deep linking params](#retrieve-session-install-or-open-parameters)
  + [Retrieve the user's first deep linking params](#retrieve-install-install-only-parameters)
  + [Setting the user id for tracking influencers](#persistent-identities)
  + [Logging a user out](#logout)
  + [Tracking custom events](#register-custom-events)
  + [Sending commerce events](#commerce-events)

4. Branch Universal Objects
  + [Instantiate a Branch Universal Object](#branch-universal-object)
  + [Register user actions on an object](#register-user-actions-on-an-object)
  + [List content on Spotlight](#list-content-on-spotlight)
  + [Configuring link properties](#link-properties-parameters)
  + [Configuring control parameters](#control-parameters)
  + [Creating a short link referencing the object](#shortened-links)
  + [Triggering a share sheet to share a link](#share-sheet)
  + [Releasing native resources](#releasing-native-resources)

5. Referral rewards methods
  + [Get reward balance](#get-reward-balance)
  + [Redeem rewards](#redeem-all-or-some-of-the-reward-balance-store-state)
  + [Get credit history](#get-credit-history)

## Installation

Note that the `react-native-branch` module requires `react-native` >= 0.40.

### Pure React Native app (using react-native link)

1. `yarn add react-native-branch` or `npm install --save react-native-branch`
2. (Optional) Add a branch.json file to the root of your app project. See https://rnbranch.app.link/branch-json.
3. `react-native link react-native-branch`
4. Follow the [setup instructions](#setup).

### Native iOS app using the React pod

Only follow these instructions if you are already using the React pod from node_modules. This is usually
done in native apps that integrate React Native components.

1. Add the following to your Podfile:
    ```Ruby
    pod "react-native-branch", path: "../node_modules/react-native-branch"
    pod "Branch-SDK", path: "../node_modules/react-native-branch/ios"
    ```
    Adjust the path if necessary to indicate the location of your `node_modules` subdirectory.
2. Run `pod install` to regenerate the Pods project with these new dependencies.
2. (Optional) Add a branch.json file to the your app project. See https://rnbranch.app.link/branch-json.
4. Follow the [setup instructions](#setup).

### Updating from an earlier version

As of version 2.0.0, the native Branch SDKs are included in the module and must not be installed
from elsewhere (CocoaPods, Carthage or manually). When updating from an earlier
versions of `react-native-branch`, you must remove the Branch SDK that was
previously taken from elsewhere.

#### Android

Open the `android/app/build.gradle` file
in your project. Remove:

```gradle
compile 'io.branch.sdk.android:library:2.+'
```

And add this line if not already present:

```gradle
compile fileTree(dir: 'libs', include: ['*.jar'])
```

The result should be something like
```gradle
dependencies {
    compile project(':react-native-branch')
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:+"  // From node_modules
}
```

#### iOS

Remove the external Branch SDK from your project depending on how you originally
integrated it.

##### CocoaPods

###### Apps built using react-native link

Remove "Branch" from your Podfile. Run `pod install` after updating the Podfile. This is
necessary to regenerate the Pods project without the Branch pod.

If you added CocoaPods to your project just for the Branch pod, you can remove CocoaPods entirely from your app using the [pod deintegrate](https://guides.cocoapods.org/terminal/commands.html#pod_deintegrate) command.

###### Using the React pod

Replace `pod "Branch"` in your Podfile with `pod "Branch-SDK", path: "../node_modules/react-native-branch/ios"`.
```Ruby
pod "React", path: "../node_modules/react-native"
# The following line is necessary with use_frameworks! or RN >= 0.42.
pod "Yoga", path: "../node_modules/react-native/ReactCommon/yoga"
pod "react-native-branch", path: "../node_modules/react-native-branch"
pod "Branch-SDK", path: "../node_modules/react-native-branch/ios"
```

Run `pod install` after making this change.

##### Carthage

Remove Branch.framework from your app's dependencies. Also remove Branch.framework from your `carthage copy-frameworks` build phase.

##### Manually installed

Remove Branch.framework from your app's dependencies.

### Register Your App

You can sign up for your own app id at [https://dashboard.branch.io](https://dashboard.branch.io).

## Setup

### iOS project

Modify your AppDelegate as follows:

#### Objective-C
In AppDelegate.m

```objective-c
#import <react-native-branch/RNBranch.h> // at the top

// Initialize the Branch Session at the top of existing application:didFinishLaunchingWithOptions:
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    // Uncomment this line to use the test key instead of the live one.
    // [RNBranch useTestInstance]
    [RNBranch initSessionWithLaunchOptions:launchOptions isReferrable:YES]; // <-- add this

    NSURL *jsCodeLocation;
    //...
}

// Add the openURL and continueUserActivity functions
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    if (![RNBranch.branch application:app openURL:url options:options]) {
        // do other deep link routing for the Facebook SDK, Pinterest SDK, etc
        return YES;
    }
}

- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray *restorableObjects))restorationHandler {
    return [RNBranch continueUserActivity:userActivity];
}
```

#### Swift

If you are using `react-native link`, your AppDelegate is probably written in Objective-C. When you use
`react-native link`, the Branch dependency is added to your project as a static library. If instead
you are using Swift in a pure React Native app with `react-native link`, you will require a
[bridging header](https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html)
in order to use any React Native plugin in Swift.

Add `#import <react-native-branch/RNBranch.h>` to your Bridging header if you have one.

If you are using the `React` pod in a native app with `use_frameworks!`, you may simply use
a Swift import:

```Swift
import react_native_branch
```

In AppDelegate.swift:
```Swift
// Initialize the Branch Session at the top of existing application:didFinishLaunchingWithOptions:
func application(_ application: UIApplication, didFinishLaunchingWithOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Uncomment this line to use the test key instead of the live one.
    // RNBranch.useTestInstance()
    RNBranch.initSession(launchOptions: launchOptions, isReferrable: true) // <-- add this

    //...
}

// Add the openURL and continueUserActivity functions
func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
    return RNBranch.branch.application(app, open: url, options: options)
}

func application(_ application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([Any]?) -> Void) -> Bool {
    return RNBranch.continue(userActivity)
}
```

These instructions are for Swift 3. Please note that Swift 2 is deprecated.

### Android project

Add RNBranchPackage to packages list in MainApplication.java (`android/app/src/[...]/MainApplication.java`)
```java
// ...

// import Branch and RNBranch
import io.branch.rnbranch.*;
import io.branch.referral.Branch;

//...

    // add RNBranchPackage to react-native package list
    @Override
    protected List<ReactPackage> getPackages() {
        return Arrays.<ReactPackage>asList(
                new MainReactPackage(),
                new RNBranchPackage(), // <-- add this

    // ...

    // add onCreate() override
    @Override
    public void onCreate() {
      super.onCreate();
      Branch.getAutoInstance(this);
    }
```

Override onStart and onNewIntent in MainActivity.java to handle Branch links (`android/app/src/[...]/MainActivity.java`)
```java
import io.branch.rnbranch.*; // <-- add this
import android.content.Intent; // <-- and this

public class MainActivity extends ReactActivity {

    @Override
    protected String getMainComponentName() {
        return "base";
    }

    // Override onStart, onNewIntent:
    @Override
    protected void onStart() {
        super.onStart();
        RNBranchModule.initSession(getIntent().getData(), this);
    }

    @Override
    public void onNewIntent(Intent intent) {
        setIntent(intent);
    }
    // ...
}
```

### iOS Project Setup

After modifying your AppDelegate:

1. [Add a String entry branch_key](https://dev.branch.io/references/ios_sdk/#add-your-branch-key-to-your-project) with your Branch key to your info.plist

2. [Configure for Universal Linking](https://dev.branch.io/references/ios_sdk/#support-universal-linking-ios-9)

3. If using a custom domain in the Branch Dashboard or one or more non-Branch domains, [add the `branch_universal_link_domains`
   key to your Info.plist](https://dev.branch.io/getting-started/universal-app-links/advanced/ios/#custom-continueuseractivity-configuration).

### Android Project Setup

1. [Configure AndroidManifest.xml](https://dev.branch.io/getting-started/sdk-integration-guide/guide/android/#configure-manifest). Be sure to set `android:launchMode="singleTask"` on your main activity.

2. [Register for Google Play Install Referrer](https://dev.branch.io/getting-started/sdk-integration-guide/guide/android/#register-for-google-play-install-referrer). The "receiver" element needs to be added to the "application" node in AndroidManifest.xml

3. [Register a URI scheme](https://dev.branch.io/getting-started/sdk-integration-guide/guide/android/#register-a-uri-scheme)
- The "intent-filter" element needs to be added to the activity node, whose android:name is "com.yourAppName.MainActivity". This node is in the "application" node.
- If you already have an intent-filter tag, this has to be added as an additional one.
- Make sure to replace "yourApp" with the scheme you specified in the Branch dashboard.

4. [Enable Auto Session Management](https://dev.branch.io/getting-started/sdk-integration-guide/guide/android/#enable-auto-session-management). Simply add the "android:name" attribute to your "application" node in your AndroidManifest.xml

5. [Enable App Links for Android M and above](https://dev.branch.io/getting-started/universal-app-links/guide/android/) (optional but recommended)

6. Add your Branch key to AndroidManifest: Inside of application node add `<meta-data android:name="io.branch.sdk.BranchKey" android:value="your_branch_key" />`

Please see the Branch [SDK Integration Guide](https://dev.branch.io/getting-started/sdk-integration-guide/) for complete setup instructions.

### Automated setup using Fastlane

<a href="https://docs.fastlane.tools" target="_blank"><img src="https://github.com/fastlane/fastlane/blob/master/fastlane/assets/fastlane_text.png?raw=true" alt="Fastlane" width="200" /></a>

Alternately, you can automatically perform all project configuration setup using Fastlane.
See the [Branch Fastlane plugin](https://github.com/BranchMetrics/fastlane-plugin-branch)
for details.

## Example apps

There are five example apps in this repo. See the [examples](./examples) subdirectory for more details.

## SDK Documentation

### Register a subscriber

To be called back when a link is opened, register a subscriber callback function
using the `branch.subscribe` method. Note that unlike the underlying native SDKs,
you do not have to initialize the Branch session from JavaScript. This is done
in native code at app launch, before the JavaScript initializes. If the app was
opened from a link, the initial link is cached by the native layer and returned
to the JavaScript subscriber afterward. This method may be called repeatedly in
different app components. To route links in a pure React Native app, call this
method in `componentDidMount` in a component that is mounted at app launch.

In a native app that includes a React Native component, link routing will usually
be done at the native level. This method may still be called from
JavaScript for purposes other than link routing (e.g. custom analytics). Other
Branch SDK methods may be called in this case without calling `branch.subscribe`
at all in JavaScript.

The callback function you supply to the `branch.subscribe` method is called
whenever a link is opened and at certain other times, such as successful SDK
initialization without a deferred deep link.

#### Method

```js
branch.subscribe(listener)
```

##### Parameters

**listener**: A function taking an object argument with the shape `{ error, params }`.
The `error` argument is a string. The `params` argument is an object. See
[Params object](#params-object) for details on the contents.

##### Return

The return value of `branch.subscribe` is a function that cancels the subscription
when called.

#### Example

```js
import branch from 'react-native-branch'

branch.subscribe(({ error, params }) => {
  if (error) {
    console.error('Error from Branch: ' + error)
    return
  }

  // params will never be null if error is null

  if (params['+non_branch_link']) {
    const nonBranchUrl = params['+non_branch_link']
    // Route non-Branch URL if appropriate.
    return
  }

  if (!params['+clicked_branch_link']) {
    // Indicates initialization success and some other conditions.
    // No link was opened.
    return
  }

  // A Branch link was opened.
  // Route link based on data in params.
})
```

### Unregister a subscriber

The return value of `branch.subscribe` is a function that cancels the
subscription when called. Call this in `componentWillUnmount`.

#### Example

```js
import branch from 'react-native-branch'

class MyApp extends Component {
  _unsubscribeFromBranch = null

  componentDidMount() {
    _unsubscribeFromBranch = branch.subscribe({ error, params } => {
      // ...
    })
  }

  componentWillUnmount() {
    if (_unsubscribeFromBranch) {
      _unsubscribeFromBranch()
      _unsubscribeFromBranch = null
    }
  }
}
```

### Adjust cached link TTL

Any initial link cached by the native layer will be returned to the callback
supplied to `branch.subscribe` immediately if the JavaScript method is called
within a certain time from app launch. This allows components to unsubscribe and
resubscribe without receiving the initial link after resubscription. By default,
the TTL for any initial link is 5000 ms. To use a different value, set
`branch.initSessionTtl` to that value, in ms. To have the desired effect, this
parameter should be set at app launch.

#### Property

```js
branch.initSessionTtl
```

An integer specifying the maximum amount of time, in milliseconds, to cache
the initial link from app launch. Defaults to 5000.

#### Example

```js
import branch from 'react-native-branch'

branch.initSessionTTL = 10000
branch.subscribe({ error, params } => {
  // ...
})
```

### Retrieve session (install or open) params

These session parameters will be available at any point later on with this command. If no parameters are available then Branch will return an empty dictionary. This refreshes with every new session (app installs AND app opens).

#### Method

```js
branch.getLatestReferringParams()
```

##### Return

A promise. On resolution, the promise returns an object containing the parameters
from the latest link open or install. See [Params object](#params-object) for
details on the contents.

#### Example

```js
import branch from 'react-native-branch'

const latestParams = await branch.getLatestReferringParams()
```

### Retrieve Install (Install Only) Parameters

If you ever want to access the original session params (the parameters passed in for the first install event only), you can use this line. This is useful if you only want to reward users who newly installed the app from a referral link. Note that these parameters can be updated when `setIdentity:` is called and identity merging occurs.

#### Method

```js
branch.getFirstReferringParams()
```

##### Return

A promise. On resolution, the promise returns an object containing the referring
parameters from the initial app installation. See [Params object](#params-object)
for details on the contents.

#### Example

```js
import branch from 'react-native-branch'

const latestParams = await branch.getFirstReferringParams()
```

### Persistent Identities

Often, you might have your own user IDs, or want referral and event data to persist across platforms or uninstall/reinstall. It's helpful if you know your users access your service from different devices. This where we introduce the concept of an 'identity'.

#### Method

```js
branch.setIdentity(userIdentity)
```

##### Parameters

**userIdentity**: A string specifying the user identity to use.

#### Example

```js
import branch from 'react-native-branch'

branch.setIdentity('theUserId')
```

### Logout

If you provide a logout function in your app, be sure to clear the user when the logout completes. This will ensure that all the stored parameters get cleared and all events are properly attributed to the right identity.

**Warning**: This call will clear the promo credits and attribution on the device.

#### Method

```js
branch.logout()
```

#### Example

```js
import branch from 'react-native-branch'

branch.logout()
```

### Register custom events

The `branch.userCompletedAction` method may be used for custom events that do not
involve specific content views in the app. To record custom events related to
content views, use the `userCompletedAction` method on the Branch Universal Object.

#### Method

```js
branch.userCompletedAction(event, params)
```

##### Parameters

**event**: A string specifying a custom event name  
**params**: (optional) An object containing custom key-value pairs to associate with the event

#### Example

```js
import branch from 'react-native-branch'

branch.userCompletedAction('Level Complete', {level: 'Level 1'})
```

### Commerce events

Use the `branch.sendCommerceEvent` method to record commerce events.

#### Method

```js
branch.sendCommerceEvent(revenue, metadata)
```

##### Parameters

**revenue**: A decimal number represented as a string or a float, e.g. "20.00" or 20.00  
**metadata**: (Optional) Metadata to associate with this event. Keys must be strings.

#### Example

```js
import branch from 'react-native-branch'

branch.sendCommerceEvent("20.00")
branch.sendCommerceEvent(50, {key1: "value1", key2: "value2"})
```

## Branch Universal Object (for deep links, content analytics and indexing)

The Branch Universal Object represents an item of content in your app, e.g. an article,
a video, a user profile or a post.

### Branch Universal Object best practices

Here are a set of best practices to ensure that your analytics are correct, and your content is ranking on Spotlight effectively.

1. Set the `canonicalIdentifier` to a unique, de-duped value across instances of the app.
2. Ensure that the `title`, `contentDescription` and `contentImageUrl` properly represent the object.
3. Initialize the Branch Universal Object and call `userCompletedAction` with the `RegisterViewEvent` **on page load** (in `componentDidMount`).
4. Call `showShareSheet` and `generateShortLink` later in the life cycle, when the user takes an action that needs a link.
5. Call the additional object events (purchase, share completed, etc) when the corresponding user action is taken.

Practices to _avoid_:
1. Don't set the same `title`, `contentDescription` and `contentImageUrl` across all objects.
2. Don't wait to initialize the object and register views until the user goes to share.
3. Don't wait to initialize the object until you conveniently need a link.
4. Don't create many objects at once and register views in a loop.

#### Params object

The params object is returned by various linking methods including subscribe, getLatestReferringParams, and getFirstReferringParams. Params will contain any data associated with the Branch link that was clicked before the app session began.  

Branch returns explicit parameters every time. Here is a list, and a description of what each represents.  
* `~` denotes analytics  
* `+` denotes information added by Branch

| **Parameter** | **Meaning**
| --- | ---
| ~channel | The channel on which the link was shared, specified at link creation time
| ~feature | The feature, such as `invite` or `share`, specified at link creation time
| ~tags | Any tags, specified at link creation time
| ~campaign | The campaign the link is associated with, specified at link creation time
| ~stage | The stage, specified at link creation time
| ~creation_source | Where the link was created ('API', 'Dashboard', 'SDK', 'iOS SDK', 'Android SDK', or 'Web SDK')
| ~referring_link | The referring link that drove the install/open, if present
| ~id | Automatically generated 18 digit ID number for the link that drove the install/open, if present
| +match_guaranteed | True or false as to whether the match was made with 100% accuracy
| +referrer | The referrer for the link click, if a link was clicked
| +phone_number | The phone number of the user, if the user texted himself/herself the app
| +is_first_session | Denotes whether this is the first session (install) or any other session (open)
| +clicked_branch_link | Denotes whether or not the user clicked a Branch link that triggered this session
| +click_timestamp | Epoch timestamp of when the click occurred
| +url | The full URL of the link that drove the install/open, if present (e.g. bnc.lt/m/abcde12345)

See also [Deep Link Routing](https://dev.branch.io/getting-started/deep-link-routing/guide/react/#branch-provided-data-parameters-in-callback)
on the Branch documentation site for more information.

Any additional data attached to the Branch link will be available unprefixed.

### Branch Universal Object

To create a Branch Universal Object, use the `branch.createBranchUniversalObject` method. Note
that unlike the underlying SDKs, all parameters to the Branch Universal Object must be supplied
at creation. These parameters are not represented as properties on the JavaScript object
returned by this method. They are stored on the underlying native Branch Universal Object.

#### Method

```js
branch.createBranchUniversalObject(canonicalIdentifier, properties)
```

##### Parameters

**canonicalIdentifier**: A string that uniquely identifies this item of content  
**properties**: An object containing properties defining the Branch Universal Object. See
[Branch Universal Object Properties](#branch-universal-object-properties) for a list of
available properties.

##### Return

A promise. On resolution, the promise returns an object with a number of methods, documented
below.

#### Example

```js
import branch from 'react-native-branch'

let branchUniversalObject = await branch.createBranchUniversalObject('canonicalIdentifier', {
  automaticallyListOnSpotlight: true,
  metadata: {prop1: 'test', prop2: 'abc'},
  title: 'Cool Content!',
  contentDescription: 'Cool Content Description'})
```

### Register user actions on an object

We've added a series of custom events that you'll want to start tracking for rich
analytics and targeting. Use this method to record events associated with a
content item (Branch Universal Object).

#### Method

```js
branchUniversalObject.userCompletedAction(event, params)
```

##### Parameters

**event**: A string representing a standard event from the list below or a custom
event.

| Event | Description
| ----- | ---
| RegisterViewEvent | User viewed the object
| AddToWishlistEvent | User added the object to their wishlist
| AddToCartEvent | User added object to cart
| PurchaseInitiatedEvent | User started to check out
| PurchasedEvent | User purchased the item
| ShareInitiatedEvent | User started to share the object
| ShareCompletedEvent | User completed a share

**params**: (Optional) A dictionary of custom key-value pairs to associate with
this event.

```js
import branch, {
  AddToCartEvent,
  AddToWishlistEvent,
  PurchasedEvent,
  PurchaseInitiatedEvent,
  RegisterViewEvent,
  ShareCompletedEvent,
  ShareInitiatedEvent
} from 'react-native-branch'

let branchUniversalObject = await branch.createBranchUniversalObject(...)

branchUniversalObject.userCompletedAction(RegisterViewEvent)
branchUniversalObject.userCompletedAction('Custom Action', { key: 'value' })
```

### List content on Spotlight

To list content on Spotlight (iOS), the preferred practice is to set the `automaticallyListOnSpotlight`
property to `true` in the `createBranchUniversalObject` method and call `userCompletedAction(RegisterViewEvent)`
on the Branch Universal Object when the view appears. The content will be listed on Spotlight
when `userCompletedAction` is called. There is also a `listOnSpotlight()` method on the Branch Universal
Object that can be used for this purpose. Note that this method and the
`automaticallyListOnSpotlight` property are ignored on Android.

**Note**: Listing on Spotlight requires adding `CoreSpotlight.framework` to your
Xcode project.

#### Method

```js
branchUniversalObject.listOnSpotlight()
```

#### Example

```js
import branch, { RegisterViewEvent } from 'react-native-branch'

let branchUniversalObject = await branch.createBranchUniversalObject('canonicalIdentifier', {
  automaticallyListOnSpotlight: true,
  // other properties
})

branchUniversalObject.userCompletedAction(RegisterViewEvent)
```

or

```js
import branch, { RegisterViewEvent } from 'react-native-branch'

let branchUniversalObject = await branch.createBranchUniversalObject(...)

branchUniversalObject.listOnSpotlight()
```

### Shortened Links

Once you've created your `Branch Universal Object`, which is the reference to the content you're interested in, you can then get a link back to it with the mechanisms described below.

#### Method

```js
branchUniversalObject.generateShortUrl(linkProperties, controlParams)
```

##### Parameters

**linkProperties**: An object containing properties to define the link. See [Link
Properties Parameters](#link-properties-parameters) for available properties.  
**controlParams**: (Optional) An object containing control parameters to override
redirects specified in the Branch Dashboard. See
[Control Parameters](#control-parameters) for a list of available parameters.

##### Return

A promise. On resolution, the promise returns an object with the shape `{ url }`.
The `url` property is a string containing the generated short URL.

#### Example

```js
import branch from `react-native-branch`

let branchUniversalObject = await branch.createBranchUniversalObject(...)

let linkProperties = { feature: 'share', channel: 'RNApp' }
let controlParams = { $desktop_url: 'http://example.com/home', $ios_url: 'http://example.com/ios' }

let {url} = await branchUniversalObject.generateShortUrl(linkProperties, controlParams)
```

### Share sheet

Once you've created your `Branch Universal Object`, which is the reference to the
content you're interested in, you can then automatically share it _without having
to create a link_ using the mechanism below.

The Branch SDK includes a wrapper around the system share sheet that will generate a
Branch short URL and automatically tag it with the channel the user selects
(Facebook, Twitter, etc.). Note that certain channels restrict access to certain
fields. For example, Facebook prohibits you from pre-populating a message.

#### Method

```js
branchUniversalObject.showShareSheet(shareOptions, linkProperties, controlParams)
```

##### Parameters

**shareOptions*: An object containing any of the following properties:

|        KEY         |   TYPE   |       MEANING
| ------------------ | -------- | --------------------
| messageHeader      | `string` | The header text
| messageBody        | `string` | The body text
| emailSubject       | `string` | The subject of the email channel if selected

**linkProperties**: An object containing properties to define the link. See [Link
Properties Parameters](#link-properties-parameters) for available properties.  
**controlParams**: (Optional) An object containing control parameters to override
redirects specified in the Branch Dashboard. See
[Control Parameters](#control-parameters) for a list of available parameters.

##### Return

A promise. On resolution, the promise returns an object with the shape
`{ channel, completed, error }`. The `completed` property is a boolean specifying
whether the operation was completed by the user. The `channel` property is a
string specifying the share channel selected by the user. The `error` property
is a string. If non-null, it specifies any error that occurred.

#### Example

```js
import branch from `react-native-branch`

let branchUniversalObject = await branch.createBranchUniversalObject(...)

let linkProperties = { feature: 'share', channel: 'RNApp' }
let controlParams = { $desktop_url: 'http://example.com/home', $ios_url: 'http://example.com/ios' }

let shareOptions = { messageHeader: 'Check this out', messageBody: 'No really, check this out!' }

let {channel, completed, error} = await branchUniversalObject.showShareSheet(shareOptions, linkProperties, controlParams)
```

### Releasing native resources

The Branch Universal Object is a construct in the underlying native SDK that is
exposed using a JavaScript object that is returned by the
`createBranchUniversalObject` method. For best performance, call the `release()`
method on the Branch UniversalObject when the Branch Universal Object is no
longer in use. Native resources will eventually be reclaimed without calling
this method. Calling it ensures that the resources are reclaimed promptly.

#### Method

```js
branchUniversalObject.release()
```

#### Example

```js
import branch, { RegisterViewEvent } from `react-native-branch

class CustomComponent extends Component {
  buo = null

  componentDidMount() {
    this.buo = await branch.createBranchUniversalObject(...)
    this.buo.userCompletedAction(RegisterViewEvent)
  }

  componentWillUnmount() {
    if (this.buo) {
      this.buo.release()
      this.buo = null
    }
  }
}
```

#### Branch Universal Object Properties

|         Key                  | TYPE   |             DESCRIPTION                                |
| ---------------------------- | ------ | ------------------------------------------------------ |
| automaticallyListOnSpotlight | Bool   | List this item on Spotlight (iOS). Ignored on Android. |
| canonicalIdentifier          | String | The object identifier                                  |
| contentDescription           | String | Object Description                                     |
| contentImageUrl              | String | The Image URL                                          |
| contentIndexingMode          | String | Indexing Mode 'private' or 'public'                    |
| currency                     | String | A 3-letter ISO currency code (used with price)         |
| expirationDate               | String | A UTC expiration date, e.g. 2018-02-01T00:00:00        |
| keywords                     | Array  | An array of keyword strings                            |
| metadata                     | Object | Custom key/value                                       |
| price                        | Float  | A floating-point price (used with currency)            |
| title                        | String | The object title                                       |
| type                         | String | MIME type for this content                             |

#### Link Properties Parameters

|    KEY   |   TYPE   |          MEANING
| -------- | -------- |------------------------
| alias    | `string` | Specify a link alias in place of the standard encoded short URL (e.g., `[branchsubdomain]/youralias or yourdomain.co/youralias)`. Link aliases are unique, immutable objects that cannot be deleted. **Aliases on the legacy `bnc.lt` domain are incompatible with Universal Links and Spotlight**
| campaign | `string` | Use this field to organize the links by actual campaign. For example, if you launched a new feature or product and want to run a campaign around that
| channel  | `string` | Use channel to tag the route that your link reaches users. For example, tag links with ‘Facebook’ or ‘LinkedIn’ to help track clicks and installs through those paths separately
| feature  | `string` | This is the feature of your app that the link might be associated with. eg: if you had built a referral program, you would label links with the feature `referral`
| stage    | `string` | Use this to categorize the progress or category of a user when the link was generated. For example, if you had an invite system accessible on level 1, level 3 and 5, you could differentiate links generated at each level with this parameter
| tags     | `array`  | This is a free form entry with unlimited values. Use it to organize your link data with labels that don’t fit within the bounds of the above

### Control parameters

Specify control parameters in calls to `generateShortUrl` and `showShareSheet`.

All Branch control parameters are supported. See [here](https://dev.branch.io/getting-started/configuring-links/guide/#link-control-parameters) for a complete list. In particular, these control parameters determine where the link redirects.

|        KEY         |   TYPE   |       MEANING
| ------------------ | -------- | --------------------
| $fallback_url      | `string` | Change the redirect endpoint for all platforms - so you don’t have to enable it by platform
| $desktop_url       | `string` | Change the redirect endpoint on desktops  
| $android_url       | `string` | Change the redirect endpoint for Android
| $ios_url           | `string` | Change the redirect endpoint for iOS
| $ipad_url          | `string` | Change the redirect endpoint for iPads
| $fire_url          | `string` | Change the redirect endpoint for Amazon Fire OS
| $blackberry_url    | `string` | Change the redirect endpoint for Blackberry OS
| $windows_phone_url | `string` | Change the redirect endpoint for Windows OS

## Referral System Rewarding Functionality

### Get Reward Balance

Reward balances change randomly on the backend when certain actions are taken (defined by your rules), so
you will need to make an asynchronous call to retrieve the balance.

#### Method

```js
branch.loadRewards()
```

##### Return

#### Example

```js
import branch from 'react-native-branch'

let rewards = await branch.loadRewards()
```

### Redeem All or Some of the Reward Balance (Store State)

Redeeming credits allows users to cash in the credits they've earned. Upon successful redemption, the user's balance will be updated reflecting the deduction.

#### Method

```js
branch.redeemRewards(amount, bucket)
```

##### Parameters

**amount**: The amount to redeem  
**bucket**: (Optional) The bucket to redeem from

#### Example

```js
import branch from 'react-native-branch'

let redeemResult = await branch.redeemRewards(amount, bucket)
```

### Get Credit History

This call will retrieve the entire history of credits and redemptions from the individual user.

#### Method

```js
branch.getCreditHistory()
```

##### Return

A promise. On resolution, the promise returns an array containing the current user's credit history.

#### Example

```js
let creditHistory = await branch.getCreditHistory()
```
