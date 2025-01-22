# Traits

## General 일반적인 내용

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


##### Traits의 역할
1.  문맥적 의미 부여 -> 어떤 상황에서 어떻게 동작할지 명확히 전달
2. 코드의 안정성 강화 -> 에러 및 스레드 문제 방지
3. 가독성 향상 -> 간결하고 직관적인 코드 작성

##### Traits 예시

- Driver -> UI 업데이트 전용 ( 에러 없음, 메인스레드에서 실행)
- Signal -> UI 이벤트 처리 ( 에러 없음, 메인스레드 실행, 상태 보존  X)
- Single -> 단일 성공/실패 결과 반환 (API 응답)
- Completable -> 성공 여부만 반환 (데이터 없음)
- Maybe -> 성공/실패/아무것도 없음 중 하나 반환


##### **Traits vs Observable**

| Observable         | Traits                  |
| ------------------ | ----------------------- |
| 범용적으로 사용가능         | 특정 상황에 최적화 된 Observable |
| 에러,스레드 제어 모두 직접 관리 | 에러 및 스레드 관리가 자동화됨       |
| 복잡한 상황에서도 사용 가능    | 간결하고 안전하게 사용 가능         |
| 추가적인 설정 필요         | 기본적으로 안전한 설정 적용         |


##### Traits의 동작방식

Traits 단순히 읽기 전용 Observable 시퀀스 속성을 가진 Wrapper(struct)입니다.
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


비유하자면 

- **Observable** → **기본 재료**
- **Trait** → **특정 용도**에 맞게 **조리**된 요리
- **.asObservable()** → 조리된 요리를 다시 **재료 상태**로 돌리는 과정

## RxSwift traits

### Single

Single은 Observable의 변형중 하나로, 여러요소를 방출하는 대신, 항상 하나의 요소 또는 에러중 하나를 방출하는것이 보장됩니다.

- 정확히 하나의 값만 방출하거나 에러를 방출합니다.
- side effects를 공유하지 않습니다.
  
Single의 일반적인 사용사례중 하나는 HTTP 요청과 같이 응답 또는 에러만 반환할수 있는 작업을 수행하는 경우입니다. 그러나 Single은 단일 요소만 필요한 모든 상황에서 사용할수 있습니다. 예를 들어 무한 스트림이 필요없는 경우 적합합니다.

방출후 바로완료 ( 1개의 값 혹은 에러 1회만 방출)

**Single이 적합한 경우**  
HTTP요청과 같이 응답이 단일 데이터 혹은 에러인경우
단일계산 결과: 복잡한 연산후, 하나의 결과만 반환하는 경우


Single을 만드는것은 Observable을 만드는것과 유사합니다. 아주 간단한 예제를 본다면 이렇습니다:
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
그후에 이런방식으로 사용할수 있습니다
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

또는 `subscribe(onSucess:onError)` 메서드를 사용해서 다음과 같이 처리할수도 있습니다.

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

이구독(subscribe)은 Swift의 Result enum을 사용합니다.
이 enum은 `.success`를 통해 Single의 타입 요소를 포함하거나, `.failure`를 통해 에러를 포함할수 있습니다. 첫번째 이벤트 이후에는 더이상 이벤트가 방출되지 않습니다.

또한 `.asSingle()` 메서드를 사용하여 원시 Observable 시퀀스를 Single로 변환하는것도 가능합니다.

```swift 
let Observable = Observable.of("A", "B", "C")
let single = observable.asSingle()
single.subscribe(onSucess: { value in 
	print("Value: ", value)
}, onError { error in 
	print("Error: ", error)
})
```
이 예제에서 observable은 여러값을 방출하지만, asSingle() 을 호출하면 첫번째 요소만 방출하거나 에러를 반환합니다.

#### 요약

- Single은 한번의 결과 또는 에러를 처리하는데 이상적인  RxSwift의 특수 Observable 입니다
- subscribe 메서드는 Result를 통해 성공/ 실패를 명확히 처리하며, onSuccess 와 onError를 별도로 정의할수도 있습니다.
- 필요시 .asSingle() 로 기존 Observable을 변환해 단일값만 처리할수 있습니다.
- API요청 및 단일 결과를 다루는 작업에서 용이합니다.

### Completable

Completable은 Observable 의 변형으로 , 완료 이벤트 또는 오류만 방출할수 있습니다.
요소를 방출하지 않는것이 보장됩니다.
- 요소를 방출하지 않음
- 완료 이벤트 또는 오류를 방출.
- 부작용을 공유하지 않음.

Completable은 작업의 완료 여부만 중요하고, 그완료로 인해 생성된 요소에는 관심이 없는경우에 유용합니다.  
이것은 요소를 방출하지않는 `Observable<Void>` 와 비슷합니다.

##### Creating a Completable

completable을 생성하는 방식은 Observable을 생성하는 방식과 유사합니다.
아래는 간단한 예제 입니다:
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
생성한 Completable을 다음과 같이 사용할수 있습니다.
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

구독(Subscribe)과정에서 CompletableEvent 열거형이 제공됩니다.
- `.completed`: 작업이 완료되었음을 나타냄 
- `.error` : 작업중 오류가 발생했음을 나타냄

이후에는 더이상 이벤트를 발생하지 않습니다.

요약: 
요소 방출이 없으며 작업의 실패/성공 여부만 중요, 작업결과는 무관합니다.
데이터 저장, 파일업로드 ,로그작성과 같은 작업에서 작업의 완료 여부만 필요할때 유용합니다.

작업 결과가 아닌 상태를 확인하려는 경우, Completable이 적합한 선택입니다.
데이터 저장, 네트워크 작업 등 “완료 여부만 확인”할 필요가 있는 작업에서 유용합니다.
