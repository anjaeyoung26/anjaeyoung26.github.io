---
title : "Rich Push Notification"
category :
    - Push
tag :
    - APNs
toc: true
toc_sticky: true
toc_label: "목차"
---

iOS 10에 탄생한 Rich Push Notification 입니다. 푸시에 담겨져 있는 URL을 통해 다운로드 하거나, 푸시를 3D Touch를 통해 상세하게 볼 수 있는 화면을 커스터마이징 할 수 있습니다. 이번 포스팅은 Rich Push Notification의 특징과 동작 과정 및 등록 과정에 관한 글입니다.

<br/>

# 1. 특징

Rich Push Notification은 App Extension의 한 종류로, 별도의 어플리케이션으로 취급하기 때문에, Rich Push Notification만의 Bundle Identifier를 소유합니다.

종류는 Notification Service Extension, Notification Content Extension이 있습니다.

<br/>

## 1-1 Notification Service Extension

화면에 표시되기 전 추가적으로 해야할 동작을 구현합니다. 주 용도는 푸시에 담긴 URL을 통해 request를 보내서 다운로드하여 표시하는 것입니다. 하지만 한 가지 제약사항이 있습니다. 추가적인 동작이 `30초 안에` 마무리되어야 합니다. 30초 안에 끝나지 않아도 크래쉬가 발생하지 않고 별도의 메소드가 호출됩니다.

<br/>

## 1-2 Notification Content Extension

푸시를 3D Touch를 통해 상세히 보여지는 화면을 커스터마이징 합니다.

<br/>
<br/>
<br/>

# 2. 동작 과정

![richpush](https://user-images.githubusercontent.com/61190690/98899303-ae4e8880-24f2-11eb-9709-bb73dbd01fca.png)   
[출처](https://swifting.io/blog/2016/08/22/23-notifications-in-ios-10/)

Provider가 APNs에게 푸시를 요청하고, 전달되기 전에 추가한 App Extension을 거쳐갑니다.
위 그림과 같이 Server side Application에서는 "지금 보내는 푸시는 App Extension을 거쳐갈 것입니다." 라고 명시해야 합니다.

<br/>

## 어떻게 명시를 할까?

Server side Application에서는 푸시의 페이로드에 `mutable-content`의 값을 1로 설정해야 합니다.
그래야 App Extension을 거쳐가도록 진행됩니다. 예를들면 아래와 같이 페이로드를 구성합니다.

~~~
{
    "aps" : { 
        "alert" : {
            "title" : "title",
            "body" : "body" 
        },
        "mutable-content" : 1
    }
}
~~~

APNs 서버에 위 그림과 같은 형태의 푸시를 요청하면 푸시를 수신하고 화면에 표시되기 전에 App Extension을 거쳐서 가공(?)됩니다.

<br/>
<br/>
<br/>

# 3. 등록 과정

등록 과정은 Rich Push Notification의 종류인 Notification Service Extension과 Notification Content Extension을 등록하는 과정을 설명하겠습니다.

<br/>

## 3-1 Notification Service Extension

(1) Xcode 상단 메뉴의 [File] - [New] - [Target] - [Notification Service Extension] 추가

![serviceextension](https://user-images.githubusercontent.com/61190690/98901793-b65cf700-24f7-11eb-9dba-d8449c26924e.png)

추가 시 NotificationService 클래스 파일과 plist 가 생성됩니다. 앞서 말했듯이 App Extension은 별도의 어플리케이션으로 처리하기 때문에 plist도 함께 생성됩니다.

![serviceextension2](https://user-images.githubusercontent.com/61190690/98901815-c5dc4000-24f7-11eb-93b0-21d9e5a75106.png)

NotificationService.m 파일에서 두 가지 메소드를 확인할 수 있습니다.

~~~swift
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler;

- (void)serviceExtensionTimeWillExpire;
~~~

추가적인 동작은 didReceiveNotificationRequest 메소드에서 구현하며, 구현한 동작의 소요시간이 30초가 경과했을 때 serviceExtensionTimeWillExpire 메소드가 호출됩니다.


(2) NotificationService를 Build Target으로 하여 빌드 진행

![serviceextension3](https://user-images.githubusercontent.com/61190690/98902529-3b94db80-24f9-11eb-8b17-47f53b381d0b.png)

진행 시 NotificationService.appex 파일이 생성됩니다.

![appex](https://user-images.githubusercontent.com/61190690/98902336-dfca5280-24f8-11eb-8cb3-e7275987c4cb.png)

<br/>

(3) 프로젝트의 [General] - [Frameworks, Libraries and Embedded Content]에 NotificationService.appex 파일을 추가합니다.

![serviceextension4](https://user-images.githubusercontent.com/61190690/98902725-9c241880-24f9-11eb-94cc-baa7aa23b7d1.png)

<br/>

## 3-2 Notification Content Extension

(1) Xcode 상단 메뉴의 [File] - [New] - [Target] - [Notification Content Extension] 추가

![contentextension](https://user-images.githubusercontent.com/61190690/98901781-ae04bc00-24f7-11eb-96c6-2c1adbd2db11.png)

추가 시 뷰 컨트롤러와 스토리보드 파일, plist 파일이 생성됩니다.

![contentextension2](https://user-images.githubusercontent.com/61190690/98902896-e4433b00-24f9-11eb-80a2-fb2823d83ae6.png)

대강 아시겠지만, MainInterface.storyboard 파일에서 상세 화면을 커스터마이징 합니다.

NotificationViewController.m 파일에서 두 가지 메소드를 확인할 수 있습니다.

~~~swift
- (void)viewDidLoad;

- (void)didReceiveNotification:(UNNotification *)notification;
~~~

viewDidLoad는 일반적인 뷰 컨트롤러의 라이프 사이클 메소드이며, didReceiveNotification 메소드는 푸시가 수신됬을 때 동작을 구현합니다. 아래와 같이 푸시에 포함된 사용자 정의 데이터를 화면에 표시하는 등의 동작을 합니다.

~~~swift
- (void)didReceiveNotification:(UNNotification *)notification {
    NSDictionary * userInfo = notification.request.content.userInfo;
    NSString * message = [userInfo objectForKey:@"custom"];
    
    self.label.text = [NSString stringWithFormat:@"[Content Extension] %@", message];
}
~~~

<br/>

(2) NotificationContent를 Build Target으로 하여 빌드 진행

![contentextension3](https://user-images.githubusercontent.com/61190690/98902840-cbd32080-24f9-11eb-91f7-eff6e05c52fe.png)

진행 시 NotificationContent.appex 파일이 생성됩니다.

![appex](https://user-images.githubusercontent.com/61190690/98902336-dfca5280-24f8-11eb-8cb3-e7275987c4cb.png)

<br/>

(3) 프로젝트의 [General] - [Frameworks, Libraries and Embedded Content]에 NotificationContent.appex 파일을 추가합니다.

![contentextension4](https://user-images.githubusercontent.com/61190690/98903350-bf9b9300-24fa-11eb-84a6-c8e95ee1d1f0.png)

<br/>

# 4. 푸시가 정상적으로 수신이 안될 때

plist에 아래와 같은 값을 추가합니다.

~~~
<key>NSAppTransportSecurity</key>
<dict>
	<key>NSAllowsArbitraryLoads</key>
	<true/>
</dict>
~~~

이로써 Rich Push Notification의 대략적인 설명을 마쳤습니다.

다음 포스팅은 node.js를 통해 직접 디바이스에 푸시를 보내는 과정을 설명하겠습니다.

끝