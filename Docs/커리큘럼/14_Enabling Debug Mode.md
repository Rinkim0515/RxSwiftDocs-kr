# 디버그 모드의 활성화 

- RxSwift의 RxSwift.Resource 를 사용해 메모리 누수 디버깅을 하거나 HTTP 요청을 자동으로 로깅하려면 디버그 모드를 활성화 해야합니다.

- 디버그 모드를 활성화 하려면 RxSwift 타겟의 빌드 설정에서 Other Swift Flags 에 TRACE_RESOURCE 플래그를 추가해야 합니다.

## 메모리누수 디버깅
- 디버깅 모드에서는 RxSwift가 할당된 모든 리소스를 전역변수 Resource.total에서 추적합니다.
- 리소스 누수 탐지 로직을 구현하려면, 주기적으로 RxSwift.Resource.total 값을 출력하는 방법이 제일 안전합니다.

```swift
/* 다음 코드를 앱 시작 시점에 추가:
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]? = nil) */
_ = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .subscribe(onNext: { _ in
        print("Resource count \(RxSwift.Resources.total)")
    })
```
메모리 누수를 테스트하는 가장 효율적인 방법은:

1. 화면으로 이동하고 사용합니다.
2. 이전 화면으로 돌아옵니다.
3. **초기 리소스 개수**를 관찰합니다.
4. 화면으로 다시 이동하고 사용합니다.
5. 다시 이전 화면으로 돌아옵니다.
6. **최종 리소스 개수**를 관찰합니다.

- 초기와 최종 리소스 개수가 다르다면, **메모리 누수(memory leak)** 가 발생했을 가능성이 있습니다.
- 두 번의 화면 전환을 권장하는 이유는, 첫 번째 전환에서 **지연 로딩(lazy loading)** 리소스가 로드되기 때문입니다.

> [!NOTE]
> RxSwift는 메모리 관리가 중요함  
> Observable체인의 구독이나 리소스 해제가 제대로 이루어지지 않으면 메모리 누수가 발생할수 있음
> 그래서 Resource.total을 활용하여서 현재 RxSwift에서 활성상태인 리소스 개수를 확인하고 누수를 탐지할수 잇음

화면전환 테스트 외에도 매초마다 현재리소스 개수를 출력하는 방법도 있음 
```swift
_ = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .subscribe(onNext: { _ in
        print("Resource count \(RxSwift.Resources.total)")
    })
```
이런식으로 매초마다 현재 리소스 개수를 출력하여 확인하는 방법도 있음 

