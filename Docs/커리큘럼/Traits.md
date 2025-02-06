# Traits

## General 

Swift는 강력한 타입 시스템을 가지고 있어, 애플리케이션의 정확성과 안정성을 향상시키고, Rx를 더직관적이고 간편하게 사용할수 있도록 도와줍니다.

Traits는 인터페이스 경계를 넘어서 Observable 시퀀스의 특성을 명확히 전달하고 보장하는 데 도움을 줍니다. 또한, 문맨적 의미, 문법적 편의성, 그리고 특정 사용 사례에 적합하도록 만들어졌습니다. 반면, 일반 Observable은 어떤 상황에서도 사용할수 있기때문에 범용적입니다.

따라서 Traits는 전적으로 선택사항 입니다.
모든 RxSwift/RxCocoa API 는 Observable 시퀀스를 지원하므로, 프로그램 어디에서든 Observable을 자유롭게 사용할수 있습니다. 

- 일부 Traits는 RxSwift의 일반 가능입니다.
- 하지만 이러한 개념은 필요하다면 다른 Rx 구현체에서도 쉽게 적용할수 있습니다.
- 비공개 API나 특별한 기술 없이도 구현할 수 있습니다.

---
##### 핵심 개념 

- Swift의 타입 시스템은 컴파일 단계에서 오류를 사전에 방지하여 코드 안정성을 높입니다.
- Traits는 특정 상황에서 Observable을 더 명확하고 안전하게 사용하도록 도와줍니다.
- 예를 들어, UI업데이트는 Main Tread에서만 실행되어야 합니다. 
- 이런 규칙 보장하기 위해 Driver와 같은 Trait이 존재합니다. 

</br>

##### Traits의 역할
1.  문맥적 의미 부여 -> 어떤 상황에서 어떻게 동작할지 명확히 전달
2. 코드의 안정성 강화 -> 에러 및 스레드 문제 방지
3. 가독성 향상 -> 간결하고 직관적인 코드 작성

</br>

##### Traits 예시

- Driver -> UI 업데이트 전용 ( 에러 없음, 메인스레드에서 실행)
- Signal -> UI 이벤트 처리 ( 에러 없음, 메인스레드 실행, 상태 보존  X)
- Single -> 단일 성공/실패 결과 반환 (API 응답)
- Completable -> 성공 여부만 반환 (데이터 없음)
- Maybe -> 성공/실패/아무것도 없음 중 하나 반환

</br>

##### `Traits` vs `Observable`

| Observable         | Traits                  |
| ------------------ | ----------------------- |
| 범용적으로 사용가능         | 특정 상황에 최적화 된 Observable |
| 에러,스레드 제어 모두 직접 관리 | 에러 및 스레드 관리가 자동화됨       |
| 복잡한 상황에서도 사용 가능    | 간결하고 안전하게 사용 가능         |
| 추가적인 설정 필요         | 기본적으로 안전한 설정 적용         |


</br>

##### Traits의 동작방식

Traits 단순히 읽기 전용 Observable 시퀀스 속성을 가진 `Wrapper(struct)`입니다.
```swift
struct Single<Element> {
 let source: Observable<Element>
}
struct Driver<Element> {
	let source: Observable<Elemnet>
}
```
Traits는 Observable 시퀀스를 위한 일종의 빌더 패턴으로 생각할수 있습니다.
즉, Trait이 만들어지면 .asObservable()을 호출하여 일반적인 Observable 시퀀스로 다시 변환할수 있습니다.

</br>


#### Traits의 구조와 역할

Traits는 특정 목적에 맞게 Observable을 감싸는 구조체 입니다.내부에는 항상 하나의 Observable 시퀀스가(source) 존재합니다
즉,  Observable의 특정상황에서의 사용을 위해 제약을 두고, 보다 안정적이고 예측 가능한 동작을 하도록 도와줍니다.


Traits의 필요성
1. 안정성의 강화: 특정동작(예: UI업데이트) 에서 에러처리 및 스레드 제어를 자동화
2. 코드 가독성 향상: 사용목적이 명확하므로 의도를 쉽게 이해
3. 유지보수 용이성: 동작방식이 일관성 있게 유지됨

