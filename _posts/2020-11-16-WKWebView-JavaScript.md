---
title : "WKWebView, JavaScript"
category :
    - "Web Application"
tag :
    - "WKWebView"
---

WKWebView 이용한 하이브리드 어플리케이션에 관한 글입니다.   
WKWebView 바탕으로 Native(iOS) <-> Web(JavaScript)간의 통신을 하여 서로의 기능을 호출할 수 있습니다.

UIWebView와의 차이점은 아래와 같습니다.

- UIWebView는 어플리케이션 프로세스 내에서 실행됩니다. 하지만 WKWebView는 어플리케이션 프로세스 밖에서 실행되므로, 별도의 메모리를 사용합니다.
- WKWebView는 JavaScript와 비동기적으로 통신합니다.
- WKWebView는 데이터를 쿠키에 저장하지 않으므로, 로딩 시간이 항상 동일합니다.
- UIWebView는 iOS 2.0, WKWebView는 iOS 8.0에 도입됐습니다.

자세한 차이점은 Zedd님 블로그의 [iOS ) UIWebView와 WKWebView의 차이](https://zeddios.tistory.com/332) 에서 확인할 수 있습니다.

<br/>

## 1. NSAppTransportSecurity 추가

Info.plist에 아래와 같은 값을 추가합니다.

~~~
<key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
    </dict>
~~~

위 방식은 모든 도메인을 허용하는 것으로, 안전하게 사용하길 원한다면 아래와 같이 제외시킬 도메인을 추가합니다.

~~~
<key>NSAppTransportSecurity</key> 
<dict>
    <key>NSExceptionDomains</key> 
    <dict> 
        <key>www.naver.com</key>
        <dict> 
            <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key> 
            <true/> 
        </dict> 
    </dict>
 </dict>
~~~

<br/>

## 2. Framework 추가

![wkwebkit](https://user-images.githubusercontent.com/61190690/99464989-b720dd80-297c-11eb-95ec-75594e395baf.png)

위와 같이 [Build Phases] - [Link Binary With Libraries] 에서 검색하여 프레임워크를 추가합니다.

<br/>

## 3. Import

~~~swift
#import <WebKit/WebKit.h>
~~~

<br/>

## 4. Delegate 채택

~~~swift
@interface ViewController: UIViewController<WKUIDelegate, WKNavigationDelegate, WKScriptMessageHandler>
~~~

- WKUIDelgate : JavaScript, 기타 플러그인 이벤트를 캐치하여 동작합니다. 웹 페이지의 기본적인 사용자 인터페이스 요소를 제공합니다.
- WKNavigationDelegate : 웹 페이지의 start, loading, finish, error 등의 트리거 이벤트를 캐치하여 사용자 정의 동작을 구현할 수 있습니다.
- WKScriptMessageHandler : 웹 페이지에서 실행되는 JavaScript Message를 수신합니다. 웹 페이지에서 메시지가 수신될 때 호출되는 userContentController를 정의하고 있습니다.

<br/>

## 5. Delegate 위임

~~~swift
@property (nonatomic, retain) IBOutlet WKWebView *webView;

WKWebViewConfiguration * config = [[WKWebViewConfiguration alloc] init];
WKUserContentController * contentController = [[WKUserContentController alloc] init];

[contentController addScriptMessageHandler:self name:@"myFunctionName"]; // JavaScript가 Native를 호출할 때 사용할 이름을 추가합니다.

[config setUserContentController:contentController];

[webView setUIDelegate:self];
[webView setNavigationDelegate:self];
~~~

<br/>

## 6. 뷰 추가

웹 뷰를 서브 뷰로 추가합니다. xib 혹은 스토리보드에서도 인터페이스 요소로 제공됩니다.

~~~swift
CGRect frame = [[UIScreen mainScreen] bounds];

webView = [[WKWebView alloc] initWithFrame:frame configuration:config];

[[self view] addSubview:webView];
~~~

<br/>

## 7. 통신

### 7-1 Web -> Native

아래의 메소드를 통해 Javascript에서 Native를 호출합니다.

~~~css
window.webkit.messageHandlers.myFunctionName1.postMessage("parameter1");
~~~

~~~swift
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
    // [message body]는 id(Any in Swift) 타입입니다. 딕셔너리 혹은 배열, 문자열 등 다양한 값이 전달될 수 있습니다.
    if ([[message name] isEqualToString:@"myFunctionName1"])
    {
        NSString * strFunctionName = [[message body] stringValue]; // parameter1
    }
}
~~~

<br/>

아래와 같이 두 개의 파라미터를 포함하여 Native를 호출할 수 있습니다.

~~~css
window.webkit.messageHandlers.myFuctionName2.postMessage(firstParam:'param', secondParam:'param');
~~~

Native에서는 딕셔너리 형태로 형변환하여 사용합니다.

~~~swift
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
    // [message body]는 id(Any in Swift) 타입입니다. 딕셔너리 혹은 배열, 문자열 등 다양한 값이 전달될 수 있습니다.
    if ([[message name] isEqualToString:@"myFunctionName2"])
    {
        NSDictionary * dictionary = (NSDictionary *) [message body];

        NSString * firstParam = [dictionary objectForKey:@"firstParam"];
        NSString * secondParam = [dictionary objectForKey:@"secondParam"];
    }
}
~~~

<br/>

### 7-2 Native -> Web

JavaScript 내에 아래와 같은 함수가 구현되어 있습니다.

~~~css
function myFunction(firstMsg, secondMsg) {
    alert(firstMsg + ' ' + secondMsg);
}
~~~

이를 Native에서 호출하고자 할 때 아래와 같은 메소드를 사용합니다.

~~~swift
[webView evaluateJavaScript:[NSString stringWithFormat:@"myFunction('%@', '%@');", @"First Message from Native", @"Second Message from Native"] completionHandler:^{
    //...
}];
~~~

<br/>

## 8. Script 삽입

WKWebView에는 UIWebView와 차별화된 최상단 혹은 최하단에 스크립트를 삽입하는 기능이 있습니다.

~~~swift
WKUserScript * script = [[WKUserScript alloc] initWithSource:@"alert('load')"
                                               injectionTime:WKUserScriptInjectionTimeAtDocumentStart
                                            forMainFrameOnly:NO];

[contentController addUserScript:script];
~~~

- injectionTime : 최상단 혹은 최하단 중 어느곳에 스크립트를 추가할지 설정합니다.
- forMainFrameOnly : 모든 프레임에 적용할 것인지 설정합니다.

추가 시 웹 뷰가 load 되었을 때, 삽입한 스크립트가 실행됩니다.