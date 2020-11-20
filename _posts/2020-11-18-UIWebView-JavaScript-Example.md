---
title : "UIWebView, JavaScript example"
category :
    - "Web Application"
tag :
    - UIWebView
toc: true
toc_sticky: true
toc_label: "목차"
---

UIWebView를 이용한 Native(iOS) - Web(JavaScript)간 통신하는 예시입니다.

<br/>

## 1. html 파일 생성

~~~css
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
            <input type="button" name="button" value="callNativeFunc" onclick="callNativeFunc()">
    </body>
</html>
~~~

작성한 html 파일을 실행해보면 아래와 같은 웹 페이지가 표시됩니다. html 파일은 Visual Studio Code로 작성하였습니다.

![html](https://user-images.githubusercontent.com/61190690/99492595-37613600-29b1-11eb-9291-87cdda930953.png)

<br/>

## 2. Native -> Web

작성한 html 파일을 웹 뷰에 load 합니다.

~~~swift
NSURL * url = [[NSURL alloc] initWithString:@"file:///Users/yourname/Desktop/Demo-Server-Web-Page.html"]; // URL은 위 이미지에서 확인 가능합니다.

NSURLRequest * urlRequest = [[NSURLRequest alloc] initWithURL:url];

[self.webView loadRequest:urlRequest];
~~~

뷰 추가, Delegate 등과 관련된 작업은 생략했습니다.   
html 파일을 웹 뷰에 load하면 웹 페이지가 아래와 같이 표시됩니다.

<img src="https://user-images.githubusercontent.com/61190690/99493217-2bc23f00-29b2-11eb-9281-01d4366cf50c.png" width="300" height="600">

웹 뷰 하단에 버튼을 추가하고, 버튼을 클릭할 시 html 파일에 구현되어 있는 javascriptFunc 메소드가 호출되도록 구현하겠습니다.

<img src="https://user-images.githubusercontent.com/61190690/99493532-b99e2a00-29b2-11eb-9fdb-488f51641842.png" width="300" height="600">

아래와 같이 버튼의 IBAction을 추가합니다.

~~~swift
- (IBAction)onClickCallJavascriptFuncButton:(id)sender {
    NSString * message = @"Message from Native";
    [self.uiWebView stringByEvaluatingJavaScriptFromString:[NSString stringWithFormat:@"javascriptFunc('%@')", message]];
}
~~~

이후 버튼을 클릭하면 html 파일에 구현되어 있는 javascriptFunc 메소드가 호출됩니다.

<img src="https://user-images.githubusercontent.com/61190690/99493721-100b6880-29b3-11eb-92d8-cc8e39b72594.png" width="300" height="600">

<br/>

## 3. Web -> Native

작성된 html 파일의 하단을 보면, 버튼 클릭 시 html 파일에 구현되어 있는 callNativeFunc 메소드가 호출됩니다.

~~~css
<input type="button" name="button" value="callNativeFunc" onclick="callNativeFunc()">
~~~

callNativeFunc 메소드는 아래와 같이 구현되어 있고, Native에게 "javascriptCall://fromJavaScript:callNativeFunc"를 전달합니다. 전달하는 값은 Native에서 처리하는 방식에 따라 상이하므로, 참고만 하시길 바랍니다.   
제가 보낸 값은 "javascriptCall://Native의 메소드명:파라미터" 입니다.

~~~css
function callNativeFunc() {
    window.location = "javascriptCall://fromJavaScript:callNativeFunc";
}
~~~

Native에서는 웹 뷰가 URL을 load하기 전 거쳐가는 델리게이트 메소드인, shouldStartLoadWithRequest에서 처리해야 합니다.

~~~swift
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{

1   if ([request.URL.absoluteString hasPrefix:@"javascriptCall:"]) {
2       NSArray * arr = [request.URL.absoluteString componentsSeparatedByString:@"://"];
        NSArray * component = [[arr objectAtIndex:1] componentsSeparatedByString:@":"];
        
        NSString * functionName = [[component objectAtIndex:0] stringByAppendingString:@":"];
        NSString * parameter = [component objectAtIndex:1];
        
        SEL selector = NSSelectorFromString(functionName);
        
        if ([self respondsToSelector:selector]) {
3           //#pragma clang diagnostic push
            //#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
4           [self performSelector:selector withObject:parameter];
            //#pragma clang diagnostic pop
            return NO;
        }
    }
    
    return YES;
}
~~~

`1` if ([request.URL.absoluteString hasPrefix:@"javascriptCall:"])로 Native 내에서 웹 뷰에 URL을 load한 것이 아닌, JavaScript로부터 call 된 것을 확인합니다.

`2` Web(JavaScript)로부터 전달된 값에서 메소드명과 파라미터를 추출하는 과정입니다.

즉, 전달된 "javascriptCall://fromJavaScript:callNativeFunc" 에서 메소드명인 "fromJavaScript"와 파라미터인 "callNativeFunc"을 추출합니다.

`3` 주석처리된 부분은 performSelector 부분에서 컴파일러가 표시하는 메모리 leak 경고를 임시적으로 표시되지 않게 합니다.   
if ([self respondsToSelector:selector]) 에서 selector의 유효성을 확인했으므로 경고가 표시되도 지장이 없지만, 보기에 흉하다고 느끼신다면 추가합니다.

`4` 추출한 메소드를 파라미터와 함께 performSelector로 호출합니다.

<br/>

Web(JavaScript)가 호출하고자 하는 Native의 메소드는 아래와 같이 구현되어 있습니다.

~~~swift
- (void)fromJavaScript:(NSString *)message
{
    NSLog(@"%@", message);
}
~~~

단순히 Web(JavaScript)로부터 전달받은 값을 Log 출력합니다.

<br/>

이제 웹 페이지의 버튼을 클릭하면 fromJavaScript 메소드가 호출되어, 전달된 값이 Log 출력되는 것을 확인할 수 있습니다.

![htmlwebView_htmlbutton_click](https://user-images.githubusercontent.com/61190690/99495412-f6b7eb80-29b5-11eb-8df7-ba29d528814b.png)

끝










