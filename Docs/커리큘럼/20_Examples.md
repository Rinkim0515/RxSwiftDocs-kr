# Examples

## Reactive Values

> 먼저 명령형 코드로 작성된 코드를 살펴보겠습니다. 이코드의 목적은 a와 b 값을 이용하여 특정 조건을 만족할때 c에 값을할당하는것 입니다.

```swift
// 일반적인 명령형 코드의 
var c: String
var a = 1
var b = 2

if a + b >= 0 { // a + b 의 0 이상이면
	c = "\(a + b) is positive" //  c에 "\(a + b) is positive"  를 저장 
}
```
> 허나 a의 값을 변경해도 c의 값은 자동으로 업데이트되지 않습니다.
```swift
a = 4  // 하지만 c의 값은 여전히  "3 is positive" 
		// 우리가 원하는 결과는 바뀐 a 값을 대입한 "6 is positive" (4 + 2 = 6 )
```
> a =4 로 변경되어도 c의 값은 자동으로 갱신되지 않습니다.  우리가 원하는 동작은 a 또는 b 가 바뀌는 경우 c또한 자동으로 바뀌는것입니다.

</br>
</br>

> RxSwift를 사용하여 개선된 코드를 보자면 
```swift
let a: Observable<Int> = BehaviorRelay(value: 1) // a = 1
let b: Obasevable<Int> = BehaviorRelay(value: 2) // b = 2

let c = Observable.combineLatest(a, b) { $0 + $1 }
	.filter { $0 >= 0 } // filter 처리를 통해서 합이 0이상만 넘김
	.map {"\($0) is positive"} // 그 값이포함된 문자열이 c에 맵핑됨 




c.subscribe(onNext: { print($0) })
```
> 이제 a 와 b 에 값을 대입하였을때 자동으로 업데이트 된다.
```swift
a.accept(4)
```
> a에 4 가 들어오게되면 filter에서 검증처리후 그에대한 값이 0 이상임으로  6으로 잘 변경될것이다.
```swift
b.accept(-8)
```
> 허나 이경우에서 합산한 값이 0 이상 이아니므로 적용되지않는다. 
- 즉 필터링 조건을 활용하여 특정 상황에서는 업데이를 막을수도 있다. 

## Simple UI bindings

- Relay에 값을 바인딩하는 대신, UITextField의 rx.text 속성을 사용하여 값을 바인딩 해보겠습니다.
- UITextField.rx.text를 사용하여 사용자가 입력한 값을 직접 옵저버블 스트림으로 변환합니다.
```
let subscription: Disposable  = primeTextField.rx.text.orEmpty
            .map { WolframAlphaIsPrime(Int($0) ?? 0) }               // 숫자로 변환 후 비동기 API 호출 (Observable<Observable<Prime>>)
            .concat()                                                // 새로운 입력이 들어오면 이전 요청을 취소하고 최신 요청 실행 (Observable<Prime>)
            .map { "number \($0.n) is prime? \($0.isPrime)" }        // 결과를 문자열로 변환 (Observable<String>)
            .bind(to: resultLabel.rx.text)                           // UILabel에 결과 바인딩
```