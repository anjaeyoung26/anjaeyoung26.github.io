---
title: "App Store Distribution - 𝖨"
categories: 
   - Distribution
tags:
   - Distribution
toc: true
toc_sticky: true
toc_label: "목차"
---

어플리케이션의 App Store 배포 과정에 대한 글 입니다.

<br/>

# 1. App ID

https://developer.apple.com/account/resources/identifiers/list 에서 App ID를 등록합니다.   
App ID는 프로젝트의 Bundle Identifier 입니다.

<br/>

# 2. App Store Connect

App Store Connect란 Apple Developer Program 회원에게 어플리케이션 제출 및 관리, TestFlight를 통한 테스트 등의 작업을 도와주는 웹 기반 도구 모음 입니다.

App Store에 기재할 어플리케이션에 대한 설명, 키워드, 등급, 저작권 등을 설정하고 제출을 위해 스크린샷을 추가합니다.
또한 로그인이 필요한 경우, 테스트 계정을 제공하고 리뷰어와 통화할 개발자의 연락처 정보를 기입하는 등의 전반적인 어플리케이션 제출 및 관리를 합니다.

App Store에 제출하기 위해 App Store Connect에 `이름`, `기본 언어`, `번들 ID`를 설정하여 신규 어플리케이션을 등록합니다.  번들 ID는 1 과정에서 설정한 App ID를 선택합니다.

<br/>

# 3. Manual

<br/>

## 3-1 CSR

CSR 이란 Certificate Signing Request(인증서 서명 요청)으로, 인증서 발급을 위해 필요한 정보를 담고있는 인증서 신청 형식의 데이터 입니다. CSR 파일 생성 시 공개 키와 개인 키가 함께 생성되며, CSR 데이터에는 공개 키의 정보가 포함되어 있습니다.

[키체인 접근] - [인증서 지원] - [인증 기관에서 인증서 요청]에 접근하여 아래와 같이 항목을 입력합니다.

1. 사용자 이메일 주소 : 법인 혹은 개인 개발자 계정의 이메일 주소를 입력합니다.
2. 일반 이름 : 아무렇게나 입력해도 됩니다.
3. CA 이메일 주소 : 공란
4. 요청 항목 : 디스크에 기입됨

입력 후 Next - 위치 설정 - 저장을 클릭하여 CSR 파일을 생성합니다. CSR 파일은 추후 인증서를 생성할 때 사용되므로 파일의 위치를 알고있어야 합니다.

<br/>

## 3-2 Certificate

생성한 CSR을 이용하여 https://developer.apple.com/account/resources/certificates/list 에서 개발 및 배포 인증서를 생성합니다.

![dis4](https://user-images.githubusercontent.com/61190690/104118787-de899a80-536e-11eb-87c0-fdfd0a3d9ede.png)

신규 생성한 인증서는 위와 같은 화면에서만 다운로드 받을 수 있으므로 주의합니다. 다운로드 받은 인증서를 더블 클릭하면 키체인 접근에 등록됩니다. 배포 인증서는 `개인 키`와 함께 등록되는데, ~~개인 키가 함께 등록되지 않는 경우 배포를 할 수 없으므로 이전 과정을 되새김질 해야합니다.~~ 개인 키는 Provisioning Profile이 생성된 이후 등록됩니다. 따라서 Provisioning Profile를 개발자 사이트에서 수동으로 생성하거나 혹은 Xcode의 Automatically manage signing을 통해 생성한 후 배포 인증서를 더블 클릭하면 키체인에 배포 인증서와 함께 개인 키가 등록됩니다.

`참고` 생성 시 다운로드 받은 경우가 아닌, 나중에 다시 다운로드를 받는 경우 개인 키가 함께 등록되지 않습니다.(확실 X)   
`참고` 인증서 관련 문제가 발생할 시 Revoke 후 다시 생성하는 방법이 있습니다.

<br/>

## 3-3 Provisioning Profile

Provisioning Profile이란 iOS 디바이스와 인증서를 연결하여 디바이스에 해당 어플리케이션이 설치 및 실행될 수 있도록 합니다.

- 생성 시 App ID, 인증서, 디바이스(개발용 생성 시에만)가 필요합니다.
- 인증서와 마찬가지로 개발, 배포용 두 가지 종류가 있습니다.
- App ID 당 한 개씩 생성할 수 있습니다.

<br/>

## 3-4 적용

Xcode 프로젝트 - [Signing & Capabilities] - [Signing] 에서 `Automatically manage signing`의 체크를 해제하면 Debug(개발), Release(배포) 시 Provisioning Profile과 인증서를 이전 과정에서 다운로드 받은 파일로 직접 설정할 수 있습니다.

<br/>

# 4. Automatic

Xcode 8.0 부터 App ID, Certificate, Provisioning Profile을 자동으로 생성해주는 Automatically manage signing 기능이 추가됐습니다. 

- 이미 생성됐다면 업데이트 합니다.
- Provisioning Profile 생성 과정에서 꼬일 수 있습니다.

<br/>

# 5. Archive

아카이브란 .app 및 App Store에 제출하는데 필요한 정보를 포함하는 디렉토리(.xarchive)를 생성하는 과정 혹은 App Store 배포에서는 .ipa 파일을 생성하여 App Store Connect에 해당 빌드 버전을 업로드하는 과정을 의미합니다.

Xcode 상단 바 - [Product] - [Archive]를 클릭해서 아카이브를 진행합니다.

> Archive가 비활성화 상태라면, 시뮬레이터를 변경한 뒤 진행합니다.

Archive가 완료되면 Organizer가 표시되고, 방금 아카이브가 완료된 정보가 표시됩니다. Distribute App을 클릭하여 배포를 진행합니다.

![arc2](https://user-images.githubusercontent.com/61190690/104119749-0e3ba100-5375-11eb-9aa7-417134f801a2.png)

<br/>

App Store Connect를 선택한 뒤 Next를 클릭합니다.

![arc](https://user-images.githubusercontent.com/61190690/104120742-93768400-537c-11eb-8698-14ea728797bf.png)

<br/>

중간에 3-4 와 동일하게 Signing을 진행하는 과정이 있습니다.

![arc6-1](https://user-images.githubusercontent.com/61190690/104121324-89568480-5380-11eb-86a7-33813e109e38.png)


App Store Connect에 현재 빌드 버전을 업로드 합니다.

![arc9](https://user-images.githubusercontent.com/61190690/104121398-45b04a80-5381-11eb-824e-1747565b91d4.png)


![arc10](https://user-images.githubusercontent.com/61190690/104121402-4812a480-5381-11eb-87e6-a59fa2b04e06.png)

20~30분 후에 App Store Connect에서 업로드된 빌드 버전을 확인할 수 있습니다.
App Store Connect에서 이후 과정은 다음 글에 포스팅 하겠습니다.