# RxSwift 

<br/>

$\it{\large{\color{#5ad7b7}“이\ 번역본은\ RxSwift\ 원문을\ 기반으로\ 작성되었습니다."}}$ [![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2FRinkim0515%2FRxSwift2025&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)         

 <br/>

---
## 참고하기 좋은 링크 

> 공식문서 링크: [바로가기](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/GettingStarted.md) 



---
## 작성하게된 계기 

Swift를 학습함에 있어서 반응형 프로그래밍은 언젠간 넘어야할 산같은 존재라고 생각합니다.   

학습함에 있어서 초보자로써 이개념이 생소하고 이해하기 어렵다는 생각이 많이 들었고  저또한 공식문서 없이 공부하기 어려웠습니다.
 
허나 영어가 익숙치 않은 한국인으로써 학습하는데에 있어서 스스로 공부도하면서 타인에게 도움도 줄수있지 않을까 생각하였고
 
이내용으로 다른사람에게 도움이 되길 바랍니다. 



--- 
## 작성완성 방법

> 일단 영어원문을 번역해달라고 chatGPT 를 이용해서 번역을 돌리고 난후 부족한 설명이나 이해가 안가는 부분에대해서 찾아보고 정리하는 식으로 작성하였습니다. 

> 학습방식은 초반에는 복붙 을 이용해서 필요한 내용만 요약하는 형식으로 가져갔는데 다시 봐야하는 케이스가 빈번헤졌습니다( 집중도 이슈)

> 블로그에서 영문번역과 함께 확인하실수 있습니다. 


## Getting Started
 
### 시작하기전 배경지식
참고로 Swift에서 Protocol인 Sequence, Collection 이 있다.
```
public protocol Collection<Element>: Sequence {}
```

 - sequence는 요소들을 순차적으로 접근할수있는 데이터의 나열  
 *eg) Range, AnyIterator, stride*

 - collection은 sequence 프로토콜을 채택하고있는 모든 요소에 효율적으로 접근할수있는구조이며 특정인덱스 요소에 직접 접근이 가능하고, 항상정해진 모든 요소를 갖고있다
 - *eg) Array, Dictionary, Set*

>[!NOTE]
> 즉 컬렉션은 시퀀스를 채택하고 있기에 컬렉션인 배열또한 하나의 시퀀스 인것이다.  
> 이내용을 기반으로 Rxswift에서 observable 이라는 타입은 시퀀스라고 일컫는데  
> 이 데이터의 흐름이 순서대로 진행되기때문에 시퀀스로 간주되며 다른 시퀀스와의 차이점은 시간에 따라 비동기적으로 변화하는 시퀀스를 표현하는방식인 것  

<br/> <br/> <br/>

## 주요개념 및 목차 
> | Chapter Subject |  | Keyword |
> |:---:| :---  | :--- |
> | **[1.Observables aka Sequences](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Observables%20aka%20Sequences.md#observables-aka-sequences)**|| **[시퀀스로 알려진 구독가능한것](https://kimrindev.tistory.com/1)**<p> | 
> | **[2.Disposing](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Disposing.md#disposing)**| ||
> | **[3.Implicit Observable guarantees](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Implict%20Observable%20guarantees.md#implicitobservableguarantees-%EC%95%94%EB%AC%B5%EC%A0%81%EC%9D%B8-observable-%EB%B3%B4%EC%9E%A5)** ||
> | **[4.Creating your first Observable (aka observable sequence)](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Creating%20your%20own%C2%A0%60Observable%60%C2%A0(aka%20observable%20sequence).md#observable-%EC%83%9D%EC%84%B1%EC%9D%98-%ED%95%B5%EC%8B%AC%EA%B0%9C%EB%85%90)** ||
> | **[5.Creating an Observable that performs work](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Creating%20an%C2%A0Observable%C2%A0that%20performs%20work.md#creating-anobservablethat-performs-work)** ||
> | **[6.Sharing subscription and share operator](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Sharing%20Subscription%20and%20the%20share%20Operator.md#sharing-subscription-and-the-share-operator)** ||
> | **[7.Operators](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Operator.md#operator)** | ||
> | **[8.Custom operators](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Operator.md#custom-operators)** | ||
> | **[9.Infallible](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Infallible.md#infallible)** | |
> | **[10.Playgrounds](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Playgrounds.md#playgrounds)** | |
> | **[11.Error handling](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Error%20handling.md#error-handling)** |  | 에러핸들링 |
> | **[12.Debugging Compile Errors](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Debugging%20Compile%20Errors.md#debugging-compile-errors)** | |
> | **[13.Debugging](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/Debugging.md#debugging)** | |디버깅 |
> | **[14.Enabling Debug Mode](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/14_Enabling%20Debug%20Mode.md#%EB%94%94%EB%B2%84%EA%B7%B8-%EB%AA%A8%EB%93%9C%EC%9D%98-%ED%99%9C%EC%84%B1%ED%99%94)** | |
> | **[15.Debugging memoryleaks](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/14_Enabling%20Debug%20Mode.md#%EB%A9%94%EB%AA%A8%EB%A6%AC%EB%88%84%EC%88%98-%EB%94%94%EB%B2%84%EA%B9%85)** |   | |
> | **[16.KVO](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/15_KVO.md#kvo)** | |||
> | **17.UI layer tips** | |||
> | **18.Making HTTP requests** ||
> | **19.RxDataSources** | 
> | **20.Driver** | 드라이버 |
> | **21.Traits: Driver, Single, Maybe, Completable** |
> | **22.Examples** | 예시 |

---
## 추가 기재 내용 




---
정리해야할 키워드 : Monad, iOS 스택/힙 의 실제 동작원리 (class, struct)


## ContactMe
* 📱 +82 10.2412.7271
* 📧 kimrindev@gmail.com
