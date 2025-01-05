
# 시작하기에 앞서 

## 배경지식
참고로 Swift에서 Protocol인 Sequence, Collection 이 있다.
```
public protocol Collection<Element>: Sequence {}
```

 - sequence는 요소들을 순차적으로 접근할수있는 데이터의 나열  
 *eg) Range, AnyIterator, stride*

 - collection은 sequence 프로토콜을 채택하고있는 모든 요소에 효율적으로 접근할수있는구조이며 특정인덱스 요소에 직접 접근이 가능하고, 항상정해진 모든 요소를 갖고있다
 - *eg) Array, Dictionary, Set*

> 즉 컬렉션은 시퀀스를 채택하고 있기에 컬렉션인 배열또한 하나의 시퀀스 인것이다.
> 이내용을 기반으로 Rxswift에서 observable 이라는 타입은 시퀀스라고 일컫는데
> 이 데이터의 흐름이 순서대로 진행되기때문에 시퀀스로 간주되며 다른 시퀀스와의 차이점은 시간에 따라 비동기적으로 변화하는 시퀀스를 표현하는방식인 것 

<br/> <br/> <br/>

# Observables aka Sequences

## Basics
The [equivalence](MathBehindRx.md) of observer pattern (`Observable<Element>` sequence) and normal sequences (`Sequence`) is the most important thing to understand about Rx.
<br/> <br/>
- 시퀀스가 일반적인 Sequence (예: 배열 등)와 어떻게 유사한지를 이해하는 것이  매우 중요합니다.
<br/>

