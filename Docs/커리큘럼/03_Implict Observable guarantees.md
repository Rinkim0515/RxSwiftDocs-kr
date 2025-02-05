# 암묵적인 Observable 보장(Implict Observable guarantees)

모든 시퀀스 생성자(Observable)는 반드시 준수해야 하는 몇 가지 추가적인 보장이 있습니다.

요소를 생성하는 스레드가 무엇인지와는 상관없이, 하나의 요소를 생성하여 observer.on(.next(nextElement)) 메서드를 통해 관찰자(observer)에게 전달한 경우, observer.on 메서드의 실행이 완료되기 전에는 다음 요소를 보낼 수 없습니다.

또한, .next 이벤트가 완료되지 않은 상태에서는 .completed 또는 .error와 같은 종료 이벤트를 보낼 수 없습니다.

간단히 말해, 다음 예제를 고려해 보십시오:

```swift
someObservable
  .subscribe { (e: Event<Element>) in
      print("Event processing started")
      // processing
      print("Event processing ended")
  }
```

이는 이렇게 출력될것 입니다.
```
Event processing started

Event processing ended

Event processing started

Event processing ended

Event processing started

Event processing ended
```

그리고 이렇게 출력될일 은 없을것 입니다.
```
Event processing started
Event processing started
Event processing ended
Event processing ended
```

핵심은 이벤트 처리의 순차성을 보장하는것 

1. observeOn 메서드가 실행중일때는 다른 이벤트가 들어올수 없고 이벤트가 겹쳐 처리되지 않도록 보장하여 데아터 처리의 안정성과 예측 가능성을 유지합니다.
이벤트가 절대 중첩해서 들어올수 없습니다. 이는 이넵ㄴ트가 병렬로 처리되거나 겹치는 경우를 방지합니다. 

2. .completed 와 .error 같은 종료이벤트는 모든 .next 이벤트가 완료된 뒤에만 발생할수 있습니다.  
이는 종료 이벤트가 발생하기전까지 모든 데이터가 안전하게 처리될수 있도록 합니다.





