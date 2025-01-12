## Infallible

- Infallible은 Observable의 또 다른 유형 으로, Observable과 동일하지만 절대 실패하지 않으며, 에러를 방출할수 없습니다.
- 즉,Infallible.create 또는 "Creating Your first Observable"에서 언급된 생성방법을 사용하여 Infallible을 만들때 , 에러를 방출하는것이 허용되지 않는다는 뜻입니다.

  Infallible의 주요 특징
  
	1. 에러방출 불가능: onNext와 onCompleted만 방출
	2. 에러 안전성 보장: 에러가 발생하지 않기에 단순한 처리가 가능
	3. 주요 사용사례 : UI이벤트, 설정값 가져오기, 로컬캐시 데이터 등

<br/>



사용예제 ) UI event
```
let buttonTap = Infallible<String>.create {
	observer(.next("Button Tapped!"))
	observer(.completed)
	return Disposable.create
}

buttonTap
	.subscribe(.onNext: { print($0) })

```
observable vs Infallible 
| 특징       | Observable                          | Infallible                          |
| -------- | ----------------------------------- | ----------------------------------- |
| 에러방출     | 가능                                  | 불가능                                 |
| 주요 사용사례  | 네트워크 요청, 파일처리 등 에러가 발생할 가능성이 있는 작업  | UI 이벤트, 설정값, 데이터 등 에러가 발생하지 않는 작업   |
| 에러 처리 여부 | 필요 (catch, retry 등 연산자 사용가능)        | 불필요( catch 등 사용 불가)                 |
| 생성 메서드   | Observable.create, Observable.just등 | Infallible.create,Infallible.just 등 |
| 전환가능여부   | asInfallible 메서드를 통해 전환가능           | 불가능 (Observable로 전환신 에러 처리 필요)      |

- Infallible 은 절대 실패하지 않는 값 스트림을 정적으로 모델링 하고 보장하고 싶을때 유용합니다.
- 이때 MainScheduler를 강제로 사용하지 않거나, Driver나 Signal처럼 share() 를 암묵적으로 사용해 리소의와 부작용을 공유하지 않으려는 경우에 특히 적합합니다.


> [!NOTE]
> Infallible은 에러를 방출하지 않는 데이터 스트림을 보장함
> Driver와 Signal은 기본적으로 MainScheduler를 사용하여 데이터를 방출하지만 Inallible은 Scheduler에 구애받지 않는 다는것이 큰 차이점
>  또한 Driver와 Signal은 기본적으로 share() 통해서 구독을 공유하여 리소스와 부작용을 방지 하지만 Infallible은 share()없이 각 구독이  독립적으로 동작할수 있음 


| 특징            | Driver              | Signal            | Infallible               |
| ------------- | ------------------- | ----------------- | ------------------------ |
| 에러 방출 여부      | 불가                  | 불가                | 불가                       |
| Scheduler     | MainScheduler       | MainScheduler     | 제한 없음                    |
| share() 사용 여부 | 암묵적 사용              | 암묵적 사용            | 사용하지 않음                  |
| 주요 사용 사례      | UI 데이터 스트림 (텍스트 입력) | 사용자 이벤트 ( 버튼 클릭 ) | 설정값 , 로컬 캐시 데이터등 독립적 스트림 |


---

### 종종 발생할수 있는 일

- 어떤 경우에는 Rx연산자 만으로 문제를 해결하기 어려울수 있습니다. 
- 이럴때는 RxMonad를 탈출하여 명령형 방식으로 작업을 수행한후,결과를 Rx로 다시 전달할수 있습니다. 이과정에서 Subjects를 사용할수 있습니다.
- 다만 이방법은 자주 사용해서는 안되며, 코드냄새가 나는 나쁜 방식으로 간주됩니다. 하지만 가능은 합니다.


