## finalizer와 cleaner 사용을 피하라

### 왜?
* 자바는 두 가지 객체 소멸자를 제공한다. 
	* finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
	* cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고 불필요하다.


* finalizer와 cleaner는 즉시 수행된다는 보장이 없다. 즉 이 두가지로는 제때 실행되어야 하는 작업은 절대 할 수 없다는 것이다.
* finalizer와 cleaner를 얼마나 신속히 수행할지는 전적으로 GC 알고리즘에 달려있다.
* 그러므로 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존하면 안된다.

---
### 정상적으로 반납하는 방법
* 파일이나 스레드 등 종료해야 할 자원을 담고 있는 객체 클래스에서 finalizer나 cleaner를 대신해줄 묘안은 **AutoCloseable을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출**해주는 방식이 있다.
* 즉, close 메서드에서 이 **객체는 더 이상 유효하지 않다는 것을 기록**한다.
* 만약 다른 메서드가 이 필드를 검사했을 때 **해당 객체가 유효하지 않다면 IllegalStateException**을 던지면 된다.
---

### Cleaner, Finalizer가 적절한 경우
```java
public class Room implements AutoCloseable{
    private static final Cleaner cleaner = Cleaner.create();


    // Room을 참조하면 순환으로 참조하기에 가비지 컬렉터의 대상이 되지 않으므로 Room을 참조해서는 안된다.
    // 청소가 필요한 자원, cleaner가 청소할 때 수거할 자원을 가진다.
    private static class State implements Runnable{
        int numJunkPiles; // 수거대

        public State(final int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        //close나 cleaner메소드가 호출한다.
        @Override
        public void run() {
            System.out.println("방청소");
            numJunkPiles = 0;
        }
    }

    private final State state; // 방 상태

    private final Cleaner.Cleanable cleanable; // 수거 대상이 된다면 방을 청소한다.

    public Room(final int numJunkFiles) {
        this.state = new State(numJunkFiles);
        cleanable = cleaner.register(this, state); 
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```
* 자원의 소유자가 close 메서드를 호출하지 않는 것에 대한 대비망
  * 즉시 호출된다고 보장은 없지만 자원 회수를 늦게라도 해주므로 안전망 역할이다. 
  * 예를 들어, FileInputStream, FileOutputStream, ThreadPoolExecutor가 존재한다.

* 네이티브 피어와 연결된 객체에서의 사용  
  * native peer란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다. 
  * 네이티브 피어는 자바 객체가 아니여서 가비지 컬렉터는 그 존재를 알지 못한다. 따라서 자바 피어를 회수할 때 네이티브 피어도 회수하지 못해서 cleaner나 finalizer가 나서서 처리할 수 있다.

---
### 요약
* **cleaner는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자.**
* **물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.**
