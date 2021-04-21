## 자바스크립트
---

### ◆ Javascript Event Loop
#### 자바스크립트가 동작하는 환경과 해석하고 실행시키는 엔진
자바스크립트를 해석하는 ```Javascript Engine```과 웹 브라우저에 화면을 그리는 Rendering Engine은 다른 엔진이다. ```Rendering Engine```은 HTML과 CSS로 작성된 마크업 코드를 콘텐츠로서 웹 페이지에 <B>Rendering</B>한다. Javascript Engine은 자바스크립트로 작성한 코드를 해석하고 실행하는 <b>인터프리터</b>다.   

추가적으로 <b>Event Loop</b>가 존재하여 Task Queue에 들어가는 Task들을 관리한다.

#### Call Stack
자바스크립트는 <b>단 하나의 호출 스택</b>을 사용한다. 그래서 자바스크립트의 함수가 실행되는 방식을 Run to Completion이라고 한다. 이는 하나의 함수가 실행되면 이 함수의 실행이 끝날 때까지 다른 어떤 Task도 수행될 수 없다는 의미다. 요청이 들어올 때마다 해당 요청을 <b>순차적으로</b> 호출 스택에 담아 처리한다. 메소드가 실행될 때, Call Stack에 새로운 프레임이 생기고 Push가 되고 메소드의 실행이 끝나면 해당 프레임은 Pop이 되는 원리이다.

```
function foo(b) {
  var a = 10;
  return a + b;
}

function bar(x) {
  var y = 2;
  return foo(x + y);
}

console.log(bar(1));
```
bar 함수를 호출하여 이에 해당하는 스택 프레임이 만들어지고, 그 안에 y와 같은 local variable과 arguments가 함께 생성된다. 그리고 bar 함수는 foo 함수를 호출하였지만 bar 함수는 아직 종료되지 않아 pop이 되지 않고 호출된 foo 함수가 Call Stack에 push가 된다.   
foo 함수에서는 a + b라는 값을 return 하여 함수의 역할을 모두 마쳤으므로 Stack에서 pop이 된다. 다시 bar 함수로 돌아아ㅗ foo 함수로부터 받은 값을 return하여 bar 함수도 종료되고 Stack에서 pop이 된다.

#### Heap
동적으로 생성된 객체(인스턴스)는 Heap에 할당된다. 대부분 구조화되지 않은 더미같은 메모리 영역을 Heap이라고 한다.

#### Task Queue (Event Queue)
자바스크립트의 런타임 환경에서는 처리해야 할 Task들을 임시로 저장하는 대기 큐가 존재한다. 이를 Task Queue (Event Queue)라고 부른다. 그리고 Call Stack이 비어졌을 때 먼저 대기열에 들어온 순서대로 수행된다.
```
setTimeout(function() {
  console.log("1");
}, 0);

console.log("2");

console> 
2
1
```
위 코드를 볼 때, 딜레이없이 바로 실행될 것 같지만 그렇지 않다.   
자바스크립트에서 비동기로 호출되는 함수들은 Call Stack에 쌓이지 않고 Task Queue에 Enqueue된다. 그리고 이벤트에 의해 실행되는 함수(Handler)들이 비동기로 실행된다. 자바스크립트 엔진이 아닌 Web API 영역에 따로 정의된 함수들은 비동기로 실행된다.

```
function first() {
  console.log("first");
  second();
}

function second() {
  let timer = setTimeout(function() {
    console.log("second");
  }, 0);

  third();
}

function third() {
  console.log("third");
}

first();

console>
first
third
second
```
위 코드에서 first가 console에 찍힌다. 그리고 second가 호출되고 setTimeout 함수 실행 후에 Call Stack에 들어간 다음 바로 빠져나온다. 그리고 내부에 걸려 있는 <b>Handler(익명 함수)는 Call Stack에 들어가서 바로 실행되지 않는다.</b> 그리고 이 Handler는 Call Stack 영역이 아닌 <b>Event Queue 영역</b>으로 들어간다. 그리고 third 함수가 Call Stack으로 들어간다.   

함수가 실행되면서 third이 console에 찍히고, 작업을 모두 마친 third 함수가 Call Stack에서 pop이 된다. 그리고 second 함수와 first 함수까지 Call Stack에서 pop이 된다. 이 때, Event Loop의 Call Stack이 비어있게 된다. 바로 이 시점에 Queue의 Head에서 하나의 이벤트를 가져와서 Call Stack으로 넣는다. 여기서 이 이벤트는 setTimeout 함수 내부에 있던 익명 함수다.   