```
let magicBeing: Observable<MagicBeing> = summonFromMiddleEarth()

magicBeings
	.subscribe(onNext: { being in  //Rx Monad 탈출
		self.doSomeStateMagic(being)
	})
	.disposed(by: DisposedBag)


let kitten = globalParty( // 명령형 방식으로 계산
	being.
	UIApplication.delegate.dataSomething.attendees
)
kitten.on(.next(kitten)) //결과를 다시 Rx로 전달

let kitten = BehaviorRelay(value: firstKitten) // 다시 Rx Monad로 돌아옴 

kittens.asObservable()
	.map {kitten in 
		return kitten.purr()
	}
// ...
```
> 이런 작업을 할때마다 다음과 같은 코드가 어딘가에 작성될 가능성이 큽니다.
```
kitten
	.subscribe( onNext: { kitten in 
	})
	.dispose(by: disposeBag)
```
그러니 이런방식은 피하는것이 좋습니다.


> [!NOTE]
> Rx Monad의 의미는 RxSwift의 선언형 프로그래밍 방식을 의미함  
> Monad는 함수형 프로그래밍의 개념으로, 데이터를 체계적으로 연결하는 방식이라고 함  
> RxSwift에서는 Observable이 Monad로 동작하며, 데이터 스트림을 함수 체인으로 처리함 
>  
> Rx Monad는 데이터를 Observable 체인 안에서 처리하며, 명령형 코드(데이터에 직접 변경/조작 ) 를 지양함
> 연산자 를 활용하여 데이터를 변환하고 처리하며 부작용이 발생하지 않는 순수한 데이터 흐름을 지향함
>  
> 즉 Rx Monad를 탈출한다는 위와 같은 내용처럼 처리 하지 않고 체인의 밖에서 직접 명령형 방식으로 처리하거나 ,상태를 변경하는것으로 볼수있다.

---

## 명령형 코드 vs 선언형 코드

일단 명령형 코드와 선언형 코드에 대한 기본지식이 어느정도 필요할것으로 생각되어서 정리를 해보자면 

명령형 코드는 
- 어떻게(**How**) 상태를 변경할지에 대한 초점.
- 상태변경과 작업과정을 명시적으로 작성

> 예시
```
button.backgroundColor = .blue //버튼의 ! 배경색은 ! 파랑
label.text = "Hello" // label의 text는 문자열 "Hello"
```

선언형 코드는 
- 무엇(**What**) 을 달성할지에 초점
- 데이터를 설명적으로 정의 하고 , 상태 변화는 내부적으로 처리.

> 예시
```
textObservable
	.bind(to: label.rx.text)
	.disposed(by: disposeBag)
```

> 명령형 코드의 예시
```
class ViewController: UIViewController {
    @IBOutlet weak var label: UILabel!
    @IBOutlet weak var button: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)
    }

    @objc func buttonTapped() {
        label.text = "Button Tapped"
        label.textColor = .red
    }
}
```

> 선언형 코드 예시
```
class ViewController: UIViewController {
    @IBOutlet weak var label: UILabel!
    @IBOutlet weak var button: UIButton!
    
    let disposeBag = DisposeBag()

    override func viewDidLoad() {
        super.viewDidLoad()
        
        button.rx.tap
            .map { "Button Tapped" }
            .bind(to: label.rx.text)
            .disposed(by: disposeBag)
    }
}
```
한발 더나아가서 생각을 해본다면,,,?

UIKit은 기본적으로 명령형 프로그래밍 방식으로 설계되었다?

UIKit vs SwiftUI 에대한 부분이지 않을까 싶다. 

UIKit은 뷰와 상태를 직접 변경하고 text나 textColor 와 같은 속성들을 명시적으로 수정한다
또한 어떠한 이벤트를 처리하기위해 addTarget 메서드와 명령형 메서드를 실행하는 방식으로 돌아가는데 

상태기반으로 뷰를 업데이트 한다 
상태가 변경되면 뷰가 자동 업데이트 할수 있도록 동작한다.

(실제로 UIKit은 SwiftUI로 대체되어지는 중 이라고 생각한다. )















