### Subject!

Observable과 Observer 역할을 둘 다 수행할 수 있는 특별한 형태입니다. Observer인 동시에 Observable이죠. 실시간으로 Observable에 새로운 값을 추가하고 subscribers에게 방출합니다. 그리고 Subjects는 Cold Observable을 Hot하게 변형하는 효과를 얻을 수 있습니다. Multicast 방식으로 구독하고 있는 Observer들에게 값을 발행할 수 있습니다.

Observable을 상속하고 있으며 ObserverType 프로토콜을 채택하고 있습니다. 이는 Subject가 **Observable + Observer** 라는 것을 의미합니다.

여기서 질문이 생깁니다. 그러면 Observable과 Subject의 차이점은 뭘까요?



#### Observable vs Subject

|                           Subject                            |                          Observable                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| multicast방식이기 때문에 여러 개의 Observer에게 이벤트를 발행 | 반대로 unicast 방식이므로 하나의 Observer에게만 이벤트를 발행 |
| Observer들의 정보를 저장하고 발행된 이벤트를 모든 Observer에게 제공 | Observer가 구독하게 되면 그때마다 새로 create하여 발행하는 과정 |
|             state를 가지고 data를 메모리에 저장              |                 단지 함수일 뿐, state 존재 X                 |
| 자주 데이터를 저장하고 수정할 때<br />여러 개의 옵저버가 data를 관찰해야할 때<br /> Observer와 Observable 사이의 프록시 역할 |     하나의 Observer에 대한 간단한 Observable이 필요할 때     |



#### Varieties of Subject

1. AysncSubject

   ![스크린샷 2021-02-21 오후 11.43.48](/Users/igayeong/Library/Application Support/typora-user-images/스크린샷 2021-02-21 오후 11.43.48.png)

   마블 다이어그램을 보게되면 Observable로부터 배출된 마지막 값을 배출하고 소스 Observable의 동작이 완료된 후에야 동작합니다. 첫번째 subscribe 라인에서 맨마지막 값을 배출하게 되면 파란색 이벤트입니다. 그리고 두번째 subscribe 라인에서 맨마지막 값 역시 파란색 이벤트이므로 위의 사진과 같이 배출하게됩니다. 

   ❗️만약 해당 Observable이 오류로 인해 종료될 경우엔 아무 항목도 배출하지 않고 발생한 오류를 그대로 전달합니다.

2. BehaviorSubject

   ![스크린샷 2021-02-21 오후 11.49.16](/Users/igayeong/Library/Application Support/typora-user-images/스크린샷 2021-02-21 오후 11.49.16.png)

   BehaviorSubject는 구독하기 시작하면 초기값을 가진 형태의 subject라고 볼 수 있습니다. 가장 최근에 발행한 항목의 발행을 시작하며 그 이후의 이벤트들에 의해 발행된 항목들을 계속 발행합니다. 

   ❗️만약 오류 때문에 종료되면 아무런 항목도 배출하지 않고 Observable에서 발생한 오류를 그대로 전달합니다.

3. PublishSubject

   ![스크린샷 2021-02-21 오후 11.56.44](/Users/igayeong/Library/Application Support/typora-user-images/스크린샷 2021-02-21 오후 11.56.44.png)

   마블 다이어그램에서 보면 구독 이후에 Observable이 배출한 이벤트들만 Observer에게 배출하는 형태를 볼 수 있습니다. PublishSubject 사용 시 주의할 점은 subject가 생성되는 시점과 Observer가 이 subject를 구독하기 시작하는 시점 그 사이에 배출되는 항목들은 잃어버릴 수 있습니다. 그때를 대비해 모든 Observer가 구독을 시작했는지 체크하는 cold observable들을 생성하거나 ReplaySubject를 사용하면 해결할 수 있습니다.

   ❗️만약 오류 때문에 종료되면 아무런 항목도 배출하지 않고 Observable에서 발생한 오류를 그대로 전달합니다.

4. ReplaySubject

   ![스크린샷 2021-02-22 오전 12.00.20](/Users/igayeong/Library/Application Support/typora-user-images/스크린샷 2021-02-22 오전 12.00.20.png)

   Observer가 구독을 시작한 시점과 관계없이 Observable들이 배출한 모든 항목들을 모든 Observer에게 배출합니다. 확실히 여태까지 봐온 다이어그램과의 차이가 있는 걸 느끼죠? 늦게 구독을 시작해도 전에 배출된 이벤트들도 배출됩니다. 

   ❗️멀티스레드 환경에서는 Observable 계약 위반과 Subject에서 어느 항목 또는 알림을 먼저 재생해야하는지 모호함이 발생하기 때문에 호출을 유발시키는 onNext 메소드는 사용하지 않도록 주의해햐합니다.



#### Example Codes

1. AysncSubject

2. BehaviorSubject

   - RxSwift

   ```swift
   let disposeBag = DisposeBag()
   let subject = BahaviorSubject<String>(value: "init")
   subject.onNext("Last Issue")
   
   subject.subscribe { event in
     print(event)  
   }
   
   subject.onNext("Issue 1")
   //Last Issue
   //Issue 1
   ```

   - RxJava

   ```java
    BehaviorSubject<Object> subject = BehaviorSubject.create("default");
   
    subject.subscribe(observer);
   
    subject.onNext("one");
    subject.onNext("two");
    subject.onNext("three");
   //default one two three
   ```

3. PublishSubject

   - RxSwift

   ```swift
   let disposeBag = DisposeBag()
   let subject = PublishSubject<String>()
   subject.onNext("Issue 1")
   
   subject.subscribe { event in
     print(event)  
   }
   
   subject.onNext("Issue 2")
   subject.onNext("Issue 3")
   //Issue 2
   //Issue 3
   ```

   - RxJava

   ```java
   PublishSubject<Integer> source = PublishSubject.create();
   
   source.subscribe(getFirstObserver()); 
   
   source.onNext(1);
   source.onNext(2);
   source.onNext(3);
   //1 2 3
   
   source.subscribe(getSecondObserver());
   
   //4
   source.onNext(4);
   source.onComplete();
   ```

4. ReplaySubject

   - RxSwift

   ```swift
   let disposeBag = DisposeBag()
   let subject = ReplaySubject<String>.create(bufferSize: 2)
   
   subject.onNext("Issue 1")
   subject.onNext("Issue 2")
   subject.onNext("Issue 3")
   
   subject.subscribe { event in
     print(event)  
   }
   
   subject.onNext("Issue 4")
   //Issue 2
   //Issue 3
   //Issue 4
   ```

   Subscribe 되는 시점에서 2개 (bufferSize 만큼의 최신 이벤트를 저장)

   - RxJava

   ```java
   ReplaySubject<Integer> source = ReplaySubject.create();
   
   source.subscribe(getFirstObserver());
   
   source.onNext(1);
   source.onNext(2);
   source.onNext(3);
   source.onNext(4);
   source.onComplete();
   
   source.subscribe(getSecondObserver());
   ```

   

