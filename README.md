# RxSwift 

<br/>

$\it{\large{\color{#5ad7b7}“이\ 번역본은\ RxSwift\ 원문을\ 기반으로\ 작성되었습니다."}}$ [![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2FRinkim0515%2FRxSwift2025&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)         

 <br/>

---
## [공식문서](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/GettingStarted.md)  




## 개요

Swift를 학습함에 있어서 반응형 프로그래밍은 언젠간 넘어야할 산같은 존재라고 생각합니다.   

학습함에 있어서 초보자로써 이개념이 생소하고 이해하기 어렵다는 생각이 많이 들었고  저또한 공식문서 없이 공부하기 어려웠습니다.
 
허나 영어가 익숙치 않은 한국인으로써 학습하는데에 있어서 스스로 공부도하면서 타인에게 도움도 줄수있지 않을까 생각하였고
 
이내용으로 다른사람에게 도움이 되길 바랍니다. 



--- 



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
> | Chapter Subject |ㅇㅇ  | Keyword |
> |:---:| :---  | :--- |
> | **[1.Observables aka Sequences](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/01_Observables%20aka%20Sequences.md#observables-aka-sequences)**|| **[시퀀스로 알려진 구독가능한것](https://kimrindev.tistory.com/1)**<p> | 
> | **[2.Disposing](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/02_Disposing.md#disposing)**| ||
> | **[3.Implicit Observable guarantees](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/03_Implict%20Observable%20guarantees.md#implicitobservableguarantees-%EC%95%94%EB%AC%B5%EC%A0%81%EC%9D%B8-observable-%EB%B3%B4%EC%9E%A5)** ||
> | **[4.Creating your first Observable (aka observable sequence)](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/04_Creating%20your%20first%C2%A0Observable%C2%A0.md#observable-%EC%83%9D%EC%84%B1%EC%9D%98-%ED%95%B5%EC%8B%AC%EA%B0%9C%EB%85%90)** ||
> | **[5.Creating an Observable that performs work](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/05_Creating%20an%C2%A0Observable%C2%A0that%20performs%20work.md#creating-anobservablethat-performs-work)** ||
> | **[6.Sharing subscription and share operator](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/06_Sharing%20Subscription%20and%20the%20share%20Operator.md#%EA%B5%AC%EB%8F%85subscription-%EA%B3%B5%EC%9C%A0%EC%99%80-share-%EC%97%B0%EC%82%B0%EC%9E%90)** ||
> | **[7.Operators](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/07_Operator.md#%EC%97%B0%EC%82%B0%EC%9E%90)** | ||
> | **[8.Custom operators](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/08_Custom%20operator.md#%EC%82%AC%EC%9A%A9%EC%9E%90%EC%A0%95%EC%9D%98-%EC%97%B0%EC%82%B0%EC%9E%90)** | ||
> | **[9.Infallible](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/09_Infallible.md#%EC%8B%A4%ED%8C%A8%ED%95%98%EA%B1%B0%EB%82%98-%EC%98%A4%EB%A5%98%EB%A5%BC-%EB%B0%9C%EC%83%9D%EC%8B%9C%ED%82%A4%EC%A7%80-%EC%95%8A%EB%8A%94%EA%B2%83)** | |
> | **[10.Playgrounds](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/10_Playgrounds.md#playgrounds)** | |
> | **[11.Error handling](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/11_Error%20handling.md#%EC%97%90%EB%9F%AC%EB%A5%BC-%EC%B2%98%EB%A6%AC%ED%95%98%EA%B8%B0)** |  | 에러핸들링 |
> | **[12.Debugging Compile Errors](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/12_Debugging%20Compile%20Errors.md#%EC%BB%B4%ED%8C%8C%EC%9D%BC-%EC%97%90%EB%9F%AC%EB%A5%BC-%EB%94%94%EB%B2%84%EA%B9%85%ED%95%98%EA%B8%B0)** | |
> | **[13.Debugging](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/13_Debugging.md#%EB%94%94%EB%B2%84%EA%B9%85)** | |디버깅 |
> | **[14.Enabling Debug Mode](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/14_Enabling%20Debug%20Mode.md#%EB%94%94%EB%B2%84%EA%B7%B8-%EB%AA%A8%EB%93%9C%EC%9D%98-%ED%99%9C%EC%84%B1%ED%99%94)** | |
> | **[15.Debugging memoryleaks](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/15_Debugging%20memory%20leaks.md#%EB%A9%94%EB%AA%A8%EB%A6%AC%EB%88%84%EC%88%98-%EB%94%94%EB%B2%84%EA%B9%85)** |   | |
> | **[16.KVO](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/16_KVO.md#kvo)** | | |
> | **[17.UI layer tips](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/17_UI%20Layer%20Tips.md#ui-layer-tips)** | |
> | **[18.Making HTTP requests](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/18_Making%20HTTP%20Requests.md#http-%EC%9A%94%EC%B2%AD-%EB%A7%8C%EB%93%A4%EA%B8%B0)** ||
> | **[19.RxDataSources](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/19_RxDataSource.md#rxdatasources)** |  |
> | **[20.Driver]()** | 드라이버 | |
> | **[21.Traits: Driver, Single, Maybe, Completable](https://github.com/Rinkim0515/RxSwift2025/blob/main/Docs/%EC%BB%A4%EB%A6%AC%ED%81%98%EB%9F%BC/Traits.md#traits)** | |
> | **[22.Examples]** | 예시 |

---
## 추가 기재 내용 




---
정리해야할 키워드 : Monad, iOS 스택/힙 의 실제 동작원리 (class, struct)


## ContactMe
* 📱 +82 10.2412.7271
* 📧 kimrindev@gmail.com