Observable로의 변환 
- Traits는 결국 Observable을 한차례 래핑 한 구조이기 때문에, .asObservable()을 사용하여 기본 Observable로 변환해서 사용할수 있습니다.
- 이로 인해서 유연성을 유지하고 필요한 상황에선 제약을 할수도 있습니다.

</br>

비유하자면 

- **Observable** → **기본 재료**
- **Trait** → **특정 용도**에 맞게 **조리**된 요리
- **.asObservable()** → 조리된 요리를 다시 **재료 상태**로 돌리는 과정

</br>
</br>


## RxSwift traits

### Single

Single은 Observable의 변형중 하나로, 여러요소를 방출하는 대신, 항상 하나의 요소 또는 에러중 하나를 방출하는것이 보장됩니다.

- 정확히 하나의 값만 방출하거나 에러를 방출합니다.
- side effects를 공유하지 않습니다.
  
Single의 일반적인 사용사례중 하나는 HTTP 요청과 같이 응답 또는 에러만 반환할수 있는 작업을 수행하는 경우입니다. 그러나 Single은 단일 요소만 필요한 모든 상황에서 사용할수 있습니다. 예를 들어 무한 스트림이 필요없는 경우 적합합니다.

방출후 바로완료 ( 1개의 값 혹은 에러 1회만 방출)

</br>
</br>

**Single이 적합한 경우**  
HTTP요청과 같이 응답이 단일 데이터 혹은 에러인경우
단일계산 결과: 복잡한 연산후, 하나의 결과만 반환하는 경우


> Single을 만드는것은 Observable을 만드는것과 유사합니다. 아주 간단한 예제를 본다면 이렇습니다:
```swift
func getRepo(_ repo: String) -> Single<[String: Any]> {
	return Single<[String: Any]>.create { single in 
		let task = URLSession.shared.dataTask(with: URL(string:  "https://api.github.com/repos/\(repo)")!) { data, _, error in
			if let error = error {
				single(.failure(error))
				return
			}
			guard let data = data,
				  let json = try? JSONSerialization.jsonObject(with: data, option: .mutableLeaves).
				  let result = json as? [String: Any] else {
				  single(.failure(DataError.cantParseJSON))
				  return
			}	
				single(.sucess(result))	
		}
		task.resume()
		
		return Disposables.create { task.cancel() }
		}
}
```
> 그후에 이런방식으로 사용할수 있습니다
```swift
getRepo("ReactiveX/RxSwift")
	.subscribe { event in 
		switch event {
			case .success(let json):
				print("JSON: ",json)
			case .failure(let error):
				print("Error: ", error)
		}
	
	}
	.disposec(by: disposeBag)
```

> 또는 `subscribe(onSucess:onError)` 메서드를 사용해서 다음과 같이 처리할수도 있습니다.

```swift
getRepo("ReactiveX/RxSwift")
	.subscribe(onSucess: { json in 
					print("JSON: ", json)
			   },
			   onError: { error in
				   print("Error: ", error)
			   })
	.disposed(by: disposeBag)
```

> 이구독(subscribe)은 Swift의 Result enum을 사용합니다.
이 enum은 `.success`를 통해 Single의 타입 요소를 포함하거나, `.failure`를 통해 에러를 포함할수 있습니다. 첫번째 이벤트 이후에는 더이상 이벤트가 방출되지 않습니다.

> 또한 `.asSingle()` 메서드를 사용하여 원시 Observable 시퀀스를 Single로 변환하는것도 가능합니다.

```swift 
let Observable = Observable.of("A", "B", "C")
let single = observable.asSingle()
single.subscribe(onSucess: { value in 
	print("Value: ", value)
}, onError { error in 
	print("Error: ", error)
})
```
> 이 예제에서 observable은 여러값을 방출하지만, asSingle() 을 호출하면 첫번째 요소만 방출하거나 에러를 반환합니다.

</br>
</br>

#### 요약

- Single은 한번의 결과 또는 에러를 처리하는데 이상적인  RxSwift의 특수 Observable 입니다
- subscribe 메서드는 Result를 통해 성공/ 실패를 명확히 처리하며, onSuccess 와 onError를 별도로 정의할수도 있습니다.
- 필요시 .asSingle() 로 기존 Observable을 변환해 단일값만 처리할수 있습니다.
- API요청 및 단일 결과를 다루는 작업에서 용이합니다.

