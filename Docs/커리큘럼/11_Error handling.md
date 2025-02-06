## 에러를 처리하기

여기에 두가지 에러 메커니즘이 있습니다. 

### Observable에서의 비동기 에러처리 메커니즘

에러처리는 매우 간단합니다.
한 시퀀스가 에러로 종료되면 , 해당 시퀀스에 의존하는 모든 시퀀스도 에러로 종료됩니다.
이는 일반적인 [단락(short - circuit)](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%B6%94%EA%B0%80%20%ED%95%99%EC%8A%B5%EC%9E%90%EB%A3%8C/Short_Circuit_Logic.md) 논리 입니다.

`Obsevable`의 실패에서 복구하려면 `catch` 연산자를 사용할수 있습니다.
`catch` 연산자에는 복구 동작을 세밀하게 지정할수 있는 다양한 오버로드가 있습니다. 

또한 시퀀스가 에러를 발생햇을때 다시 시도할수 있는 retry 연산자도 제공됩니다. 

RxSwift의 에러전파 방식 
-  RxSwift에서 한 Observable이 에러로 종료되면, 해당 Observable에 의존하는 모든 체인도 에러로 종료됩니다. 
- 이는 short - circuit logic으로 에러가 발생하면 즉시 처리 흐름이 중단됩니다.

```swift
let observable = Observable<Int>.create { observer in 
	observer.onNext(1)
	observer.onNext(2)
	observer.onNext(NSError(domain: "Test", code: 1, userInfo: nil))
	observer.onNext(3)
	return Disposable.create()
}

observable
	.map { $0 * 2 }
	.subscribe(
		onNext: { print($0) },
		onError: { print("Error: \($0)" ) }
	
	)
	
```
> 여기서 3 * 2인  6은 출력되지않고
```
2
4
Error: Error Domain=Test Code=1 "(null)"
```
> 이것 처럼 나온다.

</br>
</br>

> 만약 catch 연산자를 사용한다면 Observable에서 에러가 발생했을때, 이를 처리하고 대체 Observable을 반환하여 복구할수 있습니다.
```swift
let observable = Observable<Int>.create { observer in
    observer.onNext(1)
    observer.onNext(2)
    observer.onError(NSError(domain: "Test", code: 1, userInfo: nil)) // 에러 발생
    return Disposables.create()
}

observable
    .catch { _ in Observable.just(100) } // 에러 발생 시 대체 값 제공
    .subscribe(
        onNext: { print($0) },
        onError: { print("Error: \($0)") }
    )
```
> catch 연산자로 에러가 발생했을때 대체할 값을 제공함으로써
```
1
2
100
```
> 이것처럼 나올것이다.

</br>

> 만약 retry 연산자를 사용한다면 에러가 발생했을때, 해당시퀀스를 다시 구독하여 실행을 재시도 한다.
```swift
var attempt = 1

let observable = Observable<Int>.create { observer in
    print("Attempt \(attempt)")
    if attempt < 3 {
        attempt += 1
        observer.onError(NSError(domain: "Test", code: 1, userInfo: nil)) // 에러 발생
    } else {
        observer.onNext(1)
        observer.onNext(2)
        observer.onCompleted()
    }
    return Disposables.create()
}

observable
    .retry(3) // 최대 3번 재시도
    .subscribe(
        onNext: { print($0) },
        onError: { print("Error: \($0)") }
    )
```
> 이것을 이렇게 출력될 것이다.

```swift
Attempt 1 // 출력이 된후 if문으로 빠지고 Attempt는 1증가후 Error로 빠짐 
Attempt 2 // 재구독 하여서 1이증가한 2가 출력이 되고 if문으로 빠지고 1증가후 Error로빠짐 
Attempt 3 // 1이증가하여 3이 출력이 되고 else문으로 빠짐
1 // else에서 1을 방출함
2 //이어서 2를 방출할고 onCompleted로 종료 
```


