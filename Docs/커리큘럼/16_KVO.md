# KVO (Key-Value Observing)

KVO는 Objective-C 기반의 객체속성 변화를 감지하는 기능입니다. 하지만 타입안전성이 부족하고 메모리 관리가 복잡하다는 단점이 있습니다. 

RxSwift는 KVO를 위한 두가지 메서드를 제공합니다.
- `observe`
- `observeWeakly`

```swift
extension Reactive where Base: NSObject {
    public func observe<E>(
        type: E.Type,
        _ keyPath: String,
        options: KeyValueObservingOptions,
        retainSelf: Bool = true
    ) -> Observable<E?> {}
} 

#if !DISABLE_SWIZZLING
extension Reactive where Base: NSObject {
    public func observeWeakly<E>(
        type: E.Type,
        _ keyPath: String,
        options: KeyValueObservingOptions
    ) -> Observable<E?> {}
}
#endif
```


> UIView에서 frame을 관찰하는 예시  
> 경고) UIKit은 KVO를 완벽하게 지원하지 않지만 , 이방법은 동작합니다. 


```swift
view
  .rx.observe(CGRect.self, "frame")
  .subscribe(onNext: { frame in
    ...
  })

/////////또는/////////
view
  .rx.observeWeakly(CGRect.self, "frame")
  .subscribe(onNext: { frame in
    ...
  })
```
- UIKit 의 속성들은 KVO로 감시하는것이 일반적으로 권장되지 않습니다.
- 하지만 RxSwift로 observe, observeWeakly를 통해 UIView의 frame과 같은 속성도 안정적으로 감시할수 있습니다.

</br>
</br>

### rx.observe

rx.observe는 단순히 KVO를 감싼것이때문에 성능이 더 뛰어나지만, 사용에 제약이 있습니다.
- 강한참조 속성만 감시할수 있습니다.
- retainSelf를 false로 설정하면, 자기자신을 참조하지 않음으로써 메모리 누수를 방지 합니다.
- 약한 참조나 복잡한 객체 관계에서는 충돌이 발생할수 있습니다.

observe는 성능이 뛰어난 대신 감시할수있는 대상이 제한적입니다.
강한 참조속성만 관찰이 가능하고, 약한참조 나 복잡한관계에서는 메모리 해제 문제가 발생할수있습니다.

</br>
</br>


> Eg)
```swift
self.rx.observe(CGRect.self, "view.frame", retainSelf: false)
```

### rx.observeWeakly

rx.observeWeakly 는 객체의 메모리 해제를 안전하게 처리하기때문에 약간 느립니다.
- 약한참조 속성도 감시할수 있습니다.
- 객체의 소유관계가 불명확 해도 안전하게 동작합니다.
성능이 약간 떨어지지만 , 약한참조나 불확실한 객체관계에서도 안전성을 보장합니다.
복잡한 객체 트리나 메모리 관리가 어려운 상황에서 더안전합니다.

Eg)
```swift
someViewController.rx.observeWeakly(Bool.self, "isActive") 
```

</br>
</br>



### 구조체 감시

KVO는 Objective -C 기반이기때문에 주로 NSValue를 사용합니다.
**RxCocoa**는 **CGRect**, **CGSize**, **CGPoint**와 같은 구조체의 감시를 지원합니다.

- KVO는 **객체(Object)** 감시에 최적화되어 있어 **구조체(Struct)** 감시가 어렵습니다.
- RxCocoa는 자주 사용하는 **CGRect**, **CGSize**, **CGPoint**와 같은 구조체의 감시를 **내장 지원**합니다.
- 다른 구조체를 감시하려면 **NSValue**로 변환하거나, **KVORepresentable**을 구현해야 합니다.

</br>
</br>

### 최종요약

- **RxSwift**는 기존 **Objective-C의 KVO**의 복잡성과 불안정성을 해결합니다.
- **observe**는 성능이 뛰어나지만 **강한 참조**만 감시 가능.
- **observeWeakly**는 **약한 참조**도 감시 가능하지만 성능이 약간 느립니다.
- **UIKit** 속성도 **RxSwift**를 통해 안전하게 감시할 수 있습니다.





