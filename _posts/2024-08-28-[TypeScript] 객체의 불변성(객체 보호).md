---
title: (TypeScript) 객체의 불변성(객체 보호)
categories: TypeScript
key: 20240828_01
comments: true
tags: ts, typescript, 불변성, 불변, Immutable
---

프로그래밍에서 어떤 값이나 모델을 제공할때 의도치 않은 변화로 인해 오류가 발생되거나, 오류는 발생 되지 않았지만 잘못된 데이터로 인해 예기치 못한 결과를 만들어 낼 수 있는 상황이 있을 수 있습니다.<br/>
이때 객체를 **불변(Immutable)으로 처리** 함으로서 위와 같은 상황을 최소화 하거나 해결 할 수 있습니다.<br/>
여러 언어에서 객체를 불변으로 처리하는 방법이 있는데 TypeScript나 JavaScript에서 객체를 불변으로 다루는 방법을 알아 보겠습니다.

<!--more-->

상수 지정 (constant)
-

상수 키워드(const)는 TypeScript, JavaScript에서 모두 사용 가능한 키워드 입니다.<br/>
상수로 선언된 변수는 재 할당이 불가능 하기에 해당 변수에 초기화된 값을 그대로 사용할 수 있습니다.<br/><br/>

```ts
type Person = {
  name: string;
  address: {
    city: string;
  },
  hobby: string[]
};

const person: Person = {
  name: "John",
  address: {
    city: "New York",
  },
  hobby: ['game', 'drive']
};

// 오류 발생 (수정 불가)
person = Person = {
  name: "ming",
  address: {
    city: "Seoul",
  },
  hobby: ['programing', 'drive']
};
```

하지만, const로 선언된 객체의 경우 객체의 속성 값은 변경할 수 있습니다. 단지 객체 자체를 다른 객체로 재할당할 수 없을 뿐입니다.<br/><br/>

```ts
// name 속성 변경 가능
person.name = "ming";
```

Objest.seal()
-

**<span style="color: rgb(107, 173, 222);">Objest.seal()</span>** 메서드 또한 TypeScript, JavaScript에서 사용 가능합니다.<br/>
**<span style="color: rgb(107, 173, 222);">Objest.seal()</span>** 메서드에 대해 MDN 문서 설명을 보면 객체를 **'밀봉'** 한다고 설명 하고 있습니다.<br/>
그럼 객체를 밀봉 한다는 의미가 무엇인지 확인해 보기 위해 객체의 속성 상태를 같이 체크해 보겠습니다.<br/>
우선 해당 객체에 부여된 속성 설정 값을 확인해 보려면 **<span style="color: rgb(107, 173, 222);">Object</span>**객체의 **<span style="color: rgb(107, 173, 222);">getOwnPropertyDescriptors()</span>** 메서드로 확인해 볼 수 있습니다.<br/><br/>

```ts
const person = {
    name: "John",
    address: {
        city: "New York",
    },
    hobby: ['game', 'drive']
};
```

```ts
Object.getOwnPropertyDescriptors(person)
```

