---
title: (JavaScript) 다시 한번 기초 다지기! - JS 비동기 처리
categories: JavaScript
key: 20220826_01
comments: true
tags: JavaScript 비동기 async,await callback Promise
---

자바스크립트는 기본적으로 단일 스택으로 싱글 스레드로 처리되서 동기적으로 처리됩니다.<br/>
이것이 어떤말인가 하면 자바스크립트가 엔진에 의해 처리 될때 호이스팅(hoisting) 처리후에 한줄 한줄씩 순차적으로 처리된다는 말입니다.<br/>
> 호이스팅(hoisting) 이란?<br/>
> 변수, 일반 함수 선언된것이 최상위로 올라가는 현상

그렇기 때문에 효율적으로 처리하기 위해서는 비동기 처리 방법이 필수적인 상황이 있는데 자바스크립트에서 비동기로 처리하는 방법을 하나씩 알아보도록 해보겠습니다.

<!--more-->

비동기 Callback 처리
-

자바스크립트에서 비동기로 처리하기 위한 방법 중 하나는 콜백함수를 이용한 처리 입니다.<br/>
콜백함수는 어떤한 작업을 처리한 후에 넘겨받은 함수를 호출함으로써 비동기로 처리하는 방법 입니다.<br/>
아래 예제 코드는 데이터를 요청하기 위해 정해진 id와 password로 토큰값을 받고 받은 토큰값을 이용해 데이터를 요청해서 Log에 출력하는 코드 입니다.<br/>
네트워크로 요청한다는 가정으로 setTimeout함수로 딜레이 처리 하였습니다.

```js
class DataRequest {
	getToken(id, password, onSuccess, onError) {
		setTimeout( () => {
			if(id === 'admin' && password === '1234') {
				onSuccess('abcde');
			}
			else {
				onError(new Error('ID or password is incorrect'));
			}
		}, 2000);
	}
	
	getData(token, onData) {
		setTimeout( () => {
			if(token === 'abcde') {
				onData('Hello');
			}
			else {
				onData(null);
			}
		}, 2000);
	}
}

// 데이터 요청
const dataRequest = new DataRequest();
dataRequest.getToken('admin',
	'1234',
	token => {
		let data = dataRequest.getData(token, data => {
			console.log(data);
		})
	},
	error => {
		console.log(error);
	});
```

위와 같이 비동기로 언제 끝날지 모르는 처리결과를 함수 호출로 통보 받는 방식이 콜백함수 방식 입니다.<br/>
위 처럼 간단한 비동기 처리는 이 처럼 콜백함수로 처리해도 무난하지만 복잡한 체인 형태라면 코드가 길어지고 복잡지는 단점이 있습니다.

Promise 처리
-

Promise는 위 콜백함수 처리 방식보다 조금 더 간결하게 비동기 처리를 할 수 있는 자바스크립트에서 제공하는 비동기 처리 오브젝트 입니다.<br/>
Promise는 두가지의 포인트가 있습니다. 첫번째는 상태(State) 두번째는 데이터 제공자(Producer)와 데이터 소비자(Consumer)로 나뉠 수 있습니다.<br/>
또 State는 현재 **데이터 처리중 상태인 pending**과 데이터 처리를 **성공적으로 완료된 fulfilled상태** **오류로 인해 처리가 완료된 rejected상태**가 있습니다.<br/>
그리고 Producer는 Promise오브젝트라고 보면 됩니다.

한가지 짚고 넘어가야할 부분이 있는데 Promise오브젝트는 객체를 생성하는 순간 내부에 구현 되어 있는 executor가 바로 호출 된다는점 입니다.<br/>
```js
const promise = new Promise( (resolve, reject) => {
  console.log('Hello');
});
```

```
실행결과 : Hello
```

### 데이터 제공자(Producer)

Promise를 통한 비동기 결과 핸들링은 resolve, reject으로 결과를 통보할 수 있습니다.<br/>
```js
const promise = new Promise( (resolve, reject) => {
  setTimeout( () => resolve('Hello'), 2000);
});
```
이렇게 비동기 처리 데이터를 제공하는 Producer를 구현할 수 있습니다.

### 소비자(Consumer)

위에서 만든 Promise를 사용하는 Consumer 부분은 then, catch, finally를 통해 핸들링 할 수 있습니다.<br/>

```js
promise.then(value => {
  console.log(value);
});
```

데이터 처리 성공으로 resolve로 처리된 결과는 then API로 받아 처리할 수 있고 오류발생시 reject으로 처리된 결과는 catch API를 통해 처리할 수 있습니다.<br/>
또한 finally로 마지막 후 처리가 필요한 작업을 처리할 수 있습니다.<br/>

Promise Chaining
-

then API는 값을 Promise를 통해 받아온 값을 반환할 수도 있지만 또 다른 새로운 Promise 객체를 반환시켜 체이닝으로 연속적인 작업 처리가 가능합니다.<br/>
다음 예제는 정해진 id와 password로 로그인 시도를 하고 로그인 성공시 받아온 데이터를 사용해서 순차적으로 다른 네트워크 API를 호출한다는 가정으로 처리하는 예제입니다.<br/>

