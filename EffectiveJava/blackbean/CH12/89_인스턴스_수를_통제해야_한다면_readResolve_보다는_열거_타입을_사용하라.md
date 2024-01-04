### 방어적 복사를 사용하는 불변클래스

``` java
public final class Period {
  private final Date start;
  private final Date end;
  
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start + "after" + end); 
  }
  
  public Date start() {
    return new Date(start.getTime()); 
  }

  public Date end() {
    return new Date(end.getTime());
  }
}
```

- 직렬화 할 경우, `implements Serializable` 을 추가하여 처리가 가능해 보임
- 하지만, `implements Serializable` 을 사용할 경우 불변식을 보장하지 못하게 됨
- readObject 메서드가 실질적으로 또 다른 public 생성자
- 생성자처럼 readObject 메서드에서도 인수가 유효한지 검사해야 하고 필요하다면 매개변수를 방어적으로 복사해야 함.
- **readObject는 매개변수로 바이트 스트림을 받는 생성자**
- 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 발생


### readObject 는 public API 이다.

- 접근 제어자가 private 이지만 역직렬화 과정에서 이것은 생성자와 동일하다.
- 따라서 생성자에서 수행하는 validation check 가 동일하게 수행되어야 한다.

public class BogusPeriod { // 진짜 Period 인스터느에서는 만들어질 수 없는 바이트 스트림 
```java
public class BogusPeriod {

	private static final byte[] serializedForm = new byte[] { (byte) 0xac,
			(byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06, 0x50, 0x65, 0x72,
			0x69, 0x6f, 0x64, 0x40, 0x7e, (byte) 0xf8, 0x2b, 0x4f, 0x46,
			(byte) 0xc0, (byte) 0xf4, 0x02, 0x00, 0x02, 0x4c, 0x00, 0x03, 0x65,
			0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f,
			0x75, 0x74, 0x69, 0x6c, 0x2f, 0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c,
			0x00, 0x05, 0x73, 0x74, 0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00,
			0x01, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61,
			0x2e, 0x75, 0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68,
			0x6a, (byte) 0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
			0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte) 0xdf, 0x6e,
			0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03, 0x77, 0x08,
			0x00, 0x00, 0x00, (byte) 0xd5, 0x17, 0x69, 0x22, 0x00, 0x78 };

	public static void main(String[] args) {
		Period p = (Period) deserialize(serializedForm);
		System.out.println(p);
	}

	private static Object deserialize(byte[] sf) {
		try {
			InputStream is = new ByteArrayInputStream(sf);
			ObjectInputStream ois = new ObjectInputStream(is);
			return ois.readObject();
		} catch (Exception e) {
			throw new IllegalArgumentException(e);
		}
	}
}
```

- 코드를 실행하면 **Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984** 를 출력
-  Period를 직렬화할 수 있도록 선언한 것만으 클래스의 불변식을 깨뜨리는 객체를 만들 수 있게 된 것
- Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사할 필요가 있음


```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  
  if (start.compareTo(end) > 0) 
    throw new InvalidObjectException(start +" after " + end);
    
}
```


### 악의적인 공격, 비잔틴 결함
- **정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어 낼 수 있다.**
- Period 인스턴스를 획득하면 공격적는 이 인스턴스가 불변이라고 가정하는 클래스에 값을 수정하여 엄청난 보안 문제 발생

**가변 공격 예시**
```java
public class MutablePeriod {
	public final Period period;

	public final Date start;

	public final Date end;

	public MutablePeriod() {
		try {
			ByteArrayOutputStream bos = new ByteArrayOutputStream();
			ObjectOutputStream out = new ObjectOutputStream(bos);


			out.writeObject(new Period(new Date(), new Date()));

			byte[] ref = { 0x71, 0, 0x7e, 0, 5 };
			bos.write(ref); 
			ref[4] = 4;
			bos.write(ref); 

			ObjectInputStream in = new ObjectInputStream(
					new ByteArrayInputStream(bos.toByteArray()));
			period = (Period) in.readObject();
			start = (Date) in.readObject();
			end = (Date) in.readObject();
		} catch (Exception e) {
			throw new AssertionError(e);
		}
	}
}
```

#### 공격

```
	public static void main(String[] args) {
		MutablePeriod mp = new MutablePeriod();
		Period p = mp.period;
		Date pEnd = mp.end;

		pEnd.setYear(78);
		System.out.println(p);

		pEnd.setYear(69);
		System.out.println(p);
	}
```

#### 결과

```
Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1969
```


### 역직렬화 할때도 방어적 복사를 해야한다

- **객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.**
- readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사
- final 필드는 방어적 복사 불가
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  
  start = new Date(start.getTime());
  end = new Date(end.getTime());
  
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start +" after " + end);
}
```


### 기본 readObject 메서드 사용 여부 판별

- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은지 확인-> 아닐경우 커스텀 readObject 메서드 또는 직렬화 프록시 패턴을 사용