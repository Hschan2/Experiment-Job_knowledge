# 자바

## JVM 에 대해서, GC 의 원리
### JVM (Java Virtual Machine)
JVM은 자바 가상 머신를 부르는 용어이다. (가상머신: 프로그램을 실행하기 위해 물리적 머신과 유사한 머신을 소프트웨어로 구현한 것) JVM 역할은 자바 애플리케이션을 클래스 로더를 통해 읽어 들여 자바 API와 함께 <b>실행</b>하는 것이다. 그리고 자바와 OS 사이에서 <b>중개자 역할</b>을 수행하여 자바가 OS에 구애받지 않고 재사용을 가능하게 해준다.   

그리고 가장 중요한 <b>메모리 관리</b>, <b>Garbage Collection</b>을 수행한다. JVM은 <b>스택 기반의 가상머신</b>이다.   

자바 가상 머신을 알아야 하는 이유는 한정된 메모리를 효율적으로 사용하여 최고의 성능을 내기 위해서 알아야 한다. 메모리 관리는 프로그램 개발에서 가장 중요한 것 중 하나이기 때문이다.   

#### 자바 프로그램 실행 과정
1. 프로그램이 실행되면 JVM은 OS로부터 이 프로그램이 필요로 하는 메모리를 할당받는다. (JVM은 메모리를 용도에 따라 여러 영역으로 나누어 관리한다.)   
2. 자바 컴파일러(Javac)가 자바 소스코드를 읽어들여 자바 바이트 코드(.class)로 변환시킨다.   
3. Class Loader를 통해 class 파일들을 JVM으로 로딩한다.   
4. 로딩된 class 파일들은 Execution Engine을 통해서 해석된다.   
5. 해석된 바이트 코드는 Runtime Data Areas에 배치되어 실질적인 수행이 이루어지게 된다.   

이러한 실행 과정 속에서 JVM은 필요에 따라 Thread Synchronization과 GC(Garbage Collection)와 같은 관리 작업을 수행한다.   

#### JVM 구성
* Class Loader (클래스 로더)   
JVM 내로 클래스 파일을 로드하고, 링크를 통해 배치하는 작업을 수행하는 모듈이다. Runtime 시에 동적으로 클래스를 로드한다. jar 파일 내 저장된 클래스들을 JVM 위에 탑재하고 사용하지 않는 클래스들은 메모리에서 삭제한다. (이는 컴파일러 역할을 한다.) 자바는 동적 코드이며 컴파일 타임이 아니라 런타임에 참조한다. 즉, 클래스를 처음으로 참조할 때에 해당 클래스를 로드하고 링크하는 것이다. 그러한 역할을 클래스 로더가 수행한다.   

* Execution Engine (실행 엔진)   
클래스를 실행시키는 역할이다. 클래스 로더가 JVM 내의 런타임 데이터 영역에 바이트 코드를 배치시키고, 이것은 실행 엔진에 의해 실행된다. 자바 바이트 코드는 기계가 바로 수행할 수 있는 언어보다 비교적으로 인간이 보기 편한 형태로 기술된 것이다. 그래서 실행 엔진은 바이트 코드를 실제로 JVM 내부에서 기계가 실행할 수 있는 형태로 변경한다. 이 때, 두 가지 방식을 사용한다.   

* Interpreter (인터프리터)   
실행 엔진은 자바 바이트 코드를 명령어 단위로 읽어서 실행한다. 다만 이 방식은 인터프리터 언어의 단점을 그대로 갖고 있다. 한 줄씩 수행하기 때문에 속도가 느리다.   

