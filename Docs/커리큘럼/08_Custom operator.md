# 사용자정의 연산자

사용자 정의 연산자를 구현하는 방법은 두가지가 있습니다.

RxSwift의 내부 코드에는 고도로 최적화된 연산자가 사용되며, 이들은 학습자료로 적합하지 않으수도 있습니다.

따라서 표준 연산자를 사용하는것이 권장됩니다.

다행히도 사용자 정의 연산자를 만드는 간단한 방법이 있습니다.

새로운 연산자를 만드는것은 결국 Observable을 생성하는 과정이며, 이전 장에서 이를 다뤘습니다.

이제 최적화 되지않은 map 연산자를 구현하는 방법을 살펴 보겠습니다.

```swift
extension ObservableType {
	func myMap<R>(transform: @escaping(Element) -> R) -> Observable<R> {
		return Observable.create { observer in 
			let subscription = self.subscribe { e in 
				switch e {
					case .next(let value):
						let result = transform(value)
						observer.on(.next(result))
					case .error(let error):
						observer.on(.error(error))
					case .completed:
						observer.on(.completed)
					}
				}
				
			return subscription
		}
	}
}
```


>이제 본인만의 map연산자를 사용할수 있습니다:

```swift
let subscription = myInterval(.milliseconds(100))
	.myMap { e in
		return "This is simply \(e)"
	}
	.subscribe(onNext: { n in 
		print(n)
	})
``` 
> 이는 이렇게 출력될것입니다.

```
Subscribed
This is simply 0
This is simply 1
This is simply 2
This is simply 3
This is simply 4
This is simply 5
This is simply 6
This is simply 7
This is simply 8
...
