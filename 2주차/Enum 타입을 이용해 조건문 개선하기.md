# Enum 타입 활용

프리코스 과제에는 독특한 요구사항이 있다. 바로 ```else``` 예약어를 사용하지 않는 것이다.
지금껏 여러 객체지향 관련 도서, 강의를 보며 ```else```에 대한 부정적인 이야기를 많이 들어왔다.
유지보수, 확장에 취약하다는 점이 대표적이다.

이번 주 과제까지는 ```else```를 사용하지 않는 것이 정말 쉬웠다. 하지만 다음 과제를 미리 보니 ```Enum``` 타입을 사용하지 않으면, ```else```사용이 필수적이겠다는 생각이 들었다.
그래서 간만에 ```Enum```으로 다형성 활용, ```Enum```으로 함수형 인터페이스 사용, ```Enum```으로 함수형 인터페이스 대체 사용법을 정리하려 한다.

## 1. Enum 다형성
작년 프리 코스 기준으로 예시를 가져왔다.

```java
public enum MainFeatures {

    // 1
    MANAGE_STATION("1", new StationController()),
    MANAGE_LINE("2", new LineController()),
    MANAGE_SECTION("3", new SectionController()),
    PRINT_ROUTE_MAP("4", new RouteController()),
    QUIT("Q", null);

    private String input;
    private Controller controller;

    MainFeatures(String input, Controller controller){
        this.input = input;
        this.controller = controller;
    }

    // 2
    public Controller getController() {
        return controller;
    }

    public String getInput() {
        return input;
    }


    // 3
    public static MainFeatures getFeatureByInput(String inputNumber){
        return Arrays.stream(MainFeatures.values())
                .filter(value -> value.getInput().equals(inputNumber))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException(" 선택할 수 없는 항목입니다."));
    }
}
```
### 1 . values에 값 선언해주기.

요구사항을 확인해보면 입력한 번호에 따라 선택지가 갈린다. 만약 선택지가 한정적이라면 ```if - else```를 사용하는 것도 나쁘지 않다고 생각한다.
하지만 기능이 언제 확장될지 모르고 코드가 깔끔하게 유지되기 위해서는 ```Enum```타입에 값을 선언해주고 이를 활용하는게 좋다.

> 이 부분에서 다형성을 사용한 이유는, 각 선택지가 나름 큰 기능을 차지한다고 생각했기 때문이다.

```java
// 1
MANAGE_STATION("1", new StationController()),
MANAGE_LINE("2", new LineController()),
MANAGE_SECTION("3", new SectionController()),
PRINT_ROUTE_MAP("4", new RouteController()),
QUIT("Q", null);

private String input;
private Controller controller;

MainFeatures(String input, Controller controller){
    this.input = input;
    this.controller = controller;
}
```
기본적으로, 괄호에 값을 선언해주고 이 변수를 ```private```으로 선언해준뒤 생성자로 마무리해줘야 컴파일 에러가 발생하지 않는다.

### 2. getter
```java
// 2
public Controller getController() {
    return controller;
}

public String getInput() {
    return input;
}
```
이 부분을 활용해서, 어떤 값을 선택했고 어떤 ```Controller```를 생성해줄지 매핑해준다.


### 3. 입력값에 따른 return
```java
// 3
public static MainFeatures getFeatureByInput(String inputNumber){
    return Arrays.stream(MainFeatures.values())
            .filter(value -> value.getInput().equals(inputNumber))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException(" 선택할 수 없는 항목입니다."));
}
```
```Stream```을 활용해서, 입력받은 값에 대해 ```filter```를 걸어주고 가장 앞의 값을 찾아 ```Optional```로 반환 받는다.
이후 ```Optional```값이 ```null```이라면 바로 에러를 던져준다. 이후 ```Controller```에서 ```getController```메서드를 이용하여 다형성을 활용할 수 있다.


## 2. Enum 함수형 인터페이스 사용

Java 8 부터 Java에서도 함수형 인터페이스가 사용 가능해졌다. ```Enum```에서도 이를 활용할 수 있다.

```java
public enum StationFeatures {

    STATION_ENROLL("1", () -> {
        String input = InputView.printEnrollStation(new Scanner(System.in));
        StationRepository.addStation(new Station(input));
        InputView.completeEnrollStation();
    }),
    STATION_DELETE("2", () -> {
        String input = InputView.printDeleteStation(new Scanner(System.in));
        StationRepository.deleteStation(input);
        InputView.completeDeleteStation();
    }),
    STATION_FIND("3", () -> {
        System.out.println("## 역 목록");
        StationRepository.findAllStationName().forEach(name -> System.out.println(Constant.PREFIX_INFO + " " +name));
    }),
    BACK("B", () -> {});

    private String input;
    private Runnable runnable;

    StationFeatures(String input, Runnable runnable) {
        this.input = input;
        this.runnable = runnable;
    }

    public String getInput() {
        return input;
    }

    public Runnable getRunnable() {
        return runnable;
    }

    public static StationFeatures getStationFeature(String inputNumber){
        return Arrays.stream(StationFeatures.values())
                .filter(value -> value.getInput().equals(inputNumber))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException(Constant.PREFIX_ERROR + " 선택할 수 없습니다."));
    }

    public void run(){
       this.getRunnable().run();
    }
}
```

위의 다형성과 유사하다. 람다식을 매개변수자리에 넣어주기만 하면 된다.
여기서 ```Runnable```을 사용한 이유는 인자로 아무 값도 안넘겨주고, void 타입이기 때문이다. 상황에 맞게 사용하면 된다. ```Predicate``` ```Consumer``` 등등

## 3. 함수형 인터페이스 대체

분명 Java 7 이하 버전을 사용하는 사람이 있을 것이다. 또, 함수형 인터페이스에 익숙하지 않은 사람도 있다.
당연히 이를 대체할 수 있는 방법도 존재한다.

위에서 선언한 함수형 인터페이스 대신 다음과 같이 추상 메서드를 구현한다.
```java
abstract void run(); // 인자 x , void 타입
```

그리고 이 추상메서드를 상속받아 사용하면 된다.
```java
STATION_ENROLL("1"){
    @Override
    void run() {
        String input = InputView.printEnrollStation(new Scanner(System.in));
        StationRepository.addStation(new Station(input));
        InputView.completeEnrollStation();
    }
}
```

## 마무리
```Enum``` 타입을 사용하면 요청값을 정확히 한정지을 수 있으며, 코드를 이해하는데 정말 간편하다.
또, 개인적으로는 ```if - else``` 를 남발하는 것보다 가독성, 유지보수 측면에서 큰 장점이 있다고 생각한다.
다음 프리코스 미션부터는 ```선택지```가 포함된다. 선택지에 따라 다른 화면들을 제공해줘야 하는데, ```Enumb```은 해당 미션에는 필수적으로 반영해야 하는 사항이다.


참고 : https://techblog.woowahan.com/2527/

