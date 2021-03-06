# WeChat

## Web

### Prerequisite

To create a website application in WeChat, you can choose to setup a website application and wait for approval or a sandbox account for testing.

### Setup the Website Application \(网站应用\)

1. Register an account in [WeChat Open Platform](https://open.weixin.qq.com/).
2. Create Website Application \(网站应用\), fill in information and wait for approval \(It may take few days\).
3. View the application detail page, obtain the "AppID" and "AppSecret" on the top of the application page.
4. Go to Account Center &gt; Basic information, to obtain the "原始ID".

    ![wechat-account-id](../../.gitbook/assets/wechat-account-id.png)

### Setup the Sandbox Application \(微信公众平台接口测试帐号\)

1. Use your WeChat app to login in [微信公众平台接口测试帐号申请](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login).
2. Obtain the "appID", "appSecret" and "原始ID". The "原始ID" is the "微信号" in the top right corner.

    ![wechat-sandbox-account-id](../../.gitbook/assets/wechat-sandbox-account-id.png)

### Configure Sign in with WeChat through the Authgear portal

In the portal, go to "Single-Sign On" page, then do the following:

1. Enable "Sign in with WeChat \(Web/网站应用\)"
2. Fill in "Client ID" with the "AppID".
3. Fill in "Client Secret" with the "AppSecret".
4. Fill in "原始 ID" with the "原始 ID".
5. Check the checkbox "Is Sandbox account" if you are using sandbox account.
6. Click save.

## Mobile app \(Native iOS app, Native Android app and React Native\)

### Overview of Setting Up Wechat

Wechat integration is a bit more complicated then other social login, here are the overview of what needs to be done:

1. Register an account and create mobile application in WeChat Open Platform. Approval is needed in this process.
2. Enable and configure WeChat Login in Authgear portal.
3. Include Authgear SDK on your app.
4. Implement a delegate function to be triggered when user clicks the "Login with WeChat" button during authorization. You need to integrate WeChat SDK to open WeChat app to perform authentication in the delegate function \(we have sample code below\). After obtaining the authorization code from WeChat, call the Authgear callback with the auth code and complete the "Login With WeChat" process.

Here are the detailed steps for iOS, Android and React Native.

### Prerequisite - Setup the Mobile Application \(移动应用\)

1. Register an account in [WeChat Open Platform](https://open.weixin.qq.com/).
2. Create Mobile Application \(移动应用\), fill in information and wait for approval \(It may take few days\).
3. View the application detail page, obtain the `AppID` and `AppSecret` on the top of the page.

    ![wechat-mobile-app-id](../../.gitbook/assets/wechat-mobile-app-id.png)

4. Go to Account Center &gt; Basic information, to obtain the "原始ID".

    ![wechat-account-id](../../.gitbook/assets/wechat-account-id.png)

5. Save those values, we will need them in the section below.

### Update code based on platform

#### Native iOS application

1. Setup Authgear iOS SDK.
2. Follow [iOS接入指南](https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/iOS.html) to setup WeChat SDK. For the coding part, we will further explain in the below steps.
3. `WechatOpenSDK` is Objective-C library. If you are using swift. You will need to create bridging header. To setup bridge header, please check [Importing Objective-C into Swift](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift). Here is the example `WechatOpenSDK-Bridging-Header.h`.

   ```text
    #ifndef WechatOpenSDK_Bridging_Header_h
    #define WechatOpenSDK_Bridging_Header_h

    #import "WXApiObject.h"
    #import "WXApi.h"

    #endif
   ```

4. After setting up the `WechatOpenSDK`, universal link should be enabled in your application. We will need two links for the setup. One is for the WeChat SDK used, another is for the Authgear SDK to trigger delegate function when user click "Login with WeChat" button. Here are the suggestion of the links.
   * **WECHAT\_UNIVERICAL\_LINK**: `https://{YOUR_DOMAIN}/wechat`
   * **WECHAT\_REDICRECT\_URI\_FOR\_AUTHGEAR**: `https://{YOUR_DOMAIN}/open_wechat_app`
5. Login WeChat Open platform, open the application detail page, update the development information iOS section.

   ![wechat-development-information](../../.gitbook/assets/wechat-development-information.png)

   * Fill in "Bundle ID" field with your app bundle id.
   * Fill in "Universal Links" with "WECHAT\_UNIVERICAL\_LINK" above.

6. Login Authgear portal, go to "Single-Sign On" page, then do the following:
   * Enable "Sign in with WeChat \(Mobile/移动应用\)"
   * Fill in "Client ID" with the WeChat "AppID".
   * Fill in "Client Secret" with the WeChat "AppSecret".
   * Fill in "原始 ID" with the WeChat "原始 ID".
   * Add "WECHAT\_REDICRECT\_URI\_FOR\_AUTHGEAR" above into "WeChat redirect URIs" 
   * Click save.
7. Update the code
   * Setup WeChat SDK when app launch

     ```swift
       // Replace WECHAT_APP_ID with wechat app id
       // Replace WECHAT_UNIVERICAL_LINK with the link defined above
       WXApi.registerApp("WECHAT_APP_ID", universalLink: "WECHAT_UNIVERICAL_LINK")
       WXApi.startLog(by: .detail) { log in
           print(#line, "wechat sdk wxapi: " + log)
       }
     ```

   * Setup Authgear delegate and call WeChat SDK when `sendWeChatAuthRequest` is triggered

     ```swift
       // Replace self with the object that you implement the AuthgearDelegate
       authgear.delegate = self

       // Replace WECHAT_APP_ID with wechat app id
       extension MyClass: AuthgearDelegate {
           func sendWeChatAuthRequest(_ state: String) {
               let req = SendAuthReq()
               req.openID = "WECHAT_APP_ID"
               req.scope = "snsapi_userinfo"
               req.state = state
               WXApi.send(req)
           }
       }
     ```

   * Handle universal link

     ```swift
       // Update App Delegate
       func application(_ application: NSApplication,
                       continue userActivity: NSUserActivity,
                       restorationHandler: @escaping ([NSUserActivityRestoring]) -> Void) -> Bool {
           // wechat sdk handle, replace self with object implement WXApiDelegate
           WXApi.handleOpenUniversalLink(userActivity, delegate: self)
           // authgear sdk handle
           return authgear.application(application, continue: userActivity, restorationHandler: restorationHandler)
       }

       // If your app has opted into Scenes, Update Scene Delegate
       func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
           // wechat sdk handle, replace self with object implement WXApiDelegate
           WXApi.handleOpenUniversalLink(userActivity, delegate: self)

           // authgear sdk handle
           authgear.scene(scene, continue: userActivity)
       }

       // Implement WXApiDelegate
       extension MyClass: WXApiDelegate {
           func onReq(_ req: BaseReq) {}
           func onResp(_ resp: BaseResp) {
               // Receive code from WeChat, send callback to authgear
               // by calling `authgear.weChatAuthCallback`
               if resp.isKind(of: SendAuthResp.self) {
                   if resp.errCode == 0 {
                       let _resp = resp as! SendAuthResp
                       if let code = _resp.code, let state = _resp.state {
                           authgear.weChatAuthCallback(code: code, state: state) { result in
                               switch result {
                               case .success():
                                   // send wechat auth callback to authgear successfully
                               case let .failure(error):
                                   // failed to send wechat auth callback to authgear
                               }
                           }
                       }
                   } else {
                       // failed to obtain code from wechat sdk
                   }
               }
           }
       }
     ```

   * Provide `weChatRedirectURI` when calling `authorize` and `promoteAnonymousUser` in authgear sdk

     ```swift
       // Replace "WECHAT_REDICRECT_URI_FOR_AUTHGEAR" with link defined above
       container?.authorize(
           redirectURI: "REDIRECT_URI",
           prompt: "login",
           weChatRedirectURI: "WECHAT_REDICRECT_URI_FOR_AUTHGEAR"
       ) { result in
       }

       // For anonymous user support only
       // Replace "WECHAT_REDICRECT_URI_FOR_AUTHGEAR" with link defined above
       container?.promoteAnonymousUser(
           redirectURI: "REDIRECT_URI",
           weChatRedirectURI: "WECHAT_REDICRECT_URI_FOR_AUTHGEAR"
       ) { result in
       }

       // Open setting page
       // Replace "WECHAT_REDICRECT_URI_FOR_AUTHGEAR" with link defined above
       container?.open(
           page: .settings,
           wechatRedirectURI: "WECHAT_REDICRECT_URI_FOR_AUTHGEAR"
       )
     ```
8. Here is the completed [example](https://github.com/authgear/authgear-sdk-ios/tree/master/example).

#### Native Android application

1. Setup Authgear iOS SDK.
2. Follow [Android接入指南](https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/Android.html) to setup Wechat SDK. For the coding part, we will further explain in the below steps.
3. Login WeChat Open platform, open the application detail page, update the development information Android section.

   ![wechat-development-information](../../.gitbook/assets/wechat-development-information.png)

   * Fill in application signature, you can obtain it with command `keytool -list -v -keystore YOUR_KEYSTORE_FILE_PATH`. WeChat needs the certificate fingerprint in MD5, remove `:` in the fingerprint. It should be string in length 32.
   * Fill in your package name

4. We will need to define a custom url for Authgear SDK to trigger delegate function when user click "Login with WeChat" button. Here is the example, you should update it with your own scheme.
   * **"WECHAT\_REDICRECT\_URI\_FOR\_AUTHGEAR"**: `com.myapp://host/open_wechat_app`
5. Login Authgear portal, go to "Single-Sign On" page, then do the following:
   * Enable "Sign in with WeChat \(Mobile/移动应用\)"
   * Fill in "Client ID" with the WeChat "AppID".
   * Fill in "Client Secret" with the WeChat "AppSecret".
   * Fill in "原始 ID" with the WeChat "原始 ID".
   * Add "WECHAT\_REDICRECT\_URI\_FOR\_AUTHGEAR" above into "WeChat redirect URIs" 
   * Click save.
6. Update the code
   * Update application `AndroidManifest.xml`

     ```markup
       <!-- Your application configuration. Omitted here for brevity -->
       <application>
       <!-- Other activities or entries -->

       <!-- It should be added when setting up Authgear SDK -->
       <activity android:name="com.oursky.authgear.OauthRedirectActivity"
           android:exported="false"
           android:launchMode="singleTask">
           <intent-filter>
               <action android:name="android.intent.action.VIEW" />
               <category android:name="android.intent.category.DEFAULT" />
               <category android:name="android.intent.category.BROWSABLE" />
               <!-- This is the redirectURI, It should be added when setting up Authgear SDK -->
               <data android:scheme="com.myapp"
                   android:host="host"
                   android:pathPrefix="/path"/>
               <!-- Add this for WeChat setup, this should match the WECHAT_REDICRECT_URI_FOR_AUTHGEAR defined above -->
               <data android:scheme="com.myapp"
                   android:host="host"
                   android:pathPrefix="/open_wechat_app"/>
           </intent-filter>
       </activity>

       <!-- Add this for WeChat SDK setup, replace YOUR_PACKAGE_NAME-->
       <activity
           android:name=".wxapi.WXEntryActivity"
           android:exported="true"
           android:label="@string/app_name"
           android:launchMode="singleTask"
           android:taskAffinity="YOUR_PACKAGE_NAME"
           android:theme="@android:style/Theme.Translucent.NoTitleBar"></activity>
       </application>
     ```

   * Configure WeChat SDK

     ```java
       private IWXAPI weChatAPI;

       private setupWeChatSDK() {
           weChatAPI = WXAPIFactory.createWXAPI(app, YOUR_WECHAT_APP_ID, true);
           weChatAPI.registerApp(YOUR_WECHAT_APP_ID);
       }
     ```

   * Setup Authgear delegate

     ```java
       mAuthgear.setDelegate(new AuthgearDelegate() {
           @Override
           public void sendWeChatAuthRequest(String state) {
               if (!weChatAPI.isWXAppInstalled()) {
                   // User have not installed WeChat app, show them the error
                   return;
               }
               SendAuth.Req req = new SendAuth.Req();
               req.scope = "snsapi_userinfo";
               req.state = state;
               weChatAPI.sendReq(req);
           }
       });
     ```

   * Create wxapi directory in the directory named after your package name and create `WXEntryActivity` activity. In `WXEntryActivity`, pass the received intent and the object that implements IWXAPIEventHandler API to the `handleIntent` method of the `IWXAPI` API, as shown below:

     ```java
       api.handleIntent(getIntent(), this);
     ```

     You will be able to receive the authentication code and state in `onResp` method, call Authgear `weChatAuthCallback` with `code` and `state`.

     ```java
       mAuthgear.weChatAuthCallback(code, state, new OnWeChatAuthCallbackListener() {
           @Override
           public void onWeChatAuthCallback() {
           }

           @Override
           public void onWeChatAuthCallbackFailed(Throwable throwable) {
           }
       });
     ```

   * Provide `weChatRedirectURI` when calling `authorize` and `promoteAnonymousUser` in Authgear SDK.

     ```java
       // Replace "WECHAT_REDICRECT_URI_FOR_AUTHGEAR" with link defined above
       AuthorizeOptions options = new AuthorizeOptions(AUTHGEAR_REDIRECT_URI);
       options.setWeChatRedirectURI(WECHAT_REDICRECT_URI_FOR_AUTHGEAR);
       mAuthgear.authorize(options, new OnAuthorizeListener() {
           @Override
           public void onAuthorized(@Nullable AuthorizeResult result) {
           }

           @Override
           public void onAuthorizationFailed(@NonNull Throwable throwable) {
           }
       });

       // For anonymous user support only
       // Replace "WECHAT_REDICRECT_URI_FOR_AUTHGEAR" with link defined above
       PromoteOptions options = new PromoteOptions(AUTHGEAR_REDIRECT_URI);
       options.setWeChatRedirectURI(WECHAT_REDICRECT_URI_FOR_AUTHGEAR);
       mAuthgear.promoteAnonymousUser(options, new OnPromoteAnonymousUserListener() {
           @Override
           public void onPromoted(@NonNull AuthorizeResult result) {
           }

           @Override
           public void onPromotionFailed(@NonNull Throwable throwable) {
           }
       });

       // Open setting page
       // Replace "WECHAT_REDICRECT_URI_FOR_AUTHGEAR" with link defined above
       SettingOptions options = new SettingOptions();
       options.setWeChatRedirectURI(WECHAT_REDICRECT_URI_FOR_AUTHGEAR);
       mAuthgear.open(Page.Settings, options);
     ```
7. Here is the completed [example](https://github.com/authgear/authgear-sdk-android/tree/main/javasample).

#### React Native

1. Setup Authgear SDK
2. Follow [iOS接入指南](https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/iOS.html) and [Android接入指南](https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/Android.html) to setup WeChat SDK. For the coding part, we will further explain in the below steps.
3. In iOS, after setting up the WechatOpenSDK, universal link should be enabled in your application. We will need two links for the setup. One is for the WeChat SDK used, another is for the Authgear SDK to trigger delegate function when user click "Login with WeChat" button. Here are the suggestion of the links.
   * **IOS\_WECHAT\_UNIVERICAL\_LINK**: `https://{YOUR_DOMAIN}/wechat`
   * **IOS\_WECHAT\_REDICRECT\_URI\_FOR\_AUTHGEAR**: `https://{YOUR_DOMAIN}/open_wechat_app`
4. In android, you need to sign your app to use WeChat SDK. Obtain your application signature by running command `keytool -list -v -keystore YOUR_KEYSTORE_FILE_PATH` with your keystore file. WeChat needs the certificate fingerprint in MD5, remove `:` in the fingerprint. It should be string in length 32.
5. Login WeChat Open platform, open the application detail page, update the development information iOS and Android sections.

   ![wechat-development-information](../../.gitbook/assets/wechat-development-information.png)

   * In iOS
     * Fill in "Bundle ID" field with your app bundle id.
     * Fill in "Universal Links" with "IOS\_WECHAT\_UNIVERICAL\_LINK" above.
   * In Android
     * Fill in application signature.
     * Fill in your package name

6. For android, we will need to define a custom url for Authgear SDK to trigger delegate function when user click "Login with WeChat" button. Here is the example, you should update it with your own scheme.
   * **ANDROID\_WECHAT\_REDICRECT\_URI\_FOR\_AUTHGEAR**: `com.myapp://host/open_wechat_app`
7. Login Authgear portal, go to "Single-Sign On" page, then do the following:
   * Enable "Sign in with WeChat \(Mobile/移动应用\)"
   * Fill in "Client ID" with the WeChat "AppID".
   * Fill in "Client Secret" with the WeChat "AppSecret".
   * Fill in "原始 ID" with the WeChat "原始 ID".
   * Add "IOS\_WECHAT\_REDICRECT\_URI\_FOR\_AUTHGEAR" and "ANDROID\_WECHAT\_REDICRECT\_URI\_FOR\_AUTHGEAR" above into "WeChat redirect URIs" 
   * Click save.
8. Update the code
   * In Android, Update application `AndroidManifest.xml`.

     ```markup
       <!-- Your application configuration. Omitted here for brevity -->
       <application>
       <!-- Other activities or entries -->

       <!-- It should be added when setting up Authgear SDK -->
       <activity android:name="com.oursky.authgear.OauthRedirectActivity"
           android:exported="false"
           android:launchMode="singleTask">
           <intent-filter>
               <action android:name="android.intent.action.VIEW" />
               <category android:name="android.intent.category.DEFAULT" />
               <category android:name="android.intent.category.BROWSABLE" />
               <!-- This is the redirectURI, It should be added when setting up Authgear SDK -->
               <data android:scheme="com.myapp"
                   android:host="host"
                   android:pathPrefix="/path"/>
               <!-- Add this for WeChat setup, this should match the WECHAT_REDICRECT_URI_FOR_AUTHGEAR defined above -->
               <data android:scheme="com.myapp"
                   android:host="host"
                   android:pathPrefix="/open_wechat_app"/>
           </intent-filter>
       </activity>

       <!-- Add this for WeChat SDK setup, replace YOUR_PACKAGE_NAME-->
       <activity
           android:name=".wxapi.WXEntryActivity"
           android:exported="true"
           android:label="@string/app_name"
           android:launchMode="singleTask"
           android:taskAffinity="YOUR_PACKAGE_NAME"
           android:theme="@android:style/Theme.Translucent.NoTitleBar"></activity>
       </application>
     ```

   * In iOS, update your App Delegate

     ```text
       - (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray *))restorationHandler {
           [WXApi handleOpenUniversalLink:userActivity delegate:self];
           [AGAuthgearReactNative application:application continueUserActivity:userActivity restorationHandler:restorationHandler];
           return YES;
       }
     ```

   * Provide `weChatRedirectURI` when calling Authgear SDK `authorize` and `promoteAnonymousUser` in js

     ```javascript
       // REPLACE IOS_WECHAT_REDICRECT_URI_FOR_AUTHGEAR and ANDROID_WECHAT_REDICRECT_URI_FOR_AUTHGEAR
       const weChatRedirectURI = Platform.select<string>({
           android: 'ANDROID_WECHAT_REDICRECT_URI_FOR_AUTHGEAR',
           ios: 'IOS_WECHAT_REDICRECT_URI_FOR_AUTHGEAR',
       });

       authgear
           .authorize({
               redirectURI: "REDIRECT_URI",
               weChatRedirectURI: weChatRedirectURI

           });

       // For anonymous user support only
       authgear
           .promoteAnonymousUser({
               redirectURI: "REDIRECT_URI",
               weChatRedirectURI: weChatRedirectURI
           });

       // Open setting page
       authgear
           .open(Page.Settings, {
               weChatRedirectURI: weChatRedirectURI,
           })
     ```

   * Setup Authgear delegate and open WeChat SDK when sendWeChatAuthRequest is triggered

     ```javascript
       authgear.delegate = {
           sendWeChatAuthRequest: (state) => {
               // User click login with WeChat
               // Implement native modules to use WeChat SDK to open 
               // WeChat app for authorization.
               // After obtaining authorization code, call Authgear.weChatAuthCallback
               // to complete the authorization.
               const {WeChatAuth} = NativeModules;
               WeChatAuth.sendWeChatAuthRequest(state)
               .then((result: {code: string; state: string}) => {
                   // Native module sending back the code after login with
                   // WeChat app. Call Authgear.weChatAuthCallback
                   return authgear.weChatAuthCallback(result.code, result.state);
               })
               .then(() => {
                   // Send WeChat callback to authgear successfully
               })
               .catch((err: Error) => {
                   // error ocurred
               });
           }
       }
     ```

   * Implement the NativeModules to use WeChat SDK to open WeChat app for authorization. Here is the completed [example](https://github.com/authgear/authgear-sdk-js/tree/master/example/reactnative).

