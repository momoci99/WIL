## 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

e.g : 함수와 관련된 any 사용 예제

```tsx
function processBar(b: Bar) {
  /*...*/
}
function f() {
  const x = expressionReturningFoo();
  processBar(x); //Error - Foo 형식의 인수는 Bar 형식의 매개변수에 할당될 수 없음.
}
```

- 문맥상으로 x라는 변수가 동시에 Foo 타입과 Bar 타입에 할당가능하다면, 오류를 제거하는 방법은 두가지

e.g:하면 안되는 방법

```tsx
function f1() {
  const x: any = expressionReturningFoo(); //이렇게 하면 안됨
  processBar(x);
}
```

e.g: 그나마 나은 방법

```tsx
function f2() {
  const x = expressionReturningFoo();
  processBar(x as any); //이게 나음
}
```

- 두가지 해결책 중에서 f1에 사용된 x: any보다 f2에 사용된 x as any 형태가 권장됨.
- any 타입이 processBar 함수의 매개변수에서만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문
- f1에서는 함수의 마지막까지 x의 타입이 any인 반면, f2에서는 processBar 호출 이후에 x가 그대로 Foo 타입을 유지.

e.g : 만일 f1 함수가 x를 반환한다면 문제가 커짐

```tsx
function f1() {
	const x: any expressionReturningFoo();
	processBar(x);
	return x;
}

function g() {
	const foo = f1(); // 타입이 any
	foo.fooMethod();  // 이 함수 호출은 체크되지 않음.
}
```

- g 함수 내에서 f1이 사용되므로 f1의 반환 타입인 any타입이 foo의 타입에 영향을 미침
- 함수에서 any를 반환하면 그 영향력은 프로젝트 전반에 전염병처럼 퍼지게 됨.
- 반면 any의 사용 범위를 좁게 제한하는 f2함수를 사용한다면 any타입이 함수 바깥으로 영향을 미치지 않음.
- 비슷한 경우 맥락으로, 함수의 반환 타입을 명시하는것이 좋음. any 타입이 함수 바깥으로 영향을 미치는 것을 방지할 수 있음.

e.g : ts-ignore를 사용하여 any를 사용하지않고 오류를 제거하는 예

```tsx
function f1() {
  const x = expressionReturningFoo();
  //@ts - ignore
  processBar(x);
  return x;
}
```

- @ts-ignore를 사용한 다음 줄의 오류가 무시됨. 그러나 근본적인 원인을 해결한 것이 아니므로 다른곳에서 더 큰 문제가 발생할 수 있음.
- 타입 체커가 알려주는 오류는 문제가 될 가능성이 높은 부분이므로 근본적인 원인을 찾아 대처하는것이 바람직함.

e.g : 객체와 관련된 any의 사용법

```tsx
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value,
    // foo 속성이 Foo 타입에 필요하지만 Bar 타입에는 없습니다.
  },
};
```

- 단순히 생각한다면 config 객체 전체를 as any로 선언해서 오류를 제거할 수 있음.

e.g : config 객체 전체를 as any로 선언한 예

```tsx
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value,
  },
} as any; // 이렇게 하면 안됨.
```

- 객체 전체를 any로 단언하면 다른 속성들(a와 b) 역시 타입체크가 되지 않는 부작용이 생김

e.g : 최소한의 범위에만 any를 사용한 예제

```tsx
const config: Config = {
  a: 1,
  b: 2, //이 속성은 여전히 체크됨.
  c: {
    key: value as any,
  },
};
```

### 요약

- 의도치 않은 타입 안전성의 손실을 피하기 위해서 any의 사용 범위를 최소한으로 좁혀야함.
- 함수의 반환 타입이 any인 경우 타입 안정성이 나빠짐. 따라서 any 타입을 반환하면 절대 안됨.
- 강제로 타입 오류를 제거하려면 any 대신 @ts-ignore를 사용하는 것이 좋음.

## 39 any를 구체적으로 변형해서 사용하기

- any는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입
- 일반적인 상황에서는 any보다는 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높으므로 더 구체적인 타입을 찾아야함.

e.g : any타입의 값을 그대로 정규식이나 함수에 넣는 것은 권장되지 않음.

```tsx
function getLengthBad(array: any) {
  //이렇게 하면 안됨.
  return array.length;
}

function getLength(array: any[]) {
  return array.length;
}
```

