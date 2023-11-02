### ☁️ For-Each문을 사용하기 이전

#### 전통적 for문을 사용해서 반복하기
▶️ 컬렉션 순회
```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ){
	Element e = i.next();
    ...  //e로 무언가를 한다.
}
```

▶️ 배열 순회
```java
for (int i = 0; i < a.length; i++) {
    ..// a[i]로 무언가를 한다.
}
```

앞선 장에서 말했던 `while` 문 보다는 낫지만, 가장 좋은 방법은 아니다. 

1. 반복자와 인덱스 변수는 코드만 지저분하게 할 뿐 꼭 필요한 것은 원소들뿐이다.
2. 반복자와 인덱스 '변수'를 잘못 사용할 가능성이 높아진다.
3. 컬렉션이나 배열이냐에 따라 코드 형태가 달라진다.

이러한 문제들은 `for-each` 문을 사용하면 모두 해결된다.

### ☁️ For-Each문이란?
`for-each` 문의 정식 명칭은 `향상된 for문(enhanced for statement)` 이다. 

전통적 `for` 문과 달리 반복자와 인덱스 변수를 사용하지 않고, 하나의 관용구로 **컬렉션**과 **배열** 그리고 **Iterable 인터페이스**를 구현한 객체까지 모두 처리할 수 있다는 장점이 존재한다.

> 🫧 **Iterable 인터페이스**<br>
`Iterable` 인터페이스를  구현한 모든 객체는 `for-each` 문으로 순회할 수 있다.
```java
public interface Iterable<E> {
    // 이 객체의 원소들을 순회하는 반복자를 반환한다.
	Iterator<E> iterator();
}
```

표기법은 다음과 같고, 내부에서 `Iterator` 를 사용해서 순회하는 형식으로 동작한다.
```java
for (Element a : elements) {
	..// a로 무언가를 한다.
}
```


### ☁️ For-each문의 장점

#### 컬렉션 중첩 순회의 경우

서로 크기가 다른 컬렉션 두개를 중첩으로 순회할 때, 아래와 같은 실수가 발생할 수 있다.

```java
    enum Suit { CLUB, DIAMOND, HEART, SPADE }
    enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
        NINE, TEN, JACK, QUEEN, KING }

    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());
    
    for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
          for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
              deck.add(new Card(i.next(), j.next()));  // 문제 발생
```
마지막 줄에서 `i.next()` 는 `Suit` 의 개수만큼 불려야 하는데, 안쪽 반복문에서 호출되는 바람에 `Rank` 개수만큼 순회되어 숫자가 바닥나면 `NoSuchElementException` 예외가 터지기 때문이다. 

#### for-each 문 사용

```java
for (Suit suit : suits)
	for (Rank rank : ranks)
    	deck.add(new Card(suit, rank));
        
```

### ☁️ For-each 문을 사용할 수 없는 경우

아래의 세 가지 경우에 대해서는 일반 `for` 문을 사용하는 것이 좋다.

#### 1. 파괴적인 필터링
컬렉션을 순회하면서 선택된 원소를 제거해야 하는 경우, `for-each` 대신 반복자의 `remove` 메서드를 호출해야 한다. 자바 8부터는 반복자를 사용하지 않고 컬렉션 단에서 `removeIf` 를 통해 제거할 수 있다.

#### 2. 변형 
순회하면서 특정 원소의 값 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다. 

#### 3. 병렬 반복
여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

> 📚 핵심 정리
전통적인 for문과 비교했을 때 for-each문은 명료하고, 유연하고, 버그를 예방해준다. 성능 저하도 없으므로 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자.