</br>
</br>



### Completable

Completable은 Observable 의 변형으로 , 완료 이벤트 또는 오류만 방출할수 있습니다.
요소를 방출하지 않는것이 보장됩니다.
- 요소를 방출하지 않음
- 완료 이벤트 또는 오류를 방출.
- 부작용을 공유하지 않음.

Completable은 작업의 완료 여부만 중요하고, 그완료로 인해 생성된 요소에는 관심이 없는경우에 유용합니다.  
이것은 요소를 방출하지않는 `Observable<Void>` 와 비슷합니다.

</br>

##### Creating a Completable

>completable을 생성하는 방식은 Observable을 생성하는 방식과 유사합니다. 아래는 간단한 예제 입니다:
```swift
func cacheLocally() -> Completable {
	return Completable.create { completable in 
	// Store some data locally
	---
	---
	
	guard success else {
		completable(.error(CacheError.failedCaching))
		return Disposable.create {}
	}
	
	completable(.completed)
	return Disposable.create {}
	}
}
```
> 생성한 Completable을 다음과 같이 사용할수 있습니다.
```swift
cacheLocally()
	.subscribe { completable in 
		switch completable {
			case .completed:
				print("Completed with no error")
            case .error(let error):
                print("Completed with an error: \(error.localizedDescription)")
		}
	}
	.disposed(by: disposeBag)
```
</br>
</br>


구독(Subscribe)과정에서 CompletableEvent 열거형이 제공됩니다.
- `.completed`: 작업이 완료되었음을 나타냄 
- `.error` : 작업중 오류가 발생했음을 나타냄

이후에는 더이상 이벤트를 발생하지 않습니다.

</br>
</br>

- 요약:  
   
요소 방출이 없으며 작업의 실패/성공 여부만 중요, 작업결과는 무관합니다.
데이터 저장, 파일업로드 ,로그작성과 같은 작업에서 작업의 완료 여부만 필요할때 유용합니다.

작업 결과가 아닌 상태를 확인하려는 경우, Completable이 적합한 선택입니다.
데이터 저장, 네트워크 작업 등 “완료 여부만 확인”할 필요가 있는 작업에서 유용합니다.

</br>
</br>
</br>

### Maybe

`Maybe` 는 `Single` 과 `Completable` 의 중간형태의 Observable입니다.
하나의 요소를 방출하거나, 요소없이 완료되거나, 오류를 방출할수 있습니다.

주의: 이러한 이벤트중 하나라도 발생하면 `Maybe` 는 종료됩니다.
- 예를들어, 완료된 `Maybe` 는 요소를 방출할수없으며, **요소를 방출한 Maybe는 완료 이벤트를 보낼수 없습니다.** 

- 완료 이벤트, 단일 요소, 또는 오류 중 하나를 방출.
- 부작용을 공유하지 않음
  
`Maybe` 는 요소를 방출할수도 있고, 방출하지 않을수더 있는 작업을 모델링하는데 적합합니다.

</br>

`Maybe`와 `Single` 은 비슷해 보이지만, 동작과 의도에서 중요한 차이점이 존재.

`Single` 은 항상 결과 또는 오류중 하나를 방출해야하는 작업
`Maybe`는 결과가 없을수도 있는 작업을 표현  값이 없이 완료될수도 있음 

</br>



#### Creating a Maybe

`Maybe`를 만들어서 사용하는것은 `Observable`과 유사함
> 아주 간단한 예제를 보자면 :
```swift
func generateString() -> Maybe<String> {
	return Maybe<String>.create { maybe in 
		maybe(.success("RxSwift"))
		
		// OR
		
		maybe(.completed)
		
		// OR
		
		maybe(.error(error))
		
		return Disposable.create {}
	}

}
```
> 그리고 이런식으로 사용 하면됨:

