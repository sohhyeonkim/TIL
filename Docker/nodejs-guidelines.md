<a href="https://docs.docker.com/language/nodejs/">참고 공식문서</a>

`이 문서는 도커를 사용해 노드 애플리케이션을 컨테이너화하는 가이드를 따라하며 번역한 문서입니다.`  

이 문서를 통해 다음 내용을 실습하면서 학습한다.

1. 노드 애플리케이션을 컨테이너화하고, 실행하기
2. 컨테이너들을 사용해 로컬 환경에서 노드 애플리케이션 개발 환경 구축하기
3. 컨테이너들을 사용해 노드 애플리케이션 테스트 실행하기
4. 깃헙 액션을 사용해서 컨테이너화된 노드 애플리케이션 CI/CD 파이프라인 구축하기
5. 컨테이너화된 노드 애플리케이션을 로컬에서 쿠버네티스에 배포하고, 디버깅하기

## 1. 노드 애플리케이션을 컨테이너화하고 실행하기

### 예제 프로젝트 클론하기
    ```js
    git clone https://github.com/docker/docker-nodejs-sample
    ```

### docker-nodejs-sample 디렉토리로 이동해서 `docker init` 실행하기

    커맨드 실행 결과, .dockerignore, Dockerfile, compose.yaml, README.Docker.md 파일이 생성된다.

    ```js
    docker init
    Welcome to the Docker Init CLI!

    This utility will walk you through creating the following files with sensible defaults for your project:
    - .dockerignore
    - Dockerfile
    - compose.yaml
    - README.Docker.md

    Let's get started!

    ? What application platform does your project use? Node
    ? What version of Node do you want to use? 18.0.0
    ? Which package manager do you want to use? npm
    ? What command do you want to use to start the app: node src/index.js
    ? What port does your server listen on? 3000
    ```

### 애플리케이션 실행하기 

    `docker compose up --build` 커맨드로 애플리케이션을 실행할 수 있다. `http://localhost:3000`로 접속해 애플리케이션을 확인할 수 있다. `ctrl+C`로 애플리케이션을 종료할 수 있다.

### 백그라운드에서 애플리케이션 실행하기

    `docker compose up --build -d` 커맨드로 애플리케이션을 백그라운드에서 실행할 수 있다. `docker compose down` 커맨드로 애플리케이션을 종료할 수 있다.


## 2. 컨테이너들을 사용해 로컬 환경에서 노드 애플리케이션 개발 환경 구축하기

이번 섹션에서는 컨테이너화된 애플리케이션의 개발 환경을 구축하는 방법을 설명한다.

- 로컬 데이터베이스를 만들고, 데이터 저장하기
- 개발 환경에서 실행되는 컨테이너 설정하기
- 컨테이너화된 애플리케이션 디버깅하기 

### 로컬 데이터베이스를 만들고, 데이터 저장하기

컨테이너를 사용해 데이터베이스 등을 로컬 환경에서 구축할 수 있다. 예제 프로젝트의 compose.yml 파일을 수정해서 데이터베이스 서비스와 데이터를 저장할 볼륨을 정의해보자.

`src/persistence/postgres.js` 파일을 보면, 애플리케이션에서 Postgres 데이터베이스를 사용하고, 연겨하기 위해 환경변수가 필요한 것을 알 수 있다. 현재 compose.yml 파일에는 이러한 변수들이 정의되어있지 않다. 따라서, compose.yml 파일을 다음과 같이 수정해보자.

- DB instructions 주석 제거
- server service 하위의 환경변수 추가
- server service의 데이터베이스 비밀번호 secrets 추가

```yml
services:
  server:
    build:
      context: .
    environment:
      NODE_ENV: production
      POSTGRES_HOST: db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD_FILE: /run/secrets/db-password
      POSTGRES_DB: example
    ports:
      - 3000:3000
    depends_on:
      db:
        condition: service_healthy
    secrets:
      - db-password
  db:
    image: postgres
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
```

위와 같이 파일을 수정했다면, compose.yml에서 secrets라는 것을 사용하고, 실제 데이터는 password.txt 파일에 저장되어있는 것을 알 수 있다. 따라서, 실제로 이 파일을 생성해주어야 한다.

db 디렉토리를 만들고, 그 하위에 password.txt 파일을 만들어서 임의의 비밀번호를 저장한다.

파일을 만들어줬다면 앞에서 했던 것과 동일하게 `docker compose up --build`를 실행할 수 있고, `ctrl+C`로 애플리케이션을 종료할 수 있다. 

`docker compose rm` 커맨드는 컨테이너를 삭제하는 명령어이고, 다시 `docker compose up --build`로 애플리케이션을 실행할 수 있다.

### 개발 환경에서 실행되는 컨테이너 설정하기