* JIT (Just - In - Time)   
인터프리터 방식의 단점을 보완하기 위해 도입된 JIT 컴파일러이다. 인터프리터 방식으로 실행하다 적절한 시점에 바이트 코드 전체를 컴파일하여 <b>네이티브 코드</b>로 변경하고, 이후에는 해당 더 이상 인터프리팅 하지 않고 네이티브 코드로 직접 실행하는 방식이다. 네이티브 코드는 캐시에 보관하기 때문에 한 번 컴파일된 코드는 빠르게 수행하게 된다. JIT 컴파일러가 컴파일하는 과정은 바이트 코드를 인터프리팅하는 것보다 더 오래걸리기 때문에 한 번만 실행되는 코드라면 컴파일하지 않고 인터프리팅하는 것이 더 유리하다. 따라서 JIT 컴파일러를 사용하는 JVM들은 내부적으로 해당 메소드가 얼마나 자주 수행되는지 체크하고, 일정 정도를 넘을 때만 컴파일을 수행한다.   

* Garbage Collector   
Garbage Collection을 수행하는 모듈(스레드)가 있다.   

#### Runtime Data Area
프로그램을 수행하기 위해 OS에서 할당받은 메모리 공간   

* PC Register   
스레드가 시작될 때 생성되며 생성될 때마다 생성되는 공간으로 스레드마다 하나씩 존재한다. 스레드가 어떤 부분을 어떠한 명령으로 실행해야할 지에 대한 기록을 하는 부분으로 현재 수행 중인 JVM 명령의 주소를 갖는다.   

* JVM 스택 영역   
프로그램 실행 과정에서 임시로 할당되었다가 메소드를 빠져나가면 바로 소멸되는 특성의 데이터를 저장하기 위한 영역이다. 각 형태의 변수나 임시 데이터 그리고 스레드나 메소드의 정보를 저장한다. 메소드 호출 시, 각각 스택프레임(해당 메소드를 위한 공간)이 생성된다. 메소드 수행이 끝나면 프레임 별로 삭제한다. 메소드 안에 사용되는 값들(Local Variable)을 저장한다. 또한 호출된 메소드의 매개변수, 지역변수, Return 값과 연산 시 일어나는 값을 임시로 저장한다.   

* Native Method Stack   
자바 프로그램이 컴파일이 되어 생성되는 바이트 코드가 아닌 실제 실행할 수 있는 기계어로 작성된 프로그램을 실행시키는 영역이다. 자바가 아닌 다른 언어로 작성된 코드를 위한 공간이다. Java Native Interface를 통해 바이트 코드로 전환하여 저장하게 된다. 일반 프로그램처럼 커널이 스택을 잡아 독자적으로 프로그램을 실행시키는 영역이다. 이 부분으로 C Code를 실행시켜서 Kernel에 접근할 수 있다.   

* Method Area (= Class Area, Static Area)   
클래스 정보를 처음 메모리 공간에 올릴 때, 초기화되는 대상을 저장하기 위한 메모리 공간이다. 올라가게 되는 메소드의 바이트 코드는 프로그램의 흐름을 구성하는 바이트 코드이다. 자바 프로그램은 Main 메소드의 호출에서부터 계속된 메소드의 호출로 흐름을 이어간다. 대부분 인스턴스의 생성도 메소드 내에서 명령을 하고 호출을 한다. (컴파일된 바이트 코드의 대부분이 메소드 바이트 코드이기 때문에 거의 모든 바이트 코드가 올라간다고 볼 수 있다.) 이 공간에는 Runtime Constant Pool이라는 별도의 관리 영역도 함께 존재한다. 이는 상수 자료형을 저장하고 참조하여 중복을 막는 역할을 수행한다.   

##### 올라가는 정보의 종류
* Field Information   
멤버변수의 이름, 데이터 타입, 접근 제어자에 대한 정보   
* Method Information   
메소드의 이름, Return 타입, 매개변수, 접근제어자에 대한 정보   
* Type Information   
해당 정보가 Class인지 Interface인지의 여부를 저장, Type의 속성/전체 이름, Super Class의 전체 이름(Interface이거나 Object인 경우에는 제외)   

```
Method Area는 클래스 데이터를 위한 공간이라면 Heap 영역이 객체를 위한 공간이다. Heap과 마찬가지로 GC (Garbage Collection)의 관리 대상에 포함된다.
```