```swift
generateString()
	.subscribe { maybe in 
		switch maybe {
			case .sucess(let element):
				print("Completed with element \(element)")
			case .completed:
				print("Completed with no element")
			case .error(let error):
				print("Completed with an error \(error.localizedDescription)")
		}
	}
	.disposed(by: disposeBag)
```
> 혹은 `subscribe`를 사용해서 이렇게 쓸수도 있음
```swift
generateString()
	.subscribe(onSuccess: {elements in 
					print("Completed with element \(element)")
               },
               onError: { error in
		            print("Completed with an error \(error.localizedDescription)")
               },
               onCompleted: {
	                print("Completed with no element")
               })
    .disposed(by: disposeBag)
```
> 이것 역시 `.asMaybe()` 를 사용해서 Observable 시퀀스를 Maybe타입으로 변형시키는것이 가능함.


## RxCocoa traits

### Driver
`Driver`는 가정 정교한 Traits 중 하나로, RxCocoa에서  UI계층에서 리액티브 코드를 작성하거나, 애플리케이션의 동작을 구동하는 데이터 스트림을 모델링하려는 경우 에 적합합니다. ( 기본적으로 UI를 안정적이고 안전하게 구동하기 위한 목적으로 만들어 졌습니다. )

</br>
</br>

**Driver의 특성:**
1. 에러가 발생하지 않음: Driver는 에러를 방출하지 않는 시퀀스를 보장합니다.  (UI가 비정상적으로 중단되지 않습니다.)
2. MainScheduler에서 관찰: UI 는 주로 메인스레드에서 실행되므로 Driver는 항상 메인스케쥴러에서 관찰됨.( UI 작업은 반드시 메인스레드에서 실행되어야하며, Driver는 이규칙을 강제함)
3. 사이드 이펙트 공유: `shrare(replay:1, scope: .whileConnected)`를 통해서 사이드 이펙트를 공유합니다. (여러 UI요소 에서 같은 데이터를 사용할때 ,Driver는 데이터를 한번만 생성하고 이를 공유합니다.)

</br>
</br>

예시
- CoreData  모델로부터 UI를 구동하기
- UI 요소 간의 값을 기반으로 UI를 구동하기

</br>
</br>

운영체제 드라이버 처럼, 시퀀스에서 에러가 발생하면 애플리케이션이 사용자입력에 응답하지 않을수 있습니다.
또한, UI요소와 애플리케이션 로직은 일반적으로 스레드에 안전하지 않으므로, 메인스레드에서 요소를 관찰하는것이 매우 중요합니다. 

</br>


##### 전형적인 초보자용 예제
RxSwift를 배우는 과정에서 흔히 접하는 예제 입니다. 이코드는  UI입력을 처리하고 데이터를 가져와 화면에 표시 하는 기본적인 RxSwift 패턴을 보여줍니다.

```swift
let results = query.rx.text
	.throttle(.milliseconds(300), scheduler: MainScheduler.instance)
	.flatMapLatest { query in 
		fetchAutoCompleteItems(query)
	}
```

`query.rx.text`는 사용자의 입력을 관찰합니다.
-  .throttle(.milliseconds(300), scheduler: MainScheduler.instance)는 사용자의 입력을 **300밀리초 간격**으로 제한합니다(과도한 요청 방지).
-  .flatMapLatest { query in fetchAutoCompleteItems(query) }는 최신 검색어만 서버에 요청합니다.


```swift
results
	.map { "\($0.count)" }
	.bind(to: resultCount.rx.text)
	.disposed(by: disposeBag)

results
    .bind(to: resultsTableView.rx.items(cellIdentifier: "Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)

```

> 이 코드의 의도된 동작은 다음과 같습니다:
1. 사용자 입력을 **제한(throttle)** 한다.
2. **서버에 요청을 보내** 자동완성 결과 목록을 가져온다. (쿼리당 한 번)
3. 결과를 **두 개의 UI 요소에 바인딩**한다.

- 결과 리스트를 표시하는 UITableView
- 결과 개수를 표시하는 UILabel

> 그렇다면, 이코드의 문제점은 무엇일까요?
1. fetchAutoCompleteItems 옵저버블 시퀀스에서 오류(예: 네트워크 연결 실패 또는 파싱 오류) 가 발생하면, UI 바인딩이 해제되며, 이후 새로운 검색어 입력에 반응하지 않게됨.
2. fetchAutoCompleteItems가 백그라운드 스레드에서 결과를 반환하는 경우, UI요소가 백그라운드 스레드에서 업데이트 될수 있으며, 이로인해 예측할수 없는 충돌(non-deterministic crash)이 발생할수 있음.
3. 결과가 두개의 UI요소에 바인딩 되어 있기때문에, 각UI 요소마다 별도로HTTP 요청이 발새하여, 한번의 입력에 대해 두개의 네트워크 요청이 수행됨 -> 의도한 동작이 아님


