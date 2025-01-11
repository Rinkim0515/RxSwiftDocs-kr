# Short - Circuit Logic

- short-circuit Logic은 논리연산에서 불필요한 계산을 건너뛰는 동작방식을 의미합니다.
-  논리 연산이 이미 결과를 확정지었을때 , 나머지 연산을 수행하지 않고 종료하는 최적화 기법 입니다.

예시로는 논리연산에서 
AND 연산자(&&)와 같이 
- 두 조건중 하나라도 false라면 결과는 항상 false입니다. 
- 따라서 첫번째 조건이 false인 경우, 두번째 조건은 평가 하지 않습니다.


또는 OR연산자(||) 와 같이 
- 두조건중 하나라도 true 라면 결과는 항상 true 입니다. 
- 따라서 첫번째 조건이 true라면, 두번째 조건은 평가하지않습니다.

### RxSwift에서의 Short-Circuit Logic 

RxSwift에서 short - circuit logic은 에러가 발생하면 Observable 체인의 나머지 작업을 중단하고 즉시 에러를 전파하는 방식으로 진행됩니다.


> 예시
```
let observable = Observable<Int>.create { observer in 
	observer.onNext(1)
	observer.onError(NSError(domain: "TestError", code: 1, userInfo: nil))
	observer.onNext(2)
	return Disposable.create()
}

observable
	.map { $0 * 2}
	.subscribe(
		onNext: { print($0)},
		onError: { print ("Error: \($0)") }
	)
```
> 출력
```
2
Error: Error Domain=TestError Code=1 "(null)"
```

- 첫번째 값 1이 onNext를 통해 방출되고, map 연산자를 통해서 2로 변환후에 Observer에게 전달 
- onError 이벤트가 발생하면 Observable 체인 전체가 중단되고, 이후 onNext(2)는 실행되지 않음


---

## && 와 || 은  단락 평가에 따라 반환하는 값이 결정이 된다는 특징이 있다.

정리하자면 최적화 기법은  && 연산자 || 연산자를 사용할때 쓸수 있는 최적화 기법인셈 

&&연산자는 

두개의 값이 모든 true일때 true를 반환하기때문에  
1번째 Bool 값이 true라면 두번째 Bool 값도 검사를 해야하지만  
1번째 Bool 값이 false라면 두번째 Bool 값을 검사하지 않는다.  

||연산자 역시 마찬가지로 

이값에서는 둘중하나가 true라고 한다면 true를 반환하는데   
1번째 Bool 값이 true 일경우에 2번째 값은 검사를 하지않고   
1번째 Bool 값이 false라면 2번째값을 검사해야한다.  




&&연산의 앞에 false가 와버린다면 2번째 조건은 검사를 안하고 false를 반환하기때문에   
||연산자앞에 true가 와버린다면 2번째 조건은 검사를 안하고 true를 반환하는 셈이다.



