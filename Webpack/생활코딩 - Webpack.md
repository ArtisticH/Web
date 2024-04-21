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
    entry: "./source/index.js",
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

js번들러 파일과 별개로
css파일이 별도로 다운로드될때,
js안에 css파일까지 넣으면 좋겠다

웹팩을 잘 다룬다 = 로더를 얼마나 잘 다루는가?
```
```
- npm install --save-dev(-D) style-loader css-loader
- webpack.config.js에서
module: {
    rules: [..

test: /\.css$/, // 확장자가 .css로 끝나는 파일 처리
use: ['css-loader']

웹팩 동작 시 확장자가 .css인 파일을 만나면
웹팩이 알아서 "웹팩 안으로 로드" 시켜주는 특수한 명령

- index.html에서 <link..>지우고
index.js안에
import css from './style.css'
style.css를 source폴더 하위로 옮겨야 한다.

- 웹팩이 index.js를 보다가
확장자가 css인 파일을 설정 파일에 따라 css-loader가 .css파일을 css변수에 세팅해준다.
"import css from './style.css'"

console.log(css)를 보면
css내용이 자바스크립트 형태로 웹팩에 주입되었다. 
```
```
use: ['style-loader', 'css-loader']

css-loader: .css파일을 읽어 웹팩으로 가져오는 놈
style-loader: 가져온 css코드를 웹페이지 안에 스타일 태그<style></style>로 주입해주는 로더

서버에서 index_bindle.js만 읽어왔음에도
css가 웹페이지에 삽입되었다.

개발자 도구의
elements의
<head>의 <style>에 삽입되었다.

- 로더: 입력한 assets이 로더를 통과하면
로더가 자산들을 가공해
우리가 원하는 output으로 만들어준다.

- test:에 맞는 파일 발견시
설정한 로더들을 통과시켜라.
- 뒤쪽에 있는 로더가 먼저 실행된다. 
```

### 7. output설정
```
index.html말고 about.html파일이 있을때,
about_bundle.js을 만들고 싶다.
새로운 entry: about.js

entry: {
    index: "./source/index.js",
    about: "./source/about.js",
}

output: {
    path: path.resolve(__dirname, "public"), // public 폴더에
    filename: "[name]_bundle.js"
}
```

### 8. 플러그인의 도입
```
확장 기능
1. 로더: 최종 결과물을 만들어가기 위해 그 과정에 쓰인다.
2. 플러그인: 최종 결과물을 변형한다. 

로더는 규정되어 있고,
플러그인은 보다 자유롭다.
플러그인마다 사용방법이 다르다.
```
```
- HtmlWebpackPlugin 사용할거임
html파일을 자동으로 생성하고 싶다?
html파일을 템플릿으로 사용하고 싶다?

npm insatll -D html-webpack-plugin

- 두개의 Html파일을 source폴더에 옮기고
html파일 내의 <script "..bundie">삭제
public폴더 안에 최종적으로 최종적으로 완성된 Html 생성

- 플러그인
require()로 변수에 할당하고,
plugins: [new HtmlWebpackPlugin()]

- rm ./public/*.*; npx webpack // 기존의 public에 있던 두 개의 bundle.js파일들 삭제
- public에 이전에 없던 index.html 파일 생성됌,
그리고 번들된 자바스크립트 파일이 자동으로 script에 삽입됌
- 근데 아직 템플릿화 안됌
```
```
- plugins: [new HtmlWebpackPlugin({
    template: './source/index.html',
    filename: './index.html', // 최종 파일명
    chunks: ['index'], // index_bundle.js만 삽입
})]

- plugins: [
    new HtmlWebpackPlugin({
        template: './source/index.html',
        filename: './index.html', // 최종 파일명
        chunks: ['index'], // index_bundle.js만 삽입
    }),
    new HtmlWebpackPlugin({
        template: './source/about.html',
        filename: './about.html', // 최종 파일명
        chunks: ['about'], // about_bundle.js만 삽입
    }),
] // public에 index.html과 about.html 모두 생성
```



### 9. 선물
```
npx webpack대신
npx webpack --watch
: 파일의 변경을 감지해 자동으로 컴파일시켜준다.
```

### 10. npm 패키지 사용
```
npm패키지를 application으로 가져올때 웹팩을 사용하는 방법

- lodash 라이브러리 설치: npm i lodash
- index.js안에
import _ from "lodash";
웹팩이 node_modules에서 lodash를 찾아 _를 준다.
```

