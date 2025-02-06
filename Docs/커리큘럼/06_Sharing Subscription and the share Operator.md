# 구독(Subscription) 공유와 share 연산자(Sharing Subscription and the share Operator)
만약 여러 Observer가 하나의 구독에서 생성된이벤트(요소) 를 공유하도록 만들고 싶다면 어떻게 해야할까요? 

이를 위해선 두가지를 정의 해야 합니다.

**1. 과거 이벤트를 처리하는 방식**
- 새로운 `Subscriber`가 구독하기 전에 생성된 이벤트를 어떻게 다룰지 결정해야합니다.
- eg) 가장최근의 이벤트만 재생 (repaly), 모든 이벤트 재생, 최근 n개의 이벤트 재생.
  
**2. 공유된 구독을 시작하는 시점**
- 언제 구독을 시작하고 종료할지 결정해야합니다.
-	eg) `refCount`(구독자가 처음생기면서 시작, 마지막 구독자가 해지되면 중단)할것인지 수동으로 할것인지 또는 사용자 알고리즘으로 할것인지

일반적으로 가장 많이 사용되는 선택지는 replay(1).refCount()의 조합, 즉 share(replay: 1)입니다.

```swift
let counter = myInterval(.milliseconds(100))
	.share(replay: 1)

print("started")

let subscription1 = counter
	.subscribe(onNext: { n in 
		print("First \(n)")
	})

let subscription2 = counter
	.subscribe(onNext: { n in
		print("Second \(n)")
	})
Thread.sleep(forTimeInterval: 0.5)

subscription1.dispose()

Thread.sleep(forTimeInterval: 0.5)

subscription2.dispose()

print("Ended ----")
```
> 이것은 이렇게 출력될것입니다.
```
Started ----
Subscribed
First 0
Second 0
First 1
Second 1
First 2
Second 2
First 3
Second 3
First 4
Second 4
Second 5
Second 6
Second 7
Second 8
Second 9
Disposed
Ended ----
```

**share(replay: 1) 의 의미**
- replay: 1 -> 가장 최근의 이벤트 하나를 기억하며 새로구독하는 Observer 에게도 동일한 데이터를 전달
- refCount: 첫번째 구독자가 생기면 실행을 시작하고, 모든 구독자가 없어지면 실행을 중단.

**이걸 왜쓰는걸까?**

- 실행의 컨텍스트에 대한 차이  
- replay를 쓰고 안쓰고의 차이에 대한 포인트는 실행컨텍스트에 대한 차이이다. 

- replay를 사용하게 될경우 첫번째의 구독이 생성되면 Observable 이 시작되고, 추가 구독은 동일한 실행을 공유  즉 데이터 방출은 모든 Observer가 공유함 

- 첫번째 구독을 취소하더라도 실행은 계속 유지됨 ( 다른 구독자가 존재하기때문 ) 그리고 모든 구독자가 취소되면 Observable 실행이 종료됨 

- 허나 사용하지 않을경우에는 독립시행됨 
- subscription1, subscription2 는 각각 별도의 실행 컨텍스트를 가짐 **즉 두개의 독립적인 타이머가 생성되는것**


| **특징**        | share(replay: 1) **사용**        | share(replay: 1) **미사용**           |
| ------------- | ------------------------------ | ---------------------------------- |
| **구독 실행 방식**  | 구독자 간 **실행 컨텍스트 공유**           | 각 구독이 **독립적인 실행 컨텍스트**를 가짐         |
| **타이머 생성**    | 하나의 타이머만 실행                    | 각 구독마다 별도의 타이머 생성                  |
| **데이터 방출**    | 데이터가 모든 Observer에게 공유됨         | 각 Observer가 독립적으로 데이터를 받음          |
| **구독 취소의 영향** | 한 구독이 취소되어도 다른 구독자는 동일한 실행을 공유 | 한 구독이 취소되어도 다른 구독은 별개의 실행 컨텍스트를 유지 |
| **효율성**       | 리소스 효율적 (중복 실행 방지)             | 리소스 비효율적 (중복 실행 발생)                |
| **출력**        | 일한 데이터가 두 구독에 동일하게 전달          | 두 구독은 서로 다른 데이터 스트림을 받음            |



여기서 Subscribed**와** Disposed **이벤트가 각각 한 번씩만 발생**하는 점에 주목하세요.
</br>
</br>
</br>
URL observable의 동작은 이와 동일합니다.  
HTTP 요청은 Rx에서 이렇게 래핑 됩니다. 이는 interval 연산자와 매우 비슷한 패턴으로 작동됩니다.

```swift
extension Reactive where Base: URLSession {
	public func response(request: URLRequest) -> 
	Obsevable<(response:HTTPURLResponse, data: Data)> {
		return Observable.create { observer in 
			let task = self.base.dataTask(with: request) {
			 (data, response, error) in 
			 
			 guard let response = response, let data = data else {
				 observer.on(.error(error ?? RxCocoaURLError.unknown))
				 return
			 }
			 
			 guard let httpResponse = response as? HTTPURLResponse else {
				 observer.on(.error(
					 RxCocoaURLError.nowHTTPResponse(response: response)
					 )
				)
				return
			 }
			 
			 observer.on(.next((httpResponse,data)))
			 observer.on(.completed)
			}
		
		task.resume()
		
		return Disposable.create {
			task.cancel()
		}
		}
	
	}
}
```

이코드는 URLSession을 RxSwift 방식으로 확장하여, 네트워크 요청을 처리하는 기능을 제공합니다.

요청 결과를 **(response: HTTPURLResponse, data: Data)** 형태로 Observer에 전달.


| **코드**                                    | **설명**                                        |
| ----------------------------------------- | --------------------------------------------- |
| extension Reactive where Base: URLSession | URLSession을 Rx 방식으로 확장하여 Reactive 네임스페이스에 포함. |
| func response(request: URLRequest)        | 네트워크 요청을 수행하고 결과를 Observable 형태로 반환.          |
| Observable.create                         | 비동기 네트워크 작업을 처리하는 Observable 생성.              |
| self.base.dataTask(with: request)         | URLSession의 dataTask를 사용하여 네트워크 요청 수행.        |
| observer.on(.next((httpResponse, data)))  | 요청이 성공하면 응답 데이터를 Observer에 전달.                |
| observer.on(.error(...))                  | 요청 실패 시 Observer에 에러 전달.                      |
| task.resume()                             | 네트워크 요청을 시작.                                  |
| Disposables.create { task.cancel() }      | 구독 취소 시 네트워크 요청을 중단하여 리소스 해제.                 |

</br>
</br>

> 사용예시
```swift
let url = URL(string:"https://api.example.com/data")!

let request = URLRequest(url: url)

URLSession.shared.rx.response(request: request)
	.subscribe(onNext: { response, data in 
		print("Status Code: \(response.statusCode)")
        print("Response Data: \(data)")
    }, onError: { error in
        print("Error: \(error)")
    })
    .disposed(by: disposeBag)
```