> 보다 적절한 코드버전은 다음과 같습니다.

``` swift
let results = query.rx.text
	.throttle(.milliseconds(300), scheduler: MainScheduler.instance)
	.flatMapLatest { query in 
		fetchAutoCompleteItems(query)
			.observeOn(MainScheduler.instance)
			.catchErrorJustReturn([])
	}

results
	.map { "\($0.count)" }
	.bind(to:resultCount.rx.text)
	.disposed(by: disposeBag)

results
	.bind(to: resultsTableView.rx.items(cellIndentifier: "Cell")) { _, result. cell in cell.textLabel?.text = "\(result)"
	}
	.disposed(by: disposeBag)
```

기존의 문제점를 해결하기 위해 몇가지 추가적인 Rxswift 연산자를 활용합니다.
`observeOn(MainScheduler.instance)` 를 사용하여 네트워크 요청이 백그라운드 스레드에서 실행되더라도 그것에대한 결과는 항상 메인스레드에서 처리하도록 강제합니다.
`catchErrorJustReturn([])`을 통해서 네트워크 오류나 데이터 파싱오류로 인해 실패하면 빈배열을 반환함으로써  UI응답이 멈추는것을 방지합니다. 
`share(replay: 1)` 을 통해서 HTTP요청을 중복으로 발생하는것을 방지하고 가장최신의 결과를 모든 UI요소가 재사용할수 있습니다. 즉 `UILabel` 과 `UITableView`에 각각 새로운 HTTP 요청을 보내는것이 아니라 하나의 요청결과를 공유하게 됩니다.

이러한 요구 사항을 대규모 시스템에서 올바르게 처리하는것은 어려울수 있습니다.
그러나 `Compiler`와 `RxSwift`의 `Traits`를 사용하면 더간단한 방식으로 이를 증명할수 있습니다.



> 다음 코드는 거의 동일합겁니다. Trait의 사용 

```swift
let results = query.rx.text.asDriver()
	.throttle(.milliseconds(300), scheduler: MainScheduler.instance)
	.flatMapLatest { query in 
		fetchAutoCompleteItems(query)
			.asDriver(onErrorJustReturn: [])
		}
results
	.map { "\($0.count)" }
	.drive(resultCount.rx.text)
	.diposed(by: disposeBag)

results
	.drive(resultTableView.rx.items(cellIdentifier: "Cell")) {_, results, cell in
		cell.textLabel?.text = "\(result)"
	}
	.disposed(by: disposeBag)
```

> 이코드의 핵심변화는 `asDriver()`를 사용하여 Driver Traits 을 적용한것입니다.
```swift
query.rx.text.asDriver()
```

- `query.rx.text.asDriver()` 에서 `quert.rx.text` 는 ControlProperty (즉, UI요소의 값변화 스트림) 인데 이를 Driver로 변환하고

```
.asDriver(onErrorJustReturn:[])
```
- `.asDriver(onErrorJustReturn[])` 네트워크 요청이 실패하면 빈 배열 을 반환하여 UI . 가멈추는 문제를 방지하였습니다. 

</br>
</br>

Driver는 기본적으로 메인스레드에서 실행되고 UI와 안정적으로 연동되도록 설계되었으며
에러를 발생시키지 않도록 설계되어있어서 Observable이 Driver로 변환될때 반드시 에러처리방식을 지정해야합니다.

기존의 `.bind(to:)`가 `.drive()`로 바뀌었다는 것
`.drive()`는 **RxSwift의** Driver **트레잇을 사용해야만 호출할 수 있는 메서드**입니다.
`drive()`가 사용 가능하다는 것은 **해당 옵저버블이** Driver **속성을 만족**한다는 의미입니다.

</br>
</br>


> **기존 코드와 비교하면** Driver**를 사용하여 UI 업데이트 안정성을 보장할 수 있도록 개선됨.**

</br>

이 3가지 속성이 **UI 업데이트에 최적화된 RxSwift의** Driver**가 가진 가장 중요한 특성**입니다.

