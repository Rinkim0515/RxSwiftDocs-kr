# Design Rationale (설계 철학)

## Why error type isn't generic (에러타입이 제네릭 타입이 아닌이유)

```
enum Event<Element>  {
    case next(Element)      // next element of a sequence
    case error(Error)       // sequence failed with error
    case completed          // sequence terminated successfully 
    }
```

<br/>

- 만약 에러 타입이 제네릭이라면, 두 Observable 간에 추가적인 객체 - 관계 불일치(임피던스 미스매치)
  를 발생시킬 수 있습니다.”
- 제네릭 에러 타입을 사용하면, 각 Observable이 서로 다른 에러 타입을 가지게 될 가능성이 높아집니다. 이로 인해 두 Observable을 결합하거나 변환할 때 **호환성 문제**가 발생하여 추가적인 작업(변환 로직)을 필요로 하게 되고, 코드가 더 복잡해질 수 있습니다.
- RxSwift에서 Error가 제네릭이 아닌 이유는 바로 이러한 “임피던스 불일치”를 줄이기 위함입니다.
- 에러 타입이 고정되어 있다면, 서로 다른 Observable을 결합하거나 조작할 때 에러 타입 간 변환이 필요 없으므로 코드가 간결하고 유지보수가 쉬워집니다.
- 반대로, 제네릭 에러 타입을 사용하면 모든 에러를 직접 변환하거나 매핑해야 하므로 복잡성이 증가합니다.

`observable<String, E1>`과 `Observable<String, E2>`라는 두 Observable이 있습니다.

이 두 Observable을 조합하거나 처리하려면, 결과 에러 타입이 E1, E2, 아니면 새로운 E3가 될지 결정해야 합니다. 이를 해결하려면 새로운 연산자 집합이 필요합니다.
