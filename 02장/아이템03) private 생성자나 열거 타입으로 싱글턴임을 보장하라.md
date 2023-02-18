## 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

싱글턴을 만드는 방식은 두 가지 방법이 있는데 두 방식 모두 생성자는 private로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련한다.

```java
// 코드 3-1 public static final 방식의 싱글턴
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {}
	
	public void leaveTheBuilding() {...}
}
```

- private 생성자는 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다.

```java
// 코드 3-2 정적 팩터리 방식의 싱글턴
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {...}

	public static Elvis getInstance() {
		return INSTANCE;
	}

	public void leaveTheBuilding() {...}

	// 싱글턴임을 보장해주는 readResolve 메서드
	private Object readResolve() {
		// 진짜 Elvis를 반환하고, 가짜 Elvis는 GC에 맡긴다.
		return INSTANCE;
	}
}
```

- Elvis.getInstance는 항상 같은 객체의 참조를 반환하므로 유일성이 보장된다.
- 코드 3-1의 public 필드 방식의 큰 장점은 해당 클래스가 싱글텀임이 API에 명백히 드러난다. 또한 간결하다
- 코드 3-2의 정적 팩터리 방식의 장점은 *API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점이다.* 또한 원한다면 정적 팩터리 메서드를 제네릭 싱글턴 팩터리 메서드로 만들 수 있다는 점이다.
- 둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것으로는 부족하다. 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야한다.
- 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때 마다 새로운 인스턴스가 만들어진다.

```java
// 코드 3-3 열거 타입 방식의 싱글턴
public enum Elvis {
	INSTANCE;
	
	public void leaveTheBuilding() {...}
}
```

- public 필드 방식과 비슷하지만, 더 간결하고 추가적인 노력 없이 직렬화 할 수 있고, 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴가 생기는 일을 막아준다.
- 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.