- any를 사용하는 getLengthBad보다는 any[]를 사용하는 getLength가 더 좋은 함수.
  - 함수 내의 array.length 타입이 체크됨.
  - 함수의 반환 타입이 any 대신 number로 추론됨.
  - 함수 호출될 때 매개변수가 배열인지 체크됨.

e.g 배열이 아닌 값을 넣어서 실행했을때의 비교

```tsx
getLengthBad(/123/); //오류 없음. undefined를 반환
getLength(/123/); //Error
```

e.g : {[key: string]: any} 를 사용한 예제

```tsx
function hasTwelveLetterKey(o: { [key: string]: any }) {
  for (const key in o) {
    if (key.length === 12) {
      return true;
    }
  }
  return false;
}
```

- 함수의 매개변수를 구체화할 때, 요소의 타입에 관계없이 배열의 배열 형태라면 any[][]처럼 선언하면됨.
- 함수의 매개변수가 객체이긴 하지만 값을 할 수 없다면 {[key: string]: any} 처럼 선언.

e.g : object타입을 사용하는 예제

```tsx
function hasTwelveLetterKey(o: object) {
  for (const key in o) {
    if (key.length === 12) {
      console.log(key, o[key]); //Error
      //Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{}'.
      //  No index signature with a parameter of type 'string' was found on type '{}'.
      return true;
    }
  }
}
```

e.g : 최소한으로 구체화하는 방법 세가지

```tsx
type Fn0 = () => any; //매개 변수 없이 호출 가능한 모든 함수
type Fn1 = (arg: any) => any; //매개변수 1개
type FnN = (...args: any[]) => any; //모든 개수의 매개변수 //"Function" 타입과 동일함.
```

- any보다는 구체적인 방법.
- …args의 타입을 any[]로 선언. any로 선언해도 동작하지만 any[]로 선언하면 배열 형태라는 것을 알 수 있어 더 구체적.

e.g : …args의 타입을 any[]로 선언.

```tsx
const numArgsBad = (...args: any) => args.length; //any를 반환
const numArgsGood = (...args: any[]) => args.length; //number을 반환
```

### 요약

- any를 사용할 때는 정말로 모든 값이 허용되어야만 하는지 면밀히 검토해야함.
- any보다 더 정확하게 모델링할 수 있도록 any[] 또는 {[id:string]: any} 또는 () ⇒ any 처럼 구체적인 형태를 사용해야함.

## 40 함수 안으로 타입 단언문 감추기

- 함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 명확히 명시하는 정도로 끝내는 게 나음.

e.g : 함수 캐싱으로 알아보는 개선 기법

```tsx
declare function cacheLast<T extends Function>(fn: T): T;

//구현체
declare function shallowEqual(a: any, b: any): boolean;
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;
  return function (...args: any[]) {
    //Type '(...args: any[]) => any' is not assignable to type 'T'.
    //  '(...args: any[]) => any' is assignable to the constraint of type 'T', but 'T' could be instantiated with a different subtype of constraint 'Function'.
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  };
}
```

- 타입스크립트는 반환문에 있는 함수와 원본 함수 T타입이 어떤 관련이 있는지 알지 못하기 때문에 오류가 발생
- 그러나 결과적으로 원본 함수 T타입과 동일한 매개변수로 호출되고, 반환값 역시 예상한 결과가 되기 때문에, 타입 단언문을 추가해서 오류를 제거하는 것이 큰문제가 되지 않음.

e.g : 타입 단언문을 추가한 코드

```tsx
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;
  return function (...args: any[]) {
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  } as unknown as T;
}
```

- 함수 내부에는 any가 꽤 많이 보이지만, 타입 정의에는 any가 없기 때문에, cacheLast를 호출하는 쪽에서는 any가 사용됐는지 알지 못함.

e.g : 좀 더 구현이 복잡한 shallowObjectEqual 구현