• **에러를 허용하지 않음** → UI가 멈추는 문제 방지

• **메인 스레드에서 실행** → UI 충돌 및 백그라운드 업데이트 방지

• **결과를 공유** → 중복 네트워크 요청 방지

</br>
</br>

> 어떻게 Driver의 속성이 보장될까요? 

그냥 일반적인 RxSwift 연산자를 사용하면 됩니다.
사실 `asDriver(onErrorKustReturn:[])` 는 아래 코드와 동일한 동작을 합니다. 
즉 ,Driver는 기존의 옵저버블을 변환하여 안정적인 UI  업데이트가 가능하도록 보장되는 래퍼(Wrapper) 입니다.

```swift
let safeSequence = xs
	.observeOn(MainScheduler.instance)
	.cathErrorJustReturn(onErrorJustReturn)
	.share(replay:1, scope: .whileConnected)
return Driver(raw: safeSequence)
```
> `asDriver(onErrorJustReturn: [])`**는 위 코드의 축약형이며, 이 변환 과정을 자동으로 처리하는 역할**을 합니다.

</br>
</br>


마지막 중요한 차이점은 `bind(to:)` 대신 `drive()`를 사용하는 것입니다.
drive()는 Driver 트레잇에서만 정의되어 있습니다.

> 즉, 코드에서 drive()가 사용되고 있다면, 그 Observable은 절대 에러를 발생시키지 않으며, 항상 메인 스레드에서 실행됨을 의미합니다
따라서 UI 요소에 안전하게 바인딩할 수 있습니다.


하지만 이론적으로 누군가 ObservableType 또는 다른 인터페이스에서
drive 메서드를 정의할 수도 있습니다.

따라서 **완벽하게 안전성을 보장하려면**,바인딩 전에 `let results: Driver<[Results]> = ...` 와 같이
**임시로** Driver **타입을 명시적으로 선언하는 것이 필요할 수도 있습니다.**

다만, 이것이 현실적으로 필요한지는 독자의 판단에 맡기겠습니다.

일반적으로 drive()는 Driver에서만 사용할 수 있지만,  
 만약 누군가 **다른** ObservableType**에서도** drive()**를 정의해버리면**?  그 경우에는 Driver의 속성이 완벽하게 보장되지 않을 수도 있습니다.

</br>
</br>


> 기존의 Observable을 사용할 경우

- `observeOn(MainScheduler.instance)`
- `catchErrorJustReturn([])`
- `share(replay: 1, scope: .whileConnected)`
- `bind(to:)`

위와 같은 처리를 직접 해야 하지만,

  

> Driver를 사용하면 **자동으로** 이 모든 것이 보장

- drive()를 사용하면 **에러 발생 X 
- UI 스레드에서 실행 보장**
- UI와 연동되는 RxSwift 코드에서는 **가능하면** Driver**를 사용하는 것이 권장됨**

</br>
</br>
</br>

### `Signal`

`Signal` 은 `Driver` 와 유사하지만 한가지 차이점이 있습니다.
`Driver`는 새로운 구독자가 생길때마다 가장 최근 이벤트를 다시 전달하지만, `Signal`은 새 구독자에게 아무 이벤트도 재생하지않습니다.

- 대신 `Signal`은 이벤트가 발생할때만 데이터를 전파하고, 이벤트 발생전에는 값을 저장하지 않습니다.
- `Signal`은 `버튼 클릭`, `알림(Notification)`, `UI이벤트 처리` 등에 적합한 구조를 제공합니다.

즉 `Singal`은 이벤트 기반의 UI동작을 RxSwift로 다룰때 적합한 개념입니다.

`Signal`은 Application에서 명령형 이벤트(Imperative Events)를 반응형 방식(Reactively)으로 모델링하는 
빌더패턴으로 볼수 있습니다.

</br>

`Signal`의 특징:
- 이벤트를 항상 메인스레드에서 전달한다.
- 구독자들은 데이터 스트림을 공유한다(`share(scope: .whileConnected)`)
- 구독자가 추가 되어도 이전 이벤트를 전달하지 않는다.