#### Heap 영역
객체를 저장하는 가상 메모리 공간이다. new 연산자로 생성된 객체와 배열을 저장한다. 이는 Class Area 영역에 올라온 클래스들만 객체로 생성할 수 있다.   

##### Heap의 세 부분
* Permanent Generation 영역   
생성된 객체들의 정보의 주소값이 저장되는 공간이다. Class Loader에 의해 로드되는 Class와 Method 등에 대한 Meta 정보가 저장되는 영역이며 JVM에 의해 사용된다. Reflection을 사용하여 동적으로 클래스가 로딩되는 경우에 사용된다. 내부적으로 Reflection 기능을 자주 사용하는 Spring Framework를 이용할 경우에 이 영역에 대한 고려가 필요하다.   

* New/Young 영역   
  * Eden: 객체들이 최초로 생성되는 공간
  * Survivor 0/1: Eden에서 참조되는 객체들이 저장되는 공간   

* Young 영역   
새롭게 생성한 객체의 대부분이 위치한다. 대부분 객체가 금방 접근 불가능 상태가 되기 때문에 매우 많은 객체가 Young 영역에 생성되었다가 사라진다. 이 영역에서 객체가 사라질 때, Minor GC가 발생한다고 말한다.   

* Old 영역   
New Area에서 일정 시간이 참조되고 있고 살아남은 객체들이 저장되는 공간 Eden 영역에 객체가 가득차게 되면 첫 번째 GC (Minor GC)가 발생한다. Eden 영역에 있는 값들을 Survivor 1 영역에 복사하고 이 영역을 제외한 나머지 영역의 객체를 삭제한다.   

인스턴스는 소멸 방법과 소멸 시점이 지역 변수와의 다르기 때문에 힙이라는 별도의 영역에 할당된다. 자바 가상 머신은 매우 합리적으로 인스턴스를 소멸시킨다. 더 이상 인스턴스의 존재 이유가 없을 때는 이를 소멸시킨다.   

### GC (Garbage Collection)
* Minor GC   
새로 생성된 대부분 객체(인스턴스)는 Eden 영역에 위치한다. 이 영역에서 GC가 한 번 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동된다. 이 과정을 반복하다가 계속해서 살아남아 있는 객체는 일정시간 참조되고 있다는 뜻으로 Old 영역으로 이동시킨다.   

* Major GC   
Old 영역에 있는 모든 객체들을 검사하여 참조되지 않은 객체들을 한번에 삭제한다. 시간이 오래 걸리고 실행 중 프로세스가 정지된다. 이것을 <b>Stop-the-World</b>라고 하며 Major GC가 발생하면 GC를 실행하는 스레드를 제외한 나머지 스레드는 모두 작업을 멈춘다. GC 작업을 완료한 후에 중단했던 작업을 다시 시작한다. (GC 튜닝이란 Stop-the-World 시간을 줄이는 것을 말한다.)   

#### GC가 선정하는 소멸 대상
Garbage Collector는 힙 내의 객체 중에서 Garbage를 찾아내고 이를 처리해서 힙의 메모리를 회수한다. 참조되지 않은 객체를 <b>Garbage</b>라고 하며 객체가 이를 판단하기 위해 <b>Reachability</b>라는 개념을 사용한다. 어떠한 힙 영역에 할당된 객체가 유효한 참조가 있으면 <b>Reachability</b>, 없다면 <b>Unreachability</b>로 판단한다. 하나의 객체는 다른 객체를 참조하고, 다른 객체는 또 다른 객체를 참조할 수 있기 때문에 참조 사슬이 형성이 되며, 이 참조 사슬 중 최초에 참조한 것을 <b>Root Set</b>이라고 한다. 힙 영역에 있는 객체들은 4가지 경우에 대한 참조를 한다.   

1. 힙 내 다른 객체에 의한 참조   
2. 자바 스택. 즉, 메소드 실행 시 사용하는 지역 변수아 파라미터들에 의한 참조   
3. 네이티브 스택(JNI, Java Native Interface)에 의해 생성된 객체에 대한 참조   
4. 메소드 영역의 정적 변수에 의한 참조   

