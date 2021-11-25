# Mock으로 static test 하기 (feat.Randoms)
개발할 때 의도적으로 Test 코드를 작성하려 노력한다. 완벽한 TDD(Test 작성 -> 로직 작성 -> 리팩토링)은 조금 어려워 잘 습관화하지 못하더라도
절반정도의 TDD (로직 작성 -> Test 작성 -> 리팩토링)은 습관화하려고 노력 중이다.

이번 우아한 테크 코스 1차 미션에서는 기본으로 제공해준 테스트만 패스하면 된다고 하였지만 Test 코드를 작성했을 때의
간편한 리팩토링을 몸소 체험한 사람으로서 자연스럽게 Test 코드를 작성했다.

그런데 문제가 발생했다. 바로 ```Randoms``` 라이브러리이다. 비즈니스 요구사항 상 ```Randoms```값을 받아 테스트해야하는데,
만약 난수를 받아오면 테스트 결과가 매번 달라진다. 처음에 바로 **가짜 객체**를 떠올렸으나 애노테이션을 쓰지 않고 ```mockito```를 사용해 본적이 없어
우선 콘솔 출력 형식의 잘못된 테스트를 작성했다.

코드는 다음과 같다.
```java
class GameProviderTest {

    private GameProvider gameProvider;

    @BeforeEach
    void setUp(){
        gameProvider = new GameProvider();
        gameProvider.generateAnswer();
        System.out.println(gameProvider);
    }

    @RepeatedTest(10)
    @DisplayName("정답 확인 - debug")
    @Test
    void checkAnswer(){
        GameScore gameScore = gameProvider.checkAnswer(new int[]{1, 2, 3});
        gameScore.printResult();
    }
}
```
매번 10개의 테스트를 눈으로 확인한다는 것이 너무 불편했고, 좋은 테스트 코드가 아니라는 생각을 했다.
우선 기능이 정상적으로 작동하는 것은 확인했으니 개발을 이어나갔지만, 계속 찜찜한 마음이 들었다.

그러다 문득 제공된 라이브러리 중 ```Assertions```에서 테스트 하는 방식이 떠올랐다. 바로 이전 포스팅을 살펴보니,
아래와 같은 코드가 있었다.
```java
assertTimeoutPreemptively(RANDOM_TEST_TIMEOUT, () -> {
        try (final MockedStatic<Randoms> mock = mockStatic(Randoms.class)) {
            mock.when(verification).thenReturn(value, Arrays.stream(values).toArray());
            executable.execute();
        }
    });
```

## MockedStatic
얼른 모키토 reference에 가서 살펴보니 [Mocking static methods](https://javadoc.io/static/org.mockito/mockito-core/4.1.0/org/mockito/Mockito.html#static_mocks)
라는 내용이 있었다.
![image](https://user-images.githubusercontent.com/87312401/143450648-1fbe8e59-efb6-4952-8944-127960f3cff5.png)

간단하게 해석해보자면 
> static 메서드를 현재 thread와 지정한 범위에서 mocking 할 수 있고 테스트 간 간섭이 발생하지 않는 다고 한다.
> ```try-with-resources``` 내부에서 사용하여 범위 내에서만 mocking하는 것을 추천한다고 한다.
>> ScopedMock.close() 를 받기 때문에 ```try-with-resources``` 사용 가능

이제 static 메서드에 대해 가짜 객체를 만드는 방법에 대해 알았으니, 각 테스트 상황에 맞게 이를 선언해주면 된다.

```java
@BeforeEach
void setUp(){
    try (final MockedStatic<Randoms> mock = mockStatic(Randoms.class)) {
        mock.when(() -> Randoms.pickNumberInRange(anyInt(),anyInt())).thenReturn(1,2,3);
        gameProvider = new GameProvider();
    }

}
```
테스트가 실행되기 전에 매번 ```Randoms``` 클래스를 mocking 해주고, 이번 프리코스에서 사용한 ```pickNumberInRange()```
메서드를 받아 ```1,2,3``` 을 순차적으로 return하게 만들어줬다. 그리고 mocking을 마무리한다.

이제 랜덤값도 임의로 정해졌으니, 내 비즈니스 로직이 잘 맞는지 확인해준다.

```java
@DisplayName("정답 확인_3스트라이크")
@Test
void checkAnswer_ThreeStrike(){
    GameScore gameScore = gameProvider.checkAnswer(new int[]{1, 2, 3});
    assertThat(gameScore.getStrike()).isEqualTo(3);
    assertThat(gameScore.getBall()).isEqualTo(0);
}
```
정확하게 3개 숫자의 자리수와, 값이 일치하므로 ```3스트라이크``` 가 나와야 한다.
이외에도 볼, 볼과 스트라이크, 낫싱의 경우도 테스트해본다.

![image](https://user-images.githubusercontent.com/87312401/143451872-bd94f62f-1790-4cd2-8fd2-0f9307465607.png)

테스트가 깔끔하게 통과하는 것을 볼 수 있다. 작년과 문제가 같다면 다음 프리코스에서도 랜덤값을 사용하는 것으로 아는데,
그때는 mocking을 통해 절반쯤의 TDD를 할 수 있을것 같다!
