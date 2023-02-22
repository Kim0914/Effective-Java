# 아이템12) toString을 항상 재정의하라

## toString ?

한 번쯤은 `toString` 을 객체의 값을 출력하기 위해 사용한 적이 있을 것입니다.

![image](https://user-images.githubusercontent.com/68818952/220099309-549dc6c2-c83a-4701-a125-8c878189c38b.png)


`toString` 의 일반 규약은 **'간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'**를 반환하는 것이다.

---

## toString을 왜 재정의할까?

`@toString` 을 잘 구현한 클래스는 사용하기 편하고, 디버깅하기 쉽다.

`@toString` 메서드는 객체를 println, printf, 문자열 연결, assert 구문에 넘길 때, 디버거가 객체를 호출할 때 자동으로 호출된다.

### toString을 재정의하지 않은 경우

- System.out.println(car)를 할 때

![image](https://user-images.githubusercontent.com/68818952/220099402-b4f93df2-5a82-41c8-a62b-447aff592f0e.png)


- assert에서 디버거가 객체를 호출할 때

![image](https://user-images.githubusercontent.com/68818952/220099431-e5cba05f-fc44-4580-8959-8e59494df870.png)


---

### toString을 재정의한 경우

![image](https://user-images.githubusercontent.com/68818952/220099483-7536d634-516a-41d9-a91b-695d4e2d8fc0.png)


- System.out.println(car)를 할 때

![image](https://user-images.githubusercontent.com/68818952/220099561-116c1f13-8742-4806-b3ce-07cfc89c2477.png)


- assert에서 디버거가 객체를 호출할 때

![image](https://user-images.githubusercontent.com/68818952/220099615-c6a2d598-bd71-4e3a-80e8-30de36e9fee4.png)


---

즉, toString을 제대로 작성하지 않는다면 쓸모없는 메세지만 로그에 남는다.

## ***그렇다면 toString을 어떻게 사용하는 것이 좋을까?***

실제 toString은 해당 객체가 가진 주요 정보를 모두 반환하는 것이 좋다.
```java
public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;
  
  public String getName() {
    return this.name;
  }
  
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
    
    @Override public String toString() {
      return "Square(super=" + super.toString() + ", width=" + this.width + ", height=" + this.height + ")";
    }
  }
  
  @Override public String toString() {
    return "ToStringExample(" + this.getName() + ", " + this.shape + ", " + Arrays.deepToString(this.tags) + ")";
  }
}
```

단, 정적 유틸리티 클래스의 경우 toString 을 제공할 필요가 없다. 또한 대부분의 Enum 타입은 이미 자바가 완벽한 toString 을 제공하므로 재정의할 필요 없다.

# **toString 은 언제 사용될까?**

Stackoverflow 의 [Is it ok to add toString() to ease debugging?](https://stackoverflow.com/questions/44132918/is-it-ok-to-add-tostring-to-ease-debugging) 를 읽어보면, **`toString`** 은 디버깅을 위해 설계된 메소드라고 한다. 어떤 문제가 발생한 클래스가 **`toString`** 이 잘 구현된 클래스일 경우 스스로를 완벽히 설명하는 문자열이 로깅될 것이고 그렇지 않을 때 보다 훨씬 더 원인을 발견하기 쉽다.

그렇다면, 로깅과 디버깅의 용도 외로 프로덕션 코드에는 **`toString`** 을 사용하면 안되는 것 일까?Stackoverflow 의 [Java toString for debugging or actual logical use](https://stackoverflow.com/questions/19911290/java-tostring-for-debugging-or-actual-logical-use) 와 [Is toString() only useful for debugging?](https://stackoverflow.com/questions/563676/is-tostring-only-useful-for-debugging) 를 읽어보면, 의견이 조금 갈릴수도 있으나 특이한 케이스를 제외하면 **`toString`** 은 디버그만을 위해 사용되는 것이 올바르다고 생각된다.

또한, 캐시를 사용하는 경우 기본 값으로 클래스의 `toString`을 식별자로 사용한다. 즉, toString은 해당 객체를 식별할 수 있도록 객체가 가진 주요 정보를 모두 담고 있는 메서드라고 볼 수 있다.

## 정리
```java
모든 구체(구현) 클래스에서 Object의 toString을 재정의하자.
toString을 재정의한 클래스는 사용하기 편하고 디버깅하기도 쉽다.
```
