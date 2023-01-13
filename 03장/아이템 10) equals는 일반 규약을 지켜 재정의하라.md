# 아이템 10) equals는 일반 규약을 지켜 재정의하라

- equals 메서드는 재정의하기 쉬워보이지만 신중하게 해야한다.
    - **각 인스턴스가 본질적으로 고유하다**
    - **인스턴스의 ‘논리적 동치성’을 검사할 일이 없다.**
    - **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**
    - **클래스가 private이거나 package-private이면 equals 메서드를 호출할 일이 없다.**
    

---

## 언제 equals를 재정의 해야할까?

- 객체의 논리적 동치성을 확인해야 하는데, **상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때.**
- 주로 값 클래스들이 여기에 해당한다.

```java
public class Point {
	private final int x;
	private final int y;
	
	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override
	public boolean equals(Object o) {
		if (!(o instanceof Point)) return false;
		
		Point p = (Point)o;
		return p.x == x && p.y == y;
	}
}
```

- equals를 잘 정의해두면 Map, Set 자료구조에서 key로 사용할 수 있다.
- equals 메서드를 재정의 할때는 반드시 일반 규약을 따라야 한다.

```java
* 반사성: null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.
* 대칭성: null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true이면 y.equals(x)도 true다.
* 추이성: null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true, y.equals(z)가 true이면, x.equals(z)도 true이다.
* 일관성: null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
* Not-Null: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.
```

---

## 1. 반사성

- 단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻이다.

## 2. 대칭성

- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.
- 대소문자를 구분하지 않는 문자열을 구현한 다음 클래스를 예로 살펴보자.

```java
public final class CaseInsensitiveString {

    private final String str;
    
    public CaseInsensitiveString(String str) {
        this.str = Objects.requireNonNull(str);
    }
    
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return str.equalsIgnoreCase(((CaseInsensitiveString) o).str);
        }
    
        if (o instanceof String) { // 한 방향으로만 작동한다.
            return str.equalsIgnoreCase((String) o);
        }
        return false;
    }
}
```

- 이 클래스에서 toString 메서드는 원본 문자열의 대소문자를 그대로 돌려주지만, equals에서는 대소문자를 무시한다.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

1. cis.equals(s); // true
2. s.equals(cis); // false
```

- 1번의 경우에는 true를 반환하지만, 2번의 경우에는 false를 반환한다. String 클래스는 cis를 모르고 잇기 때문이다.

## 3. 추이성

- a → b, b → c 이면 a → c 이어야 한다는 조건이다.

```java
public class Point {
	private final int x;
	private final int y;
	
	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override
	public boolean equals(Object o) {
		if (!(o instanceof Point)) return false;
		
		Point p = (Point)o;
		return p.x == x && p.y == y;
	}
}
```

- 2차원 좌표를 가진 Point 클래스를 확장해서 점에 색상을 더해보자.

```java
public class ColorPoint extends Point {
	private final Color color;
	
	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}

	...
}
```

```java
@Override
public boolean equals(Object o) {
	if (!(o instanceof ColorPoint)) return false;
	
	return super.equals(o) && ((ColorPoint) o).color == color;
}
```

- 여기서 정의한 equals는 대칭성을 위반한다.

```java
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);

p.equals(cp); // true
cp.equals(p); // false
```

```java
@Override
public boolean equals(Object o) {
	if (!(o instanceof Point)) return false;

	// o가 일반 Point면 색상을 무시하고 비교한다.
	if (!(o instanceof ColorPoint)) return o.equals(this);
	
	// o가 ColorPoint면 색상까지 비교한다.
	return super.equals(o) && ((ColorPoint) o).color == color;
}
```

- 이런식으로 equals를 정의하게 되면 대칭성은 지켜주지만, 추이성은 깨진다.

```java
void transitivityTest() {
    ColorPoint a = new ColorPoint(2, 3, Color.RED);
    Point b = new Point(2, 3);
    ColorPoint c = new ColorPoint(2, 3, Color.BLUE);

    System.out.println(a.equals(b)); // true
    System.out.println(b.equals(c)); // true
    System.out.println(a.equals(c)); // false
}
```

- 그렇다면 어떻게 해야할까?
