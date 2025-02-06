# UI Layer Tips

obserbable을 UIkit 컨트롤에 바인딩 할떼, UI레이어에서 지켜야 할 몇 가지 사항이 있습니다.

## 쓰레드 처리

Observable은 반드시 `MainScheduler`에서 값을 전달해야합니다.
- 이는 UIKit/Cocoa의 일반적인 요구사항입니다.  
API가 MainScheduler에서  결과를 반환하도록 만드는것이 좋습니다.
백그라운드 스레드에서 UI바인딩을 시도하면, 디버그 빌드에서 RxCocoa가 예외를 발생시켜 경고합니다.
이를 해결하려면 observe.on(MainScheduler.instance) 를 추가해야합니다.
URLSession 확장은 기본적으로 MainScheduler에 결과를 반환하지 않습니다.

</br>
</br>

> RxSwift에서 UI 쓰레드로 전환하는 방법
```swift
api.fetchData()
	.observeOn(MainScheduler.instance)
	.bind(to: label.rx.text)
	.disposed(by: disposeBag)
```
> URLSession에서 네트워크 응답을 UI에 바인딩전에 스레드 전환법
```swift
URLSession.shared.rx.data(request: urlRequest)
	.observeOn(MainScheduler.instance)
	.subscribe(onNext: { data in 
	})
	.disposed(by: disposeBag)
```

</br>
</br>



## 에러 처리

실패는 UIKit컨트롤에 바인딩 할수 없습니다. 이는 정의되지 않은 동작이기 때문입니다.

특정 Observable이 실패할지 모를 경우 ,catchErrorJustReturn(에러가 발생할시 변환할값)을 사용해 에러발생시 기본값을 반환하도록 할수 있습니다.

하지만 에러발생후에드 해당 시쿼느가 종료됩니다.(completed된다.)

에러발생후에도 시퀀스를 계속해서 데이터를 생성하도록 하고싶다면, retry 연산자를 사용하면됩니다.

> [!NOTE]
> Obervable은 데이터를 방출할수 있지만 에러가 발생할수도 있음  
> UI와 연결될때에는 에러를 반영할수 없기때문에 ( crash 혹은 예기치 못한 동작)  
> 애러처리가 필수임 여기서 얘기하는것은 에러 발생시에 기본값 반환을 통해서 처리할수도 있고  
> retry연산자를 사용하는것을 얘기하는듯 

</br>
</br>

## 구독 공유

UI 레이어에서는 구독을 공유하는것이 일반적입니다.

동일한 데이터를 여러 UI요소에 바인딩 할때마다 HTTP요청을 따로 보내고  싶지 않을것입니다.

```swift
let searchResults = searchText
    .throttle(.milliseconds(300), scheduler: MainScheduler.instance) // 300ms 동안 입력 지연
    .distinctUntilChanged() // 동일한 입력 무시
    .flatMapLatest { query in
        API.getSearchResults(query) // API 호출
            .retry(3)               // 에러 발생 시 3번 재시도
            .startWith([])         // 새로운 검색어 입력 시 결과 초기화
            .catchErrorJustReturn([]) // 에러 발생 시 빈 배열 반환
    }
    .share(replay: 1) // <- 구독 결과를 공유
```

일반적으로 원하는것은 검색결과가 계산되면 이를 공유하는 방법입니다.
그래서 share연산자를 사용합니다.

UILayer에서는 변환 체인의 마지막에 share를 추가하는것이 좋습니다.
계산된 결과를 공유해야하고 동일한 데이터를 여러 UI요소에 바인딩할때 HTTP요청이 여러번 발생하는것을 방지 합니다.

또한 Driver단위를 참고하세요.
Driver는 share호출을 자동으로 감싸고, 데이터가 항상 메인 UI스레드에서 관찰되도록 보장하며, 에러가 UI에 바인딩 되지 않도록 설계되었습니다.

</br>
</br>


> [!NOTE]
> share(replay: 1) -> 마지막 데이터를 1번 캐시하여 모든 구독자가 공유  
> Driver -> UI전용 스트림으로, 스레드안전, 에러처리, 구독공유를 자동으로 처리