| 연산자   | 설명                           | 주요사용사례                       |
| ----- | ---------------------------- | ---------------------------- |
| catch | 에러 발생시 대체 Observable로 방출(복구) | 에러 발생시 기본값 제공 , 로컬 데이터로 대체   |
| retry | 에러 발생시 시퀀스를 다시 구독하여 재시도      | 네트워크 요청 또는 비동기 작업의 에러발생시 재시도 |


</br>
</br>
</br>

### Hooks와 기본 에러 처리

RxSwift는 전역 Hook을 제공하여, 사용자가 `onError` 핸들러를 제공하지 않은경우에 기본 에러처리 메커니즘을 제공합니다.

필요한 경우 , `Hooks.defaultErrorHandler`  에 사용자 정의 클로저 를 설정하여 처리되지않은 에러를 다루는 방법을 지정할수 있습니다.

예를들어 , 스택트레이스를 기록하거나 추적되지않은 에러를 분석 시스템으로 전송하는데 사용할수 있습니다. 

</br>

> [!NOTE]
>  즉 우리가 Observable 체인에서 에러가 발생할때 어떻게 할것인지 에대한 것을 클로저로 정의해두긴 하지만  이렇게 명시적으로 처리 하지않은 경우에대한 전역에러의 처리를 담당한다.

</br>


> 사용시나리오 ( 명시적 에러 처리)
```swift
Observable<String>.create { observer in
    observer.onError(NSError(domain: "Network", code: 404, userInfo: nil))
    return Disposables.create()
}
.subscribe(
    onNext: { print($0) },
    onError: { error in
        print("Network Error: \(error)")
        showAlert("Network Error", message: error.localizedDescription)
    }
)
```
> 이렇게 되었을때 구독작업에서 Error가 올때 어떻게 처리할것인지 명시적으로 다룰수 있다. 

</br>
</br>

> 사용시나리오( 전역적 애러처리 )
```swift
Hooks.defaultErrorHandler = { error, context in
    print("Unhandled Error: \(error)")
    // 에러를 서버로 전송
    ErrorReportingService.send(error: error)
}
```
> 공통적안 애로로직을 처리하기위함
```swift
Observable<String>.create { observer in
    observer.onError(NSError(domain: "Test", code: 500, userInfo: nil))
    return Disposables.create()
}
.subscribe(onNext: { print($0) })
```
> onError를 생략한것에대해서 일괄적인 처리가 가능 

</br>
</br>

이점:
- 코드 중복이 방지됨
- 중앙집중관리 할수있음 ( 에러로직을 한곳에서 관리하므로 유지보수가 용이함 )
- 모든 에러를 전역적으로 포착하여서 분석시스템으로 전송가능 
한계: 
- 특정 Observable에 대한세부처리부족 
- 우선순위: 이미onError가 설정되어 있는경우 전역 핸들러는 호출되지 않음.


결론:
- 전역적으로 동일한 에러 처리 로직이 필요한 경우 Hooks.defaultErrorHandler.

-  특정 Observable에 대한 세부 에러 처리 로직이 필요한 경우 onError.

</br>
</br>
</br>

- 기본적으로 Hooks.defaultErrorHandler 는 DEBUG 모드에서 전달된 에러를 출력하며, RELEASE 모드에서는 아무작업도 수행하지 않습니다. 
- 그러나, 이동작에 추가설정을 적용할수 있습니다.
- 상세한 호출 스택 로깅을 활성화 하려면 활성화 하려면 Hooks.recordCallStackOnError 플래그를 true로 설정하세요.


- 기본적으로,DEBUG 모드에서는 현재 Thread.callStackSymbols 를 반환하며, RELEASE 모드에서는 빈 스택 트레이스를 추적합니다.
- 이동작을 사용자 정의하려면 Hooks.customCaptureSubscriptionCallStack 을 사용하여 직접 구현을 덮어쓸수 있습니다.

- 기본적으로 Rxswift에서는 DEBUG / RELEASE 모드에서의 에러를 구분합니다.
- RELEASE 모드에서는 기본적으로 아무수행을 하지 않습니다.

스택/ 큐에대한 내용정리가 필요할것으로 보임 











