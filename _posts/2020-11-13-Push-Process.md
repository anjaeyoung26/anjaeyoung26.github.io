---
title : "Push Process"
category :
    - Push
tag :
    - APNs
---

iOS에서 이루어지는 푸시에 대한 과정에 관한 글입니다. 해당 과정들은 모두 `AppDelegate` 내에서 이루어집니다.

<br/>

# 목차


### 1. 등록

### 2. 푸시를 수신할 때 호출되는 메소드들

<br/>
<br/>

# 1. 등록

<br/>

## 1-1 사용자에게 승인 요구

사용자에게 푸시 알람을 허용할 것인지 물어봅니다.

~~~swift
UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];

[center setDelegate:self];

[center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert | UNAuthorizationOptionSound | UNAuthorizationOptionBadge) completionHandler:^(BOOL granted, NSError * _Nullable error) {
    if (granted) {
        dispatch_async(dispatch_get_main_queue(), ^{
            [application registerForRemoteNotifications];
        });
    } else {
         NSString * errorDescription = [error localizedDescription];
         NSLog(@"didFinishLaunchingWithOptions : %@", errorDescription);
    }
}];
~~~

델리게이트 위임 후, 사용자에게 UNAuthorizationOption에 대한 승인을 얻습니다.
승인에 대한 결과는 completionHandler의 granted에 반환됩니다.

사용자가 승인을 했다면, APNs 서버에 Remote Notifications를 등록합니다.

<br/>

## 1-2 Device Token 수신

정상적으로 등록이 됐다면, 델리게이트 메소드를 통해 Device Token이 전달됩니다.
전달받은 Device Token을 UserDefaults 등에 저장합니다.

~~~swift
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    if (deviceToken) {
        [[NSUserDefaults standardUserDefaults] setObject:deviceToken forKey:@"Device-Token"];
        [[NSUserDefaults standardUserDefaults] synchronize];
    }
}
~~~

만약 등록이 실패했다면, 아래의 메소드가 호출됩니다.

~~~swift
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
    //NSLog(@"%@", error.localizedDescription);
}
~~~

Device Token을 전달받으면 푸시를 수신할 수 있습니다.

<br/>
<br/>

# 2. 푸시를 수신할 때 호출되는 메소드

어플리케이션의 상태에 따라, 그리고 수신한 푸시의 페이로드에 따라 호출되는 메소드에 차이가 있습니다.

<br/>

## 2-1 Not Running

어플리케이션이 실행되지 않을 때 호출되는 메소드 입니다.

수신한 푸시를 통해 어플리케이션을 실행한다면, 당연히 최초로 실행될 떄 호출되는 didFinishLaunchingWithOptions 메소드가 호출됩니다.

~~~swift
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    NSDictionary * remoteNotification = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];

    if (remoteNotification) {
        //...
    }
}
~~~

수신한 푸시에 대한 정보는 launchOptions 딕셔너리의 `UIApplicationLaunchOptionsRemoteNotificationKey` 키 값에 포함되어 전달됩니다.

수신한 푸시를 통해 어플리케이션을 실행한 것이 아닌, 아이콘을 눌러 자체적으로 실행하거나 혹은 Silent 푸시(content-available 를 설정한 푸시를 뜻하며, Background 푸시라고도 한다.)의 경우 호출되는 메소드가 없습니다.

<br/>

## 2-2 Foreground

Foreground는 Active, Inactive 상태를 뜻합니다.

<br/>

∙ 일반적인 푸시를 수신하면 아래와 같은 메소드가 호출됩니다.

~~~swift
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions))completionHandler
{
    NSString * customMessage = [notification.request.content.userInfo objectForKey:@"Custom Message"];

    completionHandler(UNNotificationPresentationOptionAlert);
}
~~~

수신한 푸시에 대한 정보는 `notification.request.content` 에 포함되어 전달됩니다.   
completionHandler로 Presentation 옵션을 설정하여 화면에 표시합니다.

<br/>

∙ Silent 푸시를 수신하면 아래와 같은 메소드가 호출됩니다.

~~~swift
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
~~~

수신한 푸시에 대한 정보는 userInfo 딕셔너리에 포함되어 전달됩니다.  

<br/>

∙ 푸시의 배너를 클릭하면 아래와 같은 메소드가 호출됩니다.

~~~swift
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)(void))completionHandler
~~~

수신한 푸시에 대한 정보는 response.notification.request.content 에 포함되어 전달됩니다.

<br/>

## 2-3 Background

<br/>

∙ Silent 푸시를 수신하면 아래와 같은 메소드가 호출됩니다.

~~~swift
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
~~~

수신한 푸시에 대한 정보는 userInfo 딕셔너리에 포함되어 전달됩니다.  

<br/>

∙ 푸시의 배너를 클릭하면 아래와 같은 메소드가 호출됩니다.

~~~swift
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)(void))completionHandler
~~~