즉, third이 끝나고 Call Stack에서 pop이 되고 second가 끝나고 first가 마저 끝나고 나서 Event Loop에 의해 하나의 이벤트가 Dequeue가 된 다음에 Call Stack으로 들어가서 실행된다. 그러므로 <b>이벤트에 걸려 있는 핸들러는 절대로 먼저 실행될 수 없다.</b>   

Event Loop, Event Queue에서 확인과 검사를 해야할 필요가 있을 때,
```
while(queue.waitForMessage()) {
  queue.processNextMessage();
}
```
를 이용해서 Event Loop가 현재 실행 중인 Task가 있는지 없는지 Task Queue가 있는지 없는지 반복으로 확인할 수 있다.   

#### 자바스크립트 엔진
기본적으로 하나의 쓰레드에서 동작한다. 하나의 쓰레드를 가지고 있다는 것은 하나의 스택을 가지고 있다는 것이며, 하나의 스택이 있다는 의미는 <b>동시에 단 하나의 작업만 할 수 있다</b>는 의미다.   

자바스크립트 엔진은 하나의 코드 조각을 하나씩 실행하는 일을 하고, 비동기적으로 이벤트를 처리하거나 AJAX 통신을 하는 작업은 Web API에서 모두 처리된다.   

자바스크립트가 여러 작업을 비동기로 작업할 수 이유는 무엇일까? 이유는 <b>Event Loop</b>, <b>Queue</b>가 있기 때문이다.   

#### Event Loop, Queue
Loop는 반복, 순환의 의미이다. 의미처럼 반복적으로 Call Stack과 Queue 사이의 작업들을 확인하고, 비워져 있는 경우 Queue에서 작업을 꺼내어 Call Stack에 넣는다. 자바스크립트는 이를 이용하여 비동기적으로 작업을 수행한다.   

1. 직접적인 작업은 Web API에서 처리되고, 작업이 완료되면 요청 시 등록했던 Call Stack이 Queue에 등록된다.
2. Event Loop는 이 작업들을 Queue에서 꺼내어 처리한다.
3. Event Loop는 스택에 처리할 작업이 없을 경우, 우선적으로 Microtask Queue를 확인한다.
4. Microtask Queue에 작업이 있다면 Microtask 있는 작업을 꺼내서 Call Stack에 넣는다.
5. 만약 Microtask Queue가 비어서 더 이상 처리할 작업이 없을 때, Task Queue를 확인한다.
6. Task Queue의 작업도 꺼내서 Call Stack에 넣는다.
7. 이렇게 Event Loop와 Queue는 자바스크립트 엔진이 하나의 코드 조각을 하나씩 처리할 수 있도록 작업을 스케줄하는 동시에 이러한 이유로 우리는 자바스크립트에서 비동기 작업을 할수 있도록 해준다.

#### 자바스크립트 처리 과정
```
console.log("script start");

setTimeout(function() {
  console.log("setTimeout");
}, 0);

Promise.resolve().then(function() {
  console.log("promise1");
}).then(function() {
  console.log("promise2");
});

requestAnimationFrame(function {
    console.log("requestAnimationFrame");
})
console.log("script end");

console>
script start
script end
promise1
promise2
requestAnimationFrame
setTimeout
```
위 코드의 실행 결과는 다음과 같이 처리가 되기 때문에 출력된다.   