2번, 3번, 4번은 Root Set으로 참조 사슬 중 최초에 참조한 것이다.   

객체가 GC의 대상이 되었다고 해서 바로 소멸되지 않는다. 잦은 GC의 실행은 시스템에 부담이 될 수 있기 때문에 성능에 영향을 미치지 않도록 GC 실행 타이밍은 ```별도의 알고리즘```으로 계산이 되며, 이를 기반으로 GC가 수행된다.   

* Serial GC   
적은 메모리와 CPU 코어 개수가 적을 때에 적합한 방식   

* Parallel GC   
기본적인 GC 알고리즘은 Serial GC와 동일하나 Parallel GC는 GC를 처리하는 스레드가 <b>여러 개</b>이기 때문에 빠른 GC를 수행할 수 있다. 이는 메모리가 충분하고 코어의 개수가 많을 때 유리하다.   

* Parallel Old GC (Parallel Compacting GC)   
JDK 5 Update 6부터 제공하는 GC 방식이다. 별도로 살아있는 객체를 식별한다는 부분에서 보다 복잡한 단계로 수행된다. 이 방식은 <b>Mark-Summary-Compaction</b> 단계를 거친다. Summary 단계는 앞서 GC를 수행한 영역에 대해 별도로 살아있는 객체를 식별한다는 점에서 Mark-Sweep-Compaction 알고리즘의 Sweep 단계와 다르다.   

* CMS GC (+ UseConcMarkSweepGC)   
이 방식은 GC 방식보다 더욱 복잡하다. 초기 <b>Initial Mark</b> 단계에서 클래스 로더에서 가장 가까운 객체 중 살아있는 객체만 찾는 것으로 끝낸다. 따라서, 멈추는 시간은 매우 짧다. 그리고 <b>Concurrent Mark</b> 단계에서 살아있다고 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인한다. 이 단계의 특징은 다른 스레드가 실행 중인 상태에서 동시에 진행된다는 것이다. 그 다음 <b>Remark</b> 단계에서 Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다. 마지막으로 <b>Concurrent Sweep</b> 단계에서 쓰레기를 정리하는 작업을 실행한다. 이 작업도 다른 스레드가 실행되고 있는 상황에서 진행한다. CMS GC 방식은 Stop-the-World 시간이 매우 짧으며 Low Latency GC라고도 부른다.   

#### CMS GC의 단점
1. 다른 GC 방식보다 메모리와 CPU를 더 많이 사용한다.   
2. Compaction 단계가 기본적으로 제공되지 않는다.   

그렇기 때문에, CMS GC 방식은 신중하게 검토 후 사용해야 한다. 그리고 조각난 메모리가 많아 Compaction 작업을 실행하면 다른 GC 방식의 Stop-the-World 시간보다 길기 때문에 Compaction 작업이 얼마나 자주 오랫동안 수행하는지 확인해야 한다.   

* G1 GC (Garbage First)   
이 영역을 이해하기 위해서 Young 영역과 Old 영역을 알고있어야 한다. 예를 들어 바둑판이 있다고 생각해보자. G1 GC는 바둑판의 각 영역에 객체를 할당하고 GC를 실행한다. 그러다가, 해당 영역이 꽉 차면 다른 영역에서 객체를 할당하고 GC를 실행한다. 즉, 지금까지 설명한 Young의 세 가지 영역에서 데이터가 Old 영역으로 이동하는 단계가 사라진 GC 방식이다. 이 방식은 CMS GC 방식을 대체하기 위해 만들어 졌다.   

#### G1 GC 장점
G1 GC 방식의 큰 장점은 성능이다. 다른 GC 방식보다 빠르다. 그러나 (JDK 6에서) G1 GC를 Early Access라고 부르며 시험삼아 사용할 수만 있도록 한다. 그리고 JDK 7에서 정식으로 제공한다.   

