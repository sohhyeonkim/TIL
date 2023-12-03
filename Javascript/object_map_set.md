## Object

- key 타입은 string, symbol만 가능
- Ojbect 객체는 키:값 쌍을 저장하는 데이터 형태이다.
- Object는 iteration protocol을 구현하지 않기 때문에 로 개체는 for...of문을 사용하여 직접적으로 반복할 수 없습니다.
- 입력 순서가 보장되지 않는다.
    ```js
    const obj = {
        7: 'e',
        1: 'a',
        2: 'b',
        3: 'c',
        4: 'd',
    }
    Object.keys(obj) // ["1", "2", "3", "4", "7"]
    Object.values(obj) // ["a", "b", "c", "d", "e"] 
    ```

## Map

- key 타입은 모든 타입 가능 (Object는 string, symbol만 가능하다는 점에서, key의 타입이 다양할때, Map을 사용한다.)
- Map 객체는 키:값 쌍을 저장하는 데이터 형태이다.
- Map은 순회가능(iterable)하기 때문에 이므로 for...of, forEach 등의 반복문을 사용할 수 있다.
- Map 객체는 입력 순서가 보장된다. (Object는 입력 순서가 보장되지 않으므로, 입력된 key의 순서가 중요할때 Map을 사용한다.)
- Object.entries()로 객체를 Map으로 변환할 수 있고, Object.fromEntries()로 Map을 객체로 변환할 수 있다.
    ```js
    // 각 요소가 [키, 값] 쌍인 배열을 가지는 이터러블 객체를 전달하여 Map을 생성할 수 있다.
    const map1 = new Map([
        ['1', 'a'],
        [1, 'b'],
        [true, 'c'],
    ]);
    map1.set(7, 'd').set('7', 'e').set(8, 'f'); // chaining 가능
    let obj1 = {
        name: "John",
        age: 30
    };
    const params = Object.entries(obj1); // [ ["name","John"], ["age", 30] ]
    const map2 = new Map(params);

    // Map을 객체로 변환할 수 있다.
    const obj2 = Object.fromEntries(map2); // { name: "John", age: 30 }
    ```

## Set

- Set은 중복되지 않는 값으로만 이루어진 데이터 형태이다.
- Set 객체는 값의 모음이다.
- Set 객체는 순회가능(iterable)하기 때문에 이므로 for...of, forEach 등의 반복문을 사용할 수 있다.
- Set 객체는 입력 순서가 보장된다.
- has 메서드는 값이 존재하는지 확인하는데, 평균적으로 대부분의 방법보다 빠르다. 예를 들면, Set의 size와 배열의 길이가 같을때, Array.prototype.includes 메서드보다 has 메서드가 더 빠르다.
    ```js
    // 이터러블 객체를 전달받으면(대개 배열을 전달받음) 그 안의 값을 복사해 셋에 넣어준다.
    const set1 = new Set([1, 2, 3, 4, 5]);
    const set2 = new Set();
    set2.add(1).add(2).add(3).add(4).add(5); // chaining 가능
    ```