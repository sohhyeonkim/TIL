# 에러를 처리하는 두가지 방법
## throw
**throw**문은 사용자 정의 예외를 발생(throw)시킨다. throw 이후의 명령문은 실행되지 않는다. 제어 흐름은 콜스택의 첫 번째 catch 블록으로 전달된다. 호출자 함수 사이에 catch 블록이 없으면 프로그램이 종료된다.

```js
export const throwErrorFunc = () => {
  throw new Error('throw error!');
}

export const returnErrorFunc = () => {
  return new Error('return error!');
}

// try, catch가 없는 경우 
export const testFunc1 = () => {
  console.log('testfunc begins!')
  throwErrorFunc();
  console.log('testfunc is done!') // 실행되지 않음
}

// try, catch가 있는 경우 
export const testFunc2 = () => {
  try{
    console.log('testfunc begins!')
    throwErrorFunc();
    console.log('testfunc is done!') // 실행됨 
  } catch(e) {
    console.log('e: ', e) // 실행됨 
  }
  console.log('program is still working!'); // 실행됨 
}

testFunc1();
testFunc2();
```

## return
호출된 함수가 에러 객체를 리턴시키기 때문에 이 에러를 처리하기 위한 try,catch가 없어도 프로그램이 중단되지 않는다. 

```js
export const returnErrorFunc = () => {
  return new Error('return error!');
}

export const testFunc = () => {
  try{
    console.log('testfunc begins!');  
    const result = returnErrorFunc(); 
    console.log('result: ', result); // 실행됨 
    console.log('testfunc is done!'); // 실행됨 
  } catch(e) {
    console.log('e from catch: ', e); // 실행되지 않음
  }
}

testFunc();
```

## 배운 내용
service, controller 등으로 분리된 아키텍처에서 service에서는 에러를 리턴시키고 리턴된 에러를 controller에서 받아서 try문 내에서 에러 유형에 따라 분기를 나눠 에러를 핸들링한다. 따라서 depth가 깊어지는 경우, 각 단계별로 에러를 핸들링해주어야한다는 번거로움이 있다. 함수에서 에러를 throw하는 경우에는 이 함수를 호출한 함수의 catch문에서 분기를 나눠 에러를 핸들링한다. throw가 편리한 경우는, 함수의 depth가 깊어지더라도 가장 먼저 호출된 catch에서 에러를 잡을 수 있다는 점이다. 