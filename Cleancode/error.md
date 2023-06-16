# 에러 이해하기

## Types of Error
Operational Error
실행 에러는 노드의 런타임에서 발생한다. 

- failed to connect to server
- failed to resolve hostname
- invalid user input
- request timeout
- server returned a 500 response
- socket hang-up
- system is out of memory

Programmer Errors
프로그래머 에러는 버그라고 부른다. 이는 작성한 코드에 문제가 있을때 발생한다.
- called an asynchronous function without callback
- did not resolve a promise
- passed a string where an object was expected
- passed an object where a string was expected
- passed incorrect parameters in a function

## Error object

노드에는 내장 객체가 있다. 에러 객체는 name, message, stack 등의 정보를 갖는다.
```js
const error = new Error('hello');
console.log(error);
Error: hello // 인자로 전달한 메시지
    at Object.<anonymous> (/Users/test/error.js:1:15)
    at Module._compile (node:internal/modules/cjs/loader:1105:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1159:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:77:12)
    at node:internal/main/run_main_module:17:47
```

## best practices

노드에서 에러를 다루기 위한 best practices는 다음과 같다. 

### custom error 사용하기
custom error를 생성하기 앞서 async/await 패턴을 사용하면 비동기적으로 동작하는 코드를 동기적으로 작성할 수 있다.
custom error는 노드에서 기본으로 제공하는 Error 객체를 확장해서 BaseError 객체를 생성하고, 이를 확장해서 다양한 에러들을 생성할 수 있다.  

```js
const anAsyncTask = async() => {
  try{
    const user =  await getUser();
    const cart = await getCart(user);
  } catch(error) {
    console.error(error);
  } finally {
    await cleanUp();
  }
} 
```
custom erorr는 다음과 같은 방법으로 생성할 수 있다.

```js
// baseError.js
class BaseError extends Error {
  constructor(name, statusCode, isOperational, description) {
    super(description);

    Obejct.setPrototypeOf(this, new.target.prototype)
    this.name = name;
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    Error.captureStackTrace(this);
  }
}

module.exports = BaseError

// httpStatusCodes.js
const httpStatusCodes = {
  OK: 200,
  BAD_REQUEST: 400,
  NOT_FOUND: 404,
  INTERNAL_SERVER: 500
}
module.exports = httpStatusCodes

// api404Error.js
const httpStatusCodes = require("./httpStatusCode");
const BaseError = requrie("./baseError");

class Api404Error extends BaseError {
  constructor(
    name,
    statusCode = httpStatusCodes.NOT_FOUND,
    description = 'Not Found',
    isOperational = true
  ) {
    super(name, statusCode, isOperational, description)
  }
}
module.exports = Api404Error;

// usecase
const user = await User.getUSerById(req.params.id);
if(user === null) {
  throw new Api404Error(`User with id: ${req.params.id} not found`);
} 
```
<hr>

### middleware 사용하기

모든 에러들을 catch하는 미들웨어를 만들 수 있다. express 프레임워크를 사용한다면 next 함수를 사용해서 발생한 모든 에러들을 미들웨어로 보낼 수 있다. 단, 라우트 파일의 가장 끝에 .use() 메소드를 사용해서 에러를 전달받았을때 실행할 코드를 작성한다.

```js
app.post("/user", async(req, res, next) => {
  try {
    const newUser = User.create(req.body);
  } catch(error) {
    next(error);
  }
});
```
<hr>

### Programmer error를 처리하기 위해 앱 재실행하기

Programmer error는 애플리케이션의 메모리 누수나 높은 CPU 사용 등의 이슈를 일으킬 수 있다. 가장 최선의 해결책은 애플리케이션을 재실행하는 것이다. 

<hr>

### Catch all Uncaught Exceptions

예상치 못한 에러들이 발생했을때, 의도하지 않은 일이 발생하지 않도록 알림을 보내거나 애플리케이션을 재실행하도록 처리할 수 있다. 

```js
// errorHandler.js
function logError (err) {
 console.error(err)
}

function logErrorMiddleware (err, req, res, next) {
 logError(err)
 next(err)
}

function returnError (err, req, res, next) {
 res.status(err.statusCode || 500).send(err.message)
}

function isOperationalError(error) {
 if (error instanceof BaseError) {
 return error.isOperational
 }
 return false
}

module.exports = {
 logError,
 logErrorMiddleware,
 returnError,
 isOperationalError
}

const { logError, isOperationalError } = require("./errorHandler");
process.on("uncaughtException", error => {
  console.error(error);

  if(!isOperationalError(error)) {
    process.exit(1);
  }
})
```

<hr>

### Error 알림 설정 및 Logs 저장하기

구조화된 로깅의 핵심은 에러를 포맷팅해서 로그 관리 툴에 보관하는 것이다. 이를 위해서는 winston, morgan 등의 로거를 사용하고, 이를 생성된 에러를 저장, 관리하는 툴로 Sematext, Sentry 등의 툴을 사용할 수 있다.

### How to Deliver Errors: Function Patterns

다음은 노드에서 에러를 전송하는 방법들이다.

- throw the error (making it an exception)
- pass the error to a callback, a function provided specifically for handling errors and the results of ansynchronous operations
- pass the error to reject Promise function
- emit an "error" event on an EventEmitter

각각의 방법들을 하나씩 살펴보자. 

