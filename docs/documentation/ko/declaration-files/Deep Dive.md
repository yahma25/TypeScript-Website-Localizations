---
title: Deep Dive
layout: docs
permalink: /ko/docs/handbook/declaration-files/deep-dive.html
oneline: "How do d.ts files work, a deep dive"
---

## 정의 파일 이론: 심층 분석 (Definition File Theory: A Deep Dive)

원하는 API 형태를 제공하는 모듈을 만드는 것은 까다로울 수 있습니다.
예를 들어, `new`의 사용에 따라 호출할 때 다른 타입을 생성하는 모듈을 원할 수 있고,
  계층에 노출 된 다양한 명명된 타입을 가지고 있으며,
  모듈 객체에 대한 여러 프로퍼티도 가질 수 있습니다.

이 가이드에서는, 익숙한 API를 노출하는 복잡한 정의 파일에 대해 작성하는 도구를 제공합니다.
또한 옵션이 다양하기 때문에 여기서는 모듈 (또는 UMD) 라이브러리에 중점을 둡니다.

## 주요 컨셉 (Key Concepts)

TypeScript 작동 방식에 대해 여러 주요 개념을 이해하여
  정의의 형태를 만드는 방법을 완전히 이해할 수 있습니다.

### 타입 (Types)

이 가이드를 읽고 있다면, 아마도 TypeScript의 타입에 대해 이미 알고 있을 것입니다.
보다 명확하게하기 위해, 다음과 같이 *타입*이 도입됩니다:

* 타입 별칭 선언 (`type sn = number | string;`)
* 인터페이스 선언 (`interface I { x: number[]; }`)
* 클래스 선언 (`class C { }`)
* 열거형 선언 (`enum E { A, B, C }`)
* 타입을 가리키는 `import` 선언

이러한 각 선언 형태는 새로운 타입 이름을 만듭니다.

### 값 (Values)

타입과 마찬가지로 값이 무엇인지 이미 알고 있을 것입니다.
값은 표현식에서 참조할 수 있는 런타임 이름입니다.
예를 들어 `let x = 5;`에서는 `x`라고 불리는 값을 생성합니다.

다시 명확하게 말하자면, 다음과 같이 값을 만듭니다:

* `let`, `const`, 그리고 `var` 선언
* 값을 포함하는 `네임스페이스` 또는 `모듈` 선언
* `열거형` 선언
* `클래스` 선언
* 값을 참조하는 `import` 선언
* `함수` 선언

### 네임스페이스 (Namespaces)

타입은 *네임스페이스* 안에 존재할 수 있습니다.
예를 들어, `let x: A.B.C` 이란 선언이 있다면,
  타입 `C`는 `A.B` 네임스페이스에서 온 것 입니다.

이 구별은 미묘하지만 중요합니다 -- 여기서 `A.B`는 타입이거나 값일 필요는 없습니다.

## 간단한 조합: 하나의 이름, 여러 의미 (Simple Combinations: One name, multiple meanings)

`A`라는 이름이 있으면, `A`에 대해 타입, 값 또는 네임스페이스라는 세 가지 다른 의미를 찾을 수 있습니다.
이름을 해석하는 방법은 사용하는 컨텍스트에 따라 다릅니다.
예를 들어 `let m: A.A = A;` 선언에서,
  `A`는 먼저 네임스페이스로 사용 된 다음, 타입의 이름으로, 그 다음 값으로 사용됩니다.
즉 완전히 다른 선언을 의미할 수 있습니다!

약간은 혼란스러워 보이지만, 과하게 사용하지 않는 한 실제로 매우 편리합니다.
결합 동작의 유용한 측면을 살펴 보겠습니다.

### 내부 조합 (Built-in Combinations)

영리한 사람이라면, *타입*과 *값* 목록에서 `클래스`가 둘 다 나온 것을 눈치챘을 것입니다.
`class C { }` 선언은 두 가지를 만듭니다:
  클래스 인스턴스의 형태를 나타내는 *타입* `C`와
  클래스 생성자를 나타내는 *값* `C` 입니다.
열거형 선언도 비슷하게 동작합니다.

### 사용자 조합 (User Combinations)

모듈 파일 `foo.d.ts`을 작성했습니다:

```ts
export var SomeVar: { a: SomeType };
export interface SomeType {
  count: number;
}
```

그 다음 사용했습니다:

```ts
import * as foo from './foo';
let x: foo.SomeType = foo.SomeVar.a;
console.log(x.count);
```

잘 작동하지만, `SomeType`과 `SomeVar`는 이름이 같도록
  밀접하게 관련되어 있다고 상상할 수 있습니다.