![image](https://github.com/user-attachments/assets/3576b4bc-27ef-4e21-ae84-b41bb171ac5e)<br/>
결과를 보면 'address', 'hobby', 'name' 각 속성의 정보를 확인해 볼 수 있습니다.<br/>
여기서 유심히 확인해 봐야할 정보는 'writable', 'enumerable', 'configurable' 속성 입니다.<br/>
각 속성에 대해 먼저 설명을 하면 다음과 같습니다.<br/>
- writable (기본값 true) : false 인 경우 속성 값을 수정할 수 없습니다. js에서 스트릭트 모드인 경우 오류가 발생 됩니다.
- enumerable (기본값 true) : false 인 경우 iterator 반복(for in, Object.keys())을 사용할 수 없습니다.
- configurable (기본값 true) : false 인 경우 객체의 속성을 제거 및 추가할 수 없습니다.

그럼 **<span style="color: rgb(107, 173, 222);">Objest.seal()</span>** 메서드로 객체를 '밀봉' 했을때 저 속성들이 어떻게 변하고 어떤 결과를 일으키는지 확인해 보겠습니다.<br/><br/>

```ts
type Person = {
  name: string;
  address: {
    city: string;
  },
  hobby: string[]
};

const person: Person = Object.seal({
  name: "John",
  address: {
    city: "New York",
  },
  hobby: ['game', 'drive']
});
console.log(Object.getOwnPropertyDescriptors(person));
```

**[결과]** <br/>
```
[LOG]: {
  "name": {
    "value": "John",
    "writable": true,
    "enumerable": true,
    "configurable": false
  },
  "address": {
    "value": {
      "city": "New York"
    },
    "writable": true,
    "enumerable": true,
    "configurable": false
  },
  "hobby": {
    "value": [
      "game",
      "drive"
    ],
    "writable": true,
    "enumerable": true,
    "configurable": false
  }
} 
```

'configurable' 속성이 false로 변경 되었음을 알 수 있고, <br/><br/>

```ts
person.name = 'ming';  // name 속성은 변경 가능지만
person.name2 = 'aaaa2';  // 오류 발생 (다른 속성은 추가 불가능)
```

이런 결과가 발생 되는 것을 알 수 있습니다.


Objest.freeze()
-

**<span style="color: rgb(107, 173, 222);">Objest.freeze()</span>** 메서드는 **<span style="color: rgb(107, 173, 222);">Objest.seal()</span>** 메서드 보다 좀더 강력(?)하게 객체를 완전히 불변 상태로 만들 수 있습니다.<br/>
역시 직접 코드를 통해 속성 상태가 어떻게 변하는지 확인해 보겠습니다.<br/><br/>

```ts
type Person = {
  name: string;
  address: {
    city: string;
  },
  hobby: string[]
};

const person: Person = Object.freeze({
  name: "John",
  address: {
    city: "New York",
  },
  hobby: ['game', 'drive']
});
console.log(Object.getOwnPropertyDescriptors(person));
```

**[결과]** <br/>
```
[LOG]: {
  "name": {
    "value": "John",
    "writable": false,
    "enumerable": true,
    "configurable": false
  },
  "address": {
    "value": {
      "city": "New York"
    },
    "writable": false,
    "enumerable": true,
    "configurable": false
  },
  "hobby": {
    "value": [
      "game",
      "drive"
    ],
    "writable": false,
    "enumerable": true,
    "configurable": false
  }
} 
```

**<span style="color: rgb(107, 173, 222);">Objest.seal()</span>** 결과에서 'configurable' 속성이 false로 변경 되었는데, 추가로 'writable' 속성도 false로 변경 되었습니다.<br/>
'writable' 속성이 false인 경우 속성 값을 수정 할 수 없다고 설명했는데 직접 확인해 보겠습니다.<br/><br/>

```ts
person.name = 'ming';
person.name2 = 'aaaa2';
```

**[결과]**
```
[ERR]: Cannot assign to read only property 'name' of object '#<Object>'
```
```
[ERR]: Cannot add property name2, object is not extensible
```

'name' 속성 값을 변경 했을때, 그리고 새로운 속성을 추가 했을때 위와 같이 각각 오류가 발생됨을 알 수 있습니다.<br/>


Objest.seal() VS Objest.freeze()
-

정리하자면 **<span style="color: rgb(107, 173, 222);">Objest.seal()</span>** 메서드는 속성 값 수정은 허용 하지만 객체에 속성을 추가하는 것은 허용 됩니다.<br/>
**<span style="color: rgb(107, 173, 222);">Objest.freeze()</span>** 메서드는 속성 값 수정도 허용 하지 않습니다.<br/><br/>

하지만, 위 메서드 모두 중첩된 객체까지 불변으로 만들수 없는 문제가 있습니다.<br/>
위에서 사용했던 예제 코드를 그대로 중첩 객체인 'address'에 'city' 속성을 변경해 보겠습니다.<br/><br/>

```ts
person.address.city = 'Los Angeles';  // 변경 가능
console.log(person3.address.city);
```

**[결과]**
```
[LOG]: "Los Angeles"
```

이 문제에 대해 해결 방안은 **<span style="color: rgb(107, 173, 222);">Objest.freeze()</span>** 메서드로 불변 객체를 만드는 부분을 재귀 호출로 중첩된 객체까지 찾아 처리해 주어야 합니다.


Readonly&lt;Type&gt; (TypeScript)
-

TypeScript에서 자체적으로 객체를 불변으로 지정하는 방법을 제공 하는데, TypeScript의 유틸리티 타입중 하나인 **<span style="color: rgb(107, 173, 222);">Readonly&lt;Type&gt;</span>** 타입은 얕은 불변성으로 지정 할 수 있습니다.<br/>
해당 타입은 **<span style="color: rgb(107, 173, 222);">Objest.freeze()</span>** 와 결과가 같습니다.<br/><br/>

```ts
type Person = {
  name: string;
  address: {
    city: string;
  },
  hobby: string[]
};

const person: Readonly<Person> = {
  name: "John",
  address: {
    city: "New York",
  },
  hobby: ['game', 'drive']
};

//person.name = "Jane";  // 오류 발생 (수정 불가)
person.address.city = "Los Angeles";  // 내부 객체는 수정 가능 (얕은 불변성)
```


DeepReadonly 처리
-

**<span style="color: rgb(107, 173, 222);">Readonly&lt;Type&gt;</span>** 타입을 이용해서 모든 중첩된 속성까지 재귀적으로 처리해서 중첩 객체까지 불변으로 만들 수 있습니다.<br/>
이런 처리는 **매핑된 타입(mapped type)** 을 이용해서 구현할 수 있습니다.<br/>
**매핑된 타입(mapped type)** 은 키 집합의 각 키에 대한 새로운 속성을 만들어서 새로운 타입을 지정할 수 있습니다.

### 매핑된 타입(mapped type)
TypeScript 공식 문서에 설명 되고 있는 키를 다시 매핑 하는 코드 형식은 다음과 같습니다.<br/><br/>

```ts
type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

예를 들어 다음과 같은 타입이 정의 되어 있습니다.<br/><br/>

```ts
type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
};
```

**keyof** 를 통해 'Features' 타입의 모든 키 타입을 boolean으로 변경할 수 있습니다.<br/><br/>

```ts
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

