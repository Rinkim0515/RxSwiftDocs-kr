# HTTP 요청 만들기

HTTP 요청은 개발자가 가장 먼저 시도하는 작업중 하나입니다.  
URLRequest 객체를 만들어 요청 유형(GET, POST), 요청본문, 요청 쿼리파라미터 등을 정의해야합니다.

> 간단한 GET 요청예시
```swift
let req = URLRequest(url: URL(string: "http://en.wikipedia.org/w/api.php?action=parse&page=Pizza&format=json"))
```

옵저버블(Observable)을 다른 옵저버블과 연결하거나 조합(composition)하지 않고 **단독으로 요청을 실행**하고 싶을 때, 이렇게 하면 됩니다.

```swift
let responseJSON = URLSession.shared.rx.json(request: req)

// 이 시점까지는 실제 요청이 발생하지 않습니다.
// `responseJSON`은 응답을 가져오는 방법만 정의합니다.

let cancelRequest = responseJSON
    .subscribe(onNext: { json in
        print(json)  // 응답 데이터를 출력합니다.
    })

Thread.sleep(forTimeInterval: 3.0)

// 3초 후 요청을 취소하고 싶다면
cancelRequest.dispose()
```
- URLSession의 확장은 기본적으로 메인스레드에서 결과를 반환하지 않습니다.

> 응답에 대해 더 낮은 수준의 접근이 필요하다면 다음과같이 사용할수 있습니다.
```swift
URLSession.shared.rx.response(request: myURLRequest)
    .debug("my request") // 요청 정보를 콘솔에 출력
    .flatMap { (data: NSData, response: URLResponse) -> Observable<String> in
        if let response = response as? HTTPURLResponse {
            if 200 ..< 300 ~= response.statusCode {
                return just(transform(data))  // 성공 응답 처리
            } else {
                return Observable.error(yourNSError)  // 에러 처리
            }
        } else {
            rxFatalError("response = nil")  // 응답 없음 에러
            return Observable.error(yourNSError)
        }
    }
    .subscribe { event in
        print(event)  // 에러가 발생해도 출력됨
    }
```

## HTTP 트래핑 로깅

- RxCocoa는 기본적으로 디버그 모드에서 모든HTTP요청을 콘솔에 출력합니다.
- 특정 도메인만 로깅하려면 URLSession.rx.shouldLogRequest를 설정합니다.

```swift
URLSession.rx.shouldLogRequest = { request in
    // reactivex.org 도메인 요청만 로깅
    return request.url?.host == "reactivex.org" || request.url?.host == "www.reactivex.org"
}
```