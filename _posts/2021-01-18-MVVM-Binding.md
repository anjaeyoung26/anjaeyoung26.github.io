---
title: "MVVM, Binding"
categories: 
   - Design Pattern
tags:
   - MVVM
   - RxSwift
toc: true
toc_sticky: true
toc_label: "목차"
---

MVVM 디자인 패턴에서 RxSwift를 이용한 예제입니다.

<br/>

# 1. Dependency injection

뷰 모델은 뷰 컨트롤러가 소유하며 이를 주입하는 방식은 여러가지 입니다.

<br/>

## 1-1 Initializer injection

뷰 컨트롤러를 초기화할 때 직접 뷰 모델을 주입합니다. 이는 가장 간단하면서 쉬운 방식이지만, 뷰 모델이 주입받는 의존성이 많을수록 보일러 플레이트 코드가 필요합니다.(보일러 플레이트 코드란 적은 양이지만 대체할 수 없고, 여러 곳에서 사용되는 코드 섹션을 의미합니다.)

~~~swift
let authService = AuthService()
let userService = UserService()
let photoService = PhotoService()
let viewModel = ViewModel(authService: authService, 
                          userService: userService, 
                         photoService: photoService)

let viewController = ViewController(viewModel: viewModel)
~~~

단순히 뷰 컨트롤러를 사용하기 위해 위처럼 많은 양의 코드가 필요합니다.

<br/>

## 1-2 Service provider

뷰 모델에서 여러가지 서비스 계층에 의존성을 가질 때, 위처럼 각 서비스를 초기화 시 주입하는 대신 서비스 provider를 주입하는 방식이 있습니다. 뷰 모델은 서비스 계층에 직접 접근하지 않고, provider를 통해 사용하고자 하는 서비스에 접근합니다.

~~~swift
protocol ServiceProviderType: class {
    var authService: AuthServiceType { get }
    var userService: UserServiceType { get }
    var photoService: PhotoServiceType { get }
}

class ServiceProvider: ServiceProviderType {
    lazy var authService: AuthServiceType = AuthService(provider: self)
    lazy var userService: UserServiceType = UserService(provider: self)
    lazy var photoService: PhotoServiceType = PhotoService(provider: self)
}
~~~

위와 같이 provider는 각 서비스를 lazy 하게 소유하고 있습니다. provider를 통해 서비스 계층에 접근하여 사용할 때 할당하는 방식으로 뷰 모델에 서비스를 제공합니다.

뷰 모델은 초기화 시 provider를 주입받아 서비스를 이용할 수 있습니다.

~~~swift
let provider = ServiceProvider()
let viewModel = ViewModel(provider: provider)
let viewControler = ViewController(viewModel: viewModel)
~~~

<br/>

## 1-3 Container

Container에 의존성을 등록하고 사용 시 resolve 하는 방식으로 [Swinject](https://github.com/Swinject/Swinject)가 많이 사용됩니다. 이 방식은 아직 이해하고 있는 과정이여서 내용에서 제외 하겠습니다...

<br/>

# 2. Binding

뷰 컨트롤러로부터 전달된 유저 액션을 `Input`, 그에 따라 모델이 업데이트한 결과를 뷰 컨트롤러에게 전달하는 `Output`으로 구성합니다. 아래는 유저가 프로필 버튼을 클릭하여 서버에서 자신의 정보를 가져와 화면에 표시하는 상황이라고 가정하겠습니다.

~~~swift
protocol ViewModelType {
    associatedtype Input
    associatedtype Output
    
    var input : Input  { get }
    var output: Output { get }
}
~~~

<br/>

## 2-1 Input

뷰 컨트롤러에서 유저가 버튼을 누르는 액션을 바인딩 합니다.(View Controller -> View Model)

~~~swift
btnProfile.rx.tap
    .throttle(.milliseconds(500), scheduler: MainScheduler.instance)
    .bind(viewModel.input.profileButtonTapped)
    .disposed(by: disposeBag)
~~~

throttle 연산자는 첫 이벤트 발생 후 지정된 시간동안 추가로 발생되는 이벤트를 무시합니다. 이는 유저가 버튼을 여러 번 누를 때마다 이벤트가 발생되는 것을 방지합니다.

바인딩된 이벤트는 뷰 모델에서 아래와 같이 처리합니다. 아래와 같이 뷰 컨트롤러로 부터 이벤트가 전달되면 유저 서비스(모델)에서 자신의 정보를 가져와 Output으로 바인딩 합니다.

~~~swift
class ViewModel: ViewModelType {
    struct Input {
        let profileButtonTapped = PublishRelay<Void>()
    }

    struct Output {
        let userInfo = PublishRelay<User>()
    }

    let input = Input()
    let output = Output()
    let disposeBag = DisposeBag()

    init(provider: ServiceProviderType) {
        input.profileButtonTapped
            .flatMap {
                self.provider.userService.fetchMe()
            }
            .bind(to: output.userInfo)
            .disposed(by: disposeBag)
    }
}
~~~

Input, Output을 모두 Relay를 사용한 이유는, Subject는 complete, error 발생 시 구독을 종료하는 반면 Relay는 Dispose 되기 전까지 작동해서 UI 이벤트에 적합하다고 생각합니다.

<br/>

## 2-2 Output

Output으로 바인딩된 정보는 뷰 컨트롤러에서 받아 화면에 표시합니다.

~~~swift
viewModel.output.userInfo
    .subscribe(onNext: { [weak self] userInfo in
        guard let `self` = self else { return }

        self.update(userInfo: userInfo)
    })
    .disposed(by: disposeBag)
~~~

이번 글에서는 MVVM + RxSwift의 간단한 예제를 살펴봤습니다. 아직 저는 MVVM, RxSwift의 무궁무진함을 전부 알지 못하지만 여러가지 개인 프로젝트를 통해 활용성을 맛 봤습니다. 기본적으로 코드의 가독성이 좋아지며, 뷰 컨트롤러 - 뷰 모델 - 모델이 각자의 명확한 역할을 수행하면서 독립성을 가지고, RxSwift의 많은 기능을 통해 비동기 프로그래밍을 간편하게 구현할 수 있습니다.

다음 글에서는 Custom delegate proxy를 통해 기존 델리게이트 동작을 비동기 방식으로 구현하는 방법에 대해 포스팅 하겠습니다.