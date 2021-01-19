---
title : "Build with PlistBuddy"
category :
    - Study
tag :
    - PlistBuddy
toc: true
toc_sticky: true
toc_label: "목차"
---

PlistBuddy란 무엇이고, 이를 통해 빌드업 자동화를 할 수 있는 방법에 관한 글입니다. [참고](https://medium.com/@marksiu/what-is-plistbuddy-76cb4f0c262d)

<br/>

# 1. PlistBuddy 란?

Shell Script로 작성하여 Property list(plist) 파일을 편집할 수 있도록 해주는 Mac OS 내장 프로그램 입니다.

주로 어플리케이션을 업데이트할 때마다 Build Version을 자동으로 변경하고자 할 때 사용합니다.

PlistBuddy에는 Edit Mode, Direct Mode 두 가지 모드가 있습니다.

<br/>

# 2. Edit Mode

편집 모드로 진입해서 명령어를 입력하고, 해당 명령어들을 적용하려면 저장을 합니다.

<br/>

## 2-1 진입

plist 파일의 위치를 입력하여 접근합니다.

~~~
/usr/libexec/PlistBuddy <file>
~~~

<br/>

## 2-2 명령어 입력

~~~
Command: Add :Test integer 1
Command: Add :Test2 string "TEST"
~~~

<br/>

## 2-3 명령어 저장

~~~
Command: Save
~~~

<br/>

## 2-4 끝내기

~~~
Command: Exit
~~~

<br/>

# 3. Direct Mode

다이렉트 모드는 진입과 동시에 명령어를 입력하여 실행합니다.

~~~
/usr/libexec/PlistBuddy -c "Add :Test integer 1" <file>
~~~

Edit Mode와는 다르게, 입력한 명령어를 즉시 실행합니다.

<br/>

# 4. 명령어

Set, Print, Add 등 다수의 명령어가 있습니다.   

터미널에 아래와 같이 입력하면 간단한 설명과 사용법이 출력됩니다.

~~~
/usr/libexec/PlistBuddy --help
~~~

명령어의 사용법은 주로 사용하는 Direct Mode를 기준으로 작성하겠습니다.

<br/>

## 4-1 Set

plist의 key에 값을 설정합니다.   
명령어가 동작하려면 입력한 key 값이 plist 내에 존재해야 합니다.   

~~~
/usr/libexec/PlistBuddy -c "Set :Test string "TEST" "/Users/myUserName/Desktop/Demo/Demo/Info.plist"
~~~

위와 같이 plist 파일의 위치와 함께 명령어를 실행합니다. 결과로 Test라는 키에 TEST 문자열 값이 설정된 것을 확인할 수 있습니다.

![plistbuddy4](https://user-images.githubusercontent.com/61190690/99937571-6872a980-2da9-11eb-884b-9eea307e433a.png)

<br/>

## 4-2 Print

입력한 plist의 key에 해당하는 값을 출력하거나, 변수에 값을 대입할 때 사용합니다.

key에 해당하는 값 출력

~~~
/usr/libexec/PlistBuddy -c "Print :Test" "/Users/myUserName/Desktop/Demo/Demo/Info.plist"
~~~

결과로 아래와 같이 Test라는 키에 설정된 값을 출력하는 것을 확인할 수 있습니다.

![plistbuddy5](https://user-images.githubusercontent.com/61190690/99937995-49c0e280-2daa-11eb-805a-e1aaa5e86b68.png)

변수에 값을 대입

~~~
plistDir=$(/usr/libexec/PlistBuddy -c "Print :Test" "/Users/myUserName/Desktop/Demo/Demo/Info.plist")
~~~

결과로 variable에 "TEST" 라는 문자열 값이 대입되었습니다.   
이를 응용하자면, Script 내에서 Plist 파일의 주소 값을 변수에 저장하여, 아래와 같이 편리하게 값을 설정할 수 있습니다.

~~~
plistDir="${SRCROOT}/Demo/Info.plist"

/usr/libexec/PlistBuddy -c "Set :Test string 'TEST STRING'" "$plistDir"
~~~

SRCROOT는 Xcode내에서 사용할 수 있는, 현재 프로젝트가 위치한 디렉토리를 나타냅니다. [`참고`](https://developer.apple.com/library/archive/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/1-Build_Setting_Reference/build_setting_ref.html#//apple_ref/doc/uid/TP40003931-CH3-SW38)

`✓` Xcode 내에서 Script를 사용할 때, 띄어쓰기를 하면 경로를 찾지 못하는 경우가 있습니다.

<br/>

## 4-3 Add

plist 내에 값을 추가합니다.

~~~
/usr/libexec/PlistBuddy -c "Add :Test2 string 'TEST1'" "/Users/anjaeyeong/Desktop/Demo/Demo/Info.plist"
~~~

<br/>

## 4-4 Delete

plist 내에 값을 제거합니다.

~~~
/usr/libexec/PlistBuddy -c "Delete :Test" "/Users/anjaeyeong/Desktop/Demo/Demo/Info.plist"
~~~

이외에도 Help, Copy, Merge 등 여러가지 명령어가 존재합니다.

<br/>

# 5. 빌드업 자동화

Xcode 내에서는 Build, Run, Test 등 다양한 상황에 따라 Script를 사용할 수 있도록 제공합니다.

Xcode 상단 바에 [Product] - [Scheme] - [Edit Scheme] 에서 추가할 수 있습니다.

![plistbuddy1](https://user-images.githubusercontent.com/61190690/99935586-593d2d00-2da4-11eb-8f36-bcddb3ed01e4.png)

Build, Run, Test, Profile, Analyze, Archive 각 상황의 전, 후, 도중에 실행할 Script를 삽입할 수 있습니다.

![plistbuddy2](https://user-images.githubusercontent.com/61190690/99935659-88539e80-2da4-11eb-9454-71d89cc86613.png)

하단의 `+` 를 클릭해서 추가합니다.

![plistbuddy3](https://user-images.githubusercontent.com/61190690/99935838-fbf5ab80-2da4-11eb-8f82-6488f57171e2.png)

Shell 파일의 위치를 입력하거나, 직접 스크립트를 작성하여 추가합니다.

예시로, 빌드를 진행할 때 마다 CFBundleVersion 값을 증가시키는 Script를 추가할 수 있습니다.

~~~
plistDir="${SRCROOT}/Demo/Info.plist"

currentBuildNumber=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "$plistDir")
currentBuildNumber=$(($currentBuildNumber + 1))

/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $currentBuildNumber" "$plistDir"
~~~

위 Script를 Build의 Pre-actions에 추가하면 빌드가 진행될 때마다 CFBundleVersion 값이 자동으로 증가합니다.