즉 `Signal`은 UI 이벤트를 RxSwift방식으로 처리하는 최적의 Trait 이고 
`Driver`는 UI 상태를 유지하는 경우 사용하지만, `Signal`은 상태를 유지하지 않고 즉시 처리해야하는 이벤트를 다룰때 적합하다. 

</br>


## ControlProperty/ ControlEvent

### ControlProperty

`ControlProperty`는 UI 요소의 속성을 나타 내는  `Observable / Observable Type` 을 위한 트레잇 입니다.

`ControlProperty`는 RxSwift에서 UI요소의 속성을 반응형으로 다룰때 사용되는 특수한 옵저버블이고 
`UITextfield`의 `text` 속성이나 `UISwitch`의 `isOn` 속성을 RxSwift 방식으로 감지하려면 `ControlProperty`를 사용한다.

즉 UI요소의 state를 옵저버블 형태로 변환하여 반응형 프로그래밍을 가능하게 한다. 

값의 시퀀스는 초기 UI 값과 사용자가 변경한 값만을 나타냅니다. 
코드에서 직접 값이 변경된경우 (setValue 같은 프로그래밍적 변경), 이를 감지할수 없습니다
- 사용자의 입력변화만을 감지하고, 코드로 변경된값은 감지하지않음

</br>
</br>

`ControlProperty`의 주요 속성 
- `share(replay: 1)` 동작을 가진다
- 상태를 유지하며(stateful), 구독시 마지막 값을 즉시 다시전달한다.
- UI요소가 해제 될때( 메모리에서 제거. 될때) 자동으로 완료 된다. (따라서 **메모리 관리를 위한 별도의** disposeBag **처리가 필요 없음**.)
- 절대 에러를 발생시키지 않는다.
- 항상 MainScheduler.instance에서 이벤트를 전달한다. 

</br>
</br>


`ControlProperty`는 UI 요소의 속성을 다루므로, **이벤트 처리는 항상 메인 스레드에서 실행되어야 함**.

-  따라서 내부적으로 subscribeOn(ConcurrentMainScheduler.instance)**를 사용하여 메인 스레드에서 실행을 강제**.
**이로 인해 UI 관련 이벤트 처리 시 스레드 관리 문제가 발생하지 않음**

</br>
</br>

즉, ControlProperty는 UI 요소를 반응형으로 다룰 때 최적의 선택이고 
이러한 주요특징을 가진다:
-  UI 속성 변화를 옵저버블로 변환
-  사용자의 입력 변화만 감지 (코드로 변경된 값은 감지 X)
-  구독 시 마지막 값을 즉시 제공 (stateful)
-  UI 요소가 해제되면 자동으로 완료 (complete)
-  항상 메인 스레드에서 실행되어 UI 업데이트가 안전함
 ControlProperty는 RxSwift에서 UITextField, UISwitch, UISlider 등의 UI 속성을 Rx 방식으로 감지하고 다룰 때 반드시 사용해야 하는 핵심 요소

</br>
</br>

 ### Practical usage example

 UISearchBar+Rx 및 UISegmentedControl+Rx에서 매우 유용한 실용적인 예제를 찾을 수 있습니다.

 > `UISearchBar` 의 예제
 ```swift
 extension Reactive where Base: UISearchBar {

	public var value: ControlProperty<String?> {
		let source: Observable<String?> = Observable.deferred { [weak searchBar = self.base as UISearchBar] () -> Observable<String?> in 
			let text = searchBar?.text 
		
			return  (searchBar?.rx.delegate.methodInvoked(#selector(UISerachBarDelegate.searchBar(_:textDidChange:))) ?? Observable.empty())
			.map { a in 
				return a[1] as? String
			}
			.startWith(text)
		}
		let bindingObserver = Binder(self.base) { (searchBar, text: String?) in 
		searchBar.text = text
		}
		return ControlProperty(values: source, valueSink: bindingObserver)
	}
}
```
> `UISegmentControl` 의 예시 
```swift
extension Reactive where Base: UISegmentedControl {
	public var selectedSegmentIndex: ControlProperty<Int> {
		value
	}
	
	public var value: ControlProperty<Int> {
		return UIControl.rx.value(
			self.base,
			getter: { segmentControl in 
				segmentedControl.selectedSegmentIndex
			}, setter: { segmentControl, value in 
				segmentedControl.selectedSegmentIndex = value 
			}
		)
	}
}
```

