# Example applications

There are several working examples in this directory.

See each app for further details, including build instructions.

[testbed_native_android]: ./testbed_native_android
[testbed_native_ios]: ./testbed_native_ios
[testbed_simple]: ./testbed_simple
[webview_example]: ./webview_example
[webview_example_native_ios]: ./webview_example_native_ios

## Automatically set up iOS Branch configuration with Fastlane

To run any of the iOS examples ([testbed_native_ios], [testbed_simple], [webview_example], [webview_example_native_ios]) on a device, you can use Fastlane to set the following
Branch-related configuration parameters without changing settings manually in Xcode.

- Branch live key
- Branch test key
- Branch Universal Link domain(s)
- Application bundle identifier
- Code-signing team

Once this is done, you can deploy the app on a device and open Universal Links in the app on the device.

To use this tool:

1. Edit the [Branchfile](./Branchfile) in this directory to use the correct parameters for your app from the
    Branch Dashboard.
2. To update any example app, e.g. `webview_example`:
    ```bash
    bundle install
    cd webview_example
    bundle exec fastlane update_branch
    ```
    Now the project is set up to use your app parameters, including your team and bundle identifiers.
3. iOS projects: Validate the Universal Link configuration:
    ```bash
    bundle exec fastlane validate
    ```

## webview_example apps

These apps are realistic examples of a content-sharing app that includes link routing and follows best practices.

### [webview_example]

This is a pure React Native app that integrates the react-native-branch NPM module.

### [webview_example_native_ios]

This is a Swift app that integrates the Branch SDK and the react-native-branch NPM module in a React Native component.

## testbed apps

The testbed apps all include the same React Native application code. They differ by which versions of the SDK
they use and how React Native is integrated at the native level.

### [testbed_simple]

This app illustrates the simplest means of integrating react-native-branch into a React Native app, using `react-native link`.

### [testbed_native_android]

This app illustrates how to integrate the react-native-branch SDK into a React Native component within an existing native Android app.

### [testbed_native_ios]

This app illustrates how to integrate the react-native-branch SDK into a React Native component within an existing native iOS app using the React, react-native-branch and Branch-SDK pods from node_modules.