## Collection
Java Collection에는 ```List```, ```Map```, ```Set``` 인터페이스를 기준으로 여러 개의 구현체가 있다. 그리고 ```Stack```과 ```Queue``` 인터페이스가 존재한다. 이를 사용하는 이유는 다수의 Data를 다루는데 표준화된 클래스들을 제공해주기 때문에 DataStructure를 직접 구현하지 않고 편하게 사용할 수 있기 때문에 사용한다. 또한 배열과 다르게 객체를 보관하기 위한 공간을 미리 정하지 않아도 되기 때문에 상황에 따라 객체의 수를 동적으로 정할 수 있다. 이는 프로그램의 공간적인 효율성을 높여준다.   

* List
직접 ```@Override```를 통해 사용자가 정의하여 사용할 수 있으며 대표적 구현체로는 ```ArrayList```가 존재한다. ArrayList는 기존 ```Vector```를 개선한 것이다. 이외 ```LinkedList``` 등의 구현체가 있다.

* Map
대표적 구현체로 ```HashMap```이 있다. <b>Key-Value</b>의 구조로 이루어져 있으며 Map에 대한 구체적인 내용은 DataStructure 부분의 HashTable과 일치한다. Key를 기준으로 중복된 값을 저장하지 않으며 순서를 보장하지 않는다. Key에 대해 순서를 보장하기 위해서 ```LinkedHashMap```을 사용한다.

* Set
대표적 구현체로 ```HashSet```이 존재한다. ```Value```에 대해 중복된 값을 저장하지 않는다. Set 자료구조는 Map의 Key-Value 구조에서 Key 대신 Value가 들어가 Value를 Key로 하는 자료구조일 뿐이다. 마찬가지로 순서를 보장하지 않으며 순서를 보장해주기 위해 ```LinkedHashSet```을 사용한다.

* Stack, Queue
```Stack``` 객체는 직접 ```new``` 키워드로 사용할 수 있으며, ```Queue``` 인터페이스는 (JDK 1.5 부터) ```LinkedList```에 ```new``` 키워드를 적용하여 사용할 수 있다. <b>DataStructure</b> 설명에서 자세한 부분을 알 수 있다.

## Annotation
본래의 의미는 주석으로 인터페이스를 기반으로 한 문법이다. 주석과 역할이 다르지만 주석처럼 코드에 달아 클래스에 특별한 의미를 부여할 수 있으며 기능을 주입할 수도 있다. 또한 해석되는 시점을 정할 수 있다. (Retention Policy) 어노테이션은 세 가지 종류가 존재한다.   

* Built-in Annotation (JDK 내장)
* Meta Annotation (Annotation에 대한 정보를 나타내기 위함)
* Custom Annotation (개발자가 직접 만드는 것)   

Built-in Annotation은 상속을 받아서 메소드를 Override 할 때, 나타나는 @Override Annotation이 대표적인 예이다. Annotation의 동작 대상을 결정하는 Meta-Annotation에도 여러 가지가 존재한다.   

## Generic
자바에서 안정성을 맡고 있다. 다양한 타입의 객체들을 다루는 메소드나 컬렉션 클래스에서 사용하는 것으로, 컴파일 과정에서 타입 체크를 해주는 기능이다. 객체의 타입을 컴파일 시 체크하기 때문에 객체의 타입 안전성을 높이고 형변환의 번거로움을 줄여준다. 그렇기 때문에 코드도 더 간결해진다.   

예로 Collection에 특정 객체만 추가될 수 있거나 특정한 클래스의 특징을 갖고 있는 경우에만 추가될 수 있도록 하는 것이 Generic이다.   

Generic의 장점은 Collection 내부에서 들어온 값이 내가 원하는 값인지 별도의 로직처리를 구현할 필요가 없다. 그리고 API를 설계하는데 있어 보다 명확한 의사 전달이 가능해진다.

## Final Keyword
* Final Class   
다른 클래스에서 상속하지 못한다.

* Final Method   
다른 메소드에서 오버라이딩하지 못한다.

* Final Variable   
변하지 않는 상수값이 되어 새로 할당할 수 없는 변수가 된다.   

