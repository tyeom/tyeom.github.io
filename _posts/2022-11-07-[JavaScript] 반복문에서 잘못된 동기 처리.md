---
title: (JavaScript) 반복문에서 잘못된 동기 처리
categories: JavaScript
key: 20221107_01
comments: true
tags: JavaScript js 비동기 foreach 동기처리 async await
---

시간이 오래 걸리는 여러 작업을 순차적으로 처리하고자 할때 fetch API나 axios 라이브러리를 통해 비동기로 요청하고 그 결과를 await을 통해 동기처리 하여 요청하는 상황이 있을 수 있습니다.<br/>
이럴 경우 흔히 반복문 안에서 비동기 요청 후 결과를 대기하는 식으로 처리할 수 있는데 이럴 경우 잘못된 상황에 대해 설명하고자 합니다.

<!--more-->

foreach에서 비동기 결과 처리
-

여러개의 비동기의 요청을 직렬로 순차적으로 처리할때 foreach 반복문으로 처리하면 병렬 처리 될 수 있습니다.<br/>
```js
'use strict';

const baseUrl = 'http://arong.info:7003/posts/';

const foo = async (id) => {
	const response = await fetch(baseUrl + id, {
		method: 'GET',
	  	//mode: 'no-cors',
		cache: 'no-cache',
		credentials: 'same-origin',
	});
  console.log(`${id} 요청 완료`);
	const json = await response.json();
	return json;
};

const paramArr = [1, 2, 3, 4, 5];
paramArr.forEach( async (param) =>  await foo(param).then(console.log) );
```

위 코드는 5개의 파라메터 값을 이용해서 foreach 반복으로 비동기로 데이터를 요청하고 await으로 대기 후 결과를 순서대로 출력하는 간단한 코드 입니다.<br/>
즉 언뜻 볼때 결과의 기대는 1 ~ 5 데이터에 해당되는 결과를 순서대로 출력할 것 처럼 보입니다.<br/>
하지만 결과는 다음과 같습니다.<br/><br/>

![image](https://user-images.githubusercontent.com/13028129/200252766-dbad39ff-e632-4f89-b7d4-052f9bad2411.png)<br/><br/>

비동기 요청은 순차적으로 처리 했지만 그 결과는 await으로 대기 하지 않고 병렬로 처리 되서 결과가 출력되고 있습니다.<br/>
호출된 네트워크 상태를 시각적으로 보면 확실이 알 수 있습니다.<br/>
![image](https://user-images.githubusercontent.com/13028129/200253144-2afb56a6-af06-48a5-8760-20f2d488b571.png)<br/>
**(5번의 요청이 동시에 요청됨)**<br/><br/>

그 이유는 forEach 구문은 반복으로 호출될 메서드를 콜백(callback)으로 넘겨주고 바로 return 되어서 처리 되기 때문 입니다.<br/>
**[구문]**
```js
arr.forEach(callback(currentvalue[, index[, array]])[, thisArg])
```
이렇게 forEach 반복 키워드는 첫번째 인자로 콜백으로 호출될 함수를 받고 있는데 위 코드에서 await 키워드를 사용하고 foo 비동기 함수를 호출 했지만 
해당 함수는 콜백으로 등록되고 바로 return 되서 처리 된 것입니다. 그리고 이후 비동기 요청이 완료 되고 결과가 출력된 것 입니다.<br/>
그럼 병렬 처리가 아닌 순차적으로 처리되도록 하려면 어떻게 해야 할까요?

정상적인 비동기 요청 직렬 처리
-

간단히 반복 처리를 콜백으로 하지 않고 직접 호출하도록 하면 해결됩니다. 이것은 forEach가 아닌 for 키워드를 사용해서 해결할 수 있습니다.<br/>
```js
const paramArr = [1, 2, 3, 4, 5];

// for 반복문을 통해 비동기 요청
( async () => {
	for(let param of paramArr) {
		await foo(param).then(console.log);
	}
})();
```

결과를 확인해 보면 다음처럼 순서대로 요청후 결과를 기다리고 출력되는걸 확인할 수 있습니다.<br/><br/>

![image](https://user-images.githubusercontent.com/13028129/200255475-cbb75195-f9c2-4223-96a3-4dc02f48d311.png)<br/>
![image](https://user-images.githubusercontent.com/13028129/200255521-1f447021-36a2-481c-b19e-5e1a3178f61f.png)<br/>
**(5번의 요청이 순차적으로 요청됨)**<br/>

이렇게 콜백으로 동작되는 상황에서 비동기로 함수를 호출했을때 병렬로 처리될 수 있으며 순차적으로 처리되어야 하는 로직인 경우 기대한 값이 아닌 잘못된 값으로 
처리될 여지가 있으므로 확실히 짚고 넘어가야 합니다.



{% include content_adsense.html %}
