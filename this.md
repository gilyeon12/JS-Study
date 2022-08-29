# 22장 this

## 22.1 this 키워드

- 객체는 상태를 나타내는 프로퍼티와 동작을 나타내는 메서드로 구성된다. 메서드는 프로퍼티를 참조하고 변경할 수 있어야 하는데 이때, 자신이 속한 객체의 프로퍼티를 참조하려면 먼저 **자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 한다.**

- `this`는 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수다. `this`를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있다.

- `this`는 자바스크립트 엔진에 의해 암묵적으로 생성되며, 코드 어디서든 참조할 수 있다. 함수를 호출하면 `arguments` 객체와 `this`가 암묵적으로 함수 내부에 전달된다. 함수 내부에서 `arguments` 객체를 지역 변수처럼 사용할 수 있는 거처럼 `this`도 지역 변수처럼 사용할 수 있다. **단, `this`가 가리키는 값, `this` 바인딩은 함수 호출 방식에 의해 동적으로 결정된다.**

- 자바, C++ 같은 클래스 기반 언어에서 `this`는 언제나 클래스가 생성하는 인스턴스를 가리킨다. 하지만 **자바스크립트의 `this`는 함수가 호출하는 방식에 따라 `this`에 바인딩될 값, 즉 `this`이 동적으로 결정된다.** 또한 strict mode 역시 `this` 바인딩에 영향을 준다.

- `this`는 객체의 프로퍼티나 메서드를 참조하기 위한 자기 참조 변수이므로 일반적으로 객체의 메서드 내부 또는 생성자 함수 내부에서만 의미가 있다. 따라서 strict mode가 적용된 일반 함수 내부의 `this`에는 undefined가 바인딩 된다. 일반 함수 내부에서 `this`를 사용할 필요가 없기 때문이다.

<br>

## 22.2 함수 호출 방식과 this 바인딩

> 함수를 호출하는 방식

  1. 일반 함수 호출
  2. 메서드 호출
  3. 생성자 함수 호출
  4. Function.prototype.apply/call/bind 메서드에 의한 간접 호출

```
var foo = function () {
  console.dir(this);
};

// 1. 함수 호출
foo(); // window
// window.foo();

// 2. 메소드 호출
var obj = { foo: foo };
obj.foo(); // obj

// 3. 생성자 함수 호출
var instance = new foo(); // instance

// 4. apply/call/bind 호출
var bar = { name: 'bar' };
foo.call(bar);   // bar
foo.apply(bar);  // bar
foo.bind(bar)(); // bar
```

<br>

### 22.2.1 일반 함수 호출

- **기본적으로 `this`는 전역 객체가 바인딩 된다. 전역 함수는 물론 중첩 함수도 일반 함수로 호출하면 함수 내부의 `this`에는 전역 객체가 바인딩된다.**

- `this`는 객체의 프로퍼티나 메서드를 참조하기 위한 자기 참조 변수이므로 객체를 생성하지 않는 일반 함수에서는 `this`는 의미가 없다. 따라서 strict mode가 적용된 일반 함수 내부에서 `this`는 undefined가 바인딩된다.

- 전역 함수, 중첩 함수, 콜백 함수 등 어떠한 함수라도 일반 객체로 호출되면 `this`에 전역 객체가 바인딩된다. 하지만 중첩 함수 또는 콜백 함수는 외부 함수를 돕는 헬퍼 함수의 역할을 하는데 외부 함수인 메서드와 중첩 함수 또는 콜백 함수의 `this`가 일치하지 않는다는 것은 중첩 함수 또는 콜백 함수를 헬퍼 함수로 동작하기 어렵게 만든다.

