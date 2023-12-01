# regular expression 

## lastIndex

정규표현식을 만들때, global 속성을 입력하면, 정규표현식의 lastIndex라는 속성값이 0으로 초기화된다. lastIndex는 read/write 가능한 정수값으로, RegExp.exec()와 RegExp.test()를 실행할때마다 찾아낸 매치의 마지막 인덱스값으로 갱신된다. exec()와 test() 메서드는 lastIndex 속성을 실행할 시작 인덱스로 사용한다. 만약 두번째 실행한 exec()와 test()에서 조건에 만족하는 매치를 찾지 못한 경우, 0으로 다시 초기화된다.

```js
const str = 'table football';

const globalRegex = new RegExp('foo*', 'g'); // "g"가 명시된 정규표현식

console.log(globalRegex.lastIndex); // lastIndex는 0으로 초기화됨
// Expected output: 0

console.log(globalRegex.test(str)); // test() 메서드를 1회 실행한 결과
// Expected output: true

console.log(globalRegex.lastIndex); // lastIndex는 9로 변경되었고
// Expected output: 9

console.log(globalRegex.test(str)); // str의 9번째 인덱스부터 실행되어 더이상 foo*와 일치하는 매치를 찾지 못했으므로
// Expected output: false

console.log(globalRegex.lastIndex) // lastIndex는 0으로 초기화됨
// Expected output: 0
```

> reference

- <a href="https://www.tutorialspoint.com/javascript/regexp_lastindex.htm">RegExp lastIndex Property</a>
- <a href="https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test">RegExp.prototype.test()</a>

## 전후방탐색
패턴에 매칭되는 문자가 아니라, 패턴에 매칭되는 문자의 앞, 뒤를 찾을 때 사용한다.

- 전방탐색 (lookahead): (?=)
    등호 뒤에 오는 패턴 앞의 문자만 매칭된다.

    ```js
    1. 이메일에서 @ 앞의 문자를 찾고 싶을때
    .*(?=@) // @ 앞의 모든 문자를 찾는다.

    2. 가격에서 '원'을 제외한 금액만 찾고 싶을때
    .+(?=원) // '원' 앞의 모든 문자를 찾는다.
    ```
    
- 후방탐색 (lookbehind): (?<=)
    등호 뒤에 오는 패턴 뒤의 문자만 매칭된다.

    ```js
    1. '$'뒤의 금액만 찾고 싶을때
    (?<=\$)[0-9.]+ // '$' 뒤의 모든 문자를 찾는다.

    ABC01: $23.45
    HGG42: $5.31
    CFMX1: $899.00
    XTC99: $69.96
    Total items found: 4
    
    str.match(/(?<=\$)[0-9.]+/g,function(m){
        return m
    });
    // ['23.45', '5.31', '899.00', '69.96']
    ```