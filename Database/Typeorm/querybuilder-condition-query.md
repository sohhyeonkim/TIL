## querybuilder를 사용해서 쿼리 조건 만들기

아래의 예시는 쿼리 파라미터로 입력받은 값들이 있다면, 그 값들로 필터링한 결과를 조회하는 함수이다. 데이터를 필터링해서 조회하는 API를 구현하면서
찾아보며 사용한 내용들을 정리해보았다.

- 쿼리 파라미터로 전달받은 값이 있는 경우 (없다면 undefined), orWhere로 조건문을 추가할 수 있다.

- 각 조건문은 파라미터가 있다면, 전달받은 값과 일치할때, 포함할때, 범위가 이상,이하 일때의 조건을 or로 연결한다.

- take는 페이지네이션을 적용할때 limit의 개념으로, 한 번에 조회해올 데이터의 개수이며, skip은 offset의 개념으로 건너뛸 데이터의 개수이다.

```js
export interface DateRange {
  begin: string; // 2022-10-01
  end: string; // 2022-10-31
}

export interface QueryCondition {
  name?: string;
  email?: string;
  created_at?: DateRange;
  take: number;
  skip: number;
}

public async findOneByUserId(query: QueryCondition) {
  const { name, email, created_at, take, skip } = query;

  const qb = this.repository
                 .createQueryBuilder('user')
                 .leftJoinAndSelect('user.manager_assignment', 'manager_assignment')

  if(name) qb.orWhere('user.name = :name', {name}) // 일치하는 경우
  if(email) qb.orWhere('user.email LIKE :email', {email: `%${email}$`}) // 포함하는 경우
  if(created_at) {
    const { begin, end } = created_at;
    const beginDataTyped = new Date(begin).slice(0, 10);
    const endDataTyped = new Date(end).slice(0, 10);
    qb.orWhere(`DATE_FORMAT(user.created_at, '%Y-%m-%d') BETWEEN :beginDataTyped AND :endDataTyped`, {beginDataTyped, endDataTyped});
  }

  return qb.take(take).skip(skip).getManyAndCount();
}
```
