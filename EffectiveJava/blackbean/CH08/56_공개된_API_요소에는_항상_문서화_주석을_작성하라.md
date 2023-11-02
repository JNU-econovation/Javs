# item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라
> 자바독을 사용한 다양한 예시들

### 자바독 (Javadoc)
- API 문서를 사람이 직접 작성하면 코드가 변경될 때마다 매번 함께 수정해줘야 하는데, 자바독(Javadoc) 유틸리티는 이 귀찮은 작업을 도와준다

## 문서화 주석
- API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다
- 유지보수까지 고려한다면 대다수의 공개되지 않은 클래스, 인터페이스, 생성자, 메서드, 필드까지 문서화 주석을 달아야 할 것이다

### 메서드용 문서화 주석
- 메서드가 무엇을 하는지, how가 아닌 what을 기술해야 한다
- 메서드를 호출하기 위한 전제조건(precondition)을 모두 나열해야 한다. 일반적으로 전제조건은 @throw 태그로 비검사 예외를 선언하여 암시적으로 기술한다. 또한 @param 태그를 이용해 그 조건에 영향받는 매개변수에 기술할 수도 있다
- 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건(postcondition)도 모두 나열해야 한다
- 부작용도 문서화해야 한다. 부작용이란, 사후조건으로 명확히 나타나지는 않지만 시스템의 상태에 어떠한 변화를 가져오는 것을 뜻한다
- 메서드의 계약(contract)을 완벽히 기술하려면, 모든 매개변수에 @param 태그를, 반환 타입이 void 가 아니라면 @return 태그를, 발생할 가능성이 있는 모든 예외에 @throw 태그를 달아야 한다. 다만, @return 태그의 설명이 메서드 설명과 같을 때 코딩 표준에서 허락한다면 @return 태그는 생략해도 좋다

### @param, @return, @throw
- `@param` : 해당 매개변수가 뜻하는 값
- `@return` : 반환 값
- `@throw` : 해당 예외를 던지는 조건

```java
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional
 * to the element position.
 *
 * @param  index index of element to return; must be
 *         non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index) {
    return null;
}
```

```java
// 한글 버전
/**
 * 이 리스트에서 지정한 위치의 원소를 반환한다.
 *
 * <p>이 메서드는 상수 시간에 수행됨을 보장하지 <i>않는다</i>. 구현에 따라
 * 원소의 위치에 비례해 시간이 걸릴 수도 있다.
 *
 * @param  index 반환할 원소의 인덱스; 0 이상이고 리스트 크기보다 작아야 한다.
 * @return 이 리스트에서 지정한 위치의 원소
 * @throws IndexOutOfBoundsException index가 범위를 벗어나면,
 * 즉, ({@code index < 0 || index >= this.size()})이면 발생한다.
 */
E get(int index) {
    return null;
}
```

### {@code}
- `{@code}` : 코드 예시를 보여줌
- 여러 줄은 `<pre>{@code}</pre>` 형태를 사용하자
```java
/**
 * ...
 * 즉, ({@code index < 0 || index >= this.size()})이면 발생한다.
 */
E get(int index) {
    return null;
}
```
```java
/**
 * <pre>{@code 예시로
 * 사용하는 코드
 * 줄바꿈이 자유롭다}</pre>
 */
```

### @implSpec
- `@implSpec` : 해당 메서드와 하위 클래스 사이의 계약 설명
- 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 하는게 목적이다
```java
// 자기사용 패턴 등 내부 구현 방식을 명확히 드러내기 위해 @implSpec 사용
/**
 * Returns true if this collection is empty.
 *
 * @implSpec This implementation returns {@code this.size() == 0}.
 *
 * @return true if this collection is empty
 */
public boolean isEmpty() {
    return false;
}
```
```java
// 한글 버전
/**
 * 이 컬렉션이 비었다면 true를 반환한다.
 * 
 * @implSpec 
 * 이 구현은 {@code this.size() == 0}의 결과를 반환한다.
 * 
 * @return 이 컬렉션이 비었다면 true, 그렇지 않다면 false
 */
public boolean isEmpty() {
    return false;
}
```

