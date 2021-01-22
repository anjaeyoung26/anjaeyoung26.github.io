---
title: "WKWebView, Alert"
categories: 
   - Hybrid
tags:
   - WKWebView
toc: true
toc_sticky: true
toc_label: "목차"
---

WKWebView와 JavaScript 연동 시 alert 예제 입니다.

<br/>

# 1. alert() not working

정확히 말하면 alert() not showing 입니다. WKWebView에서 alert() 동작을 하는 JavaScript 함수를 호출하면 아무런 동작을 하지 않습니다. UIWebView는 단순히 호출하면 알림창을 표시하지만, WKWebView는 아닙니다.

~~~
function example() {
    alert('alert message');
}
~~~

위와 같은 JavaScript의 함수를 호출하기 위해 UIWebView는 아래의 코드를 실행했습니다.

~~~swift
uiWebView.stringByEvaluatingJavaScriptFromString("example()")
~~~

WKWebView에서 동일한 방식으로 실행하면 동작하지 않습니다.

~~~swift
wkWebview.evaluateJavaScript("example()")
~~~

<br/>

# 2. WKUIDelegate

## 2-1 Alert

WKWebView는 웹 뷰에서 alert 동작이 실행될 때 아래와 같은 메소드에서 처리해야 합니다. [참고](https://developer.apple.com/documentation/webkit/wkwebview/3656442-evaluatejavascript)

~~~swift
optional func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void)
~~~

위 메소드는 WKUIDelegate 프로토콜에 정의되어 있고, optional 타입이므로 필수적으로 구현하지 않아도 됩니다.   
JavaScript의 example 함수가 호출되면서 "alert message"가 message 파라미터에 전달됩니다.

~~~swift
func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void) {
   let alertController = UIAlertController(title: "ALERT", message: message, preferredStyle: .alert)
   let alertAction = UIAlertAction(title: "OK", style: .cancle) { _ in
      completionHandler()
   }

   alertController.addAction(alertAction)

   DispatchQueue.main.async {
      self.present(alertController, animated: true, completion: nil)
   }
}
~~~

주의할 점은 동작을 완료하면 completionHandler를 통해 escaping 해야합니다. 그렇지 않으면 crash가 발생합니다.

<br/>

## 2-2 Confirm, Cancel

추가적으로 JavaScript의 alert 뿐만 아니라, cancel, confirm를 처리할 시 아래와 같은 메소드에서 처리해야 합니다. [참고](https://developer.apple.com/documentation/webkit/wkuidelegate/1536489-webview)

~~~swift
optional func webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (Bool) -> Void)
~~~

alert를 처리하는 메소드와 동일하게 optional 타입의 메소드 입니다.   
마찬가지로 함수가 호출되면서 메시지가 message 파라미터에 전달되며 completionHandler를 통해 확인, 취소를 할 수 있습니다.

~~~swift
func webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (Bool) -> Void) {
   let alertController = UIAlertController(title: "ALERT", message: message, perferredStyle: .alert)
   let cancelAction = UIAlertAction(title: "Cancel", style: .cancel) { _ in
      completionHandler(false)
   }
   let okAction = UIAlertAction(title: "OK", style: .default) { _ in
      completionHandler(true)
   }

   alertController.addAction(cancelAction)
   alertController.addAction(okAction)

   DispatchQueue.main.async {
      self.present(alertController, animated: true, completion: nil)
   }
}
~~~