1. Script 실행 작업이 Stack에 등록된다.
2. console.log("script start")가 처리된다.
3. setTimeout 작업이 Stack에 등록되고, Web API에게 setTimeout을 요청한다. 이 때, setTimeout의 콜백 함수를 함께 전달한다. 요청 이후 Stack에 있는 setTimeout 작업은 제거된다.
4. Web API는 setTimeout 작업이 완료되면 setTimeout 콜백 함수를 Task Queue에 등록한다.
5. Promise 작업이 Stack에 등록되고, Web API에게 Promise 작업을 요청한다. 이때 Promise.then의 콜백 함수를 함께 전달한다. 요청 이후 Stack에 있는 Promise 작업은 제거된다.
6. Web API는 Promise 작업이 완료되면 Promise.then의 콜백 함수를 Microtask Queue에 등록한다.
7. requestAnimation 작업이 Stack에 등록되고, Web API에게 requestAnimation을 요청한다. 이때 requestAnimation의 콜백 함수를 함께 전달한다. 요청 이후 Stack에 있는 requestAnimation 작업은 제거된다.
8. Web API는 requestAnimation의 콜백 함수를 Animation Frame에 등록한다.
9. console.log(‘script end’)가 처리된다.
10. ‘script 실행 작업’이 완료되어 Stack에서 제거된다.
11. stack이 비워있어서 Microtask Queue에 등록된 Promise.then 의 콜백 함수를 Stack에 등록한다.
12. 첫 번째 Promise.then의 콜백 함수가 실행되어 내부의 console.log(‘promise1’)가 처리된다.
13. 첫 번째 Promise.then 다음에 Promise.then이 있다면 다음 Promise.then의 콜백 함수를 Microtask Queue에 등록한다.
14. Stack 에서 첫 번째 Promise.then의 콜백 함수를 제거하고 Microtask Queue에서 첫번째 Promise.then의 콜백 함수를 제거한다.
15. 두 번째 Promise.then의 콜백 함수를 Stack에 등록한다.
16. 두 번째 Promise.then의 콜백 함수가 실행되어 내부의 console.log(‘promise2’)가 처리된다.
17. Stack 에서 두 번째 Promise.then의 콜백 함수를 제거한다.
18. Microtask 작업이 완료되면 Animation Frame에 등록된 콜백 함수를 꺼내 실행한다
19. 이후 브라우저는 랜더링 작업을 하여 UI를 업데이트한다.
20. Stack과 Microtask Queue가 비워있어서 Task Queue에 등록된 콜백 함수를 꺼내서 Stack에 등록한다.
21. setTimeout의 콜백이 실행되어 내부의 console.log(‘setTimeout’)이 처리된다.
22. setTimeout의 콜백 함수 실행이 완료되면 Stack에서 제거된다.   

위는 굉장히 복잡하고 어려울 수 있는 절차이다. 그러나 꼭 알아야 할 것은   
* 첫째. 비동기 작업으로 등록되는 작업은 Task와 Microtask. 그리고 animationFrame 작업으로 구분된다.   
* 둘째. Microtask는 Task보다 먼저 작업이 처리된다.   
* 셋째. Microtask가 처리된 이후 requestAnimationFrame이 호출되고 이후 브라우저 랜더링이 발생한다.   

#### RxJS Scheduler와 자바스크립트 비동기 작업의 종류
* Task   
비동기 작업이 순차적으로 수행될 수 있도록 보장하는 형태의 작업 유형이다. 여기서 순차적으로 보장한다는 의미는 작업이 예약되어있는 <b>순서</b>를 보장한다는 의미이다. Task 다음에 바로 다음 Task가 실행된다는 의미는 아니다. 위의 예처럼 Task 사이에는 브라우저 랜더링과 같은 작업이 일어 날 수 있기 때문이다.   

RxJS에서 task와 같은 형태의 작업을 하려면 Rx.Scheduler.async 스케줄러를 이용하여 구현할 수 있다.   

실제 Rx.Scheduler.async는 setInterval을 이용하여 구현되어 있다.
```
protected requestAsyncId(scheduler: AsyncScheduler, id?: any, delay: number = 0): any {
  return root.setInterval(scheduler.flush.bind(scheduler, this), delay);
}
```

* Microtask   
Microtask는 비동기 작업이 현재 실행되는 스크립트 바로 다음에 일어나는 작업이다. 따라서 Task보다 항상 먼저 실행된다.   

Microtask로는 MutationObserver와 Promise가 이에 해당된다.   

RxJS에서 Microtask와 같은 형태의 작업을 하려면 Rx.Scheduler.asap 스케줄러를 이용하여 구현할 수 있다.   

실제 Rx.Scheduler.asap은 Promise로 구현되어 있다.
```
protected requestAsyncId(scheduler: AsapScheduler, id?: any, delay: number = 0): any {
    //...
    return scheduler.scheduled || (scheduler.scheduled = Immediate.setImmediate(
      scheduler.flush.bind(scheduler, null)
    ));
  }

setImmediate(cb: () => void): number {
  // ...
  Promise.resolve().then(() => runIfPresent(handle));
}
```

### ◆ Hoisting
ES6 문법이 표준화가 되면서 크게 신경쓰지 않아도 되었지만, 자바스크립트는 언어 특성을 가장 잘 보여주는 특성 중 하나이다.   

