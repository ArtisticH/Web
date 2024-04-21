## 생활코딩 - Webpack

```
파일 접속 시 정말 많은 파일들이 로드되고, 
서버와의 접속이 많을수록 애플리케이션은 느리게 로드된다.

또한 자바스크립트는 하나의 전역 객체를 공유하기 때문에 변수가 중복 선언될 수 있다.

* 번들러
묶는다.
여러 개의 파일을 묶어준다.
webpack, broserify, parcel...

하나의 자바스크립트 파일에
CSS, IMG와 같은 모듈들을 몰아넣을 수 있고,
다시 분리할 수도 있다. 
```

### 2. 웹팩이전의 세계와 모듈의 개념

```
폴더가 없는 파일 시스템에서 아비규환...

이 상황을 개선하기 위해 모듈 등장,
type="module", 확장자는 .mjs,
export와 import사용,
각자의 독자적인 모듈 스코프 => 전역 변수가 아니고 전역 객체의 프로퍼티가 아니다.

export와 import가 오래된 웹 브라우저에서 동작하지 않고,
여전히 여러 파일을 다운받아야 한다. 
```

### 3. 웹팩의 도입

```
번들러의 대표 주자 웹팩,

* 웹팩 설치
우선 프로젝트를 node.js의 패키지 프로젝트로 만들어야 한다.

- 현재 디렉토리를 node.js의 프로젝트 폴더로 선언
:npm init

- 웹팩 설치
:npm install -D webpack webpack-cli

- index.js을 만들어 진입 파일을 만든다. (애플리케이션의 입구, entry file)
```
```
* 웹팩을 이용해 index.js에서 사용하는 (from "...,js")파일들을 번들링한다.

- 프로젝트 폴더에 설치한 웹팩 실행,
index.js을 ./public/index_bindle.js에 출력

:npx webpack --entry ./source/index.js --output ./public/index_bindle.js
(index.js와 그 안에서 사용하는 파일 경로 주의)

- 그리고 index.html에 <script src="./public/index_bindle.js">

그리고 파일이 index_bindle.js만 다운로드된다.
한번만 커넥션이 이루어지지만, 모든 파일이 다 들어가있음.
import, export가 오래된 브라우저에서도 동작한다. 
```

### 4. 설정파일 도입

```
여러 자원들(.js, .css, .png) => 웹팩 => 단순한 형태의 몇몇 파일들

1. 어떻게 여러 자원들(.js, .css, .png)을 웹팩에 input시키는가?
2. 어떤 방법으로 가공(process)?
3. 결과(output)
```
```
* 재사용 + 명령을 위한 설정 파일 도입

- 설정 파일 생성
webpack.config.js

- 웹팩 홈페이지 configuration 섹션에 들어가 webpack.config.js예제 파일 참조
- module.exports = {
    entry: "index.js",
    output: {
      path: path.resolve(__dirname, "public"), // public 폴더에
      filename: "index_bundle.js"
    }
- npx webpack --config webpack.config.js // 여기 적혀있는 대로 동작

webpack.config.js 으로 이름을 정했다면,
npx webpack만 해도 알아서 번들링해준다.
```
```
두 가지 방법

1. npx webpack --entry ./source/index.js --output ./public/index_bindle.js
2. webpack.config.js 생성 후 npx webpack --config webpack.config.js
```

### 5. 모드의 도입

```
웹팩의 기본값: 개발 모드와 배포 모드(기본값), 아무것도 세팅하지 않는 모드

번들러 파일은 용량을 줄이기 위해 변수를 알아보기 힘들다.

- mode: "development"로 설정하면,
이전보단 인간적인 코드로 바뀌었다.

- webpack.config.prod.js 파일을 하나 더 만들어
..mode: "production"으로 설정,
npx webpack --config webpack.config.prod.js 로 개발 모드로 번들링

- 아니면 하나의 파일을 만들어 환경 변수를 이용해 모드가 스위칭되게끔
```

### 6. 로더의 도입

```
웹팩의 핵심,


```


