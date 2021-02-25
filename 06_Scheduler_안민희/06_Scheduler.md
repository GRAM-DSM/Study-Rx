### Scheduler

#### 스케줄러란

스케줄러는 **어떤 프로그램의 세부 일정을 주관하는 관리자**로 생각하면 이해가 쉽다

요구사항에 맞게 비동기로 동작할 수 있도록 바꿔줘야하는데 이때 스케줄러를 사용한다.

- 스케줄러는 Rx 코드를 어느 스레드에서 실행할지 지정할 수 있다.
- subscribeOn() 함수와 observeOn() 함수를 모두 지정하면 Observable에서 데이터 흐름이 발생하는 스레드와 처리된 결과를 구독자에게 발행하는 스레드를 분리할 수 있다.
- subscribeOn() 함수만 호출하면 Observable의 모든 흐름이 동일한 스레드에서 실행된다.
- 스케줄러를 별도로 지정하지 않으면 현재(main) 스레드에서 동작을 실행한다.



#### 스케줄러의 종류

1. 계산 스케줄러

CPU에 대응하는 계산용 스케줄러이다.계산 작업을 할 때는 대기 시간 없이 빠르게 결과를 도출하는 것이 중요하다. 이 말은 계산 스케줄러는 입출력(I/O) 작업을 하지 않는 스케줄러라고 생각하면 된다. 내부적으로 스레드 풀을 생성하며 스레드 개수는 기본적으로 프로세서 개수와 동일하다.



RxJava

```kotlin
String[] orgs = {"1", "3", "5"};
Observable<String> source = Observable.fromArray(orgs)
	.zipWith(Observable.interval(100L, TimeUnit.MILLISECONDS), (a,b) -> a);

// 구독 1
source.map(item -> "<<" + item + ">>")
	.subscribeOn(Schedulers.computation())
	.subscribe(Log::i);

// 구독 2
source.map(item -> "##" + item + "##")
	.subscribeOn(Schedulers.computation())
	.subscribe(Log::i);
```

데이터 흐름은 1, 3, 5로 동일하다. 앞서 zipWith() 함수로 데이터와 시간을 합성할 수 있다고 했는데 여기서도 배열에 들어있는 데이터와 interval() 함수를 합성하여 시간간격으로 데이터를 발행했다.

람다 표현식으로 입력한 (a, b) -> a 는 시간은 그대로 두고 데이터만 취한다

출력은 구독 1과 구독 2가 번갈아서 나오는 것을 볼 수 있다. 이는 첫 번째 구독과 두 번째 구독이 거의 동시에 이루어지기 때문에 내부에서 동일한 스레드에 작업을 할당했기 때문이다.



2. IO 스케줄러

**네트워크상의 요청을 처리하거나 각종 입출력 작업을 실행**하기 위한 스케줄러이다. 계산 스케줄러와 다른 점은 기본으로 생성되는 스레드 개수가 다르다는 것이다.

즉 계산 스케줄러는 CPU 개수만큼 스레드를 생성하지만 IO 스케줄러는 필요할 때마다 스케줄러를 계속 생성한다. 입출력 작업을 비동기로 실행되지만 결과를 얻기까지 대기 시간이 길다.

- 계산 스케줄러: 일반적인 계산 작업
- IO 스케줄러: 네트워크상의 요청, 파일 입출력, DB 쿼리 등



RxJava

```kotlin
// c 드라이브 루트 디렉터리에 파일 목록 생성
String root = "C:\\";
File[] files = new File(root).listFiles();
Observable<String> source = Observable.fromArray(files)
	.filter(f -> !f.isDirectory())
	.map(f -> f.getAbsolutePath())
	.subscribeOn(Schedulers.io());

source.subscribe(Log::i);
```

해당 경로의 File 객체를 생성하여 lisFiles() 메서드를 호출하면 파일 목록을 File[] 배열로 리턴한다. 파일 중에 디렉터리는 제외하고 파일들만 필터링 한다.

source 변수는 File[] 배열에 있는 파일의 절대 경로를 발행하는 Observable로 IO 스케줄러에서 실행된다



3. 트램펄린 스케줄러

**새로운 스레드를 생성하지 않고 현재 스레드에 무한한 크기의 대기 행렬를 생성**하는 스케줄러이다.

새로운 스레드를 생성하지 않는다는 것과 대기 행렬을 자동으로 만들어 준다는 것이 계산 스케줄러, IO 스케줄러와 다르다.

```kotlin
String[] orgs = {"1", "3", "5"};
Observable<String> source = Observable.fromArray(orgs);

// 구독 1
source.subscribeOn(Schedulers.trampoline())
	.map(data -> "<<" + data + ">>")
	.subscribe(Log::i);

// 구독 2
source.subscribeOn(Schedulers.trampoline())
	.map(data -> "##" + data + "##")
	.subscribe(Log::i);
```

배열 데이터를 트램펄린 스케줄러를 활용해 Observable에서 발생한다. subscribeOn() 함수의 호출 위치가 IO 스케줄러의 예제보다 앞에 있는데 호출 위치는 상관이 없다. 처음에 지정한 스레드로 구독자에게 데이터를 발행한다.

실행 결과는 구독 1을 모두 발행한 뒤 구독 2를 순차적으로 발행한다. 큐에 작업을 넣은 후 1개씩 꺼내어 동작하므로 첫 번째 구독과 두 번째 구독의 실행 순서가 바뀌는 경우는 발생하지 않는다.



#### observeOn() 함수의 활용

스케줄러의 핵심은 **제공되는 스케줄러의 종류를 선택한 후 subscribeOn() 과 observeOn() 함수를 호출**하는 것이다.

subscribeOn() 함수

- Observable에서 구독자가 subscribe() 함수를 호출했을 때 데이터 흐름을 발행하는 스레드를 지정
-  observeOn() 함수는 처리된 결과를 구독자에게 전달하는 스레드를 지정
- 처음 지정한 스레드를 고정시키므로 다시 호출해도 무시한다.

observeOn() 함수는 여러 번 호출할 수 있으며 호출되면 그다음부터 동작하는 스레드를 바꿀 수 있다.