### {@literal}
- `{@literal}` : HTML 마크업이나 자바독 태그를 무시하게 해준다
- API 설명에 <, >, & 등의 HTML 메타문자를 포함시키려면 {@literal} 태그로 해당 메타 문자를 감싸면 된다
```java
/**
 * A geometric series converges if {@literal |r| < 1}.
 */
```

### {@summary}
- `{@summary}` : 요약 설명
```java
/**
 * 머스터드 대령이나 Mrs. 피콕 같은 용의자.
 * 
 * 최종 HTML에는 '머스터드 대령이나 Mrs.' 까지만 나온다.
 */
```
```java
// 방법 1
/**
 * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
 */
```
```java
// 방법 2
/**
 * {@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
 */
```

### {@index}
- `{@index}` : 자바독 문서에 색인 추가
- `{@Index 색인에 사용할 이름}` 형식으로 사용한다
```java
/**
 * This method complies with the {@index IEEE 754} standard.
 */
```

### 제네릭 타입, 제네릭 메서드
- 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 문서화 주석을 기술하자
```java
/**
 * 키와 값을 매핑하는 객체, 맵은 키를 중복해서 가질 수 없다
 * 즉, 키 하나가 가리킬 수 있는 값은 최대 1개다.
 * 
 * @param <K> 이 맵이 관리하는 키의 타입
 * @param <V> 매핑된 값의 타입
 */
public interface Map<K, V> { ... }
```

### 열거 타입
- 열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다
- 열거 타입 자체와 그 열거 타입의 public 메서드도 물론이다
```java
/**
 * An instrument section of a symphony orchestra.
 */
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe. */
    WOODWIND,

    /** Brass instruments, such as french horn and trumpet. */
    BRASS,

    /** Percussion instruments, such as timpani and cymbals. */
    PERCUSSION,

    /** Stringed instruments, such as violin and cello. */
    STRING;
}
```
```java
// 한글 버전 
/**
 * 심포니 오케스트라의 악기 세션.
 */
public enum OrchestraSection {
    /** 플루트, 클라리넷, 오보 같은 목관악기. */
    WOODWIND,

    /** 프렌치 호른, 트럼펫 같은 금관악기. */
    BRASS,

    /** 탐파니, 심벌즈 같은 타악기. */
    PERCUSSION,

    /** 바이올린, 첼로 같은 현악기. */
    STRING;
}
```

### 애너테이션 타입
- 애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다
- 요약 설명은 이 애너테이션을 사용하는 것이 어떤 의미인지를 설명한다
```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * The exception that the annotated test method must throw
     * in order to pass. (The test is permitted to throw any
     * subtype of the type described by this class object.)
     */
    Class<? extends Throwable> value();
}
```
```java

// 한글 버전
/**
 * 이 애너테이션이 달린 메서드는 명시한 예외를 던져야만 성공하는
 * 테스트 메서드임을 나타낸다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * 이 애너테이션을 단 테스트 메서드가 성공하려면 던져야 하는 예외.
     * (이 클래스의 하위 타입 예외는 모두 허용된다.)
     */
    Class<? extends Throwable> value();
}
```

### 패키지 문서화와 모듈 문서화
- 패키지를 설명하는 문서화 주석은 package-info.java 파일에 작성한다
- 모듈 관련 설명은 module-info.java 파일에 작성하면 된다

### 스레드 안전성과 직렬화 가능성
-  클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다
- 직렬화 가능한 클래스라면 직렬화 형태도 API 설명에 기술해야 한다. (@serial 태그를 활용해볼 수 있다.)

### 메서드 주석 상속 & {@inheritDoc}
- 자바독은 메서드 주석을 상속 시킬 수 있다
- 문서화 주석이 없는 API 요소를 발견하면 Javadoc이 가장 가까운 문서화 주석을 찾아준다