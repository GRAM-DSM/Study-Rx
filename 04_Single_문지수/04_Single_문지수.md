## Single

- Observable의 특수한 형태
- Observable 클래스는 데이터를 무한하게 발행할 수 있지만 Single 클래스는 오직 1개의 데이터만 발행하도록 한정
  - ex) 결과가 유일한 서버 API 호출할때 사용

- 데이터 하나가 발행과 동시에 종료 -> 종료시 onSuccess 호출됨

<hr></hr>

###  구성

- onSuccess( ) - onNext( ), onComplete( ) 함수가 통합됨
  - Single은 자신이 배출되는 하나의 값을 이 메서드를 통해 전달함
  - 이 함수가 호출되면 Single이 완료된 것을 의미함

- onError( )
  - Single은 항목을 배출할 수 없을 때 이 메서드를 통해 Throwable 객체 전달

<hr></hr>

### 마블다이어그램

![img](https://t1.daumcdn.net/cfile/tistory/9932EE3C5C1F4C6117)

- Single 클래스의 시간 표시줄은 왼쪽에서 오른쪽으로 흐름
- 왼쪽 위 빨간별은 Single 클래스에서 발행한 데이터 결과
- 왼쪽 아래의 빨간별은 flip( ) 함수의 결과값
- 오류가 발생하거나 Single이 비정상적으로 중단되는 경우에는 X로 표기

<hr></hr>

### 연산자

- Observable과 마찬가지로 Single도 다양한 연산자들을 제공

#### doOnError

- onError 메서드가 호출될 때 전달한 메서드를 실행하는 Single 리턴

![image-20210218155801178](/Users/munjisu/Library/Application Support/typora-user-images/image-20210218155801178.png)

#### doOnSuccess

- onSuccess 메서드가 호출될 때 전달한 메서드를 실행하는 Single을 리턴

![image-20210218155818951](/Users/munjisu/Library/Application Support/typora-user-images/image-20210218155818951.png)

#### onErrorReturn

- 명시된 항목을 배출하는 Single을 오류 알림을 내보내는 Single로 변환

  ![image-20210218155828089](/Users/munjisu/Library/Application Support/typora-user-images/image-20210218155828089.png)

#### zip & zipWith

- 두개 이상의 Single들이 배출한 항목에 적용된 함수의 결과를 담은 항목을 하나 배출하는 Single을 리턴

![image-20210218155837300](/Users/munjisu/Library/Application Support/typora-user-images/image-20210218155837300.png)

