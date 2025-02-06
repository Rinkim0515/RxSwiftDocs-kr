# Disposing

관찰되는 시퀀스가 종료되는 또 다른 방법이 있습니다.  
시퀀스 작업이 끝났고, 다가올 요소들을 계산하는 데 할당된 모든 리소스를 해제하려는 경우, 구독(subscription)에서 dispose를 호출할 수 있습니다.

<br/>

> 다음은 interval 연산자를 사용한 예제입니다.
```swift
let scheduler = SerialDispatchQueueScheduler(qos: .default)
let subscription = Observable<Int>.interval(.milliseconds(300), scheduler: scheduler)
    .subscribe { event in
        print(event)
    }

Thread.sleep(forTimeInterval: 2.0)

subscription.dispose()
```
이것은 이렇게 출력될것 입니다.

```
0
1
2
3
4
5
```

보통은 수동으로 dispose를 호출하지 않기를 권장합니다.이는 교육적 목적의 예제일 뿐입니다.  
수동으로 dispose를 호출하는 것은 일반적으로 냄새나는 코드로 간주됩니다. 대신, 구독을 해제하기 위한 더 나은 방법들이 있습니다.   
예를 들어, **DisposeBag**, takeUntil **연산자**, 또는 기타 다른 메커니즘을 사용할 수 있습니다.

<br/>

>disposeBag의 예시 
```
let disposeBag = DisposeBag()

let observable = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)

observable
    .subscribe(onNext: { value in
        print("Next value: \(value)")
    })
    .disposed(by: disposeBag)
```

<br/>

>takeUntil의 예시
```
let stopSignal = PublishSubject<Void>()

let observable = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)

observable
    .takeUntil(stopSignal)
    .subscribe(onNext: { value in
        print("Next value: \(value)")
    })
    .disposed(by: disposeBag)

// 5초 후에 stopSignal을 방출
DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    stopSignal.onNext(())
    print("Subscription stopped!")
}
```

<br/>
<br/>


> 다른 메커니즘
- **Lifecycle 기반 처리**: 예를 들어 Combine이나 RxCocoa의 바인딩 작업은 bind를 사용하면 생명 주기에 따라 자동으로 해제됩니다.
- **ViewModel과 Dispose 관리**: MVVM 패턴에서는 ViewModel 내에서 Observable을 관리하고 ViewController에 불필요한 구독을 넘기지 않는 방식으로 리소스를 정리할 수 있습니다.

그렇다면, 이 코드가 dispose 호출 이후에 무언가를 출력할 수 있을까요?  
정답은: **상황에 따라 다릅니다.**

- 만약 스케줄러가 **직렬 스케줄러(serial scheduler)**이고, dispose가 동일한 직렬 스케줄러에서 호출된다면, 정답은 **“아니오”**입니다.
- 그렇지 않은 경우, 정답은 **“예”**입니다.

스케줄러에 대해 더 알고 싶다면 [여기](%EB%A7%81%ED%81%AC)를 참조하세요. (추후 작성 )


<br/>


두 개의 프로세스가 병렬로 실행되고 있다고 할때
- 하나는 요소를 생성하고
-  다른 하나는 구독(subscription)을 해제(dispose)합니다

이 두 프로세스가 서로 다른 스케줄러에서 실행되는 경우 “그 이후에 무언가가 출력될 수 있나?“라는 질문은 의미가 없습니다.

RxSwift에서는 **병렬 실행**이 가능하므로 요소 생성(Producer)과 구독 해제(Dispose)가 독립적으로 실행될 수 있습니다.

- 이때 두 작업이 서로 다른 **스케줄러**에서 실행되면, 작업의 순서를 보장할 수 없기 때문에 dispose 이후에 무언가 출력될 가능성은 없습니다.
- 스케줄러의 특성에 따라 작업이 언제 실행될지 달라지기 때문입니다.

```swift
let subscription = Observable<Int>.interval(.milliseconds(300), scheduler: scheduler)
            .observe(on: MainScheduler.instance)
            .subscribe { event in
                print(event)
            }
// ....

subscription.dispose() // called from main thread
```

dispose 호출이 반환된 이후에는 아무것도 출력되지 않습니다. 이는 보장된 동작입니다.   
(subscription.dispose()가 호출되면, 해당 구독은 즉시 해제되므로 **더 이상 이벤트를 받지 않습니다**.)


<br/>

