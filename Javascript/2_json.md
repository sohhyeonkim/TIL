# JSON

JSON은 Javascript Object Notation으로 이름으로 봐서 알 수 있듯이 key와 value를 갖는 자바스크립트 객체 문법으로 구조화된 데이터를 표현하기 위한 문자 기반의 표준 포맷이다. JSON이 자바스크립트 문법과 유사하지만 자바스크립트가 아닌 다른 언어에서도 JSON을 읽고 쓸 수 있다.

## 언제 사용할까?

클라이언트에서 서버로 데이터를 전송하거나 서버로부터 데이터를 받을 때 사용한다. 문자열 형태로 존재하기 때문에 네트워크를 통해 데이터를 주고받을때 유용하다.

## 특징

- JSON 안에는 자바스크립트의 기본 데이터 타입인 문자열, 숫자, 배열, 불리언, 다른 객체를 포함할 수 있다.

- JSON은 순수한 데이터 포맷으로 메서드는 포함할 수 없다.

- JSON은 문자열 형태로 존재하며 프로퍼티 이름 작성시 큰 따옴표만 사용해야한다. 작은 따옴표는 사용할 수 없다.
  자바스크립트는 JSON 전역 객체를 제공하며 stringify 메소드를 사용해서 네트워크를 통해 전달할 수 있게 객체를 문자열로 변환(serialization)할 수 있다. parse 메소드를 사용해서 문자열에서 네이티브 객체로 변환(deserialization)할 수 있다.

```js
// stringify
const json1 = JSON.stringify(true);
console.log(json1); // 'true'

const json2 = JSON.stringify(2);
console.log(json2); // '2'

const json3 = JSON.stringify(['apple', 'kiwi']);
console.log(json3); // '["apple","kiwi"]'

const rabbit = {
  name: 'tori',
  color: 'white',
  size: null,
  birthDate: new Date(),
  jump: () => {
    console.log(`${name} can jump!`);
  },
};
const json4 = JSON.stringify(rabbit);
console.log(json4); // {"name":"tori","color":"white","size":null,"birthDate":"2022-09-06T12:59:25.462Z"}
json4.birthDate.getDate(); // Uncaught TypeError: Cannot read properties of undefined (reading 'getDate')

// parse
const parsed1 = JSON.parse(json4);
console.log(parsed1); // {name: 'tori', color: 'white', size: null, birthDate: '2022-09-06T12:59:25.462Z'}
```

- JSON 객체를 파싱하면, 벨류는 문자열이 된다.
  예를들어, 기존에 Date 타입이었던 벨류도 stringify, parse 과정을 거치면 string으로 나타난다. (위의 에시에서 json4.birthDate.getDate() 에서 타입에러가 발생한 이유이다) 이때 parse 메소드의 두 번째 인자인 콜백함수를 사용해 기존의 데이터 타입으로 만들어줄 수 있다.

```js
const json5 = JSON.parse(json4, (key, value) => {
  console.log(`key: ${key}, value: ${value}`);
  return key === 'birthDate' ? new Date(value) : value;
});

console.log(json5); // {name: 'tori', color: 'white', size: null, birthDate: Tue Sep 06 2022 21:59:25 GMT+0900 (Korean Standard Time)}
```
