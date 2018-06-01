# 모나드 첫걸음

## 람다 함수

```javascript
(function (x, y) {
  return x + y;
})(1, 2);
```

- es6 arrow function

```javascript
((x, y) => x + y)(1, 2)
```

## 모나드

`컨텍스트`가 있는 `연속된 계산`을 `읽기 쉽게` 만들기 위한 `순수 함수형` 패턴

- 컨텍스트
- 연속된 계산
- 읽기 쉽게
- 순수 함수형

- null 체크라는 컨텍스트가 있는 연속된 계산의 단순한 예

```javascript
var value1 = getValue("key1");
if (value1 !== null) {
  var value2 = getValue("key2");
  if (value2 !== null) {
    ...
  }
}
else {
  return null;
}
```

- 객체 지향과 같은 언어에서도 다양한 방식으로 문제를 풀고 있다.
- 계산 속에 컨텍스트가 섞여 있다면 계산의 본질을 파악하기(읽기) 어렵다.
- 컨텍스트와 계산을 분리하면 컨텍스트를 재사용할 수 있다.
- 순수 함수형으로 절차적으로 보이는 연속된 계산을 쉽게 표현하기 어렵다.
- 쉬운 순서대로 알아보자
  - 읽기 쉽게
  - 순수 함수형
  - 연속된 계산
  - 컨텍스트

## 컨텍스트가 있는 연속된 계산을 `읽기 쉽게` 만들기 위한 순수한 함수형 패턴

- 모든 코드를 하나의 함수에 쓰지 않는 이유는 복잡하기 때문
- 추상화(abstraction)

```clojure
(str "Hello" "World" "!")
;; str????
```

