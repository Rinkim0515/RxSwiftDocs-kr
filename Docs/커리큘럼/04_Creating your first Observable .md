# Observable 생성의 핵심개념(Creating your first Observable)

observable에 대해서 알아야할 중요한점이 하나 있습니다.

observable이 생성된다고 해서 곧바로 작업을 수행하는것은 아닙니다. 

지연실행(`Lazy execution`) 방식으로 동작하고 단순히 작업이 어떻게 수행될지 정의하는 객체일뿐 생성만으로는 어떠한 작업도 실행되지 않으며 Observer가 구독할때 비로소 시작됨

- `Observable`은 지연 실행의 특성을 가집니다.  단순히 생성하는 것만으로는 아무작업도 수행하지않으며,  데이터 생성이나 방출은 특정조건이 충족될때, 즉 구독(subscribe)이 이뤄질때 시작이됩니다. 이는 불필요한 연산을 방지하고 필요할때만 작업을 수행하도록 설계된 효율적인 구조
- `Observable`은 여러가지의 방식으로 요소를 생성할수 있습니다. 일부는 부수효과를 일으킬수 있으며, 일부는 마우스 이벤트와 같은 기존의 실행중인 프로세스에 접근할수도 있습니다.

이러한 데이터는 내부적으로 생성 되거나 , 외부 환경에서 발생한 이벤트를 기반으로 생성될수 있습니다. 

-  observable은 개발자가 정의한 새로운 데이터를 내부적으로 생성하고 Observer에게 전달할수 있습니다.이경우 observabledms 데이터 시퀀스 생성 역할을 하고 일반적인 데이터 흐름을 정의합니다.
-  observable은 외부환경에서 발생하는 이벤트를 감지하여 observer에게 전달할수 있습니다. 이방식은 이벤트 기반 프로그래밍에서 매우 유용합니다
  
  </br>
  </br>


```swift
  let button = UIButton()
button.rx.tap
    .subscribe(onNext: { print("Button clicked") })
```
- 여기서의 외부환경은 사용자 입력처리 즉 버튼클릭, 스크롤, 텍스트 입력과 같은 UI이벤트를 생성합니다.
- button.rx.tap 은 버튼 클릭 이벤트를 감지하여 Observable로 제공하고 사용자가 버튼을 클릭하면 observer가 이벤트를 처리합니다. 

  </br>

> 다른 예시로는  네트워크 요청응답 같은 경우로는 서버에서 데이터를 요청하고 응답을 Observable로 처리할수도 있습니다.
```swift
let url = URL(string: "https://api.example.com/data")!
URLSession.shared.rx.data(request: URLRequest(url: url))
    .subscribe(onNext: { data in
        print("Received data: \(data.count) bytes")
    })
```
- observable은 네트워크 요청의 결과를 방출합니다. 
- observer는 데이터의 크기나 내용을 출력하거나 추가로 처리할수 있습니다!

  </br>

> 다른예시로는 센서데이터 처리와 같은 경우가 있는데 
```swift
let motionManager = CMMotionManager()
motionManager.rx.accelerometerData
    .subscribe(onNext: { data in
        print("Acceleration: \(data.acceleration)")
    })
```
```
Acceleration: x=0.01, y=0.02, z=-9.8
```
- 가속도, GPS 와 같이 하드웨어 센서에서 발생하는 데이터를 observable로 처리할수도 있습니다. 
  
</br>
</br>

>부수 효과 (`side effects` ) 발생
- observable이 리소스 접근 또는 부수효과를 포함한 작업을 수행하는 도중 데이터를 생성할수도 있습니다
- 이경우 observable은 데이터 방출 외에도 외부리소스에 영향을 줄수도 있습니다.

그러나 Observable을 반환하는 메서드를 호출하기만 한다면 시퀀스 생성은 수행되지 않으며 부수효과도 발생하지 않습니다. 
Observable은 단지 시퀀스 생성하는 방식과 요소 생성을 위한 매개변수를 정의할뿐입니다. 
즉 observable은 기본적으로 정의만 수행하고 , 구독전까지는 어떠한 작업도 실행되지않습니다. 
> 예를들자면 observable이 파일을 읽거나 네트워크 요청을 수행하도록 정의되었다고 해도, 구독을 하지않는다면 작업은 시작되지않는다는 얘기 
결국은  Observable이 데이터 생성의 정의 단계에 머무르고 있음을 의미합니다. 실제로 데이터를 생성하고 방출하려면 **구독(subscribe)**이 필요합니다.

</br>
</br>
</br>

