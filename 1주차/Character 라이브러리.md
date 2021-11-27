# Character 레퍼런스
이번 프로젝트를 진행하며, 입력된 ```char```값을 비교하거나, 숫자로 변경하는 경우가 있었다. 지금까지는 래퍼클래스인
```Character```의 메서드를 사용하지 않고 , ```char```을 이리저리 만져 사용했는데, 이번 프로젝트에서는 라이브러리도
뒤져보고 새롭게 알게된 메서드가 있어 추가적으로 어떤 메서드가 유용할지, 알아보는 시간을 가졌다.

## Character
>The Character class wraps a value of the primitive type char in an object. An object of class Character contains a single field whose type is char.
In addition, this class provides a large number of static methods for determining a character's category (lowercase letter, digit, etc.) and for converting characters from uppercase to lowercase and vice versa.

해석
> ```Character``` 클래스는 기본 자료형 ```char```을 wrapping 하는 클래스이다. ```Character``` 객체는
> 단일필드 ```char```을 갖고 있다. ```Character```클래스는 ```char```값의 카테고리를 구분하거나, 소문자 대문자로 변경할 수 있는 등의
> 다양한 정적 메서드를 제공하고 있다. 

#### 즉, 우리가 필요한 메서드를 정말 많이 갖고 있다는 뜻이다. 

#### * 참고 - wrapper class  
> 
> These are known as wrapper classes because they "wrap" the primitive data type into an object of that class
> 
>* wrapper class 존재 이유
> 
>   - primitive 타입을 wrapping 하여 Collections에 넣어줄 수 있음 -> Collections는 Generic E를 사용하는데, 오토박싱 가능한 래퍼 클래스만 받을 수 있음
>   - primitive 타입을 wrapping 하여 형변환, 이진 변환, 8진 , 16진 변환 등 다양한 기능을 제공함
## Method
- codePointAt(char[] c, int index)
> 배열 인덱스값을 code point로 전환했을때의 값
```java
char[] cArr = {'a','b','c','z'};
System.out.println(Character.codePointAt(cArr,3));

// return 122
```
- compare(char a, char b)
> a와 b값을 비교, a가 작으면 -, b가 작으면 +
```java
System.out.println(Character.compare('b','a'));
// return 1
```

- getNumericValue(char ch)
> character의 int값을 return 해준다. (숫자일 경우 숫자 그대로)
```java
System.out.println(Character.getNumericValue('1'));
System.out.println(Character.getNumericValue('c'));
// return 1
// return 12 -> character 값이면 return 안해줄 줄 알았는데, 까보니 c를 int로 형변환하여 코드포인트로 전환해 리턴해줌
```

- isAlphabetic(int codePoint)
> codePoint를 받아 알파벳인지 아닌지 확인 가능하다.
```java
System.out.println(Character.isAlphabetic('c'));
System.out.println(Character.isAlphabetic('1'));

// return true
// return false
```

- isDigit(char ch)
> character 값이 숫자인지 확인해준다.
```java
System.out.println(Character.isDigit('1'));

// return true
```

- isLetter(char ch), isLetterOrDigit(char ch)
> 문자인지, 문자 혹은 숫자인지 확인해준다.
```java
System.out.println(Character.isLetter('c'));
System.out.println(Character.isLetter('1'));
System.out.println(Character.isLetterOrDigit('1'));

// return true
// return false
// return true
```

- isWhiteSpace(char ch)
> 공백인지 아닌지 확인해준다.
```java
System.out.println(Character.isWhitespace(' '));

// return true
```

- isLower/UpperCase(char c), toLower/UpperCase(char c)
> 대/소문자 확인 , 대/소문자 변환
```java
Character c = Character.toUpperCase('c');
System.out.println(Character.isUpperCase(c));
c = Character.toLowerCase(c);
System.out.println(Character.isLowerCase(c));

// return true
// return true
```

### 마무리
지금까지 ```char```을 다룰때 Wrapper 클래스 메서드를 잘 사용하지 않았던 것 같다. 이번 프리코스를 하며
```getNumericValue``` 라거나, ```isDigit```을 처음 알게되었고 사용하게 됐다. 추가적으로 지금 회고를 하며
다양한 메서드들을 보게되었고, 생각보다 쓸모있는게 많다고 생각했다. 특히 나중에 테스트 코드를 작성할 때, ```char``` 값으로
테스트 코드를 작성한다면 꼭 래퍼클래스를 사용하여 명확하고 간결한 테스트 코드를 작성할 수 있을 것이라 생각된다.