노드 개발환경에서 nodemon을 사용해서 코드 변경사항을 서버에 바로 반영하는 것처럼, 컨테이너 내부의 파일 시스템 상의 변경사항을 서버에 바로 반영할 수 있다. 이를 위해 compose.yml와 Dockerfile을 수정해보자. nodemon을 사용하려면 dev dependencies에 nodemon을 우선 추가해주어야 한다.

- Docerfile 수정

    운영과 개발용 Dockerfile을 분리하기보다, 하나의 도커 파일로 멀티 환경을 관리할 수 있다. 
    <a href="https://docs.docker.com/build/building/multi-stage/">멀티환경 더 알아보기</a> 

    ```Dockerfile
        # syntax=docker/dockerfile:1

        ARG NODE_VERSION=18.0.0

        FROM node:${NODE_VERSION}-alpine as base
        WORKDIR /usr/src/app
        EXPOSE 3000

        FROM base as dev
        RUN --mount=type=bind,source=package.json,target=package.json \
            --mount=type=bind,source=package-lock.json,target=package-lock.json \
            --mount=type=cache,target=/root/.npm \
            npm ci --include=dev
        USER node
        COPY . .
        CMD npm run dev

        FROM base as prod
        ENV NODE_ENV production
        RUN --mount=type=bind,source=package.json,target=package.json \
            --mount=type=bind,source=package-lock.json,target=package-lock.json \
            --mount=type=cache,target=/root/.npm \
            npm ci --omit=dev
        USER node
        COPY . .
        CMD node src/index.js
    ```
    이 도커 파일에서 우선, `FROM node:${NODE_VERSION}-alpine as base`을 통해, 다른 빌드 스테이지에서 이 빌드 스테이지를 참조할 수 있도록 한다. 그리고, `FROM base as dev`를 통해, dev 빌드 스테이지를 생성하고, 여기에 dev dependencies를 설치하고, `npm run dev`로 컨테이넝를 시작한다. `FROM base as prod`를 통해, prod 빌드 스테이지를 생성하고, 여기에서는 dev dependencies를 설치하지 않고 `node src/index.js`로 컨테이너를 시작한다.

- compose.yml 수정

    - dev 스테이지를 실행하기 위해서는 compose.yml 파일을 수정해야 한다. `target: dev`를 추가해주어서 멀티 스테이지 중 특정 스테이지를 선택해 빌드할 수 있도록 한다.

    - server service에 새로운 volume을 추가해서 로컬의 ./src 디렉토리를 컨테이너의 /usr/src/app/src 디렉토리로 마운트하도록 한다. 이렇게 하면, 로컬에서 코드를 수정하면 컨테이너 내부의 코드도 바로 수정되어 서버에 반영된다.

    - `9229:9229`를 추가해서 디버깅을 위한 포트를 열어준다.

    ```yml
    services:
    server:
        build:
        context: .
        target: dev
        environment:
        NODE_ENV: production
        POSTGRES_HOST: db
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD_FILE: /run/secrets/db-password
        POSTGRES_DB: example
        ports:
        - 3000:3000
        - 9229:9229
        depends_on:
        db:
            condition: service_healthy
        secrets:
        - db-password
        volumes:
        - ./src:/usr/src/app/src
    db:
        image: postgres
        restart: always
        user: postgres
        secrets:
        - db-password
        volumes:
        - db-data:/var/lib/postgresql/data
        environment:
        - POSTGRES_DB=example
        - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
        expose:
        - 5432
        healthcheck:
        test: [ "CMD", "pg_isready" ]
        interval: 10s
        timeout: 5s
        retries: 5
        volumes:
        db-data:
        secrets:
        db-password:
            file: db/password.txt
    ```

    - 개발 컨테이너 실행하고 애플리케이션 디버깅하기

    `docker compose up --build`를 실행하면, 로컬에서 생긴 소스코드 변경사항이 즉시 실행중인 컨테이너에 반영된다.

## 3. 컨테이너들을 사용해 노드 애플리케이션 테스트 실행하기

개발에서 테스트는 중요한 부분이다. 유닛 테스트, 통합 테스트, e2e 테스트 등 다양한 테스트가 있다. 이번 섹션에서는 도커 환경에서 유닛 테스트를 실행하는 방법을 알아보자.

### 로컬 개발환경에서 테스트 실행하기

예제 애플리케이션은 이미 Jest 패키지를 사용해 테스트를 실행할 수 있도록 설정되어있다. spec 디렉토리 하위에 이미 테스트 파일들이 있다. 로컬에서 개발할때, `compose`로 테스트를 실행할 수 있다. 

아래의 커맨드로 테스트 스크립트를 실행할 수 있다.

`docker compose run server npm run test`

