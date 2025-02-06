# Creating an Observable that performs work

> 이제 조금더 흥미로운 작업을 해보겠습니다. 이전 예제에서 사용된 interval 연산자를 직접 구현해보겠습니다.

```swift
func myInterval(_ interval: DispatchTimeInterval) -> Observable<Int> {
    return Observable.create { observer in
        print("Subscribed")

        // 글로벌 큐에서 타이머 생성
        let timer = DispatchSource.makeTimerSource(queue: DispatchQueue.global())

        // 타이머 시작 및 반복 간격 설정
        timer.schedule(deadline: DispatchTime.now() + interval, repeating: interval)

        // 구독 취소를 위한 Disposable 생성
        let cancel = Disposables.create {
            print("Disposed")
            timer.cancel()
        }

        // 타이머가 실행될 때마다 방출할 정수 값 초기화
        var next = 0

        // 타이머 이벤트 핸들러 설정
        timer.setEventHandler {
            // 구독이 취소된 경우 타이머 이벤트 중단
            if cancel.isDisposed {
                return
            }
            // Observer에 값 전달
            observer.on(.next(next))
            next += 1
        }

        // 타이머 실행
        timer.resume()

        // Disposable 반환
        return cancel
    }
}
```
- myInterval 함수는 지정된 간격동안 Observable이 데이터를 방출하는 비동기 반복작업을 구현함 
  
- 이함수는 타이머 기반 반복작업으로써 Observable.create 를 통해서 정수를 방출하는 사용자 정의 Observable을 만들고 

`timer.schedule(deadline: DispatchTime.now() + interval, repeating: interval)`
- 현재 시간으로부터 interval 만큼 지연된 시점에서 시작하여, 지정된 간격으로 반복 실행됩니다.


타이머가 실행될때마다 .next 이벤트를 통해서 현재 정수값을 Observer로 전달한후 정수값 ( next) 는 
1씩 증가합니다. 

`cancel` 을 통해서 `Dispoables.create`로 구독취소시에 타이머를 중단하고 리소스를 정리하며 
`cancel.isDisposed`가 `true`일경우 더이상 값을 방출하지 않습니다. 


- 100밀리 세컨드 = 0.1초 

> 단일 구독에대한 예제
```swift
let counter = myInterval(.milliseconds(100))

print("Started ---")

let subscription = counter
	.subscribe(onNext: { n in 
		print(n)
	})
	
Thread.sleep(forTimeInterval: 0.5)

subscription.dispose()

print("Ended ----")
```

> 이것은 이렇게 출력될것입니다.
```
Started ----
Subscribed
0
1
2
3
4
Disposed
Ended ----
```
</br>
</br>

> 순서, 실행되는스레드, 동작되는내용

| 순서  | 코드                               | 스레드            | 동작                                  |
| --- | -------------------------------- | -------------- | ----------------------------------- |
| 1   | let counter = myInterval(...)    | 메인 스레드         | Observable 생성 (백그라운드 큐에서 실행 대기 상태). |
| 2   | print("Started ---")             | 메인 스레드         | "Started ---" 출력.                   |
| 3   | subscription = counter.subscribe | 메인 → 백그라운드 스레드 | 구독 시작. 백그라운드 타이머 작동, 데이터 방출 시작.     |
| 4   | Thread.sleep(0.5)                | 메인 스레드         | 메인 스레드 정지. 백그라운드에서 데이터 계속 방출        |
| 5   | subscription.dispose()           | 메인 스레드         | 구독 취소. 백그라운드 타이머 중단 및 리소스 정리.       |
| 6   | print("Ended ---")               | 메인 스레드         | "Ended ---" 출력.                     |

</br>
</br>

> 만약 이렇게 사용한다면  **(다중 구독에 대한 예제)**

```swift
let counter = myInterval(.milliseconds(100))

print("Started ----")

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

print("Disposed")

Thread.sleep(forTimeInterval: 0.5)

subscription2.dispose()

print("Disposed")

print("Ended ----")
```
>이렇게 출력될것입니다.
```
Started ----
Subscribed
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
Disposed
Second 5
Second 6
Second 7
Second 8
Second 9
Disposed
Ended ----
```

여기서는 각 구독마다 새로운 타이머를 생성하므로, 각 구독은 서로 독립적으로 동작합니다.  

각 구독은 별도의 타이머를 싱핼하고, 독립적으로 데이터를 방출함  
`Observable`은 한번 생성된 상태를 공유하지만, 각 구독은 실행 컨텍스트를 독립적으로 가지고 `Observable`의 동작은 구독마다 독립적으로 실행된다. 

- 각 구독자는 보통 자신만의 독립적인 요소 시퀀스를 생성합니다.
- 연산자는 기본적으로 상태를 가지지 않는 형태 입니다.
- 상태를 가지지 않는 연산자가 상태를 가지는 연산자보다 훨씬 많습니다.



