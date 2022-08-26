---
title: (JavaScript) EventEmitter 구현
categories: JavaScript
key: 20220826_02
comments: true
tags: JavaScript Event EventEmitter, Emitter, EventSubscribe
---

Node.js의 Event Emitter를 순수하게 간단히 구현해 보고 어떻게 작동되는지 그 기본 동작을 파악해 보겠습니다.<br/>
자바스크립트 자체는 이벤트를 구독하고 관리하는 객체가 없습니다. 그래서 Event Emitter를 직접 만들면서 살펴 보겠습니다.

<!--more-->

Event Emitter 란?
-

자바스크립트에서 addEventListener로 동작되는 기본 이벤트, 파일 I/O관련 이벤트나 네트워크 요청 이벤트 등으로 발생되는 이벤트 수준은 브라우저나 Node.js의 libuv에서 처리되는<br/>
시스템 이벤트라고 합니다. 이외 개발자가 직접 이벤트를 처리하는것은 모두 Event Emitter를 통해 관리되고 있습니다.<br/>
자바스크립트 자체에는 해당 역할의 기능 제공이 없기 때문에 Node.js에서는 이벤트 관리/처리 하는 매커니즘의 Event Emitter를 구현해서 제공하고 있습니다.

간단하게 구현해보기
-

직접 한번 만들어 볼까요? 우선 다음과 같이 EventEmitter 클래스를 구현해볼 수 있습니다.<br/>
```js
class EventEmitter {
	constructor() {
		this.events = {};
	}
	
	on( eventName, fn ) {
		if( !this.events[eventName] ) {
			this.events[eventName] = [];
		}
		this.events[eventName].push(fn);
		
		return () => {
			this.events[eventName] = this.events[eventName].filter(eventFn => fn !== eventFn);
		}
	}
	
	emit(eventName, data) {
		const event = this.events[eventName];
		if( event ) {
			event.forEach(listener => {
				listener.call(null, data);
			});
		}
	}
}
```

EventEmitter에 events객체를 초기화 해줘서 events객체는 구독되는 이벤트를 이름과 함수로 보관하고 관리하도록 합니다.<br/>
그리고 on() 함수를 통해 이벤트 구독 역할을 수행할 수 있도록 구현합니다. 이벤트의 이름, 이벤트 리스터 함수를 파라메터로 받고 events객체에 보관합니다.<br/>
on() 함수는 함수를 반환하는 고차 함수로 filter를 통해 해당 이벤트를 구독 취소 할 수 있는 역할의 함수를 반환하게 되어 있습니다.<br/>

이벤트 발생
-

이벤트 발생은 emit함수로 구독중인 이벤트 리스너 함수를 호출합니다. 단순히 등록된 이벤트가 있는지 체크 후 구독하는 리스터 함수를 호출하는 것만으로 끝납니다.

EventEmitter 사용
-

위에서 만든 EventEmitter를 이용해 간단한 이벤트 예제를 다음과 같이 구현해 보았습니다. 간단하게 브라우저로 테스트 해보기 위해<br/>
html을 다음과 같이 작성했습니다.<br/>

```html
<html>
	<head>
		<script src="test.js" defer></script>
	</head>
	<body>
		<div>
			<input type="text" /> <br/>
			<h1>text..</h1>
		</div>
		<div>
			<button id="Emit">Click</button>
			<button id="Unsubscribe">Unsubscribe</button>
		</div>
	</body>
</html>
```

그리고 자바스크립트 사용측 부분은 다음과 같습니다.<br/>
(위에 EventEmitter 코드 포함)<br/>
**[test.js]**<br/>
```js
class EventEmitter {
	constructor() {
		this.events = {};
	}
	
	on( eventName, fn ) {
		if( !this.events[eventName] ) {
			this.events[eventName] = [];
		}
		this.events[eventName].push(fn);
		
		return () => {
			this.events[eventName] = this.events[eventName].filter(eventFn => fn !== eventFn);
		}
	}
	
	emit(eventName, data) {
		const event = this.events[eventName];
		if( event ) {
			event.forEach(listener => {
				listener.call(null, data);
			});
		}
	}
}

const input = document.querySelector('input[type="text"]');
const emitButton = document.querySelector('#Emit');
const unsubscribeButton = document.querySelector('#Unsubscribe');
const h1 = document.querySelector('h1');

emitButton.addEventListener('click', () => {
	emitter.emit('event:name-changed', {name: input.value});
});

const emitter = new EventEmitter();
const unsubscribe = emitter.on('event:name-changed', data => {
	h1.innerHTML = `Your name is: ${data.name}`;
});

unsubscribeButton.addEventListener('click', () => {
	unsubscribe();
});
```

text input 객체를 갖고 Emit, Unsubscribe 이름을 갖는 각 버튼 객체, 그리고 h1객체를 생성합니다.<br/>
EventEmitter의 on() 함수로 'event:name-changed' 이름으로 이벤트 리스너 함수를 등록시면 해당 이벤트를 사용할 수 있게 됩니다.<br/>
Emit 버튼 클릭시 EventEmitter의 emit() 함수로 이벤트를 발생시켜 보면 정상적으로 처리 되는걸 확인할 수 있습니다.<br/>
그리고 on() 함수의 반환 함수 호출로 해당 이벤트의 구독을 취소해서 메모리 누수를 방지 처리합니다.<br/>
브라우저 결과<br/><br/>

![image](https://user-images.githubusercontent.com/13028129/186836343-61fb44f8-9574-492a-a907-5990ed6a35c2.png)<br/><br/>

Unsubscribe 버튼 클릭시 이벤트는 해제 되어 더 이상 이벤트 발생은 되지 않은걸 확인할 수 있습니다.<br/>

> 위 코드는 다음 사이트를 참고하였음을 알려드립니다.
> [https://netbasal.com/javascript-the-magic-behind-event-emitter-cce3abcbcef9](https://netbasal.com/javascript-the-magic-behind-event-emitter-cce3abcbcef9)


{% include content_adsense.html %}