```tsx
//타입 정의
declare function shallowObjectEqual<T extends object>(a: T, b: T): boolean;

declare function shallowEqual(a: any, b: any): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== b[k]) {
      //Error '{}' 형식에 인덱스 시그니처 형식가 없으므로 요소에
      //암시적으로 any 형식이 있음.
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

- if 구문의 k in b 체크로 b 객체에 k 속성이 있다는 것을 확인했지만 b[k] 부분에서 에러가 발생하는것이 이상함.
- 개발자가 실제 오류가 아니라는것을 알고 있기 때문에 any로 단언하는 수밖에 없음.

e.g : any로 단언한 코드

```tsx
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
	for (const [k, aVal] of Object.entries(a)){
		if(!(k in b) || aVal !== (b as any)[k] {
			return false;
		}
	}
	return Object.keys(a).length === Object.keys(b).length;
}
```

- b as any 타입 단언문은 안전하며(k in b)체크를 했으므로), 결국 정확한 타입으로 정의되고 제대로 구현된 함수가 됨.
- 객체가 같은지 체크하기 위해 객체 순회와 단언문이 코드에 직접 들어가는 것 보다, 앞의 코드처럼 별도의 함수로 분리해 내는 것이 훨씬 좋은 설계.

### 요약

- 타입 선언문은 일반적으로 타입을 위험하게 만들지만, 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도함.
- 불가피하게 사용해야 한다면, 정확한 정의를 가지는 함수 안으로 숨기도록 할것.

## 41 any의 진화를 이해하기

- 타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정됨.
- 이후에 정제될 수 있지만 (예를 들어 null인치 체크), 새로운 값이 추가되도록 확장할 수 없음.
- 그러나 any타입과 관련해서 예외인 경우가 존재.

e.g: 자바스크립트에서 일정 범위의 숫자들을 생성하는 함수를 예로 드는경우

```tsx
function range(start, limit) {
  const out = [];
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;
}
```

- 이 코드를 타입스크립트로 변환시 예상한대로 정확히 동작

e.g : 타입스크립트로 변환한 코드

```tsx
function range(start: number, limit: number) {
  const out = [];
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;
}
```

- 자세히 보면 이상한 점을 알 수 있음. 처음에는 any타입 배열인 []로 초기화 되었는데, 마지막에는 number[]로 추론되고 있음.

e.g : 코드에 out이 등장하는 세가지 위치

```tsx
function range(start: number, limit: number) {
  const out = []; //타입이 any
  for (let i = start; i < limit; i++) {
    out.push(i); //out의 타입이 any[]
  }
  return out; //타입이 number[]
}
```

- out의 타입은 any[]로 선언되었지만, number 타입의 값을 넣는 순간부터 타입은 number[]로 진화(evolve)
- 타입의 진화는 타입 좁히기와는 다름.

e.g : 배열에 다양한 타입의 요소를 넣어서 배열의 타입이 확장되며 진화하는 예

```tsx
const result = []; // 타입이 nay[]
result.push("a");
result.result // 타입이 string[]
  .push(1);
