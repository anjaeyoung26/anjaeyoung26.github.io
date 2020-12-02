---
title : "URLScheme"
category :
    - Study
tag :
    - URLScheme
toc: true
toc_sticky: true
toc_label: "목차"
---

URLScheme에 관한 글입니다.

<br/>

## 1. URLScheme 란?

서로 다른 어플리케이션간 통신을 할 수 있는 수단을 제공하기 위해 미리 정하는 URL 입니다.   

<br/>

## 2. 내장 스키마

Apple에서 기본적으로 제공하는 내장 스키마가 있습니다.

~~~swift
let url = URL(string: "http://www.naver.com")

guard UIApplication.sharedApplication.canOpenURL(url) else { return }
UIApplication.sharedApplication.open(url, options:[:], completionHandler:nil)
~~~

위와 같이 http로 시작하는 URL을 Safari로 표시하는 기능도 내장 스키마의 한 종류입니다.

### 2-1 종류

|URL|내용|
|------|---|
| http://웹사이트 URL, https://웹사이트 URL | Safari 앱을 통해 웹사이트 표시 |
| mailto:이메일 주소 | 메일 앱을 통해 새로운 메일 작성 화면 표시 |
| tel:전화번호 | 전화 연결 |
| sms:전화번호 | 메시지 앱을 통해 새로운 메시지 입력 화면 표시 |
| facetime://FaceTime ID | FaceTime 연결 |
| facetime-audio://FaceTime ID | FaceTime Audio 연결 |
| http://maps.apple.com/?q=검색어, http://maps.apple.com/?ll=위도, 경도 | 지도 앱을 통해 지역 표시 |
| itms://itunes.apple.com/us/app/apple-store/앱 ID | App Store 앱을 통해 앱 정보 표시 |

[참고](http://blog.naver.com/horajjan/220893876471)

### 2-2 사용

4 번째 항목의 스키마를 사용하여 디바이스 내의 메시지 어플리케이션을 실행하기 위한 코드입니다.

~~~swift
let url = URL(string: "sms:")

guard UIApplication.sharedApplication.canOpenURL(url) else { return }
UIApplication.sharedApplication.open(url, options:[:], completionHandler:nil)
~~~

<br/>

![urlscheme2](https://user-images.githubusercontent.com/61190690/100412110-05d62200-30b7-11eb-8b6d-ae29c586833b.gif)

간단하게 메시지 어플리케이션을 실행할 수 있으며 "sms:010-1234-1234" 형식으로 보내는 대상의 전화번호를 입력할 수 있습니다.

<br/>

## 3. 커스텀 스키마

커스텀 스키마를 추가하여 원하는 어플리케이션과 통신할 수 있습니다.

임의로 호출하는 어플리케이션을 Client, 호출당하는 어플리케이션을 Server라고 했을 때,   
각각의 준비사항을 설명하겠습니다.

### 3-1 Client

Info.plist에 아래와 같은 항목을 추가합니다.

~~~
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>demo-server</string>
</array>
~~~

demo-server는 Server 어플리케이션을 호출하기 위한 스키마의 이름이며, 이는 Server 어플리케이션과 같은 이름으로 추가해야 합니다.

Info.plist에 직접 값을 추가하지 않고, [Info] - [URLTypes] 에서 추가해도 됩니다.

<br/>

### 3-2 Server

Info.plist에 아래와 같은 항목을 추가합니다.

~~~
<key>CFBundleURLTypes</key>
<array>
	<dict>
		<key>CFBundleURLNames</key>
		<string>bundle url name</string>
		<key>CFBundleURLSchemes</key>
		<array>
			<string>demo-server</string>
		</array>
	</dict>
</array>
~~~

CFBundleURLNames에는 Client 어플리케이션의 Bundle Identifier를 입력합니다.

CFBundleURLSchemes에는 앞서 Client 어플리케이션에서 설정한 스키마의 이름과 동일한 값을 입력합니다.

<br/>

### 3-3 Client -> Server

Client, Server 어플리케이션 모두 설정을 완료했으면, 아래와 같은 코드로 Client 어플리케이션에서 Server 어플리케이션을 실행할 수 있습니다.

~~~swift
let url = URL(string: "demo-server://")

guard UIApplication.sharedApplication.canOpenURL(url) else { return }
UIApplication.sharedApplication.open(url, options:[:], completionHandler:nil)
~~~

<br/>

![urlscheme3](https://user-images.githubusercontent.com/61190690/100416380-ad585200-30c1-11eb-8a83-e7d3d819a03a.gif)

"demo-server://name=jaeyoungan" 과 같은 URL로 추가적인 데이터를 전달할 수 있습니다.

Server는 Client로부터 호출되어 실행될 때 전달된 데이터를 AppDelegate의 application:didFinishLaunchingWithOptions 메소드에서 추출할 수 있습니다.

~~~swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
   let name = launchOptions["name"] as? String // "jaeyoungan"

    return true
}
~~~

<br/>

### 3-4 Client <-> Server

서로 간 호출을 위해서는, 양쪽 모두 Info.plist에 LSApplicationQueriesSchemes 와 CFBundleURLTypes을 추가해야 합니다. 이후 과정은 동일합니다.