- 메서드 내부의 중첩 함수나 콜백 함수의 `this` 바인딩을 메서드의 `this` 바인딩과 일치시키기 위한 방법은 다음과 같다.
```
var value = 1;

var obj = {
  value: 100,
  foo: function() {
    var that = this;  // Workaround : this === obj

    console.log("foo's this: ",  this);  // obj
    console.log("foo's this.value: ",  this.value); // 100
    function bar() {
      console.log("bar's this: ",  this); // window
      console.log("bar's this.value: ", this.value); // 1

      console.log("bar's that: ",  that); // obj
      console.log("bar's that.value: ", that.value); // 100
    }
    bar();
  }
};

obj.foo();
```

- 이 방법 외에도 자바스크립트는 `this`를 명시적으로 바인딩할 수 있는 `Function.prototype.apply`, `Function.prototype.call`, `Function.prototype.bind` 메서드를 제공한다. 

```
var value = 1;

var obj = {
  value: 100,
  foo: function() {
    console.log("foo's this: ",  this);  // obj
    console.log("foo's this.value: ",  this.value); // 100
    function bar(a, b) {
      console.log("bar's this: ",  this); // obj
      console.log("bar's this.value: ", this.value); // 100
      console.log("bar's arguments: ", arguments);
    }
    bar.apply(obj, [1, 2]);
    bar.call(obj, 1, 2);
    bar.bind(obj)(1, 2);
  }
};

obj.foo();
```

- 화살표 함수를 사용해서 `this` 바인딩을 일치시킬 수도 있다. (추후 학습)

<br>

### 22.2.2 메서드 호출

- 메서드 내부의 `this`에는 메서드를 호출한 객체, 즉 메서드를 호출할 때 메서드 이름 앞의 마침표(.) 연산자 앞에 기술한 객체가 바인딩 된다. 주의할 것은 **메서드 내부의 `this`는 메서드를 소유한 객체가 아닌 메서드를 호출한 객체에 바인딩된다는 것이다.**

