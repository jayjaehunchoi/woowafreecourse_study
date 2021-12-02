# Layered Architecture, 싱글톤

이번 주 과제의 핵심 요구사항은 ```클래스 분리```라고 생각한다. ```클래스 분리```라는 단어를 듣고,
바로 ```Layered Architecture```를 떠올렸다. 지난 주에는 실제 야구게임이 어떻게 동작할까에 대해 생각했다면,
이번 주 과제에는 조금 더 프로그래밍적으로 생각하여, ```Controller``` ```Service``` ```Repository```로 이어지는 ```Layered Architecture```를 구현하기로 마음 먹었다.

## What is Layered Architecture
이 단어에서의 ```Layered```는 물리적인 층이 아닌, 논리 층이다. 서버 계층을 ```presentation```, ```business logic``` ```data access```로 분리하여 3단계 논리 계층으로 구분한다.

### Presentation
- 화면 표현 및 전환처리, 서버에서는 ```Controller```를 이용하여 이 역할을 수행한다.
- 브라우저에서 데이터가 넘어올 때, ```DTO```에 대한 검증이 발생한다. 
(개인적으로 Controller가 복잡해지는 것을 선호하지 않아, 이외의 검증은 ```domain```혹은 ```Service```에서 하려 한다.)

### Business Logic
- 프로그램을 지탱하는 핵심 비즈니스 구현사항이 이 레이어에 속한다. ```Service```가 이 역할을 수행한다.
- 다양한 규칙, 검증, 흐름을 제어하고 ```data access```하는 영역과 밀접하게 연결되어 있다.

### Data Access
- ```dao``` ```repository```등 실제 데이터베이스와 가장 가까운 위치에 있는 클래스이다.
- 모든 데이터 처리를 이곳에서 수행한다. 순수하게 데이터와 관련된 일을 해야한다.

### 응집성을 높이고 의존도를 낮추라
- 상위 계층이 하위 계층을 호출하는 단방향성 유지
- 바로 밑의 근접 계층만 활용
- 상위 계층이 하위 계층에 영향을 받지 않게 설계
- 하위 계층은 자신을 사용하는 상위 계층을 몰라야 함
- 인터페이스를 통한 호출 (약한 결합)

이번 프로그램은 작은 단위의 프로그램이기 때문에, 모든 레이어가 하나의 패키지에 존재해도 크게 상관 없을 것이라 판단했다. 

## Singleton Repository
스프링 같은 경우는 스프링 컨테이너에 올릴 모든 객체를 싱글턴으로 관리한다. (```bean definition```에 따라 다르지만 default가 singleton) 이는 대규모 트래픽을 관리하기 위함이다. 
이번 과제는 대규모 트래픽이 들어올 일은 물론 없지만, 굳이 ```Repository```가 요청될때마다 생성될 이유가 없다. 따라서 이번 과제의 ```Repository```를 싱글톤으로 생성하였다.

하지만 문제가 있었다. 일반적으로 ```data base```를 사용하면 싱글턴 ```repository```를 무상태로 유지할 수 있다. 하지만 지금과 같은 경우에는 ```memory```에 저장소 역할을 하는 자료구조를 넣어야 하기 때문에 무상태가 유지되지 않는다. 자료구조 ```List```를 사용하기 때문에, ```List```이면서 ```thread-safe```를 보장해주는 방법을 알아봤다.

- Collections.synchronized()
   - 리스트 자체에 락이 걸림
   - 쓰기 동작에 있어 , CopyOnWriteArrayList보다 성능이 우월함
   
- CopyOnWriteArrayList<>()
   - 쓰기 동작시 요소를 복사하여 임시 배열을 만들고 동작 수행 후 원본 배열 만듦 (lock)
   - 읽기 동작은 락이 안걸림 성능 우월

쓰기, 읽기가 비슷한 수준으로 발생하고, 리스트의 크기가 얼마나 커질지 모르는 상황이기에, ```Collections.synchronized```를 선택했다.

동기화를 한다는 것 자체가 일반 ArrayList의 사용보다 성능이 안좋지만, 싱글턴을 사용하면서 ```thread-safe```하지 않은 자료구조를 사용한다는 것은 싱글턴의 의미를 완전히 상쇄시키는 행동이라고 생각했다.

사실 동시성을 고려해서 이런 자료구조를 사용하는 것이 처음이라 오류는 있을 수 있지만, 이번 기회를 통해 ```thread-safe```한 설계에 대해 조금이나마 알게된 것 같아 기쁘다. 꼭 동시성을 고려한 설계를 고민하고 실제로 해보고 싶었는데 이번에 도전할 수 있어 좋은 경험이 된 것 같다.

### 구조 
![image](https://user-images.githubusercontent.com/87312401/144345105-4e1a0bb4-73b9-4293-b5e0-b9932d329207.png)


참고 : https://engineering-skcc.github.io/microservice%20inner%20achitecture/inner-architecture-2/
https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedList-java.util.List-