#### 정의
<b>hoist</b>라는 단어의 사전적 정의는 끌어올리기다. 자바스크립트에서는 이는 변수이다. <b>var</b> Keyword로 선언된 모든 변수 선언은 ```호이스트```가 된다. 호이스트란 변수의 정의가 그 범 위에 따라 <b>선언</b>과 <b>할당</b>으로 분리되는 것을 의미한다. 즉, 변수가 함수 내 정의되었을 경우, 선언이 함수의 최상위로 함수 바깥에서 정의되었을 경우에 전역 컨텍스트의 최상위로 변경이 된다.   

선언과 할당을 이해하기 위해 끌어올려지는 것은 선언이다.
```
function getX() {
  console.log(x); // undefined

  var x = 100;

  console.log(x); // 100
}

getX();
```

다른 언어와 달리, 자바스크립트는 위 처럼 작성해도 오류가 발생하지 않고 undefined를 출력한다. ```var = 100;``` 이 구문이 ```var x;```를 호이스트하기 때문이다. 즉, 작동 순서에 맞게 코드를 재구성을 하게 된다면

```
function getX() {
  var x;

  console.log(x);

  var x = 100;

  console.log(x);
}

getX();
```
선언문은 자바스크립트 엔진 구동 시 가장 최우선으로 해석하기 때문에 호이스팅되고, <b>할당 구문은 런타임 과정에서 이루어지기 때문에</b> 호이스팅 되지 않는다.   

함수가 자신이 위치한 코드에 상관 없이 함수 선언문 형태로 정의한 함수의 유효 범위는 전체 코드의 <b>맨 처음</b>부터 시작한다. 함수 선언이 함수 실행보다 뒤에 있어도 자바스크립트 엔진이 함수 선언을 끌어올리는 것을 의미한다. 함수 호이스팅은 <b>함수</b>를 끌어올리지만 <b>변수 값</b>은 끌어올리지 않는다.

```
foo();

function foo() {
  console.log("Hello");
};

// console> Hello
```
위 는 foo()에 대한 선언을 호이스팅하여 ```global``` 객체에 등록시키기 때문에 <b>Hello</b>가 출력된다.

```
foo( );

var foo = function( ) {
  console.log(‘hello’);
};

// console> Uncaught TypeError: foo is not a function
```
위 는 함수 리터럴을 할당하는 구조이기 때문에 호이스팅되지 않으며 런타임 환경에서 <b>Type Error</b>를 발생시킨다.

### ◆ Closure
Closure은 <b>두 개의 함수로 만들어진 환경</b>으로 이루어진 특별한 객체의 한 종류이다. <b>환경</b>은 클로저가 생성될 때, 그 <b>범위</b>에 있던 여러 지역 변수들이 포함된 ```Context```이다. 이를 통해서 자바스크립트에는 없는 private 속성과 메소드, 공개 속성과 메소드를 구현할 수 있다.   

#### 클로저 생성
* 내부 함수가 익명 함수로 되어 외부 함수의 반환값으로 사용
* 내부 함수는 외부 함수의 실행 환경에서 실행
* 내부 함수에서 사용되는 변수 x는 외부 함수의 변수 스코프에 존재   

위는 클로저가 생성되는 조건이다. 클로저를 예제로 확인해보면

```
function outer() {
  var name = `closure`;

  function inner() {
    console.log(name);
  }

  inner();
}

outer();

// console> closure
```
위에서 <b>outer</b> 함수를 실행시키는 <b>context</b>에는 <b>name</b>이라는 변수가 존재하지 않는다.

```
var name = `Warning`;

function outer() {
  var name = `closure`;

  return function inner() {
    console.log(name);
  };

}

var callFunc = outer();

callFunc();

// console> closure
```
위에서 <b>callFunc</b>를 클로저라고 한다. callFunc 호출로 name 값이 console에 출력되지만 값은 <b>Warning</b>이 아닌 <b>closure</b>다. 즉, outer 함수의 context에 속해있는 변수를 참조한다. 여기서 outer 함수의 지역 변수로 존재하는 <b>name</b> 변수를 ```자유변수, free variable```이라고 한다.   

외부 함수 호출이 종료되더라도 외부 함수의 지역 변수 및 변수 스코프 객체의 체인 관계를 유지할 수 있는 구조가 클로저이며 외부 함수에 의해 반환되는 내부 함수를 가리킨다.   