결합을 사용하여 같은 이름 `Bar`로 두 가지 다른 객체 (값과 타입)를 표시 할 수 있습니다:

```ts
export var Bar: { a: Bar };
export interface Bar {
  count: number;
}
```

이 경우 사용하는 코드를 구조 분해할 수 있는 아주 좋은 기회입니다:

```ts
import { Bar } from './foo';
let x: Bar = Bar.a;
console.log(x.count);
```

여기서도 `Bar`를 타입과 값으로 사용했습니다.
`Bar` 값을 `Bar` 타입으로 선언할 필요가 없다는 점을 유의하세요 -- 저 둘은 독립적입니다.

## 고급 결합 (Advanced Combinations)

선언은 여러 개의 선언에 걸쳐 결합될 수 있습니다.
예를 들어, `class C { }`와 `interface C { }` 같이 결합할 수 있으며 둘 다 `C` 타입에 프로퍼티를 추가합니다.

충돌을 일으키지 않는다면 충분히 합법적입니다.
일반적인 경험 법칙은 값의 이름이 `네임스페이스`로 선언되지 않는 한 항상 같은 이름의 다른 값과 충돌하고,
  타입 별칭 선언(`type s = string`)으로 선언 된 경우 타입이 충돌하며,
  네임스페이스와는 절대로 충돌하지 않는 것입니다.

어떻게 사용되는지 살펴보겠습니다.

### `인터페이스`를 사용하여 추가하기 (Adding using an `interface`)

`인터페이스`에 다른 `인터페이스` 선언을 사용하여 멤버를 추가할 수 있습니다:

```ts
interface Foo {
  x: number;
}
// ... 다른 위치 ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // 성공
```

클래스와도 같이 동작합니다:

```ts
class Foo {
  x: number;
}
// ... 다른 위치 ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // 성공
```

단, 타입 별칭 (`type s = string;`)에는 인터페이스를 사용해서 추가할 수 없습니다.

### `네임스페이스`를 사용하여 추가하기 (Adding using a `namespace`)

`네임스페이스` 선언은 충돌을 일으키지 않는 방식으로 새로운 타입, 값 그리고 네임스페이스를 추가할 수 있습니다.

예를 들어, 클래스에 정적 멤버를 추가할 수 있습니다:

```ts
class C {
}
// ... 다른 위치 ...
namespace C {
  export let x: number;
}
let y = C.x; // 성공
```

위 예제에서 `C`의 *정적* 측면(생성자 함수)에 값을 추가했습니다.
*값*을 추가 했고 모든 값에 대한 컨테이너가 다르기 때문입니다.
  (타입은 네임스페이스에 포함되고 네임스페이스는 다른 네임스페이스에 포함됩니다).

네임스페이스 타입을 클래스에 추가할 수 있습니다:

```ts
class C {
}
// ... 다른 위치 ...
namespace C {
  export interface D { }
}
let y: C.D; // 성공
```

이 예제에서 `namespace` 선언을 작성할 때까지 네임스페이스 `C`는 없었습니다.
네임스페이스 `C`는 클래스에 의해 생성된 `C`의 값 또는 타입과 충돌하지 않습니다.

마지막으로 `namespace` 선언을 사용하여 다양한 병합을 할 수 있습니다.
특히 현실적인 예는 아니지만, 흥미로운 동작을 확인할 수 있습니다:

```ts
namespace X {
  export interface Y { }
  export class Z { }
}

// ... 다른 위치 ...
namespace X {
  export var Y: number;
  export namespace Z {
    export class C { }
  }
}
type X = string;
```

위 예제에서 첫 번째 블록은 다음 이름의 의미를 만듭니다:

* 값 `X` (`네임스페이스` 선언은 값 `Z`를 포함하기 때문입니다)
* 네임스페이스 `X` (`네임스페이스` 선언은 타입 `Y`를 포함하기 때문입니다)
* `X` 네임스페이스 안의 타입 `Y`
* `X` 네임스페이스 안의 타입 `Z` (클래스의 인스턴스 형태)
* `X` 값의 프로퍼티인 값 `Z` (클래스의 생성자 함수)

두 번째 블록은 다음 이름의 의미를 만듭니다:

* `X` 값의 프로퍼티인 값 `Y` (`number` 타입)
* 네임스페이스 `Z`
* `X` 값의 프로퍼티인 값 `Z`
* `X.Z` 네임스페이스 안의 타입 `C`
* `X.Z` 값의 프로퍼티인 값 `C`
* 타입 `X`

## `export =` or `import` 사용하기 (Using with `export =` or `import`)

중요한 규칙은 `export`와 `import` 선언이 대상의 *모든 의미* 를 내보내거나 가져온다는 것 입니다.

<!-- TODO: Write more on that. -->
