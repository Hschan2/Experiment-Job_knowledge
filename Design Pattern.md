# Design Pattern (디자인 패턴)

## ◆Singleton (싱글톤)
### ●필요성
```Singleton Pattern (싱클톤 패턴)```은 애플리케이션에서 인스턴스를 <b>하나</b>만 만들어서 사용하기 위한 패턴이다. Connection Pool (커넥션 풀), Thread Pool (스레드 풀), 디바이스 설정 객체 등의 경우에는 인스턴스를 여러 개 만들게 되면 자원을 낭비하거나 버그를 발생시킬 수 있기 때문에 오직 하나만 생성하고 사용하도록 하는 것이 이 패턴의 목적이다.   

### ●구현
하나의인스턴스만을 유지하기 위해 인스턴스 생성에 특별한 제약을 걸어둬야 한다. ```new```를 실행할 수 없도록 생성자에 ```private``` 접근 제어자를 지정하고, 유일한 단일 객체를 반환할 수 있도록 <b>정적 메소드</b>를 지원해야 한다. 또한 유일한 단일 객체를 참조할 정적 참조 변수가 필요하다.

```
public class Singleton {
    private static Singleton singletonObject;

    private Singleton() {}

    public static Singleton getInstance() {
        if(singletonObject == null) {
            singletonObject = new Singleton();
        }
        return singletonObject;
    }
}
```
위 코드는 멀티스레드 환경에서 싱글톤 패턴을 적용하면 문제가 발생할 수 있다. 동시에 접근을 하게 되면 하나만 생성되어야 하는 인스턴스가 두 개가 생성될 수 있다. 그렇기 때문에 ```getSingletonObject()``` 메소드를 동기화시켜야 멀티스레드 환경에서 문제가 해결된다.

```
public class Singleton {
    private static Singleton singletonObject;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if(singletonObject == null) {
            singletonObject = new Singleton();
        }
        return singletonObject;
    }
}
```

```synchronized``` 키워드를 사용하면 성능상 문제가 발생한다. 조금 더 효율적으로 제어하기 위해서

```
public class Singleton {
    private static Singleton singletonObject;

    private Singleton() {}

    public static Singleton getInstance() {
        if(singletonObject == null) {
            synchronized (Singleton.class) {
                if(singletonObject == null) {
                    singletonObject = new Singleton();
                }
            }
        }
        return singletonObject;
    }
}
```

```DCL (Double Checking Locking)```을 사용하여 ```getInstance()```에서 <b>동기화되는 영역을 줄일 수 있다</b>. 초기에 객체에 생성하지 않으면서도 동기화하는 부분을 작게 만들었다. 그러나 이 코드는 <b>멀티코어 환경에서 동작</b>할 때, 하나의 CPU를 제외하고 다른 CPU가 Lock이 걸리게 된다. 그렇기 때문에 다른 방법이 필요하게 된다.

```
public class Singleton {
    private static volatile Singleton singletonObject = new Singleton();

    private Singleton() {}

    public static Singleton getSingletonObject() {
        return singletonObject;
    }
}
```

클래스가 로딩되는 시점에 미리 객체를 생성해두고 그 객체를 반환한다.   

* volatile: 컴파일러가 특정 변수에 대해 옵티마이져가 캐싱을 적용하지 못하도록 하는 키워드   

[출처](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/DesignPattern#singleton)