### ◆ this
자바스크립트 내 모든 함수는 실행 때마다 함수 내부에 <b>this</b> 객체가 추가된다. <b>arguments</b>라는 유사 배열 객체와 함께 함수 내부로 암묵적으로 전달된다. 때문에 자바스크립트에서 this는 함수가 호출된 상황에 따라 그 모습을 다르게 한다.   

this를 사용하는 상황들을 볼 때,   

* 객체의 메서드를 호출   
객체의 프로퍼티가 함수일 때 이를 <b>메서드</b>라고 부른다. this는 함수를 실행할 때 함수를 소유하고 있는 객체(메서드를 포함하는 인스턴스)를 참조한다. 즉, 해당 메서드를 호출한 객체로 바인딩된다. ```A.B```일 때, <b>B</b> 함수 내의 this는 <b>A</b>를 가리킨다.   

```
var myObject = {
  name: "foo",

  sayName: function() {
    console.log(this);
  }

};

myObject.sayName(); // myObject의 object 객체를 호출

// console> Object {name: "foo", sayName: sayName()}
```

* 함수를 호출   
특정 객체의 메서드가 아닌 함수를 호출할 때, 해당 함수 내부 코드에서 사용된 this는 전역 객체에 바인딩된다. ```A.B```일 때, <b>A</b>가 전역 객체가 되어 <b>B</b> 함수 내부에서 this는 당연히 전역 객체에 바인딩된다.   

```
var value = 100;

var myObj = {
  value: 1,

  func1: function() {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function() {
      console.log(`func2's this.value ${this.value}`);
    };

    func2();
  }

};

myObj.func1();

// console> func1's this.value: 1
// console> func2's this.value: 100
```
<b>func1</b>에서 this는 객체의 메서드를 호출 때와 같다. 때문에 <b>myObj</b>가 this로 바인딩되고 myObj의 value인 1이 console에 출력된다. 그러나 <b>func2</b>는 함수를 호출로 해석해야 한다. ```A.B``` 구조에서 <b>A</b>가 없기 때문에 함수 내부에서 this가 전역 객체를 참조하게 되고 value는 100이 된다.   

* 생성자 함수를 통해 객체를 생성   
함수 호출이 아니라 <b>new</b> 키워드를 통해 생성자 함수를 호출할 때는 this가 다르게 바인딩된다. new 키워드를 통해 호출된 함수 내부에서 this는 객체 자신이다. 생성자 함수를 호출할 때 this 바인딩은 생성자 함수가 동작하는 방식을 통해 이해할 수 있다.   

new 연산자를 통해 함수를 생성자로 호출하면 우선 빈 객체가 생성이 되고 this가 바인딩된다. 이 객체는 함수를 통해 생성된 객체이며, 자신의 부모인 프로토타입 객체와 연결되어 있다. 그리고 Return 문이 명시되어 있지 않은 경우 this로 바인딩된 새로 생성한 객체가 Return된다.   

```
var Person = function(name) {
  console.log(this);
  this.name = name;
};

var foo = new Person("foo"); // Person 생성자로 생성

console.log(foo.name); // foo
```

* apply, call, bind를 통한 호출   
위의 상황들에 의존하지 않고 this를 자바스크립트 코드로 주입 또는 설정할 수 있다. 두 번째 상황에서 사용했던 예제 코드를 다시 한 번 봤을 때, <b>func2</b>를 호출할 때는 <b>func1</b>에서 this를 주입하기 위해 위 세 가지 메소드를 사용할 수 있다. 그리고 세 가지 메소드의 차이점을 파악하기 위해 func2에 파라미터를 받을 수 있도록 수정한다.   

1. ```bind``` 메소드 사용   

```
var value = 100;

var myObj = {
  value: 1,

  func1: function() {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function(val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    }.bind(this, `param1`, `param2`);

    func2();
  }

};

myObj.func1();

// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

2. ```call``` 메소드 사용   

```
var value = 100;

var myObj = {
  value: 1,

  func1: function() {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function(val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    };

    func2.call(this, `param1`, `param2`);
  }

};

myObj.func1();

// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

3. ```apply``` 메소드 사용   

```
var value = 100;

var myObj = {
  value: 1,

  func1: function() {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function(val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    };

    func2.apply(this, [`param1`, `param2`]);
  }

};

myObj.func1();

// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