</br>
</br>

### ControlEvent 

`ControlEvent`는 UI요소에서 발생하는 이벤트를 나타내는 `Observable / ObservableType` 을 위한 Trait 입니다.

버튼 클릭, 텍스트 입력, 스크롤 이벤트와 같은 UI이벤트를 RxSwift 방식으로 다루는 특수한 옵저버블입니다. 
`ControlProperty`는 UI 요소의 속성을 감지하는 역할을 하지만, `ControlEvent`는 사용자가 발생시키는 이벤트를 감지하는 역할 

즉 사용자의 입력 / 상호작용을 RxSwift 방식으로 처리할때 사용된다.

</br>
</br>

`ControlEvent`의 주요 속성:
- 구독시 초기값을 보내지 않는다.
- UI 요소가 메모리 해제 될때 시퀀스가 Complete 된다. 
- 절대 에러를 발생시키지 않는다.
- 항상 MainScheduler.instance에서 이벤트를 전달한다.

</br>
</br>

`ControlEvent`의 구현은 이벤트 시퀀스가 항상 메인 스레드에서 구독되도록 보장합니다.
(`subscribeOn(ConcurrentMainScheduler.instance)` 동작을 따릅니다.)

 `ControlEvent`를 사용하면 UI 이벤트가 백그라운드에서 실행되는 문제를 걱정할 필요가 없다!

 즉 UI 이벤트를 RxSwift에서 다룰 때 가장 적합한 트레잇!

-  UI 이벤트(버튼 클릭, 텍스트 입력 등)를 반응형으로 처리할 수 있음.
- 이전 값을 저장하지 않고 새로운 이벤트가 발생할 때만 값을 방출
- UI 요소가 해제되면 자동으로 완료(**complete**)되므로 메모리 관리가 용이.
- 항상 메인 스레드에서 실행되므로 UI 업데이트가 안전하게 이루어짐.

</br>
</br>


#### Practical usage example

이것은 ControlEvent를 사용할 수 있는 전형적인 사례입니다.

> `UIViewController` 의사례
```swift
public extension Reactive where Base: UIViewController {
	public var viewDidload: ControlEvent<Void> {
		let source = self.methodInvoked(#selector(Base.viewDidLoad)).map { _ in}
		return ControlEvent(events: source)
	}
}
```
-  UIViewController의 viewDidLoad()는 기본적으로 한 번 실행되는 라이프사이클 메서드입니다.
-  이 코드는 viewDidLoad가 호출될 때마다 **RxSwift의** ControlEvent**를 통해 이벤트를 방출할 수 있도록 변환**합니다.
- methodInvoked(#selector(Base.viewDidLoad))를 사용하여 viewDidLoad()가 호출될 때마다 옵저버블을 실행하도록 만듬.
-  .map { _ in }을 사용하여 반환값을 Void로 변환 (이벤트가 발생했다는 사실만 전달).
-  최종적으로 ControlEvent(events: source)를 반환하여 **RxSwift 방식으로** viewDidLoad**를 감지 가능**하게 함.

>  `UICollectionView` 의 사례
```swift
extension Reactive where Base : UICollectionView {
	public var itemSelected: ControlEvent<IndexPath> {
		let source = delegate.methodInvoked(#selector(UICollectionViewDelegate.collectionView(_:didSelecItemAt:)))
			.map { a in 
				return a[1] as! IndexPath
			}
		return ControlEvent(events: soure)
	}
}
```

- UICollectionView의 **셀 선택 이벤트**(didSelectItemAt)를 **RxSwift 방식으로 감지할 수 있도록** ControlEvent**로 변환**한 코드입니다.
- delegate.methodInvoked(#selector(UICollectionViewDelegate.collectionView(_ :didSelectItemAt:)))를 사용하여셀을 선택할 때 호출되는 델리게이트 메서드의 이벤트를 옵저버블로 변환.
-  .map { a in a[1] as! IndexPath }을 사용하여 이벤트의 두 번째 파라미터(선택된 셀의 IndexPath)를 추출하여 변환**.
- ControlEvent(events: source)를 반환하여 **Rx 방식으로 안전하게 셀 선택 이벤트를 감지 가능하게 만듦**.