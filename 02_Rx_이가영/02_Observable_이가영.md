## 02_Observable

ReactiveX에서는 옵저버에 의해 임의의 순서에 따라 병렬로 실행되고 결과는 나중에 연산된다!



#### Observer pattern

객체의 상태 변화를 관찰하는 **관찰자 목록**을 객체에 등록하는 것이다

ex) 버튼을 누르면 버튼에 미리 등록해둔 action 메서드를 호출해 원하는 처리를 하는 것!

#### Observer?

관찰자라는 뜻으로 Observable에서 관찰할 수 있는 형태로 전달한 데이터를 받고 이에 대한 행동을 취한다. Observable에서 획득한 이벤트나 값을 수신하는 객체라고 한다.

#### Observable?

'관찰할 수 있는' 이라는 뜻으로 어떤 데이터를 Observer가 관찰하는 대상이라고 볼 수 있다. 상태 변화가 있을 때마다 메서드를 호출하여 객체가 직접 목록의 각 옵저버에게 여러 이벤트나 값을 보내는 역할을 한다. 쉽게 말해 옵저버에게 알림을 보내는 개체라고 볼 수 있다. 또 이벤트 스트림을 Observable 이라는 객체로 표현한 후 비동기 이벤트 기반의 프로그램 작성을 돕는다.



##### + 마블 다이어그램

Observable과 Observable의 전환을 표현하는지 시각적으로 보여준다



#### Observer Create

비동기 모델에서의 메소드 호출 흐름

1. 비동기 메소드 호출로 결과를 리턴받고 필요한 동작을 처리하는 메소드를 정의한다. = 해당 메소드는 Observer의 일부!
2. Obervable로 비동기 호출을 정의한다
3. 구독을 통해 옵저버를 Observable 객체에 연결시킨다(+ 동시에 Observable의 동작을 초기화)
4. 나머지 코드들을 모두 정의하고 메소드 호출로 결과가 리턴될 때마다 Observer의 메소드는 리턴 값 또는 Observable이 배출하는 항목들을 사용해서 연산을 시작한다

코드로 구현한 결과물

```python
// 옵저버의 onNext 핸들러를 정의한다, 하지만 실행하지는 않는다
// (이 예제에서는, 단순히 옵저버에 onNext 핸들러만 구현한다)
def myOnNext = { it -> /* 필요한 연산을 처리한다 */ };
// Observable을 정의하지만, 역시 실행하지는 않는다
def myObservable = someObservable(itsParameters);
// 옵저버가 Observable을 구독한다. 그리고 Observable을 실행한다
myObservable.subscribe(myOnNext);
// 필요한 코드를 구현한다
```



#### Observer와 Observable 연결 ! - Subscribe

- onNext

  Observable은 새로운 항목들을 배출할 때마다 이 메소드를 호출한다. 

  이 메소드는 Observable이 배출하는 항목을 파라미터로 전달받는다

- onError

  Observable은 기대하는 데이터가 생성되지 않았거나 다른 이유로 오류가 발생할 경우 오류를 알리기 위해 이 메소드를 호출한다. 만약 이 메소드가 호출되면 onNext나 onCompleted는 더 이상 호출되지 않는다. 

  이 메소드는 오류 정보를 저장하고 있는 객체를 파라미터로 전달받는다

- onCompleted

  오류가 발생하지 않았다면 Observable은 마지막 onNext를 호출한 후 이 메소드를 호출한다.

<code>onNext</code> 는 0번 이상 호출 될 수 있고 <code>onError</code>나 <code>onCompleted</code> 는 둘 중 하나를 마지막으로 호출한다(둘 모두를 호출하진 않음)



코드로 구현한 예제

```python
def myOnNext     = { item -> /* 필요한 연산을 처리한다 */ };
def myError      = { throwable -> /* 실패한 호출에 대응한다 */ };
def myComplete   = { /* 최종 응답 후 정리 작업을 한다 */ };
def myObservable = someMethod(itsParameters);
myObservable.subscribe(myOnNext, myError, myComplete);
// 필요한 코드를 계속 구현한다
```



현재 구독 중인 Observable 중 옵저버가 더 이상 구독을 원하지 않는 경우에는 이 메소드를 호출해서 구독을 해지할 수 있다. <code>unsubscribe</code> 를 이용하여 더 이상 관심있는 다른 옵저버가 존재하지 않는다면 Observable들은 새로운 항목을 배출하지 않는다.



#### Hot & Cold Observable

Q. Observable은 연속된 항목들을 언제 배출할까?

A. Observable에 따라 다르다



**Hot Observable** 은 생성되자마자 항목들을 배출하기도 하기 때문에 이 Observable을 구독하는 옵저버들은 어떤 경우에는 항목들이 배출되는 중간부터 구독할 수 있다

**Cold Observable** 은 옵저버가 구독할 때까지 항목을 배출하지 않기 떄문에 이 Observable을 구독하는 옵저버는 Observable이 배출하는 항목 전체를 구독할 수 있도록 보장받는다



#### Observable 연산자를 활용한 구성

Observable 연산자들로 Observable이 배출하는 연속된 항목들을 변환시키고 결합하고 조작하는 기능들을 제공한다 이 연산자들은 콜백이 제공하는 효율적인 장점들을 바탕으로 선언적인 방법을 통해 연속된 비동기 호출을 구성할 수 있는 방법을 제공한다

- Observable 생성

  `Create`, `Defer`, `Empty`/`Never`/`Throw`, `From`, `Interval`, `Just`, `Range`, `Repeat`, `Start`, 그리고 `Timer`

- Observable 항목 변환

  `Buffer`, `FlatMap`, `GroupBy`, `Map`, `Scan`, 그리고 `Window`

- Observable 필터

  `Debounce`, `Distinct`, `ElementAt`, `Filter`, `First`, `IgnoreElements`, `Last`, `Sample`, `Skip`, `SkipLast`, `Take`, 그리고 `TakeLast`

- Observable 결합

  `And`/`Then`/`When`, `CombineLatest`, `Join`, `Merge`, `StartWith`, `Switch`, 그리고 `Zip`

- 오류 처리 연산자

  `Catch` 그리고 `Retry`

- 유틸리티 연산자

  `Delay`, `Do`, `Materialize`/`Dematerialize`, `ObserveOn`, `Serialize`, `Subscribe`, `SubscribeOn`, `TimeInterval`, `Timeout`, `Timestamp`, 그리고 `Using`

- 조건 및 불린(Boolean) 연산자

  `All`, `Amb`, `Contains`, `DefaultIfEmpty`, `SequenceEqual`, `SkipUntil`, `SkipWhile`, `TakeUntil`, 그리고 `TakeWhile`

- 수학과 조합 연산자

  `Average`, `Concat`, `Count`, `Max`, `Min`, `Reduce`, 그리고 `Sum`

- 변환 Observable

  `To`

- 연결 가능한 Observable 연산자

  `Connect`, `Publish`, `RefCount`, 그리고 `Replay`

- 역압(backpressure) 연산자

  특정 제어흐름 원칙들을 적용하는 다양한 연산자들