* <b>bind</b> vs <b>apply</b>, <b>call</b>. 우선 bind는 함수를 선언할 때, this와 파라미터를 지정해줄 수 있으며, call과 apply는 함수를 호출할 때, this와 파라미터를 지정해준다.   
* <b>bind</b> vs <b>apply</b>, <b>call</b>. apply 메소드에는 첫 번째 인자로 this를 넘겨주고 두 번째 인자로 넘겨줘야 하는 파라미터를 배열의 형태로 전달한다. bind 메소드와 call 메소드는 각각 파라미터를 하나씩 넘겨주는 형태다.   

### ◆ Promise
자바스크립트에서 대부분 작업들이 <b>비동기</b>로 이루어진다. 콜백 함수로 처리하면 되지만 최근에는 프론트엔드 규모가 커지면서 코드의 복잡도가 높아지는 상황이 발생한다. 콜백이 중첩되는 경우가 발생하였고, 이를 해결할 수 있는 방안으로 <b>Promise 패턴</b>이 등장하였다.   

Promise 패턴을 사용하면 비동기 작업들을 순차적으로 진행하거나, 병렬로 진행하는 등의 제어보다 수월하다. 또한 예외 처리에 대한 구조가 존재하기 때문에 오류 처리 등에 대해보다 가시적으로 관리할 수 있다. Promise 패턴은 <b>ECMAScript6</b>에 정식으로 포함되어 있다.   

[ECMAScript6 학습](https://jaeyeophan.github.io/categories/ECMAScript6/)   

### ◆ Async / Await
비동기 코드를 작성하는 새로운 방법이다. Promise의 처리 방안에 만족하지 못하여 고안해낸 방법이다. 다만, Async / Await는 Promise 기반으로 만들어졌다.   

절차적 언어에서 작성하는 코드와 같이 사용법도 간단하며 이해하기도 쉽다.   

<b>function 키워드 앞에</b> async를 붙이고 <b>function 내부의 promise를 반환하는 비동기 처리 함수 앞에</b> await를 붙여 사용하면 된다.   

Async / Await의 큰 <b>장점</b>은 Promise보다 비동기 코드의 겉모습을 더 깔끔하게 작성할 수 있다. (ES8 공식 스펙이며 NODE8LTS에서 지원한다.)   

* BABEL에서 Async / Await를 지원하여 바로 사용할 수 있다.   

#### Promise & Async / Await로 구현하기

<b>promise로 구현하기</b>
```
function makeRequest() {
    return getData()
        .then(data => {
            if(data && data.needMoreRequest) {
                return makeMoreRequest(data)
                  .then(moreData => {
                      console.log(moreData);
                      return moreData;
                  }).catch((error) => {
                      console.log('Error while makeMoreRequest', error);
                  });
            } else {
                console.log(data);
                return data;
            }
        }).catch((error) => {
          console.log('Error while getData', error);
        });
}
```
   
<b>Async / Await로 구현하기</b>
```
async function makeRequest() { // 함수 앞에 async
    try {
      const data = await getData(); // promise를 반환하는 비동기 처리 앞에
      if(data && data.needMoreRequest) {
          const moreData = await makeMoreRequest(data);
          console.log(moreData);
          return moreData;
      } else {
          console.log(data);
          return data;
      }
    } catch (error) {
        console.log('Error while getData', error);
    }
}
```

### ◆ Arrow Function
화살표 함수 표현식은 기존 function 표현식보다 간결하게 함수를 표현할 수 있다. 이 함수는 항상 익명이며, 자신의 this, arguments, super와 new.target을 바인딩하지 않는다. 그래서 생성자로 사용할 수 없다.   

* 화살표 함수 도입의 영향: 짧은 함수, 상위 스코프 this   

1. 짧은 함수   
```
var materials = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

materials.map(function(material) { // 화살표 표현식 전
  return material.length;
});
// [8, 6, 7, 9]

materials.map((material) => { // 화살표 표현식 후
  return material.length;
});
// [8, 6, 7, 9]

materials.map(({length}) => length); // 화살표 표현식을 더 간결하게
// [8, 6, 7, 9]
```
기존에 사용하던 function을 생략하고 <b>=></b>로 대체하여 표현한다.   

2. 상위 스코프 this   
```
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // 여기서 this는 person 객체를 참조
  }, 1000);
}

var p = new Person();
```

일반 함수에서 this는 자기 자신의 this를 정의한다. 하지만 화살표 표현식 함수에서의 this는 Person의 this와 <b>동일한 값</b>을 갖는다. setInterval로 전달된 this는 Person의 this이며, Person 객체의 age에 접근한다.   

[정보 출처](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/JavaScript#javascript-event-loop)
