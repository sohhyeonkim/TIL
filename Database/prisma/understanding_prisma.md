## Prisma란?

간단하게 말하자면, Prisma는 schema.prisma에 정의한 객체를 데이터베이스의 테이블에 맵핑해주는 ORM이다.  

## Schema

### schema.prisma 파일 생성하기

Schema는 Prisma가 데이터베이스에 매핑될때 필요한 정보들을 담고 있는 파일이다. 아래의 `npx prisma init` 명령어를 실행하면 schema.prisma 파일이 생성된다.

```js
// add Prisma cli as dev dependency
npm install prisma typescript ts-node @types/node --save-dev

// can invoke Prisma cli by prefixing it with npx:
npx prisma

// initialize Prisma
npx prisma init 
-> will create a new direcotry called prisma
-> inside prisma direcotry, schema.prisma file will be created
-> .env. file will be created in the root direcotry of the project
```

### schema.prisma 구성

```js
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id            String   @id @default(cuid())
  email         String   @unique                
  password      String   @db.VarChar(500)
  nickName      String   @unique
  countryCode   String
  phoneNumber   String
  role          Role     @default(USER)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  deletedAt     DateTime?

  @@unique([countryCode, phoneNumber])
}

enum Role {
  USER
  ARTIST
  AGENCY
  ADMIN
}
```

Prisma가 연결할 데이터베이스의 호스트 및 계정정보, 연결된 데이터베이스의 구조와 테이블, 컬럼 정보 등이 담겨있다. 

각 정보들은 블록 단위로 정의되는데, PSL(Prisma Schema Language)로 작성된다.

- datasource

    datasource 블록은 매핑할 DB의 정보를 작성한다. url은 `connector://user:password@host:port/db-name` 형식으로 작성되어야한다. 자동으로 생성된 .env 파일에서 DATABASE_URL을 자동으로 가져온다. 만약, .env.local 또는 .env.development 등 다른 이름의 파일을 사용하고 싶다면, <a href="https://www.prisma.io/docs/orm/more/development-environment/environment-variables/using-multiple-env-files">Using multiple .env files</a>를 참고하면 된다. provider에는 호환할 DB 타입(mysql, postgreSQL, SQLite 등)을 작성한다. 

- generator
    generator는 client를 생성하는 요소이다. client에는 schema 파일에 정의된 모든 model을 객체로 저장되어있고, 각 모델 객체에는 우리가 SQL을 대체해 쉽게 데이터를 CRUD할 수 있는 메서드가 내장되어있다.

## Migration

Schema 파일에 정의된 모델들을 데이터베이스에 매핑하기 위해서는 migration이 필요하다. 
package.json에 아래와 같은 명령어를 추가해서 환경 별로 마이그레이션을 진행할 수 있다. 

- `npx prisma migrate dev --name`

    ```js
    "mg:local": "dotenv -e .env.local -- npx prisma migrate dev --name",
    "mg:dev": "dotenv -e .env.development -- npx prisma migrate dev --name",
    "mg:prod": "dotenv -e .env.production -- npx prisma migrate deploy --name"
    ```

    각 명령어를 실행하면 prisma/migrations 폴더에 마이그레이션 파일이 생성되고, 데이터베이스에 적용된다. 그리고, 무엇보다 `npx prisma generate` 명령어가 자동으로 실행되어 매번 수동으로 실행해줄 필요가 없다. 이는 내부적으로 schema 파일에 정의된 prisma-client-js generator를 찾고, 만약 정의되어 있다면, @prisma/client가 설치되어 있는지 확인하고, 없다면 자동으로 설치해주기 때문!

- `npx prisma db pull`

    데이터베이스의 구조를 schema.prisma에 반영할 수 있다.

- `npx prisma db push`

    schema.prisma의 구조를 마이그레이션 파일 생성 없이 데이터베이스에 반영할 수 있다. `npx prisma migrate`와 마찬가지로 Prisma client를 자동으로 생성해준다.

- `npx prisma generate`
    <img src="../images/prisma_generate.png"/>

    schema.prisma에 정의된 모델들을 기반으로 client를 생성한다. schema.prisma를 변경한 이후에는 `npx prisma generate` 명령어를 실행해서 client를 업데이트해줘야한다. 그래야지 애플리케이션에서 업데이트된 스키마에 맞는 데이터를 CRUD할 수 있다.

> reference

- <a href="https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-mysql">공식문서</a>

- <a href="https://medium.com/@olafdev/prisma-schema-%ED%8E%B8-2401c9609ae2">Prisma-schema편</a>

- <a href="https://www.daleseo.com/prisma/">Prisma 처음 시작하기</a>