result; // 타입이 (string | number)[]
```

e.g : 분기에 따라 타입이 변하는 예

```tsx
let val;
if (Math.random() < 0.5) {
  val = /hello/;
  val; // 타입이 RegExp
} else {
  val = 12;
  val; // 타입이 number
}
val; //타입이 number | RegExp
```

- 변수의 초깃값이 null인 경우도 any의 진화가 일어남
- 보통 try, catch 블록 안에서 변수를 할당하는 경우에 나타남

e.g : 예외처리구문으로 인해 타입이 변경되는 코드

```tsx
let val = null; //타입이 any
try {
  somethingDangerous();
  val = 12; //타입이 number
  val;
} catch (e) {
  console.warn("alas!");
}
val; // 타입이 number | null
```

- any 타입의 진화는 noImplicityAny가 설정된 상태에서 변수의 타입이 암시적인 any인 경우에만 발생
- 그러나 명시적으로 any를 선언하면 타입이 그대로 유지

e.g : 명시적으로 any를 선언하면 타입이 그대로 유지되는 예

```tsx
let val: any; // 타입이 any
if (Math.random() < 0.5) {
  val = /hello/;
  val; // 타입이 any
} else {
  val = 12;
  val; // 타입이 any
}
val; //타입이 any
```

e.g: 암시적 any 상태인 변수에 어떠한 할당도 하지 않고 사용하려고 하면 암시적 any오류가 발생

```tsx
function range(strat: number, limit: number) {
  const out = [];

  if (start === limit) {
    return out;
    //error - Variable 'out' implicitly has type 'any[]'
    // in some locations where its type cannot be determined.
  }

  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;
}
```

- any 타입의 진화는 암시적 any 타입에 어떤 값을 할당할 때만 발생.
- 그리고 어떤 변수가 암시적 any상태일 때 값을 읽으려고 하면 오류가 발생.
- 암시적 any 타입은 함수 호출을 거쳐도 진화하지 않음.

e.g : forEach안의 화살표 함수는 추론에 영향을 미치지 않음.

```tsx
function makeSquares(start: number, limit: number) {
  const out = [];
  //Error -  out 변수는 일부 위치에서 암시적으로 any[] 형식

  range(start, limit).forEach((i) => {
    out.push(i * i);
  });
  return out;
  // Error - out 변수에는 암시적으로 any[] 형식이 포함됨.
}
```

- 루프로 순회하는 대신, 배열의 map과 filter 메서드를 통해 단일 구문으로 배열을 생성하여 any 전체를 진화시키는 방법을 생각해 볼 수 있음.
- any가 진화하는 방식은 일반적인 변수가 추론되는 원리와 동일
- 진화한 배열의 타입이 (string | number)[]라면, 원래 number[] 타입이어야하지만 실수로 string이 섞여서 잘못 진화한 것일 수도 있음.
- 타입을 안전하게 지키기 위해서는 암시적 any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 더 좋은 설계

## 요약

- 일반적인 타입들은 정제되기만 하는 반면, 암시적 any와 any[]타입은 진화할 수 있음. 이러한 동작이 발생하는 코드를 인지하고 이해할 수 있어야함.
- any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법

## 42 모르는 타입의 값에는 any 대신 unknown을 사용하기

- unknown에는 함수의 반환값과 관련된 형태, 변수 선언과 관련된 형태, 단언문과 관련된 형태가 존재.

**함수의 반환값과 관련된 unknown**

e.g : YAML 파서인 parseYAML 함수를 작성. JSON.parse의 반환 타입과 동일하게 parseYAML메서드의 반환타입을 any로 생성

```tsx
function parseYAML(yaml: string): any {
  //...
}
```

- 함수의 반환 타입으로 any를 사용하는 것은 좋지 않은 설계

e.g : parseYAML를 호출한 곳에서 반환값을 원하는 타입으로 할당하는 것이 이상적

```tsx
interface Book {
  name: string;
  author: string;
}
const book: Book = parseYAML(`
	name: Wuthering Heights
	author: Emily Bronte
`);
```

- 그러나 함수의 반환값에 타입 선언을 강제할 수 없으므로 호출한 곳에서 타입 선언을 생략하게 되면 book 변수는 암시적으로 any타입이 되고, 사용되는 곳 마다 타입 오류가 발생하게 됨.

e.g : 에러 발생예

```tsx
const book = parseYAML(`
	name: Jane Eyre
	author: Charlotte Bronte
`);
alert(book.title); // 오류 없음. 런타임에 "undefined" 경고
book("read"); //오류 없음, 런타임에 "TypeError: book은 함수가 아닙니다" 예외 발생
```

e.g : parseYAML이 unknown 타입을 반환하는게 더 안전함

```tsx
function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}
const book = safeParseYAML(`
	name: The Tenat of Wildfell Hall
	author: Anne Bronte
`);
alert(book.title); //개체가 unknown 형식입니다
book("read"); //개체가 unknown 형식입니다.
```

unknown 타입을 이해하기 위해서는 할당 가능성의 관점에서 any를 생각해볼 필요가 있음.

any가 강력하면서도 위험한 이유는 다음 두 가지 특징으로부터 비롯됨.

1. 어떠한 타입이든 any 타입에 할당 가능
2. any 타입은 어떠한 타입으로도 할당 가능

타입을 집합으로 생각하기의 관점에서, 한 집합은 다른 모든 집합의 부분 집합이면서 동시에 상위집합이 될 수 없기 때문에, 분명히 any는 타입 시스템과 상충되는 면을 가지고 있음. 이러한 점이 any의 강력함의 원천이면서 동시에 문제를 일으키는 원인이 됨.

타입 체커는 집합 기반이기 때문에 any를 사용하면 타입 체커가 무용지물이 된다는 것을 주의해야함.

unknown은 any 대신 쓸 수 있는 타입 시스템에 부합하는 타입.
