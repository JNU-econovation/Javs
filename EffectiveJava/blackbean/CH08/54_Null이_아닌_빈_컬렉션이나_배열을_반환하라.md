# item54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

아래 코드는 흔히 볼 수 있는 코드이다. (따라하지 말 것!)
```java
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null
            : new ArrayList<>(cheesesInStock);
}
```
이 코드는 재고가 없을 때 null을 반환한다.</br>
이렇게 null을 반환하는 메서드를 사용하는 클라이언트는 항상 null 상황을 처리하는 코드를 추가로 작성해야 한다.</br>
이는 코드를 어지럽히고 오류를 유발할 가능성이 높다.

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON)) {
    System.out.println("Jolly good, just the thing.");
}
```

이러한 문제를 해결하기 위해 null 대신 빈 컬렉션을 반환하면 된다.
```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

가능성은 적지만, 사용패턴에 따라 빈 컬렉션을 할당하는 데서 성능이 저하될 수 있다.</br>
이런 경우 아래와 같이 매번 똑같은 빈 '불변' 컬렉션을 반환하면 된다.
```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
            : new ArrayList<>(cheesesInStock);
}
```

배열을 쓸대도 마찬가지이다.</br>
null을 반환하는 대신 길이가 0인 배열을 반환하면 된다.
```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

배열도 마찬가지로 성능이 떨어질것이 우려된다면 길이가 0인 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다.
```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

### 핵심 정리
null이 아닌 빈 배열이나 컬렉션을 반환하라.</br>
null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다.</br>
그렇다고 성능이 좋은 것도 아니다.
