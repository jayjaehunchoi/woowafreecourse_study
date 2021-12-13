# Optional 잘 사용하기
- 있거나 없는 값 표현
- null 대체
- 자바 8부터 추가된 표현

#### -> 편리한 방식으로 null을 비교할 수 있다.

### Optional 생성
```java
// 만약 Optional.of(null); 이면 nullPointerExaception
Optional<String> opt = Optional.of("val");

// null 가능한 생성
Optional<String> optNullable = Optional.ofNullable(null);
```

### Optional 값 가져오기
```java
Optional<String> opt = Optional.of("val");
opt.get(); // return "val"

Optional<String> optEmpty = Optional.empty();
optEmpty.get() // return NoSuchElementException
```

```NoSuchElementException```을 방지하기 위해 값이 있는지, 없는지 확인해야한다.

```java
opt.isPresent(); // 존재?
opt.isEmpty(); // 자바 11부터, 존재하지 않음?
```

존재하는 것만 확인하지 않고, 존재하면 특정 로직을 수행하게끔 만들어줄 수 있음
```java
opt.isPresent(val -> print(val));
opt.isPresentOrElse(val -> print(val), () -> throw new IllegalArgumentException());
```

존재하지 않으면 특정 로직을 수행시켜 주는 예약어도 존재한다.
```java
opt.orElse("default"); // 기본값 자체
opt.orElseGet(() -> "default") // 기본값
opt.orElseThrow(() -> new IllegalArgumentException()); // 없을때 예외처리
```

### Optional 값 변경
- ```map``` 메서드를 통해 입력한 함수를 실행시켜 ```Optional``` return
- 값이 없을 경우 빈 ```Optional``` 리턴

> 멤버 데이터를 가져와 생일로부터 얼만큼 시간이 지났는지 확인
```java
Member member = memberRepository.findById(1L).orElse(null);
LocalDate birthday = member.getBirthday();
int passedDays = calPassDays(birthday);
```

> 아래와 같이 ```map```을 사용하여 개선이 가능하다.
```java
Optional<Member> memberOpt = memberRepository.findById(1L);
Integer passedDays = memberOpt.map(member -> member.getBirthday)
                            .map(birthday -> calPassDays(birthday))
                            .orElse(0);
```
```stream```의 ```filter```와 같이 ```filter```를 걸어줄 수도 있다.
```java
Optional<String> filtered = opt.filter(str -> str.length() > 3) // filter를 충족하면, 기존 Optional을 반환
```

### 마무리
```ifPresent```를 사용하는 것은 마치 ```null```을 직접 사용하는 것과 같다.
이보다는 다양하게 ```Optional```의 메서드를 사용하자.
그리고 얼마든지 위 ```Optional``` 메서드를 유연하게 조합하여 사용하면 더 좋은 ```Optional```코드를 짤 수 있을 것이다.
