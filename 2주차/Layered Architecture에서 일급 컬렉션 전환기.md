# Layerd Architecture에서 일급 컬렉션으로 리팩토링

먼저 이번 프로젝트에서 ```Layerd Architecture```를 선택한 이유는 두 가지 정도였다.

1. ```Layerd Architecture```를 가장 많이 사용해봤지만, 이 구조에 깊은 영역에 대해서는 모르고 있었음.
2. 코드를 짜는 것 자체는 익숙해서, 빠르게 코드를 짜고 깊이 공부할 수 있다는 선택지

하지만 막상 코드를 짜보니 조금 고민들이 여러가지 생겼다.

1. 실제 ```rdb```에 ```data access```가 없는데```Repository```라는 이름을 사용하는 게 맞을까?
2. ```Repository```와 ```Service``` 레이어로 기능상 분리하였는데, ```Repository```가 하는 역할이 클래스를 만들 정도로 필요한가?
3. 오히려 분리된 레이어가 코드를 더 복잡하게 만들고 있지 않을까?
4. 일급 컬렉션을 만들어 ```data structure```와 ```logic```이 함께 있다면 더 편하지 않을까?

이 네 가지 고민에 자답을 해보니 , ```layerd architecture```에 대한 물음엔 모두 부정적 생각 뿐이었고 일급 컬렉션 사용에 대한 물음에는 긍정적인 대답이 나왔다.
여지없이 리팩토링을 결정했다.

## 일급 컬렉션 선택 이유
원래도 일급 컬렉션의 존재를 알고 있었으나, 이번 프리코스 과제에 사용할 생각을 전혀 하지 못하고 있었다.
그런데 코드가 아무리 봐도 좋아보이지 않았고 이를 개선할 방법이 뭐가 있을까 고민하다보니 문득 일급 컬렉션이라는 것이 떠올랐다.

일급 컬렉션을 선택한 이유는 다음과 같다.

1. ```Repository```에 있던 ```List```를 ```Wrapping```하여 우리 프리코스 과제에 딱 맞는 자료구조를 만들 수 있다.
2. ```Service```와 ```Repository``` 로 나뉘게 되어 떨어진 가독성을 하나의 일급 컬렉션을 통해 높일 수 있다. 
3. 상태와 행위를 한 클래스에 함께 배치함으로써 불필요한 코드들을 제거할 수 있다.

이렇게 리팩토링을 결정하고 나니, ```Repository```를 사용하는 것이 연습으로 용인되는 것이 아니라, 잘못된 사용이라는 생각이 강하게 들었다.

프로그램 설계를 완전히 뒤바꿔야 했기 때문에 ```TDD``` 를 통해 신뢰도 높은 리팩토링을 진행했다.

### 1. Test 코드 작성
이번 ```Refactoring```은 ```TDD```로 진행했다.

먼저 어떤 기능들이 필요할까 정리해봤다.
1. save
2. getAll
3. update(mock test)
4. getWinners
5. save_false(중복이름, 빈칸입력, 이름 제한 초과)
6. removeAll(다른 테스트에 영향 주지 않기 위함)


먼저 ```save```와 ```save_false```에 대한 테스트 코드를 작성해본다.

```java
@Test
void saveCars() {
    List<Car> carList = new ArrayList<>();
    String input = "jae,hun,choi"
    String[] tempCars = input.split(DELIMITER);
    NameValidator.isRightName(tempCars); // 기존의 검증 로직
    List<String> names = Arrays.asList(tempCars);
    names.forEach(name ->
            carList.add(new Car(name)));
}
```

그리고 테스트가 통과될 수 있게, ```Cars``` 클래스를 만들고 save로직을 짜준다.

```java
public class Cars {
    private final List<Car> carList = Collections.synchronizedList(new ArrayList<>());

    public void saveCars(String input) {
        String[] tempCars = input.split(DELIMITER);
        NameValidator.isRightName(tempCars);
        List<String> names = Arrays.asList(tempCars);
        names.forEach(name ->
                carList.add(new Car(name)));
    }
}
```

테스트가 통과하면 코드를 ```refactoring``` 해준다. 메서드를 분리해서 하나의 메서드가 하나의 기능만 수행하게끔 리팩토링한다.

```java
public void saveCars(String input) {
    validateNames(input).forEach(name ->
            carList.add(new Car(name)));
}
```

위와 같은 방식으로 ```Cars```라는 일급 컬렉션을 만들게 됐다. 전체 ```Test```코드를 짜고, ```Cars``` 클래스를 만들고 난 뒤, 기존 코드들을 살피며 ```Service```, ```Repository``` 지우기에 돌입했다.

> 이 때 추가적으로 코드를 리팩토링 해줬다, Cars에 싱글톤이 반영되어 있었는데, 이를 제거했다. (컬렉션인데 싱글톤인게 말이 안된다고 생각했음)
> 또 , Test 코드 몇몇 부분도 리팩토링 해줬다. 
> >```TestConstant```를 만들고, ```TestInstance```를 클래스 단위로 둬 반복적으로 생성 삭제가 되는 것을 막았다. 여러 ```assertThat```이 있는 경우 ```SoftAssertions.assertSoftly```를 사용하여 중간에 테스트가 끊기지 않게끔 코드를 작성했다.

### 마무리
좋은 개발자는 자신의 코드를 항상 되돌아보고 개선해야한다. 이번 기회에 좋은 경험을 해본 것 같아서 뿌듯하다. 
존경하는 개발자 분들이 많이 리팩토링, 장애 해결 등을 개발자가 가져야 할 필수 덕목이라고 말하시기에 내 코드를 돌아보고, 개선하는 습관이 생겼다.
지금껏 그랬듯 앞으로도, 리팩토링 양이 엄청나게 많더라도 여지없이 리팩토링을 선택하는 개발자가 되자.

#### 참고 
https://jojoldu.tistory.com/412  
https://www.baeldung.com/java-beforeall-afterall-non-static  
https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/SoftAssertions.html