type FeatureOptions = OptionsFlags<Features>;
```
```
type FeatureOptions = {
    darkMode: boolean;
    newUserProfile: boolean;
}
```

또한 매핑 수정자를 통해 고유 속성을 제거 할 수도 있습니다.<br/>
다음 코드는 **keyof**로 타입의 모든 키를 이용해 **Indexed Access Types(인덱스드 접근 타입)** 으로 해당 키의 타입을 그대로 적용하고 readonly 특성만 제거 하는 코드 입니다.<br/><br/>

```ts
// 'readonly' attributes 제거
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
 
type UnlockedAccount = CreateMutable<LockedAccount>;
```
```
type UnlockedAccount = {
    id: string;
    name: string;
}
```

```ts
// 'optional' attributes 제거
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};
 
type User = Concrete<MaybeUser>;
```
```
type User = {
    id: string;
    name: string;
    age: number;
}
```

DeepReadonly&lt;Type&gt; (TypeScript) 구현
-

그럼 위에서 알아본 매핑된 타입으로 중첩 객체 까지 완전한 불변 객체를 만드는 타입을 정의해 보겠습니다.<br/><br/>

```ts
type DeepReadonly<T> = {
  readonly [P in keyof T]: DeepReadonly<T[P]>;
};

const deepPerson: DeepReadonly<Person> = {
  name: "John",
  address: {
    city: "New York",
  },
  hobby: ['game', 'drive']
};

deepPerson.name = 'ming';  // 오류 발생 (수정 불가)
deepPerson.address.city = 'Los Angeles';  // 오류 발생 (중첩 객체도 수정 불가)
```

타입의 모든 키를 가져와 **DeepReadonly&lt;Type&gt;** 타입을 재귀적으로 적용되도록 합니다.



***



{% include content_adsense.html %}
