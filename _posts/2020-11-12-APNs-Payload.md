---
title: "APNs-Payload"
category:
    - Push
tag:
    - APNs
toc: true
toc_sticky: true
toc_label: "목차"
---

APNs의 Payload에 관한 글입니다.   
Payload의 구조를 중점적으로 포스팅 하겠습니다.

<br/>

# 1. 특징

APNs에서 푸시를 요청할 때, 푸시에 관련된 내용을 담는 곳입니다. Apple 측에서 지정한 최대 허용 크기가 있으며, 지정된 항목에 값을 넣습니다.

<br/>

## 1-1 크기

∙ Token-based APNs provider API

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;일반 : 4KB(4096 bytes)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Voice over Internet Protocol(VOIP) : 5KB(5120 bytes)

∙ Legacy APNs binary interface

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;일반 : 2KB(2048 bytes)

Legacy APNs binary interface는 APNs의 Certificate-based 방식을 뜻하며, 2021년 3월 31일 부로 지원을 중단하니 참고만 하시길 바랍니다. 관련된 내용이 궁금하면 [링크](https://developer.apple.com/kr/news/)를 참고하세요.

<br/>

## 1-2 형태

APNs의 Payload는 JSON 형태입니다.

<br/>
<br/>

# 2. 구조

일반적인 구조는 아래와 같습니다.

~~~
{
    "aps" : { "alert" : "Message" }
}
~~~

aps 딕셔너리는 Apple에서 지정한 항목에 값을 지정하여 푸시에 대한 옵션을 설정합니다. 설정할 수 있는 항목은 아래와 같습니다.

<img width="745" alt="aps1" src="https://user-images.githubusercontent.com/61190690/98885617-b2b87880-24d5-11eb-92f4-553e2abf17e4.png">

<br/>

i. alert : 알림의 제목, 내용 등 사용자에게 알림을 표시하는 값입니다. alert에 설정할 수 있는 항목은 아래와 같습니다.

<img width="742" alt="aps2" src="https://user-images.githubusercontent.com/61190690/98885821-12af1f00-24d6-11eb-8e6a-e856bad3ed9d.png">

ii. sound, badge 등 기타 항목들은 설명에 잘 나와있습니다.

iii. 실제로 사용하는 페이로드의 형태는 아래와 같습니다.

~~~
{
    "aps" : {
        "alert" : {
            "loc-key" : "LOCALIZED KEY",
            "loc-args": [ "ARGS1", "ARGS2" ]
        },
        "sound" : "ping.aiff"
    },
    "custom" : "data"
}
~~~

loc-key 와 loc-args는 프로젝트의 Localizable.strings 파일의 loc-key에 해당하는 키 값을 찾아서, 해당 키의 value에 있는 가변 문자열 값을 loc-args로 대체합니다. 만약 Localizable.string에 아래와 같이 정의되어 있다면,

~~~
"LOCALIZED KEY" = "%@ and %@";
~~~

가변 문자열이 loc-args의 값으로 대체되어 "ARGS1 and ARGS2" 라는 문자열이 완성됩니다.

custom은 푸시를 수신한 곳에서 사용할 수 있는 사용자 정의 데이터 입니다. custom 이라는 이름은 임의의 값으로 정해진 형식이 아닙니다.

~~~swift
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions))completionHandler
{
    NSString * customMessage = [notification.request.content.userInfo objectForKey:@"custom"];
    NSLog(@"willPresentNotification : %@", customMessage); // "data" 출력
    
    completionHandler(UNNotificationPresentationOptionAlert);
}
~~~

위와 같이 푸시를 수신했을 때 호출되는 메소드를 통해 전달된 사용자 정의 데이터를 추출할 수 있습니다.

iv. content-available를 통해 `Silent Push`를 사용할 수 있습니다.

사용자에게 알리지 않고, 어플리케이션을 백 그라운드 상태에서 깨워서 application:didReceiveRemoteNotification:fetchCompletionHandler를 호출하여 콘텐츠 다운로드 등을 수행합니다.

Silent Push를 사용하기 위해서는 Payload의 aps에 alert, sound, badge가 포함되지 않아야 합니다.
만약 포함될 경우, 비정상적으로 처리됩니다.

최대 30초의 작업 시간이 주어지며, 작업을 마친 후 Handler가 제 시간 내에 호출되지 않으면 어플리케이션이 일시 중지 됩니다. [참고](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/pushing_background_updates_to_your_app?language=objc)

<br/>

이외에도 Actionable Notification을 구현하기 위해 category를 담을 수 있지만, 관련된 내용은 추후에 포스팅 하겠습니다.

끝