$\it{\large{\color{#5ad7b7}RxSwift에서 Observable <Element> 모든\ Observable\ 시퀀스는\ 단지\ 시퀀스입니다 }}$

<br/>

- Observable과 Swift의 시퀀스간의 주요 장점은 Observable이 비동기적으로 요소를 받을 수 있다는 것입니다.  
- 이것이 RxSwift의 핵심이며, 여기서부터는 이 아이디어를 확장하는 방법에 대한 문서입니다.”


<br/> 
 
 - “Observable(ObservableType)는 Sequence와 동등하다.”
 - “ObservableType의 subscribe 메서드는 Sequence의 makeIterator 메서드와 동등하다.”
 - “Observer(콜백)는 ObservableType의 subscribe 메서드에 전달되어 시퀀스 요소를 받기 위해 필요하다. 반환된 반복자에서 next()를 호출하는 대신에.”


<br/>

"시퀀스는 단순하고 익숙한 개념으로 쉽게 시각화할 수 있습니다. 사람들은 시각적 피질이 매우 큰 생물입니다. 개념을 쉽게 시각화할 수 있을 때, 이를 이해하는 것이 훨씬 쉬워집니다. (마블 다이어그램)

<br/> 

우리는 모든 Rx 연산자에서 이벤트 상태 머신을 시뮬레이션하려는 인지적 부담을 덜어낼 수 있으며, 시퀀스를 사용하여 고수준의 연산을 수행할 수 있습니다.”(복잡한 이벤트 상태를 일일히 추적하지 않아도 된다)

<br/>


> 보통 observable은 어떤 event 를 emit 하고 나서 onNext,onCompleted,onError로 상태를 추적해서 관리하는것은  
>
> 개발자가 그것에대한 대응을 직접 핸들링 해야되는 resourceCost가 들지만  
> *(상태의 명시적인 호출과, 상태를 계속 추적하며 그에따라 필요한 동작을 일일히 작성해야하는 resource cost )*  
>
> 그와 반대로  operator를 사용하게될경우 일일히 상태 관리와 흐름이 자동으로 이뤄지기에 이벤트의상태의 하나하나를 신경쓰지 않아도 된다.
>
> *즉 새로운데이터가 오는경우 onNext로,오류가 발생할시 onError로, 완료될시 onCompleted로 자동 상태 전이를 한다고 이해*

<br/>

$\it{\large{\color{#5ad7b7}결론이 무엇이냐? }}$

 <br/>
 
> 즉 opeartor를 사용하지 않고 직접 방출을 관리하게 되는 경우에는
>
> 데이터를 방출하는 시점마다  `onNext`, `onError`, `onCompleted` 상태를 직접 호출하여 방출해야하는것이고  상태가 변화할 때마다 그에 맞는 처리를 일일이 각 단계에서 직접 제어해야 한다.
>
> 허나 연산자를 사용하게 될경우
>
> 중간단계에서 상태관리를 자동화 할수 있어서 각 연산자가 자동으로 데이터를 전달하고 오류, 완료 상태를 다음 연산자로 넘기기 때문에,  
> 매번 방출을 직접 관리할 필요가 없고 최종 subscribe에서만 최종 데이터와 상태(onNext, onError, onCompleted)를 구독하여 한 번만 처리해주면 된다는게 이말의 요지이다.

<br/>

만약 우리가 Rx를 사용하지 않고 비동기 시스템을 모델링한다면, 그것은 아마도 우리의 코드가 추상화하는 대신 시뮬레이션해야 하는 상태 기계와 일시적인 상태로 가득 차 있다는 것을 의미할 것이다.

Rx를 사용하지 않은 비동기시스템을 구현하게 될경우에 상태관리나 상태전환에 대한 케이스를 하나하나 관리해주어야하고 코드양이 많아질수록 관리해야하는 상태 및 상태전환이 많아짐에 따라서 코드의 복잡성이 증가되며  설계와 유지보수가 어려워지고 문제발생시의 원인추적의 어려움이 높아진다.  

허나 RxSwift를 사용함으로써  상태를 직접 관리하지 않고 이벤트 스트림으로 추상화 해서 사용한다는것이 포인트이고 흐름으로 표현되기에 전환과정이 명확하게 드러난다. 





Here is a sequence of numbers: **정상적으로 종료**


```
--1--2--3--4--5--6--| // terminates normally
```

Another sequence, with characters: **오류로 종료**

```
--a--b--a--a--a---d---X // terminates with error
```

Some sequences are finite while others are infinite, like a sequence of button taps: 무한히 지속

```
---tap-tap-------tap--->
```


이것들은 마블다이어그램이라고 불리우며 더많은 마블다이어 그램은 [여기서](http://rxmarbles.com) 확인할수 있습니다.

1. **next** 

- next는 **0회 이상 반복**을 의미합니다.  
- next는 시퀀스가 요소를 방출할 때의 onNext **이벤트**를 의미합니다.  
- 즉, 시퀀스는 **0번 또는 여러 번** 요소를 방출할 수 있다는 뜻입니다.  
- next* 부분은 시퀀스가 **0번 또는 여러 번의** onNext **이벤트**를 방출할 수 있음을 나타냅니다.  
- 예: 요소가 없을 수도 있고, 10개 이상 방출할 수도 있습니다.

<br/>

2. **(error | completed)?**

- | 는 **“또는”을 의미합니다.  
- error는 onError **이벤트**로, 시퀀스가 에러로 종료되었음을 나타냅니다.  
- completed는 onCompleted **이벤트**로, 시퀀스가 정상적으로 완료되었음을 나타냅니다.  
- ?는 **0회 또는 1회만 발생**할 수 있음을 의미합니다.  
- 즉, 시퀀스는 종료 이벤트로 onError **또는** onCompleted **중 하나만 발생할 수 있다**는 뜻입니다.  

<br/>

3. **(error | completed)?**

- onError**나** onCompleted **이벤트가 발생하면 시퀀스가 종료**됨을 나타냅니다.  
- 종료 이벤트가 발생한 후에는 더 이상 **다른 이벤트(onNext 포함)**를 방출할 수 없습니다.


> [!NOTE]
> **4가지의 Case**
> 
> **1. 요소 없이 완료되는 시퀀스**  
> **2. 요소를 방출한 후 완료되는 시퀀스**  
> **3. 요소를 방출하다가 에러로 종료되는 시퀀스**  
> **4.요소 없이 에러로 종료되는 시퀀스**  

```swift+
enum Event<Element>  {
    case next(Element)      // next element of a sequence
    case error(Swift.Error) // sequence failed with error
    case completed          // sequence terminated successfully
}

class Observable<Element> {
    func subscribe(_ observer: Observer<Element>) -> Disposable
}

protocol ObserverType {
    func on(_ event: Event<Element>)
}
```

- 시퀀스가 completed 또는 error 이벤트를 보낼때, 시퀀스 요소를 계산하는데 사용된 모든 내부 리소스가 해제 됩니다.
- 시퀀스 요소의 생성을 즉시 중단하고 리소스를 해제하려면 반환된 구독(subscription) 객체에서 dispose를 호출해야합니다.
- 시퀀스가 유한한 시간내에 종료된다면, dispose 를 호출하지 않거나 dispose(by: disposeBag)를 사용하지 않아도 **영구적인 리소스 누수는 발생하지 않습니다. 하지만, 시퀀스가 요소생성을 완료하거나 에러를 반환하여 종료할때 까지 해당 리소스는 계속 사용됩니다.

- 시퀀스가 자체적으로 종료되지 않는 경우(예: 버튼 클릭 이벤트 시퀀스), dispose를 수동으로 호출하거나, DisposeBag 내부에서 자동으로 처리하거나, takeUntil 연산자를 사용하는 등의 방법으로 정리하지 않으면 리소스가 영구적으로 할당됩니다.

-  DisposeBag이나 takeUntil 연산자를 사용하는 것은 리소스를 정리하는 확실한 방법입니다. 유한 시간 내에 종료되는 시퀀스라 하더라도, 프로덕션 환경에서는 이러한 방법들을 사용하는 것이 권장됩니다.
-  Swift의 Error 타입이 왜 제네릭이 아닌지 궁금하다면, [여기](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Design%20Rationale.md#design-rationale-%EC%84%A4%EA%B3%84-%EC%B2%A0%ED%95%99)에서 설명을 확인할 수 있습니다.



