## Swagger Plugin 사용하기

swagger를 사용하면 API 명세서를 보기 쉽게 작성할 수 있다. 하지만, swagger를 사용하다보면 DTO 관련해서 많은 코드 중복이 발생한다. 이런 문제를 해결하기 위해 swagger plugin을 사용할 수 있다.

### Swagger plugin 설정

- nest-cli.json파일에 아래와 같이 플러그인을 추가한다.

    ```js
    {
    "collection": "@nestjs/schematics",
    "sourceRoot": "src",
    "compilerOptions": {
        "plugins": ["@nestjs/swagger"] // 추가된 부분
    }
    }
    ```
- 필요에 따라 options를 설정한다.
    ```js
    "plugins": [
        {
            "name": "@nestjs/swagger",
            "options": {
            "classValidatorShim": false,
            "introspectComments": true
            }
        }
    ]

    export interface PluginOptions {
        dtoFileNameSuffix?: string[];
        controllerFileNameSuffix?: string[];
        classValidatorShim?: boolean;
        dtoKeyOfComment?: string;
        controllerKeyOfComment?: string;
        introspectComments?: boolean;
    }
    ```

    - dtoFileNameSuffix
        - default: ['.dto.ts', '.entity.ts']
        - DTO 파일의 이름에 붙는 접미사를 설정한다.

    - controllerFileNameSuffix
        - default: ['.controller.ts']
        - Controller 파일의 이름에 붙는 접미사를 설정한다.

    - classValidatorShim
        - default: true
        - true로 설정된 경우, 모듈은 class-validator 라이브러리의 데코레이터를 재사용한다.

    - dtoKeyOfComment
        - default: 'description'
        - `ApiProperty`에 설명을 작성하기 위한 키를 설정한다.

    - controllerKeyOfComment
        - default: 'description'
        - `ApiOperation`에 설명을 작성하기 위한 키를 설정한다.

    - introspectComments
        - default: false
        - true로 설정된 경우, 플러그인이 코멘트를 기반으로 설명과 예제를 생성한다.

- plugin 옵션을 수정할 경우, /dist 폴더를 삭제하고, 애플리케이션을 다시 빌드해야한다!
    
    package.json에 로컬에서 서버를 실행할때, swagger 옵션에 변경사항이 있을 수도 있으므로  서버를 실행하기 전에 rimraf를 사용해서 dist 폴더를 삭제하고, nest build를 실행하도록 설정했다.

    ```js
    "build:clean": "rimraf dist && nest build",
    "start:local": "NODE_ENV=local && npm run build:clean && nest start --watch",
    ```