- 기호와 집단의 언어 (https://ko.wikipedia.org/wiki/%EC%88%98%ED%95%99_%EA%B8%B0%ED%98%B8)
- 축약의 단순함과 풀어쓰는 것과 장황함
  - multiplication vs `*`
    - `1 multiplication 2` vs `1 * 2`
  - uniqueness vs `∃!`
    - `∃! n ∈ ℕ: n + 5 = 2n.` vs `uniqueness n exists ....`

```clojure
(defn str
  "With no args, returns the empty string. With one arg x, returns
  x.toString().  (str nil) returns the empty string. With more than
  one arg, returns the concatenation of the str values of the args."
  {:tag String
   :added "1.0"
   :static true}
  (^String [] "")
  (^String [^Object x]
   (if (nil? x) "" (. x (toString))))
  (^String [x & ys]
     ((fn [^StringBuilder sb more]
          (if more
            (recur (. sb  (append (str (first more)))) (next more))
            (str sb)))
      (new StringBuilder (str x)) ys)))
```

## 컨텍스트가 있는 연속된 계산을 읽기 쉽게 만들기 위한 `순수한 함수형` 패턴

- 순수
  - 부수 효과가 없다.
    ```javascript
    function inc(x) {
      console.log("x:", x); // 부수 효과!
      return x + 1;
    }
    ```

  - 상태가 없다.
    ```javascript
    var state = 1;
    state = 2; // 상태를 변경!
    ```

  - 글로벌 값도 없다.
    ```javascript
    var y = 1;

    function calc(x) {
      return x + y; // 글로벌 y를 참조!
    }
    ```

  - 절차도 없다.
    ```javascript
    // 아레 절차 대로 실행!
    var x = 1;
    var y = 2;
    x + y;
    ```

- 대부분의 언어
  - 순수하지 않은 기능을 제공한다.

- 순수한 프로그래밍의 장점은 여기서 이야기하지 않는다.
- * 모나드는 순수함을 전제로 한다. 따라서 순수한 것과 그렇지 않은 것을 적절하게 섞어 사용한다면 의미가 없다.

## 컨텍스트가 있는 `연속된 계산`을 읽기 쉽게 만들기 위한 순수한 함수형 패턴

### 절차적

```javascript
var x = 1;
var y = 2;
var result = x + y;
```

### 순수 함수형 절차적 표현

```javascript
((x =>
  (y =>
    x + y))(1))(2)
```

- 절차적인 스타일에 비해 읽기 어렵다.

## 들여쓰기를 이용해서 보기 좋게 만들기

```javascript
((x =>
    (y =>
           x + y))
 (1))
    (2)
```

### helper 함수를 이용해서 더 보기 좋게 만들기

- 인자 하나와 함수를 받아 함수에 인자를 적용하는 함수

```javascript
function bind (mv, f) {
  return f(mv);
};
```

- `bind` 함수를 사용해 보기

```javascript
bind(1, x =>
          bind(2, y =>
                    x + y));
```

- 들여쓰기를 보기 좋게 해보자

```javascript
bind(1, x =>
bind(2, y =>
     x + y));
```

- 절차적 스타일과 비교해보기

```javascript
var x = 1;
var y = 2;
var result = x + y;
```

- `bind` 함수도 있고 괄호도 있고 해서 복잡해 보인다?
- `var`도 있고 `=`도 있고 `;`은 왜 복잡해 보이지 않는가?
- `var`, `=`, `;`는 이미 마음 속에 추상화 하지만 `bind` 함수는 추상화 되어있지 않기 때문이다.
- 이제 `bind` 함수나 괄호 같은 것은 봐도 못본 척하자. 필요한 것은 x에 1을 y에 2를 바인딩하고 x와 y를 더하는 것

```
1 x
2 y
  x + y
```

- 억지스럽다고 생각하겠지만 순수 함수로 연속된 계산을 읽기 쉽게 표현했다.

### 모나드 패턴

- 연속된 계산을 표현하는 추상 패턴

```javascript
bind(계산, 계산 결과를 바인딩할 기호 => // 줄
bind(계산, 계산 결과를 바인딩할 기호 => // 줄
     결과));                      // 줄
```

- `bind` 함수를 다시 보자

```javascript
function bind (mv, f) {
  return f(mv);
};
```

- 모나드 패턴은 유지하고 `bind` 함수의 구현 방식에 따라 모나드 특성이 결정된다.
- 위 예제는 단순히 절차적 컨텍스트만 가지는 모나드이고 `identity` 모나드라고 부른다.

## `컨텍스트`가 있는 연속된 계산을 읽기 쉽게 만들기 위한 순수한 함수형 패턴

- 계산 과정을 감싸고 있는 공통된 이야기
- 컨텍스트의 예
  - 계산 과정에서 결과가 `nil`인 경우 (maybe 모나드)
  - 계산 과정에 로그를 기록해야하는 경우 (writer 모나드)
  - 계산 과정 중에 `config`가 필요한 경우 참조해야 하는 경우 (reader 모나드)
  - 계산 과정 중 상태가 필요한 경우 (state 모나드)

## 계산 과정에서 결과가 `nil`인 경우 다음 계산을 하지 않는 `컨텍스트` 예제

- 계산 과정 `null`이 나올 수 있는 예제

```javascript
function getName (id) {
  if (id == 1) {
    return "WORLD";
  }
  else {
    return null;
  }
}

bind( getName(2)      , x =>
bind( x.toLowerCase() , y =>
      "Hello, " + y + "!"
));
```

```
Uncaught TypeError: Cannot read property 'toLowerCase' of null
    at monad.html:77
    at bind (monad.html:64)
    at monad.html:76
    at monad.html:81
```

- `bind` 함수 수정

```javascript
function bind (mv, f) {
  if (mv != null) {
    return f(mv);
  }
  return null;
};
```

```javascript
bind( getName(2)      , x =>
bind( x.toLowerCase() , y =>
      "Hello, " + y + "!"
));
// => null
```

- 계산 중 `null`이 나오면 다음 계산을 진행하지 않고 최종 결과를 `null`로 처리하는 컨텍스트를 가지는 모나드를 `maybe` 모나드라고 한다.

- 하스켈에서 `maybe` 모나드

```haskell
Just 3 >>= (\x ->
Just 4 >>= (\y ->
Just (x + y)))  
```

## 계산 과정에 로그와 같이 뭔가 계속 쓸 수 있어야 하는 경우 (writer 모나드)

```javascript
var result =
bind( 1,                   x =>
bind( logging("x: " + x),  _ =>
bind( 2,                   y =>
      x + y
)));

// result는 로그와 결과가 함께 들어 있어야 한다.

getLog(result);
// => ["x: 1"]
// 로그는 계속 쌓여야 하니 배열에 넣자!

getValue(result);
// => 3
```

- 로그와 계산 결과를 담을 데이터 (타입?)

```javascript
// 객체에 담을 수도 있지만 예제를 간단하게 하기 위해서
var result = [3, ["x: 1"]];

// v는 [x, y] 형태로 여야한다는 암묵적 타입
function getLog(v) {
  return v[1];
}

function getValue(v) {
  return v[0];
}
```

- `bind` 함수는 어떻게 생겨야 하나?

```javascript
// oldResult는 [x, []] 여야 한다는 암묵적 타입
function bind(oldResult, f) {
  var oldValue = oldResult[0];
  var oldLogs = oldResult[1];
  var newResult = f(oldValue);
  var newValue = newResult[0];
  var newLogs = newResult[1];
  return [newValue, oldLogs.concat(newLogs)];
}
```

```javascript
var result =
bind( 1,                   x =>
bind( logging("x: " + x),  _ =>
bind( 2,                   y =>
      x + y
)));
```

- 체크는 안해 주지만 타입 에러가 발생

```javascript
function bind(oldResult, f) { ... }

bind(1, function(x) { ..

// 1 != oldResult 기대하는 타입이 아니다!
```

- 일반 값을 `bind` 함수가 받을 수 있는 타입으로 만들어 주자!

```javascript
function bind(oldResult, f) { ... }

bind([1, []], function(x) { ..
```

- 일반 값을 `bind` 함수가 받을 수 있는 타입으로 만들어주는 helper 함수를 만들자!

```javascript
function mreturn(v) {
  return [v, []];
}
```

- 일반적으로 사용하기 위해서 함수 이름을 `newLogType` 대신 `mreturn`을 사용했다.
  모나드에서 보통 `return`이라고 부르지만 자바스크립트에서 `return`이 예약어라서 `mreturn`으로 했다.

```
function bind(oldResult, f) { ... }

bind(mreturn(1), function(x) { ..
```

```javascript
var result =
bind( mreturn(1),          x =>
bind( logging("x: " + x),  _ =>
bind( mreturn(2),          y =>
      mreturn(x + y) // 최종 결과도 타입을 맞춰 준다.
)));
```

- `logging` 함수는 어떻게 만들어야 하나?

```javascript
function logging(log) {
  return [null, [log]];
}
```

- 값이 있어야하는 부분이 `null`인 이유는 다음 계산 과정에서 사용하지 않기(`_`) 때문이다.
- 완성 버전

```javascript
function bind(mv, f) {
  var vv = f(mv[0]);
  return [vv[0], mv[1].concat(vv[1])];
}

function mreturn(v) {
  return [v, []];
}

function logging(log) {
  return [null, [log]];
}

function getLog(v) {
  return v[1];
}

function getValue(v) {
  return v[0];
}

var result =
bind( mreturn(1),          x =>
bind( logging("x: " + x),  _ =>
bind( mreturn(2),          y =>
      mreturn(x + y)
)));

getLog(result);
// ["x: 1"]
getValue(result);
// 3
```

- 계산 과정중 계산 결과에 영향을 주지 않고 다른 곳에 값을 기록할 수 있는 컨텍스트를 가지는 모나드를 writer
  모나드라고 한다.
- `mreturn` helper 함수는 일반 값을 모나드 컨텍스트 안에서 사용할 수 있는 형태(타입)으로 바꿔주는 함수다.
- 모나드는 `bind` 함수와 `return` 함수에 따라 특성이 결정된다.
- writer 모나드에서 `logging` 함수는 `tell` 함수라고 부른다.

### 모노이드
- 합치는 것에 대한 추상화(인터페이스)
- `concat` 대신 추상적인 함치는 방법에 대한 `append` 함수 정의
  ```javascript
  function bind(mv, f) {
    var vv = f(mv[0]);
    return [vv[0], mv[1].concat(vv[1])]; // -> mv가 모노이드면 append를 사용
  }
  ```
- 빈 배열 대신 합쳐도 자신이 나오는 `empty` 타입 정의
  ```javascript
  function mreturn(v) {
    return [v, []]; // -> [] 대신 mv 타입에 대한 empty 함수를 사용, 타입 추론이 약한 언어에서는 재사용하려면 조금 문법이 복잡하다.
  }
  ```

## 계산 과정 중에 `config`와 같이 언제든 참조할 수 있는 환경 값이 필요한 경우 (reader 모나드)

```javascript
var config = { debug: true };

function calc1 (x) {
  return 1 + x;
}

function calc2 (y) {
  if (config.debug == true) { // 컨텍스트 외부 값을 값을 참조
    return 2 + y;
  }
  else {
    return 0;
  }
}

var x = calc1(1);
var y = calc2(x);
var result = x + y;
```

- `calc2`는 전역 변수인 `config`를 참조한다.

- 전역 변수를 사용하지 않는 `calc2`

```javascript
function calc1 (x) {
  return 1 + x;
}

function calc2 (y, config) {
  if (config.debug == true) {
    return 2 + y;
  }
  else {
    return 0;
  }
}

var config = { debug: true };
var x = calc1(1);
var y = calc2(x, config);
var result = x + y;
```

- 모나드로 풀어보자.

```javascript
var result =
bind( mreturn(calc1(1))         , x =>
bind( ask()                     , config =>
bind( mreturn(calc2(2, config)) , y =>
      mreturn(x + y)
)));

result({debug: true});
```

- `bind` 함수와 `mresult` 함수

```javascript
function bind(mv, f) {
  return r => f(mv(r))(r);
}

function mreturn(v) {
  return r => v;
}
```

- 구조가 어렵지만 writer 타입과 다르게 환경 값을 저장하기 위해 함수 값을 사용한다는 것을 알 수 있다.

- 환경 값을 읽어오기 위한 도우미 함수

```javascript
function ask() {
  return r => r;
}
```

```javascript
function calc1 (x, config) {
  return 1 + x;
}

function calc2 (y, config) {
  if (config.debug == true) {
    return 2 + y;
  }
  else {
    return 0;
  }
}

var result =
bind( mreturn(calc1(1))         , x =>
bind( ask()                     , config =>
bind( mreturn(calc2(2, config)) , y =>
      mreturn(x + y)
)));

result({debug: true});
// 6
result({debug: false});
// 2
```

- 처음에 환경 값을 넘기고 계산 과정 중 언제든 참조할 수 있도록 하는 컨텍스트를 가지는 모나드를 `reader`
  모나드라고 한다.

## 계산 과정 중 읽고 쓸 수 있는 상태가 필요한 경우 (state 모나드)

- 순수 함수에는 상태가 없지만 모나드로 상태를 표현할 수 있다.

```javascript
var state = undefined;
var x = 1;
state = 1; // put state
y = state; // get state
var result = x + y;
```

- 모나드로 상태를 표현해 보기

```javascript
var result =
bind( mreturn(1)     , x =>
bind( put(x)         , _ =>
bind( get()          , y =>
      mreturn(x + y)
)));

getValue(result);
```

- `bind`와 `mreturn` 함수

```javascript
function bind(mv, f) {
  return s => {
      [v, ss] = mv(s);
      return (f(v))(ss);
  }
};

function mreturn(v) {
  return s => [v, s];
};
```

- `reader` 모나드처럼 모나드 값이 함수로 되어 있다.
- 상태 값을 쓰거나 읽을 수 있는 helper 함수

```javascript
function put(s) {
  return _ => [null, s];
};

function get() {
  return s => [s, s];
};

function getValue(s) {
  return s()[0];
};

function getState(s) {
  return s()[1];
};

var result =
bind( mreturn(1)     , x =>
bind( put(x)         , _ =>
bind( get()          , y =>
      mreturn(x + y)
)));

getValue(result);
// 2
```

- 계산 과정 중 언제든 하나의 상태를 쓰거나 읽을 수 있는 컨텍스트를 가지는 것을 `state` 모나드라고 부른다.

## 정리

`컨텍스트`가 있는 `연속된 계산`을 `읽기 쉽게` 만들기 위한 `순수한 함수형` 패턴

```javascript
bind(mreturn(1), x => bind( mreturn(2), y => mreturn(x + y)));
```

```
모나드 값 -> bind -> 값 -> f1 -> 값 -> return -> 모나드 값 -> bind -> 값 -> f2 -> 값 -> return -> 모나드 값 -> ....

값 -> f -> 값 ==변환==> 값 -> f2 -> 값
```

## 모나드와 타입과 다형성

### 데이터와 함수를 묶기

- writer 모나드에서 `bind` 함수와 `result` 함수

```javascript
function bind(v, f) {
  var vv = f(v[0]);
  return [vv[0], v[1].concat(vv[1])];
}

function mreturn(v) {
  return [v, []];
}
```

- `bind`에 전달되는 `v` 데이터는 `[x, []]` 타입이다.
- `mreturn`에서 나오는 데이터는 `[x, []]` 타입이다.

- `[x, []]` 타입 대신 Writer 타입을 만든다면 ...

```javascript
function Writer(value, extra) {
  this.value = value;
  this.extra = extra;
  this.bind = function(f) {
    var w = f(this.value);
    return new Writer(w.value, this.extra.concat(w.extra));
  };
}

Writer.return = function(v) {
  return new Writer(v, []);
};

Writer.logging = function(log) {
  return new Writer(null, [log]);
};

var result =
Writer.return(1)         .bind(x =>
Writer.logging("x: " + x).bind(_ =>
Writer.return(2)         .bind(y =>
Writer.return(x + y);
)));

// result.extra;
```

### 데이터와 함수를 분리

```javascript
var writer = {
  bind : (mv, f) => {
    var vv = f(mv[0]);
    return [vv[0], mv[1].concat(vv[1])];
  },
  return : v => [v, []];
};

var domonad = (monad, f) => f(monad.bind, monad.return);

domonad(writer, (bind, mreturn) =>
  bind( mreturn(1),         x =>
  bind( logging("x: " + x), _ =>
  bind( mreturn(2),         y =>
  mreturn(x + y)
))));
```

- `bind`, `mreturn` 인터페이스
- `do` 노테이션
- 데이터와 함수를 묶거나 데이터와 함수를 분리하거나 다형성은 가능하다. expression problem(https://en.wikipedia.org/wiki/Expression_problem)
- 하스켈 타입 추론에 의한 다형성
  ```haskell
  type GameValue = Int
  type GameState = (Bool, Int)

  playGame :: String -> State GameState GameValue
  playGame []     = do
      (_, score) <- get
      return score

  playGame (x:xs) = do
      (on, score) <- get
      case x of
           'a' | on -> put (on, score + 1)
           'b' | on -> put (on, score - 1)
           'c'      -> put (not on, score)
           _        -> put (on, score)
      playGame xs

  startState = (False, 0)

  main = print $ evalState (playGame "abcaaacbbcabbab") startState
  ```

## 다루지 않은 내용

- 펑터
- 어플리커티브 펑터
- 모나드 트랜스포머
- 모나드의 추가 인터페이스
- 모나드의 법칙

## 가치

- 순수 함수안에서 가치를 가진다
  - 언어에서 순수하지 않은 것을 허용하지 않는다면 코드의 표현력을 높혀준다.
  - 계산 과정에 중복 함수를 제거
  - 계산 과정에서 컨텍스트를 로직과 분리해서 생각
- 함수형 프로그래밍 센스
- 비 순수 함수형 언어라면 언어의 철학에 맞게 언어에서 제공하는 비순수 기능과 함수형 기능을 적절히 활용하는 것이 좋다.