* Finally   
```Try-Catch```와 ```Try-Catch-Resource``` 구문을 사용할 때, 정상적으로 작업을 한 경우와 에러가 발생했을 경우를 포함하여 마무리를 해줘야 하는 작업이 존재하는 경우에 해당하는 코드를 작성해주는 코드 블록이다.

* Finalize()   
Keyword, CodeBlock도 아닌 메소드다. ```GC```에 의해 호출되는 함수로 절대 호출해서는 안되는 함수이다. ```Object``` 클래스에 정의되어 있으며 GC가 발생하는 시점이 불분명하기 때문에 해당 메소드가 실행된다는 보장이 없다. 그리고 ```Finalize()``` 메소드가 오버라이딩되어 있으면 GC가 이루어질 때 바로 Garbage Collecting이 되지 않는다. GC가 지연되면서 OOME (Out of Memory Exception)이 발생할 수 있다.

## Overriding vs Overloading
둘 모두 다형성을 높여주는 개념이지만 전혀 다른 개념을 갖고 있을 만큼 차이가 있다. (Overloading은 다른 시그니쳐를 만든다는 관점에서 다형성으로 보지 않는 의견도 존재) 이 둘의 공통점은 같은 이름의 다른 함수를 호출하는 것이다.   

* Overriding (오버라이딩)   
상위 클래스 혹은 인터페이스에 존재하는 메소드를 하위 클래스에서 필요에 맞게 재정의하는 것이다. 자바의 경우 오버라이딩 할 때, 동적바인딩이 된다.   

오버라이딩 예시.   
```
SuperClass object = new SubClass();

object.fun();
```
위와 같은 경우에 SuperClass의 fun이라는 인터페이스를 통해 SubClass의 fun이 실행된다.   

* Overloading (오버로딩)   
이름처럼 Return 타입은 동일하지만, 매개변수만 다른 메소드를 만드는 것을 의미한다. 자바의 경우에 오버로딩은 다른 시그니쳐를 만드는 것으로 전혀 다른 함수를 만드는 것과 같다고 생각하면 이해하기 쉽다. 시그니쳐가 다르기 때문에 정적바인딩으로 처리가 가능하며, 자바의 경우에 정적으로 바인딩된다.   

오버로딩 예시.   
```
main(blabla) {
  SuperClass object = new SubClass();
  fun(object);
}

fun(SuperClass super) {
  ...
}

fun(SubClass sub) {
  ...
}
```
위와 같은 경우에 fun (SuperClass super)가 실행된다.

## Access Modifier
변수 또는 메소드의 접근 범위를 설정해주기 위해 사용하는 자바의 예약어를 의미하며 총 네가지 종류가 있다.   

* Public   
어떤 클래스에서라도 접근이 가능하다.   

* Protected   
클래스가 정의되어 있는 해당 패키지 내, 해당 클래스를 상속받은 외부 패키지의 클래스에서 접근이 가능하다.   

* Default   
클래스가 정의되어 있는 해당 패키지 내에서만 접근이 가능하도록 접근 범위를 제한한다.   

* Private   
정의된 해당 클래스에서만 접근이 가능하도록 접근 범위를 제한한다.   

## Wrapper Class
기본 자료형 (Primitive Data Type)에 대한 클래스 표현이다. ```Integer```, ```Float```, ```Boolean``` 등이 해당한다.   

<b>int를 Integer라는 객체로 감싸서 저장해야 하는 이유가 무엇일까?</b>   

우선 컬렉션에서 Generic을 사용하기 위해서 Wrapper Class를 사용해야 한다. 그리고 ```Null```값을 반환해야만 하는 경우에는 Return Type을 Wrapper Class로 지정하여 ```Null```을 반환하도록 할 수 있다. 그러나 이런 상황을 제외하고 일반적인 상황에서 Wrapper Class를 사용해야 하는 이유는 객체지향적인 프로그래밍을 위한 프로그래밍이 아니고서야 없다. 해당 값을 비교할 때, 기본 자료형인 경우에는 <b>==</b>로 바로 비교해줄 수 있다. 하지만 Wrapper Class인 경우에는 ```.intValue()``` 메소드를 통해 해당 Wrapper Class의 값을 가져와서 비교해야 한다.   