```swift
let subscription = Observable<Int>.interval(.milliseconds(300), scheduler: scheduler)
            .observe(on: MainScheduler.instance)
            .subscribe { event in
                print(event)
            }

// ...

subscription.dispose() // executing on same `serialScheduler`
```

같은 serialScheduler에서 dispose가 호출되면, 더 이상 아무것도 출력되지 않습니다. 이는 **보장된 동작**입니다.

> [!NOTE]
> 일단 SerialScheduler 와 MainScheduler는 둘다 직렬적으로 처리하는것은 동일하지만  
> 차이점은 SerialScheduler는 특정큐에서 사용하기 위함 이고 병렬적으로 들어 오더라도 직렬적으로 처리하는 특성이 있기때문에 백그라운드 처리를 해야할때 사용하면 좋다  
> 일반적으로 파일I/O 나 네트워크 처리같은경우는 백그라운드 스레드로 사용하는것이 효율적이고 일반적이지만  
> 작업순서가 중요한 비UI 작업 처리 거나 경우에따라서는 직렬처리가 필요한 경우에 는 Serial 로 요청할수도 있다.


## Dispose Bags

disposeBag은 RxSwift 에서 ARC와 유사한 동작을 제공하기위해 사용됩니다.

disposeBag이 **해제(deallocate)** 될 때, 그 안에 추가된 모든 **Disposable**에 대해 자동으로 dispose를 호출합니다.

DisposeBag 자체에는 dispose 메서드가 없으며, 명시적으로 dispose를 호출할 수 없도록 설계되었습니다.

즉시 리소스를 정리해야 할 경우, 새 **DisposeBag**을 생성하면 됩니다.

1. **자동 리소스 관리**:

- DisposeBag은 RxSwift의 구독(Subscription) 관리에서 **자동 정리** 역할을 수행합니다.
- DisposeBag이 메모리에서 해제될 때, 관련된 구독도 함께 해제되어 **메모리 누수**를 방지합니다.

2. **ARC와 유사한 동작**:

- DisposeBag은 ARC처럼 동작하여, 객체의 생명주기 동안 Disposable을 관리합니다.
- 객체가 소멸되면(deallocated) 구독도 자동으로 해제됩니다.

<br/>

self.disposeBag = DisposeBag()는 기존 DisposeBag을 새로운 인스턴스로 교체합니다.

- 이 작업은 기존 DisposeBag에 저장된 모든 Disposable을 정리(dispose)합니다.

> 일반적으로 VC의 라이프사이클을 따라서 관리하는것으로 알고 있는데 왜 이렇게 해야하는걸까 ? 

DisposeBag은 일반적으로 VC의 라이프 사이클에 따라 자동으로 Disposable을 관리하지만 
특정상황에서는 self.disposeBag = DisposeBag() 로 기존작업을  강제로 취소 하고 새로운 상태를 초기화 해야할 필요가 있음 

그에 따른 예시로 
1. 기존 Disposable의 즉시 해제: VC는 계속 유지되지만 특정작업을 취소하고 새로운 작업을 설정해야할때 
2. 리소스 효율성: 더이상 필요하지않은 작업을 방치하지않고 정리하여 메모리 누수를 방지
3. 동적 요구사항 처리: 화면의 Refresh 혹은 데이터 재요청등 특정조건 상황에서 상태를 초기화'
4. 코드 간결성: DisposeBag = DisposeBag()으로 기존 Disposable을 정리하는로직을 간단히 구현


```swift
var disposeBag = DisposeBag()

Observable.from([1, 2, 3])
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)  // 기존 DisposeBag에 추가

// DisposeBag 교체
disposeBag = DisposeBag()  // 기존 Disposable들이 해제됨
```


명시적으로 리소스를 정리해야 할 경우, CompositeDisposable을 사용하세요.   
CompositeDisposable은 dispose **메서드를 호출하면 즉시 Disposable을 해제**합니다.
dispose가 호출된 이후에 추가된 Disposable도 **즉시 해제**됩니다.

## Take until

> takeUntil 연산자는 **Observable의 수명을 특정 이벤트나 조건과 연동**하여, 조건이 만족되면 구독을 자동으로 해제합니다.

```swift
sequence
    .take(until: self.rx.deallocated)
    .subscribe {
        print($0)
    }
```
self.rx.deallocated와 함께 사용하면 **객체가 메모리에서 해제될 때 구독을 종료**합니다.





