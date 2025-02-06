# 컴파일 에러를 디버깅하기(Debugging compile Errors)

- RxSwift/RxCocoa 코드를 작성할때, 컴파일러가 Observable의 타입을 유추하도록 크게 의존하게 됩니다. 이는 Swift가 멋진이유중 하나이지만, 때론 좌절감을 줄수도 있습니다. 
> (아마 타입 추론과 연관된 얘기인듯? )

- RxSwift는 클로저와 제네릭을 자주사용함, 이로인해서 Swift 컴파일러가 타입추론을함으로써 복잡한타입연산에서 타입이 명확하지 않으면 컴파일러가 오류가 발생할수 있음
- 이러한 오류를 해결하기 위해서 타입을 명시적으로 지정하여 컴파일러가 타입추론을 명확히 할수있도록 도울필요가 있는것을 설명하는것
> ( 명시적 타입지정인 type annotation을  타입주석이라고 일컫는듯 함 )


```swift
images = word
    .filter { $0.containsString("important") }
    .flatMap { word in
        return self.api.loadFlickrFeed("karate")
            .catchError { error in
                return just(JSON(1))
            }
      }
```

> 컴파일러가 이표현식에서 어딘가에 오류가 있다고 보고하는경우, 먼저 반환타입을 명시적으로 주석처리하는것을 추천합니다. 

```swift
images = word
    .filter { s -> Bool in s.containsString("important") }
    .flatMap { word -> Observable<JSON> in
        return self.api.loadFlickrFeed("karate")
            .catchError { error -> Observable<JSON> in
                return just(JSON(1))
            }
      }
```
> 이렇게 해도 문제가 해결되지 않는다면, 더많은 타입 주석을 추가하여 오류를 지역화 할수도 있습니다.

```swift
images = word
    .filter { (s: String) -> Bool in s.containsString("important") }
    .flatMap { (word: String) -> Observable<JSON> in
        return self.api.loadFlickrFeed("karate")
            .catchError { (error: Error) -> Observable<JSON> in
                return just(JSON(1))
            }
      }
```

>반환 타입과 클로저의 인수에 주석을 다는것부터 시작해보세요.
>보통 오류를 해결한 후에는 코드를 정리하기 위해 타입주석을 제거 할수 있습니다.




