```js
const getData = (id, password) =>
	new Promise( (resolve, reject) => {
		setTimeout( () => {
			if(id === 'admin' && password === '1234') {
				resolve('A')
			}
			else {
				reject(new Error('ID or password is incorrect'));
			}
		}, 2000);
	});

const process1 = (data) =>
	new Promise( (resolve, reject) => {
		setTimeout( () => {
			resolve(data + 'B');
		}, 1000);
	});

const process2 = (data) =>
	new Promise( (resolve, reject) => {
		setTimeout( () => {
			resolve(data + 'C');
		}, 1000);
	});

getData('admin', '12345')
	.then(value => process1(value))
	.catch(error => {
		console.log(error);
		console.log('다시 시도합니다.');
		return getData('admin', '1234');
	})
	.then(value => process1(value))
	.then(value => process2(value))
	.then(value => console.log(value))
	.catch(error => console.log(error));
```

**결과**
```
Error: ID or password is incorrect
    at test.js:8:12
다시 시도합니다.
ABC
```

코드를 보면 id와 password를 통해 데이터를 비동기로 요청하는 Promise와 그 데이터를 이용해서 비동기 요청으로 사용되는 총 2개의 Promise가 있습니다.<br/>
각 순차적으로 체이닝 호출로 Promise를 사용하면서 동시에 catch를 통해 오류 발생시 한번더 getData함수를 호출해 처리되는걸 볼 수 있습니다.<br/>
Promise 오브젝트를 사용면 콜백처리 보다는 간략하게 비동기 처리를 사용할 수 있지만 이 역시도 then의 연속적인 체이닝으로 다소 복잡하게 보여질 수 있는 부분이 있습니다.

async / await
-

async / await은 자바스크립트에서 제공하는 Promise오브젝트를 이용해서 좀더 간결한 비동기 문법을 제공해주는 API입니다.<br/>
즉 자바스크립트 엔진에 의해 Promise를 사용하는 Syntactic sugar 입니다. (자바스크립트에서 사용 가능한 class도 마찬가지로 prototype 기반으로 제공되는 Syntactic sugar 입니다.)<br/>

```js
async function asyncFun() {
  return 'Test';
}

const fun = asyncFun();
console.log(fun);
```

위 코드를 보면 asyncFun 함수가 자동으로 Promise오브젝트로 처리되서 호출되는것을 확인해 볼 수 있습니다.<br/><br/>

![image](https://user-images.githubusercontent.com/13028129/186806765-dbcc94a2-add2-4781-9234-44ceb6cde23b.png)<br/><br/>

이렇게 함수에 async키워드를 추가함으로써 Promise State는 fulfilled로 되어 있고 결과는 Test로 표시 됩니다.

async함수 안에서 Promise의 결과를 대기 할때는 await 키워드를 사용할 수 있습니다.<br/>
```js
const delay = (ms) => {
	return new Promise(resolve => setTimeout(resolve, ms));
};

const foo1 = async () => {
	await delay(2000);
	return 'completion1';
}

const foo2 = async () => {
	await delay(1000);
	return 'completion2';
}

const result = async () => {
	const f1 = await foo1();
	const f2 = await foo2();
	return `${f1}_${f2}`;
}

result().then(console.log);  // 약 3초 후 completion1_completion2 출력
```

위 코드는 약 2초 지연 foo1함수와 약 1초 지연 foo2함수를 순차적으로 호출하고 그 결과를 출력하는 코드 입니다. 하지만 위 코드를 병렬로 처리하려면 다음과 같이 수정할 수 있습니다.<br/>
```js
const result = async () => {
  // 병렬로 foo1(), foo2() 동시 호출
  const f1_Promise = foo1();  // Promise오브젝트 생성시 바로 executor 호출
  const f2_Promise = foo2();  // Promise오브젝트 생성시 바로 executor 호출
	const f1 = await f1_Promise;
	const f2 = await f2_Promise;
	return `${f1}_${f2}`;
}

result().then(console.log);  // 약 2초 후 completion1_completion2 출력
```

Promise오브젝트는 객체를 생성하는 순간 executor가 바로 호출 된다고 설명했었는데 위 처럼 await 없이 즉시 호출해서 Promise 생성과 동시에 실행함으로써 병렬로 처리되도록 할 수 있습니다.<br/>
하지만 병렬처리를 하기 위해서 위 방식보다 더 좋은 방식은 Promise에서 제공되는 병렬처리 API를 이용하는 것 입니다.<br/>
그중 Promise.all() API가 있는데 여러개의 Promise를 배열로 받아서 병렬로 처리하고 모든 Promise가 완료 될때까지 비동기로 처리할 수 있습니다.<br/>
```js
const result = () => {
  // foo1(), foo2() Promise를 병렬로 처리
	return Promise.all( [foo1(), foo2()] )
		.then(value => value.join('_')
		);
}

result().then(console.log);  // 약 3초 후 completion1_completion2 출력
```

Promise.all() 외에도 Promise.race API도 있는데 여러개의 Promise중 가장 먼저 처리된 결과만을 반환받아 처리할 수 있습니다.<br/>
```js
const result = () => {
  // foo1(), foo2() Promise를 병렬로 처리하고 가장 먼저 끝나는 함수의 결과 반환
	return Promise.race( [foo1(), foo2()] )
}

result().then(console.log);  // 약 1초 후 completion2 출력
```

이 밖에도 Promise관련해서 allSettled(), any() 등이 있습니다. 관련해서는 [MDN Web Docs 사이트를 참고하면 됩니다.](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)

{% include content_adsense.html %}