- `docker compose run [OPTIONS] SERVICE [COMMAND] [ARGS...]`

    서비스에 대해 일회성 명령을 실행한다. `run`으로 실행된 명령어들은 service에 대해 정의된 설정들로 새로운 컨테이너에서 실행된다. `run`으로 실행된 명령어는 service에 대한 설정을 덮어쓴다. 그리고, 서비스 설정에 명시된 포트는 생성되지 않는다. 이는 이미 사용중인 포트와의 충돌을 방지하기 위함이다. 만약 서비스의 포트를 사용하고 싶다면, `--service-ports` 옵션을 명시적으로 사용하면 된다.(`docker compose run --service-ports web python manage.py shell`) 또는 `--publish` 또는 `-p` 옵션으로 포트를 지정할 수 있다. (`docker compose run --publish 8080:80 -p 2022:22 -p 127.0.0.1:2021:21 web python manage.py shell`)

### 빌드 시에 테스트 실행하기

빌드 시에 테스트를 진행하기 위해서는 Dockerfile에 새로운 스테이지를 추가해야한다.

```dockerfile
# syntax=docker/dockerfile:1

ARG NODE_VERSION=18.0.0

FROM node:${NODE_VERSION}-alpine as base
WORKDIR /usr/src/app
EXPOSE 3000

FROM base as dev
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci --include=dev
USER node
COPY . .
CMD npm run dev

FROM base as prod
ENV NODE_ENV production
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci --omit=dev
USER node
COPY . .
CMD node src/index.js

FROM base as test
ENV NODE_ENV test
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci --include=dev
USER node
COPY . .
RUN npm run test
```

테스트 스트에지에서는 `CMD` 대신 `RUN`을 사용한다. 그 이유는 `CMD`는 컨테이너가 실행될때 실행되는 명령어이고, `RUN`은 이미지를 빌드할때 실행되는 명령어이기 때문이다. 테스트가 실패하면 이미지 빌드가 실패하게 된다.

`docker build -t node-docker-image-test --progress=plain --no-cache --target test .`

위의 커맨드를 실행하면 테스트 스테이지를 타겟으로 새로운 이미지를 빌드하고, 테스트 결과를 확인할 수 있다. `--progress=plain`을 포함하면, 빌드 결과를 볼 수 있고, `--no-cache`를 포함하면, 테스트가 항상 실행되는 것을 보장한다. `--target test`를 포함하면, 테스트 스테이지를 타겟으로 빌드한다.

## 4. 깃헙 액션을 사용해서 컨테이너화된 노드 애플리케이션 CI/CD 파이프라인 구축하기

이번 섹션에서는 깃헙에 새로운 레포지토리를 만들고, 깃헙 액션의 workflow를 정의하고 실행해보자.

### 깃헙 레포지토리 만들고 시크릿 등록하기

레파지토리를 만들고 도커 허브 시크릿을 설정한 후, 소스코드를 푸시한다.

```js
1. 깃헙에 새로운 repository를 만든다.
2. repository 설정에서 secrets and variables > actions로 이동한다.
3. 새로운 시크릿 DOCKER_USERNAME을 만들고, DockerID를 벨류로 저장한다.
4. 도커 허브에서 사용할 Personal Access Token(PAT)를 만들고 `node-docker`로 이름을 설정한다.
5. repository의 시크릿으로 PAT를 추가한다. 이름은 DOCKERHUB_TOKEN으로 설정한다.
6. 로컬 repository에서 아래의 커맨드를 실행해 원격 repository와 연결한다.
  `git remote set-url origin https://github.com/your-username/your-repository.git`
7. 변경사항을 원격 repository에 푸시한다. `git push -u origin main`
```

### workflow 설정하기

깃헙 액션으로 빌드, 테스트, 도커 허브에 이미지 푸시하기 위해서는 workflow를 만들어야한다.

```js
1. repository의 actions 탭으로 이동한다.
2. `set up a workflow yourself`를 클릭한다.
  `.github/workflows/main.yml`의 workflow 파일이 자동으로 생성된다.
3. 아래의 코드를 복사해 yml 파일에 붙여넣는다.
```

```yml
name: ci

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v4
      -
        name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      -
        name: Build and test
        uses: docker/build-push-action@v5
        with:
          context: .
          target: test
          load: true
      -
        name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          target: prod
          tags: ${{ secrets.DOCKER_USERNAME }}/${{ github.event.repository.name }}:latest
```

### workflow 실행하기

- yml 파일 변경사항을 커밋하고 main 브랜치에 푸시한다. workflow가 자동으로 실행된다.
- actions 탭에서 workflow 실행을 확인할 수 있다.
- workflow가 성공적으로 실행되면 나의 도커 허브에 Repositoriesㅇ로 이동해서 이미지를 확인할 수 있다. 즉, 깃헙 액션이 도커 허브로 이미지를 성공적으로 푸시했다는 것을 알 수 있다.
