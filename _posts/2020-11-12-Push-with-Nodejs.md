---
title : "Push with Node.js"
category :
    - Push
tag :
    - APNs
toc: true
toc_sticky: true
toc_label: "목차"
---

Node.js를 이용해 직접 디바이스에 푸시를 보내는 방법에 대해 설명하겠습니다.

<br/>

# 1. 준비 사항

## 1-1 Node.js를 설치합니다.

당연한 얘기지만, Node.js를 설치합니다. https://nodejs.org/ko/download/ 에서 설치할 수 있습니다.

<br/>

## 1-2 apn을 설치합니다.

터미널에서 아래와 같이 입력하여 설치합니다.

~~~
mkdir apns
cd apns
npm init --yes
npm install apn --save
~~~

진행 후 /Users/이름/ 으로 이동하면, apns 폴더와, node_modules 등의 하위 파일이 생성된 것을 확인할 수 있습니다.

<br/>

## 1-3 p8 파일을 apns 폴더로 옮깁니다.

필수적인 과정은 아니지만 node.js 파일 내에서 p8 파일의 경로를 입력해야 하므로 편리한 방향으로..

<br/>

## 1-4 apns 폴더에 .js 파일을 생성합니다.

생성한 후 아래와 같이 값을 입력합니다.


~~~
var apn = require('apn');

// Set up apn with the APNs Auth Key
var apnProvider = new apn.Provider({  
     token: {
        key: 'apns.p8', // Path to the key p8 file
        keyId: 'ABCDE12345', // The Key ID of the p8 file (available at https://developer.apple.com/account/ios/certificate/key)
        teamId: 'ABCDE12345', // The Team ID of your Apple Developer Account (available at https://developer.apple.com/account/#/membership/)
    },
    production: false // Set to true if sending a notification to a production iOS app
});

// Enter the device token from the Xcode console
var deviceToken = '5311839E985FA01B56E7AD74444C0157F7F71A2745D0FB50DED665E0E882';

// Prepare a new notification
var notification = new apn.Notification();

// Specify your iOS app's Bundle ID (accessible within the project editor)
notification.topic = 'my.bundle.id';

// Set expiration to 1 hour from now (in case device is offline)
notification.expiry = Math.floor(Date.now() / 1000) + 3600;

// Set app badge indicator
notification.badge = 3;

// Play ping.aiff sound when the notification is received
notification.sound = 'ping.aiff';

// Display the following message (the actual notification text, supports emoji)
notification.alert = 'Hello World \u270C';

// Send any extra payload data with the notification which will be accessible to your app in didReceiveRemoteNotification
notification.payload = {id: 123};

// Actually send the notification
apnProvider.send(notification, deviceToken).then(function(result) {  
    // Check the result for any failed devices
    console.log(result);
});
~~~

필수적으로 수정되야 할 항목은 `key, keyId, teamId, deviceToken, topic` 입니다.

key : p8 파일의 경로를 입력합니다. 앞서 p8 파일을 apns 폴더로 옮겼으므로, 파일의 이름만 입력합니다.   
keyId : APNs 등록 당시 발급받았던 키의 ID 입니다. [URL](https://developer.apple.com/account/ios/certificate/key)   
teamId : 개발자 계정의 Team ID 입니다. [URL](https://developer.apple.com/account/#/membership/)   
topic : 푸시를 수신할 어플리케이션의 Bundle Identifier 입니다.   
deviceToken : 현재 디바이스의 토큰 입니다. 디바이스 토큰을 얻는 과정은 아래와 같습니다.

~~~swift
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
    [center setDelegate:self];
    [center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert | UNAuthorizationOptionSound | UNAuthorizationOptionBadge)
                          completionHandler:^(BOOL granted, NSError * _Nullable error) {
        if (granted) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [application registerForRemoteNotifications];
            });
        } else {
            NSString * errorDescription = [error localizedDescription];
            NSLog(@"didFinishLaunchingWithOptions : %@", errorDescription);
        }
    }];
    
    return YES;
}

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSString * tokenString = @"";

    if (deviceToken) {
        const unsigned *tokenBytes = [deviceToken bytes];
        tokenString = [NSString stringWithFormat:@"%08x%08x%08x%08x%08x%08x%08x%08x",
                              ntohl(tokenBytes[0]), ntohl(tokenBytes[1]), ntohl(tokenBytes[2]),
                              ntohl(tokenBytes[3]), ntohl(tokenBytes[4]), ntohl(tokenBytes[5]),
                              ntohl(tokenBytes[6]), ntohl(tokenBytes[7])];
        NSLog(@"deviceToken : %@", tokenString);
    }
}
~~~

registerForRemoteNotification 메소드를 호출함으로써, 정상적으로 등록이 됬다면, didRegisterForRemoteNotificationsWithDeviceToken 메소드가 호출되어 디바이스 토큰을 전달받습니다.

Rich Push Notification, Background Modes, Actionable Notification를 사용하고 싶다면 아래와 같이 값을 설정합니다.

~~~swift
notification.mutableContent = 1;
notification.contentAvailable = 1;
notification.category = "YOUR CATEGORY";
~~~

alert에 loc-key, loc-args 혹은 title, body를 설정하고 싶다면 아래와 같이 값을 설정합니다.

~~~swift
notification.alert = {
	"title": "Title",
	"body": "Body"
};

notification.alert = {
    "loc-key" : "LOCALIZED KEY",
    "loc-args": [ "ARGS1", "ARGS2" ]
};
~~~

<br/>

각 항목을 입력한 뒤 터미널에서(apns 폴더에 접근한 상태여야 합니다.) 아래와 같이 입력합니다.

~~~
node 파일이름.js
~~~

Node.js 파일이 실행되면서 디바이스에 푸시가 수신됩니다.