### AutoBoxing
(JDK 1.5 부터) ```AutoBoxing```과 ```AutoUnBoxing```을 제공한다. 이는 각 Wrapper Class에 상응하는 기본 자료형일 경우에만 가능하다.   

사용 예시.
```
List<Integer> lists = new ArrayList<>();

lists.add(1);
```
```Integer```라는 Wrapper Class로 설정한 Collection에 데이터를 추가할 때, Integer 객체로 감싸서 넣지 않는다. 이유는 자바 내부에서 ```AutoBoxing```해주기 때문이다.   

## Multi-Thread 환경에서의 개발
멀티 스레드를 고려한 프로그램을 작성할 일이 많지 않으며 실제로 부딪히기 힘든 문제이기 때문에 많은 개발 입문자가 잘 모르고 문제 중 하나이다. 그러나 가장 중요하며 고려하지 않을 경우, 많은 버그를 만들어 낼 수 있기 때문에 중요하다.   

### Field Member
```필드(Field)```는 클래스에 변수를 정의하는 공간이다. 필드에 변수를 만들면 <b>메소드</b>끼리 변수를 주고 받는 데 있어서 참조하기 쉽고 편리한 공간 중 하나이다. 다만 객체가 여러 스레드가 접근하는 싱글톤 객체라면 필드에서 상태값을 갖고 있으면 안된다. 모든 변수를 파라미터로 넘겨 받고 Return하는 방식으로 코드를 구성해야 한다.   

### 동기화 (Synchronized)
필드에 Collection이 불가피하게 필요할 때, 자바에서는 ```Synchronized``` 키워드를 사용하여 스레드 간 <b>Race Condition</b>을 통제한다. 이 키워드를 기반으로 구현된 Collection들도 많이 존재한다. ```List```를 대신하여 ```Vector```를 사용할 수 있고, ```Map```을 대신하여 ```HashTable```을 사용할 수 있다. 하지만 이런 Collection들을 제공하는 <b>API</b>가 적고 성능도 좋지 않다.   

기본적으로 ```Collections```이라는 Util 클래스에서 제공되는 Static 메소드를 통해 이를 해결할 수 있다. ```Collections.synchronizedList()```, ```Collections.synchronizedSet()```, ```Collections.synchronizedMap()``` 등이 존재한다. (JDK 1.7 부터) ```Concurrent Package```를 통해 ```ConcurrentHashMap```이라는 구현체를 제공한다. Collections Util을 사용하는 것보다 ```Synchronized``` 키워드가 적용된 범위가 좁아서 보다 좋은 성능을 낼 수 있는 자료구조이다.   

### ThreadLocal
스레드 사이에 간섭이 없어야하는 데이터에 사용한다. 멀티 스레드 환경에서는 클래스의 필드에 멤버를 추가할 수 없고 매개변수로 넘겨받아야 하기 때문이다. 즉, 스레드 내부의 ```싱글톤을 사용하기 위해``` 사용한다. 주로 <b>사용자 인증</b>과 <b>세션 정보</b>, <b>트랜잭션 컨텍스트</b>에 사용한다.   

스레드 풀 환경에서 ThreadLocal을 사용하는 경우 ThreadLocal 변수에 보관된 데이터의 사용이 끝나면 반드시 해당 데이터를 삭제해야 한다. 만약에 그렇지 않을 경우에는 재사용되는 스레드가 올바르지 않은 데이터를 참조하는 문제가 발생한다.   

ThreadLocal을 사용하는 방법   
* ThreadLocal 객체 생성
* ThreadLocal.set() 메소드를 이용해서 현재 스레드의 로컬 변수에 값을 저장
* ThreadLocal.get() 메소드를 이용해서 현재 스레드의 로컬 변수 값을 읽기
* ThreadLocal.remove() 메소드를 이용해서 현재 스레드의 로컬 변수 값을 삭제   

### 자료 출처 [Link](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/Java)
