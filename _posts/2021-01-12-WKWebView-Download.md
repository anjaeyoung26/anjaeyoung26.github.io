---
title : "WKWebView, Download"
category :
    - "Hybrid"
tag :
    - "WKWebView"
toc: true
toc_sticky: true
toc_label: "목차"
---

WKWebView에서 파일 다운로드하는 과정에 대한 글입니다.

<br/>

# 1. WKNavigationDelegate

웹 뷰에서 페이지 관련 이벤트(start, load, finish, error)를 캐치하고, 각 이벤트에 사용자 지정 동작을 구현하는데 사용합니다.

![download1](https://user-images.githubusercontent.com/61190690/104268776-12340400-54d8-11eb-942c-2eb970bbb94c.jpeg){: .align-center}

위 그림에서 표시된 부분과 같이 첨부 파일을 클릭하여 다운로드 받는 기능은 기본적으로 웹 뷰에 없습니다.   
WKNavigationDelegate 메소드를 통해서 다운로드 기능을 추가해야 합니다.

<br/>

> webView:decidePolicyForNavigationResponse:decisionHandler:

웹 뷰에서 네비게이션 탐색 요청이 발생하고 그에 대한 응답이 도착하면 콘텐츠로 이동할지 여부를 결정할 수 있습니다.   
그림에서 파일을 클릭하면(초록색 표시), 위 메소드가 호출되고, 응답에 파일과 관련된 정보가 담겨있습니다.

<br/>

## 1-1 Response Casting

Delegate 메소드의 응답(WKNavigationResponse)를 NSURLResponse로 캐스팅 합니다.

~~~objc
NSURLResponse * response = [navigationResponse response];
~~~

<br/>

## 1-2 MIME

캐스팅한 NSURLResponse에서 파일의 MIME 타입을 추출합니다.

~~~objc
NSString * mimeType = [response MIMEType];
~~~

추출한 MIME 타입에 따라 처리를 할 수 있습니다.

~~~objc
if ([mimeType isEqualToString:@"image/jpeg"]) {
    
} else if ([mimeType isEqualToString:@"application/pdf"]) {

}
~~~

<br/>

## 1-3 File name

~~~objc
NSString * fileName = [response suggestedFilename];
~~~

suggestedFilename는 응답 Header의 Content-Disposition에서 파일의 이름을 추출합니다.

`Content-Disposition` : 응답 Body를 웹 브라우저가 어떻게 표시할지 명시한 헤더입니다.

- 웹 페이지 화면에 표시

~~~objc
Content-Disposition: inline
~~~

- 다운로드 파일

~~~objc
Content-Disposition: attachment; filename="filename.jpg"
~~~

<br/>

## 1-4 Download

~~~objc
NSMutableURLRequest * request =[[NSMutableURLRequest requestWithURL:[response URL]]];

NSURLSessionDownloadTask * downloadTask = [[NSURLSession sharedSession] downloadTaskWithRequest:request completionHandler:^(NSURL * location, NSURLResponse * response, NSError * error) {
    if ([httpResponse statusCode] == 200 && error == nil) {
        NSError * errReplace = nil;
        NSError * errMove = nil;
        NSURL * documentDirectory = [fileManager URLForDirectory:NSDocumentDirectory inDomain:NSUserDomainMask appropriateForURL:nil create:YES error:&error];
        NSURL * fileURL = [documentDirectory URLByAppendingPathComponent:fileName];

        if ([fileManager fileExistsAtPath:[fileURL path]]) {
            NSLog(@"File already exists. Replace a new one.");
            [fileManager replaceItemAtURL:fileURL withItemAtURL:location backupItemName:nil options:NSFileManagerItemReplacementUsingNewMetadataOnly resultingItemURL:nil error:&errReplace];
        } else {
            NSLog(@"Create a new file. Move to document directory.");
            [fileManager moveItemAtURL:location toURL:fileURL error:&errMove];
        }

        if (errReplace) {
            NSLog(@"Replace file failed with error : %@", [errReplace localizedDescription]);
        }
                    
        if (errMove) {
            NSLog(@"Move file failed with error : %@", [errMove localizedDescription]);
        }
                    
        decisionHandler(WKNavigationResponsePolicyCancel);
    } else {
        NSLog(@"Download task failed with error : %@", [error localizedDescription]);
                    
        decisionHandler(WKNavigationResponsePolicyCancel);
    }
}];

[downloadTask resume];
~~~

`NSURLSessionDownloadTask`는 Temp Directory에 파일을 저장합니다. Temp Directory는 Document Directory와 동일한 상위 폴더에 위치하지만, 다운로드가 완료되는 즉시 파일을 옮기지 않으면 삭제됩니다. 또한 Apple iOS 파일 시스템의 권장 사항에서 사용자 생성 파일은 Document Directory에 저장할 것을 권장하니 파일을 옮깁니다.

File Manager를 통해 파일이 이미 존재하는 경우, 새로 다운로드 받은 파일로 대체합니다. 파일을 다운로드 받는 경우, 해당 URL을 load 할 필요가 없으니, decisionHandler를 통해 취소합니다.

하지만 위 코드를 입력하고 실행해도 에러는 발생하지 않지만, 다운로드 받은 파일이 '파일' 어플리케이션에서 보이지 않습니다.   
Info.plist에 값을 추가해야 합니다.

<br/>

# 2. Info.plist

다운로드 받은 파일은 '파일' 어플리케이션에 노출된다면 다른 개발자의 어플리케이션에서도 사용될 수 있으므로, 기본적으로 노출되지 않습니다.   

하지만 Info.plist에 아래와 같은 값을 추가하면 다운로드 받은 파일을 '파일' 어플리케이션에서 확인할 수 있습니다.

~~~
<key>UIFileSharingEnabled</key>
<true/>
<key>LSSupportsOpeningDocumentsInPlace</key>
<true/>
~~~
[참고](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/uid/TP40009252-SW20)

`UIFileSharingEnabled` : iTunes에서 파일을 공유할 수 있도록 지원합니다.   
`LSSupportsOpeningDocumentsInPlace` : 복사본이 아닌 파일 공급자의 원본 문서를 열 수 있도록 합니다.

두 키 값을 YES(true)로 설정 시 Document Directory에 있는 파일이 '파일' 어플리케이션 혹은 Document Browser에 나타납니다.

