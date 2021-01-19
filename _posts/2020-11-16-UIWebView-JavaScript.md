---
title : "UIWebView, JavaScript"
category :
    - "Web Application"
tag :
    - UIWebView
toc: true
toc_sticky: true
toc_label: "목차"
---

UIWebView 이용한 하이브리드 어플리케이션에 관한 글입니다.   
UIWebView 바탕으로 Native(iOS) <-> Web(JavaScript)간의 통신을 하여 서로의 기능을 호출할 수 있습니다.

<br/>

# 1. NSAppTransportSecurity 추가

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

# 2. Delegate 채택

~~~swift
@interface ViewController: UIViewController<UIWebViewDelegate>
~~~

<br/>

# 3. Delegate 위임

~~~swift
@property (nonatomic, retain) IBOutlet UIWebView *webView;

_webView.delegate = self;
~~~

<br/>

# 4. 뷰 추가

웹 뷰를 서브 뷰로 추가합니다. xib 혹은 스토리보드에서도 인터페이스 요소로 제공됩니다.

~~~swift
CGRect frame = [[UIScreen mainScreen] bounds];

webView = [[UIWebView alloc] initWithFrame:frame];

[[self view] addSubview:webView];
~~~

<br/>

# 5. 통신

<br/>

## 5-1 Web -> Native

아래의 메소드를 통해 Javascript에서 Native를 호출합니다.

~~~css
function jsFunction() {
    window.location = "javascriptcall://myFunctionName:parameter1";
}
~~~

아래의 메소드는 웹 뷰가 URL을 load되기 전 호출됩니다.

~~~swift
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest::(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    NSString * requestURLString = [[request URL] absoluteString]; // 웹 뷰가 load 하고자 하는 URL을 가져옵니다.

    if ([requestURLString hasPrefix:@"javascriptcall:"]) {
        NSArray * arr = [requestURLString componentsSeparatedByString:@"://"];
        NSString * component = [arr objectAtIndex:1];

        arr = [component componentsSeparatedByString:@":"]; 

        NSString * strFunctionName = [[[arr objectAtIndex:0] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding] stringByAppendingString:@":"]; // myFunctionName:
        NSString * strParameterName = [[arr objectAtIndex:1] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding]; // parameter1

        if ([self respondsToSelector:NSSelectorFromString(strFuncName)])
        {
            [self performSelector:NSSelectorFromString(strFunctionName) withObject:strParameterName];
        }
    }
}
~~~

<br/>

## 5-2 Native -> Web

Javascript 내에 아래와 같은 함수가 구현되어 있습니다.

~~~css
function myFunction(firstMsg, secondMsg) {
    alert(firstMsg + ' ' + secondMsg);
}
~~~

이를 Native에서 호출하고자 할 때 아래와 같은 메소드를 사용합니다.

~~~swift
[_webView stringByEvaluatingJavaScriptFromString:[NSString stringWithFormat:@"myFunction('%@', '%@');", @"First Message from Native", @"Second Message from Native"]];
~~~

JavaScript에 구현되어 있는 myFunction 함수를 호출합니다. 괄호안에 함수의 파라미터를 전달할 수 있습니다.   
웹 뷰의 Delegate 메소드가 아닌 곳에서 호출이 가능합니다.

<br/>

## 5-3 JavaScript 파일 내용

JavaScript 파일 내용입니다. 웹에 대한 지식이 없어서 구글 검색을 통해 Visual Studio Code로 간단하게 만들었습니다.

~~~html
<html>
    <head>
        <title> button </title>
        <meta charset = "utf-8">

        <script>
            function javascriptFunc(message) {
                alert(message);
            }

            function callNativeFunc() {
                window.location = "javascriptCall://fromJavaScript:callNativeFunc";
            }
        </script>
    </head>
    <body>
    <form action="a.html">
            <input type="button" name="button" value="callNativeFunc" onclick="callNativeFunc()"> <!-- onclick="alert('ALERT')"> -->
    </body>
</html>
~~~

<br/>

## 6. 웹 뷰 내에서 Alert 커스터마이징

JavaScript에서 alert 하는 경우, 알림창의 제목에 해당 페이지의 주소가 표시됩니다.

![javaalert1](https://user-images.githubusercontent.com/61190690/99223519-fdb0f380-2827-11eb-86c7-047381a57d2a.png){: .align-center}

이를 방지하고자 UIWebView의 카테고리를 생성합니다.

아래 메소드는 JavaScript로부터 Alert panel(예, 아니오와 같은 사용자의 액션에 따라 처리하는 알림이 아닌, 단순한 메시지 전달의 목적으로 표시하는 알림을 말합니다.)이 표시될 때 호출됩니다.

~~~swift
- (void)webView:(UIWebView *)sender runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(id)frame 
{
    UIAlertView *jsAlert = [[[UIAlertView alloc] initWithTitle:@"test" message:message delegate:nil cancelButtonTitle:@"확인" otherButtonTitles:nil] autorelease];
    [jsAlert show];
}
~~~

<br/>

아래 메소드는 JavaScript로부터 Confirm panel이 표시될 때 호출됩니다.

~~~swift
- (void)webView:(UIWebView *)sender runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(id)frame
{
    UIAlertView *confirmDiag = [[UIAlertView alloc] initWithTitle:nil message:message delegate:self cancelButtonTitle:@"예" otherButtonTitles:@"아니오", nil];
    [confirmDiag show];
}
~~~

UIAlertView의 Delegate를 위임할 시 사용자가 버튼을 클릭하면 아래의 메소드가 호출됩니다.   
사용자가 클릭한 버튼에 따라 처리를 할 수 있습니다.

~~~swift
- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
    if (buttonIndex == 0) {
        //...
    } eles if (buttonIndex == 1) {
        //...
    }
}
~~~


![javaalert2](https://user-images.githubusercontent.com/61190690/99223522-00134d80-2828-11eb-9760-d6e82ac55ea9.png){: .align-center}

✴︎ iOS 12에서 UIWebView가 deprecated, WKWebView로 JavaScript와 통신하는 방법은 추후에 포스팅 하겠습니다.