시퀀스 생성은 `subscribe` 메서드가 호출될때 비로소 시작됩니다
(RxSwift의 지연 실행 특성을 나타내며 ,효율적인 리소스 관리를 가능하게 만듬)
```swift
func searchWikipedia(searchTerm: String) -> Observable<Results> {}
```

```swift
let searchForMe = searchWikipedia("me")

//요청이 수행되지 않았습니다. 어떤 작업도 실행되지않았으며 , URL 요청도 전송되지 않았습니다.

let cancel = searchForMe
  // sequence generation starts now, URL requests are fired
  .subscribe(onNext: { results in
      print(results)
  })
```
observable은 지연실행을 따르고 searchWikipedia 메서드가 호출되더라도 구독이 이뤄지지 않았기에 작업또한 시작되지않았습니다.
( 단순히 observable이 정의가 되었을뿐 , 요청도 작업도 실행되지않았습니다.)

observable 시퀀스를 생성하는 방법은 여러가지가 있는데 그중 가장 간단한 방법은 create 함수를 사용하는것입니다. 

</br>
</br>

> RxSwitf는 구독시에 단일요소를 반환하는 시퀀스를 생성하는 메서드를 제공합니다.
그메서드는 just 라고 불리고 이제 이를 직접 구현해보겠습니다.

just는 단일 데이터를 방출하고 종료하는 Observable을 생성합니다.

```swift
func myJust<E>( _ element: E) -> Observable<E> {
	return Observable.create { observer in 
		observer.on(.Next(element))
		observer.on(.Completed)
		return Disposables.create()
	}
}

myJust(0)
	.subscribe(onNext: { n in 
		print(n)
	})
```

- myJust는 하나의 요소를 방출하고 종료하는 사용자 정의 `observable`입니다. 
- `observer.on(.next(element))` -> `Observer` 에게 단일요소를 전달하고 
- `observer.on(.Completed)` -> 시퀀스를 완료하며 
- `Disposables.create()` -> 리소스 해제를 위한 Disposable 을 반환합니다. 

</br>

>이는 이렇게 출력될것입니다!
```
0
```
즉 myJust에 `INT` 타입인 0 들어오고 0을 방출하는 `Observable`을 생성하고 `onNext`로 전달한후 `onCompleted`로  종료된뒤 리소스 해제가 됩니다 

</br>
</br>

>`create` 함수는 무엇일까요?

- 이것은 Swift 클로저를 사용하여 `subscribe` 메서드를 간단히 구현할수 있도록 도와주는 편의 메서드입니다.
- `subscribe` 메서드와 마찬가지로 ,`observer`를 인자로 받고, `disposable`을 반환합니다.

이렇게 구현된 시퀀스는 실제로 동기적으로 동작합니다. 
요소를생성하고 종료하는 작업은 `subscribe` 호출이 `disposable`을 반환하기전에 완료됩니다. 
따라서, 생성된 요소를 방출하는 과정은 중단될수 없으므로, 반환된 `disposable`의 타입은 중요하지않습니다.


`create`로 구현된 `observabled`은 동기적으로 작동할수있으며 `subscribe`호출시 바로요소를 방출하고 종료하고 동기적인 `Observable`에서는 `disposable`을 반환하더라도 작업이 즉시 완료되므로 구독중단은 의미가 없습니다. 

동기적 시퀀스를 생성할때, 일반적 으로 반환되는 `disposable`은 `NopDisposable`의 단일 인스턴스 입니다.
동기적 Observable은 구독중단이 필요하지 않으므로 반환값으로 아무작업도 하지않는 Disposabledls NopDispoable을 사용합니다. 

이는 Observable의 정의에만 사용되며, 특별한 리소스 해제작업이 푤요하지않은경우에 적합합니다.

> 예제

```swift
func myFrom<E>( _sequence: [E]) -> Observable<E> {
	return Observable.create { observer in 
		for element in sequence {
			observer.on(.next(element))
		}
	observer.on(.completed)
	return Disposables.create()
	}
}

let stringCounter = myFrom(["first", "second"])

print("Started ----")
// first time
stringCounter
    .subscribe(onNext: { n in
        print(n)
    })

print("----")

// again
stringCounter
    .subscribe(onNext: { n in
        print(n)
    })

print("Ended ----")
```

> 이는 이렇게 출력될것입니다.
```
Started ----
first
second
----
first
second
Ended ----
```

- myFrom(`["first", "second"]`) 은 "first" 와 "second" 를 순차적으로 방출하는 `Observable`을 생성합니다. 
- 첫번째 구독과 두번째 구독은 각각 독립적으로 실행되며 동일한 데이터를 방출합니다.












