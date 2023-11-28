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
