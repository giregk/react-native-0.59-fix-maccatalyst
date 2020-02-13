Apple released Mac Catalyst, also known as UIKitForMac. It enables iOS applications to run on MacOS 10.15+.

This is brilliant, but if your app is a react-native app, it does not work out of box. Here are the few steps I performed to make it work on react-native 0.59.10.

# Activate MacCatalyst

Open your project in XCode, (make sure to select "Targets" > yourapp to see the "General" tab)

In Deployement info section, select iPad and Mac.

# Fix Linking package

- `node_modules/react-native/Libraries/LinkingIOS/RCTLinkingManager.m`
  - replace
    ```
    BOOL opened = [RCTSharedApplication() openURL:URL];
    if (opened) {
      resolve(nil);
    } else {
      reject(RCTErrorUnspecified, [NSString stringWithFormat:@"Unable to open URL: %@", URL], nil);
    }
    ```
    with
    ```
    [RCTSharedApplication() openURL:URL options:@{} completionHandler:^(BOOL opened) {
      if (opened) {
        resolve(nil);
      } else {
        reject(RCTErrorUnspecified, [NSString stringWithFormat:@"Unable to open URL: %@", URL], nil);
      }
    }];
    ```

# Fix NetInfo package

- `node_modules/react-native/Libraries/Network/RCTNetInfo.m`
  - replace
    ```
    if ([netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyGPRS] ||
        [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyEdge] ||
        [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyCDMA1x]) {
      effectiveConnectionType = RCTEffectiveConnectionType2g;
    }else if([netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyWCDMA] ||
              [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyHSDPA] ||
              [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyHSUPA] ||
              [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyCDMAEVDORev0] ||
              [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyCDMAEVDORevA] ||
              [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyCDMAEVDORevB] ||
              [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyeHRPD]){
      effectiveConnectionType = RCTEffectiveConnectionType3g;
    } else if ([netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyLTE]) {
      effectiveConnectionType = RCTEffectiveConnectionType4g;
    }
    ```
    with
    ```
    #if TARGET_OS_MACCATALYST
        if (!!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyGPRS] ||
            !!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyEdge] ||
            !!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyCDMA1x]) {
          effectiveConnectionType = RCTEffectiveConnectionType2g;
        } else if (!!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyWCDMA] ||
                   !!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyHSDPA] ||
                   !!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyHSUPA] ||
                   !!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyCDMAEVDORev0] ||
                   !!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyCDMAEVDORevA] ||
                   !!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyCDMAEVDORevB] ||
                   !!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyeHRPD]) {
          effectiveConnectionType = RCTEffectiveConnectionType3g;
        } else if (!!netinfo.serviceCurrentRadioAccessTechnology[CTRadioAccessTechnologyLTE]) {
          effectiveConnectionType = RCTEffectiveConnectionType4g;
        }
    #else
        if ([netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyGPRS] ||
            [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyEdge] ||
            [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyCDMA1x]) {
          effectiveConnectionType = RCTEffectiveConnectionType2g;
        }else if([netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyWCDMA] ||
                 [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyHSDPA] ||
                 [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyHSUPA] ||
                 [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyCDMAEVDORev0] ||
                 [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyCDMAEVDORevA] ||
                 [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyCDMAEVDORevB] ||
                 [netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyeHRPD]){
          effectiveConnectionType = RCTEffectiveConnectionType3g;
        } else if ([netinfo.currentRadioAccessTechnology isEqualToString:CTRadioAccessTechnologyLTE]) {
          effectiveConnectionType = RCTEffectiveConnectionType4g;
        }
    #endif
    ```

# Fix TextView package

- `node_modules/react-native/Libraries/Text/Text/RCTTextView.m`
  - replace
    ```
    [menuController setTargetRect:self.bounds inView:self];
    [menuController setMenuVisible:YES animated:YES];
    ```
    with
    ```
    #if TARGET_OS_MACCATALYST
      [menuController showMenuFromView:self rect:self.bounds];
    #else
      [menuController setTargetRect:self.bounds inView:self];
      [menuController setMenuVisible:YES animated:YES];
    #endif
    ```

# Fix WebSocket package

- `node_modules/react-native/Libraries/WebSocket/RCTSRWebSocket.m`
  - delete line
    ```
    #import <Endian.h>
    ```
  - replace all `EndianU16_BtoN` with `NSSwapBigShortToHost`
  - replace all `EndianU64_BtoN` with `NSSwapBigLongLongToHost`

# Remove UIWebView

- `node_modules/react-native/Libraries/Components/WebView/WebView.ios.js`
  - delete line
    ```
    const RCTWebViewManager = require('NativeModules').WebViewManager;
    ```
  - replace
    ```
    if (this.props.useWebKit) {
      viewManager = viewManager || RCTWKWebViewManager;
    } else {
      viewManager = viewManager || RCTWebViewManager;
    }
    ```
    with
    ```
    viewManager = viewManager || RCTWKWebViewManager;
    ```
- `node_modules/react-native/React.podspec`
  - delete line
    ```
    "React/Views/RCTWebView\*"
    ```
  - don't forget to remove the trailing coma of the previous line too
- from XCode (so that files are also unlinked), delete files
  - `node_modules/react-native/React/Views/RCTWebView.h`
  - `node_modules/react-native/React/Views/RCTWebView.m`
  - `node_modules/react-native/React/Views/RCTWebViewManager.h`
  - `node_modules/react-native/React/Views/RCTWebViewManager.m`

# Run patch-package to save fixes

If you don't already have it, add `patch-package` to your dev-dependencies.

```
yarn patch-package react-native --use-yarn --exclude 'android|third-party|third-party-podspecs'
```

Your react-native project should now work on macOS. If not, try to clean build folder.

> If you have issues with glog, I noticed it helped to build the project on an iOS simulator first.

NB: using package @react-native-community/async-storage currently may break the build. If so, make sure you have added it as a dependency of the build process. This is not done automatically on linking, but is necessary in some cases to force XCode to built it before building your app itself.
