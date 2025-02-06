# Debugging

디버거를 사용하는것도 유용하지만, 보통 debug 연산자를 사용하는것이 더 효율적입니다.

`debug` 연산자는 모든 이벤트를 표준 출력에 출력하며, 이벤트에 레이블을 추가할수도 있습니다.

`debug` 연산자는 마침 탐침 처럼 작동합니다. 아래는 debug를 사용하는 예제 입니다. 
> (여기서의 probe는 뭔가를 캐묻다의 의미인듯, 뭔가를 자꾸 체크하는?)

```swift
let subsription = myInterval(.millisecond(100))
	.debug("my probe")
	.map { e in 
		return "This is simply \(e)"
	}
	.subscribe(onNext: { n in 
		print(n)
	})

Thread.sleepForTimeInterval(0.5)

subscription.dispose()
```
> 이렇게 출력될것 입니다.
```swift
[my probe] subscribed               // 구독 시작
Subscribed                          // myInterval에서 구독 시작
[my probe] -> Event next(Box(0))    // 첫 번째 이벤트(Box로 감싸진 0)
This is simply 0                    // 변환된 데이터 출력
[my probe] -> Event next(Box(1))    // 두 번째 이벤트(Box로 감싸진 1)
This is simply 1                    // 변환된 데이터 출력
[my probe] -> Event next(Box(2))    // 세 번째 이벤트(Box로 감싸진 2)
This is simply 2
[my probe] -> Event next(Box(3))    // 네 번째 이벤트(Box로 감싸진 3)
This is simply 3
[my probe] -> Event next(Box(4))    // 다섯 번째 이벤트(Box로 감싸진 4)
This is simply 4
[my probe] dispose                  // 구독 취소
Disposed                            // myInterval 구독 취소
```

- debug는 스트림에서 발생하는 모든 이벤트(next, error, completed, disposed)를 출력합니다.
- 스트림의 상태를 실시간으로 확인할 수 있어 디버깅에 유용합니다.
- 이런식으로 레이블과 함꼐 어떤 Observable에서 이벤트가 발생했는지 쉽게 파악할수 있다. 
- 이벤트 흐름을 “탐침”하듯이 관찰하여, 특정 Observable이 예상대로 동작하는지 확인합니다.

> 본인만의 debug 연산자를 쉽게 만들어 낼수도 있음
```swift
extension ObservableType {
    public func myDebug(identifier: String) -> Observable<Self.E> {
        return Observable.create { observer in
            print("subscribed \(identifier)")
            let subscription = self.subscribe { e in
                print("event \(identifier)  \(e)")
                switch e {
                case .next(let value):
                    observer.on(.next(value))

                case .error(let error):
                    observer.on(.error(error))

                case .completed:
                    observer.on(.completed)
                }
            }
            return Disposables.create {
                   print("disposing \(identifier)")
                   subscription.dispose()
            }
        }
    }
 }
```

- debug **연산자**는 Observable 스트림에서 발생하는 모든 이벤트를 출력하여 디버깅과 스트림 분석에 유용합니다.

- 복잡한 RxSwift 코드에서 이벤트 흐름을 추적하거나, 에러 발생 원인을 파악할 때 유용하게 활용할 수 있습니다.

- debug는 개발 중에만 사용하고, 배포 코드에서는 제거하여 성능에 영향을 주지 않도록 하는 것이 좋습니다.




