![image](https://user-images.githubusercontent.com/109563072/187112675-61fc8f2f-c0f2-4f77-9d2f-67a9f66d7bac.png)

![image](https://user-images.githubusercontent.com/109563072/187112728-8618ca95-162c-480c-8eea-0a74f773a352.png)

- 위 코드에서 getName 메서드는 person 객체의 메서드로 정의되었다. 메서드는 프로퍼티에 바인딩된 함수다. 즉, person 객체의 getName 프로퍼티가 가리키는 함수 객체는 person 객체에 포함된 것이 아니라 독립적으로 존재하는 별도의 객체다. getName 프로퍼티가 함수 객체를 가리키고 있을 뿐이다. 따라서 getName 프로퍼티가 가리키는 함수 객체, 즉 getName 메서드는 다른 객체의 프로퍼티에 할당하는 것으로 다른 객체의 메서드가 될 수도 있고 일반 벼눗에 할당하여 일반 함수로 호출될 수도 있다. 따라서 **메서드 내부의 `this`는 프로퍼티로 메서드를 가리키고 있는 객체와는 관계가 없고 메서드를 호출한 객체에 바인딩된다.**

![image](https://user-images.githubusercontent.com/109563072/187113119-6456672b-4a9c-44e0-a975-cfe2097ea0f6.png)

- 프로토타입 메서드 내부에서 사용된 `this`도 일반 메서드와 마찬가지로 해당 메서드를 호출한 객체에 바인딩된다.

![image](https://user-images.githubusercontent.com/109563072/187113203-28f3719e-1dd1-4e89-be4c-6c7439d759e0.png)

![image](https://user-images.githubusercontent.com/109563072/187113297-4d11b4a6-da30-48a7-9c64-e160f97bff76.png)

<br>

### 22.2.3 생성자 함수 호출

- 생성자 함수 내부의 `this`에는 생성자 함수가 (미래에) 생성할 인스턴스가 바인딩된다.

- 생성자 함수는 이름 그대로 객체(인스턴스)를 생성하는 함수다. 일반 함수와 동일한 방법으로 생성자 함수를 정의하고 `new`연산자와 함꼐 호출하면 해당 함수는 생성자 함수로 동작한다. 만약 `new`연산자와 함께 생성자 함수를 호출하지 않으면 생성자 함수가 아니라 일반 함수로 동작한다.

```
// 생성자 함수
function Person(name) {
  this.name = name;
}

var me = new Person('Lee');
console.log(me); // {name: "Lee"}

// new 연산자와 함께 생성자 함수를 호출하지 않으면 생성자 함수로 동작하지 않는다.
var you = Person('Kim');
console.log(you); // undefined
```

<br>

### 22.2.4 Function.prototype.apply/call/bind 메서드에 의한 간접 호출

- `apply`, `call`, `bind` 메서드는 `Function.prototype`의 메서드다. 즉, 이들 메서드는 모든 함수가 상속받아 사용할 수 있다.

![image](https://user-images.githubusercontent.com/109563072/187113959-fde49ae1-512b-4d5c-b987-1b383129c870.png)

- `Function.prototype.apply`, `Function.prototype.call` 메서드는 `this`로 사용할 객체와 인수 리스트를 인수로 전달받아 함수를 호출한다. 

```
func.apply(thisArg, [argsArray])

// thisArg: 함수 내부의 this에 바인딩할 객체
// argsArray: 함수에 전달할 argument의 배열


func.call(thisArg[, arg1[, arg2[, ...]]])

// thisArg: 함수 내부의 this에 바인딩할 객체
// arg1, arg2, ... : 함수에 전달할 argument의 리스트
```

![image](https://user-images.githubusercontent.com/109563072/187114498-12e34bef-fae0-48cc-8487-cd61b706cdfc.png)

- **apply와 call 메서드의 본질적인 기능은 함수를 호출하는 것이다.** apply와 call 메서드는 함수를 호출하면서 첫 번째 인수로 전달한 특정 객체를 호출한 함수의 `this`에 바인딩한다. 

- apply와 call 메서드는 호출할 함수에 인수를 전달하는 방식만 다를 뿐 동일하게 동작한다. apply 메서드는 호출할 함수의 인수를 배열로 묶어 전달한다. call 메서드는 호출할 함수의 인수를 쉼표로 구분한 리스트 형식으로 전달한다. 위 코드는 호출할 함수, 즉 `getThisBinding`함수에 인수를 전달하지 않는다. 

![image](https://user-images.githubusercontent.com/109563072/187114842-f38aac1f-3b84-4282-aec7-783e6370dd76.png)

- apply와 call 메서드의 대표적인 용도는 `arguments` 객체와 같은 유사 배열 객체에 배열 메서드를 사용하는 경우다. `arguments` 객체는 배열이 아니기 때문에 `Array.prototype.slice` 같은배열의 메서드를 사용할 수 없으나 apply와 call 메서드를 이용하면 가능하다.

![image](https://user-images.githubusercontent.com/109563072/187115139-e604e623-e316-4f75-bf09-2f7a016b7da7.png)

- `Function.prototype.bind` 메서드는 apply와 call 메서드와 달리 함수를 호출하지 않는다. 다만 첫 번째 인수로 전달한 값으로 `this` 바인딩이 교체된 함수를 새롭게 생성해 반환한다.

![image](https://user-images.githubusercontent.com/109563072/187115346-44384920-cfbf-43eb-9753-c6ae65d167ee.png)

- bind 메서드는 메서드의 `this`와 메서드 내부의 중첩 함수 또는 콜백 함수의 `this`가 불일치하는 문제를 해결하기 위해 유용하게 사용된다.

![image](https://user-images.githubusercontent.com/109563072/187115464-0987a227-a231-4aec-9a6d-c7431df32bc5.png)

![image](https://user-images.githubusercontent.com/109563072/187115525-9df123f4-dfd3-40d0-ab45-321d6850f3be.png)

- 함수 호출 방식에 따라 `this` 바인딩이 동적으로 결정되는 것을 정리해놓은 표이다.

![image](https://user-images.githubusercontent.com/109563072/187115608-7651504b-cfe5-4285-9e60-84ca7bfa6c79.png)



