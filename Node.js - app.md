# Node.js - app.js

📍 **`express.static`의 역할**
```node.js
app.use('요청 경로', express.static('실제 경로'));

app.use(express.static(path.join(__dirname, 'public')));
app.use('/img', express.static(path.join(__dirname, 'uploads')));
```
- static 미들웨어는 정적인 파일들을 **제공**하는 라우터 역할. `public/css/style.css`는 `http://localhost:8080/css/style.css`로 접근할 수 있다.
- public 폴더를 만들고 css나 js, 이미지 파일들을 public폴더에 넣으면 **브라우저에서 접근**할 수 있다.
- 서버의 폴더 경로와 요청 경로가 다르므로(public노출여부) 외부인이 서버의 구조를 쉽게 파악할 수 없고 이는 보안에 도움이 된다.
- 정적 파일들을 알아서 제공해주므로 `fs.readFile`로 파일을 직접 읽어서 전송할 필요가 없다.
- 만약 요청 경로에 해당하는 파일이 없으면 알아서 내부적으로 `next`를 호출한다.
- 만약 파일을 발견했다면 다음 미들웨어는 실행되지 않는다. 응답으로 파일을 보내고 next를 호출하지 않는다.
- `app.use('/img', express.static(path.join(__dirname, 'uploads')));`는 예를 들어 `uploads` 파일에 `image.jpeg` 파일이 있다면 브라우저에서 `uploads/image.jpeg`을 `http://localhost:8080/img/image.jpeg`로 접근할 수 있다.

📍 **`body-parser`의 역할**
```node.js
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
```
- 요청의 본문에 있는 데이터를 해석해서 `req.body`객체로 만들어주는 미들웨어
- 보통 폼 데이터나 AJAX 요청의 데이터를 처리한다.
- 단 멀티파트(동영상, 이미지, 파일) 데이터는 multer 모듈을 사용해야 한다.
- 익스프레스 4.16.0 버전부터 body-parser 미들웨어의 일부 기능이 익스프레스에 내장되었다.
- 단, Raw(버퍼 데이터), Text(텍스트 데이터) 형식의 데이터를 해석할땐 body-parser를 설치해야 한다.
```node.js
const bodyParser = require('body-parser');
app.use(bodyParser.raw());
app.use(bodyParser.text());
```
- URL-encoded는 주소 형식으로 데이터를 보내는 방식이다. 폼 전송은 이 방식을 사용한다. 
- `extended: false`는 false 라면 노드의 querystring 모듈을 사용해 쿼리스트링을 해석하고, true면 qs모듈을 사용해 쿼리스트링을 해석한다. qs는 내장 모듈이 아니라 npm패키지이며, querystring 모듈의 기능을 좀 더 확장한 모듈이다.
- 요청의 본문을 전달받을때 req.on('data')와 req.on('end')로 스트림을 사용할 필요 없이, 내부적으로 스트림을 처리해 req.body에 추가한다.
- URL-encoded 형식은 데이터를 전송하거나 저장할 때 사용되는 데이터 인코딩 방식 중 하나이다. 예를 들어 스페이스 문자를 %20으로 인코딩해서 데이터를 전달한다.
- URL-encoded 데이터는 URL의 **쿼리 문자열** 부분에서 매개변수를 전달하는데도 사용된다. 예를 들어 `http://example.com/search?q=keyword`에서 `q=keyword`

📍 **`app`, `app.set`, `app.get`, `app.use`**
```node.js
const express = require('express');
const app = express();
app.set('port', process.env.PORT || 3000);

app.get('/', (req, res) => {
  res.send('Hello');
});

app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기 중');
});
```
- express 모듈을 실행해 이를 app 변수에 할당한다. express내부에 **http 모듈이 내장**되어 있으므로 **서버**의 역할을 할 수 있다.
- `express()` 함수 내부적으로 `createApplication`이라는 함수를 호출해 새로운 애플리케이션 객체를 생성하고, 이 객체는 요청 및 응답 처리, 미들웨어 관리, 라우팅 등을 위한 다양한 메서드와 속성을 포함한다. 
- 요청을 처리하기 위한 라우팅 시스템이 설정된다. `app.get()`, `app.post()`, `app.use()`등의 메서드를 통해 라우트를 정의할 수 있다.
- Express는 요청을 처리하기 위해 미들웨어와 라우트를 **스택 형태**로 관리한다. 각 미들웨어와 라우트는 특정 경로 및 HTTP 메서드와 연결된다.
- `app.set(키, 값)`을 통해 데이터를 저장하고 `app.get(키)`로 가져올 수 있다.
- `app.get(주소, 라우터)`, app.post, app.put, app.patch 등의 메서드가 있다. 

📍 **`미들웨어, 라우터`**
* **미들웨어**
  * 요청과 응답의 중간에 위치하여, 요청과 응답 객체를 조작해 다음 미들웨어 함수 또는 라우트로 제어를 전달하는 함수
  * `app.use(미들웨어)`꼴이다. `app.use(미들웨어)`에 `req, res, next`인 함수를 넣는다.
  * 미들웨어는 위에서부터 아래로 순서대로 실행되면서 요청과 응답 사이에 특별한 기능을 추가할 수 있다.
  * `next`를 실행하지 않으면 다음 미들웨어가 실행되지 않는다.
  * 요청 처리 전/후 작업: 인증, 로깅, 파싱, 에러 핸들링 등
  * 요청과 응답 객체의 조작: 요청 본문 파싱, 쿠키 읽기/쓰기, 사용자 세션 관리 등
  * 흐름 제어: 다음 미들웨어 또는 라우트로의 제어 전달
  * 전역적이거나 특정 경로에 대해 적용될 수 있다.
  * 같은 라우터에 미들웨어를 여러 개 장착할 수 있다. 이때도 `next`를 호출해야 다음 미들웨어로 넘어갈 수 있다.
  * 에러 처리 미들웨어는 매개변수가 `err, req, res, next`로 반드시 네 개이다. 특별한 경우가 아니면 가장 아래에 위치하도록 한다.
  * 대부분의 패키지는 미들웨어이고, app.use에 연결해 사용한다. `req, res, next`는 내부에 들어있고, next도 내부적으로 호출하기에 다음 미들웨어로 넘어갈 수 있다. 

```node.js
app.use(미들웨어) // 모든 요청에서 실행되는 전역 미들웨어
app.use('/abc', 미들웨어) // abc로 시작하는 요청에서 미들웨어 실행
app.post('/abc', 미들웨어) // abc로 시작하는 POST 요청에서 미들웨어 실행
```
* **라우터**
  * 특정 경로와 HTTP메서드에 대해 요청을 처리하는 함수 또는 함수 집합.
  * 경로 기반 요청 처리: 요청 경로에 따라 다른 핸들러 함수로 요청을 처리
  * 경로 정의 및 처리: 특정 경로와 HTTP메서드에 대한 요청을 처리
  * 일반적으로 `next`를 호출하지 않는다.
```node.js
// GET 요청에 대해 응답하는 라우터
app.get('/', (req, res) => {
  res.send('Hello World!');
});

// POST 요청에 대해 응답하는 라우터
app.post('/submit', (req, res) => {
  res.send('Form submitted!');
});
```

📍 **`미들웨어의 특성`**
- 미들웨어는 `req, res, next`를 매개변수로 가지는 함수로서 `app.use`나 `app.get` 등으로 장착한다.
- 여러 개의 미들웨어를 동시에 `app.use(, , , ...)`안에 장착할 수도 있다. 보통 다음 미들웨어로 넘어가려면 `next`함수를 호출해야 하지만, 패키지들은 내부적으로 next를 호출하고 있으므로 연달아 쓸 수 있다.
- next로 다음 미들웨어를 호출하지 않는 경우 `res.send`나 `res.sendFile` 등의 메서드로 응답해야 한다.
- `express.static`같은 미들웨어는 next 대신 `res.sendFile`로 응답한다.
- `next('router')`는 다음 라우터의 미들웨어로, 그 외의 인수를 넣는다면, 예를 들면 `next(error)`는 에러 핸들러로, 이때 error인수는 에러 처리 미들웨어의 err 매개변수가 된다.
- 미들웨어 간에 데이터를 전달할 수도 있다. 세션을 사용하면 `req.session`에 넣어 세션이 유지되는 동안 데이터도 계속 유지되지만, 요청이 끝날 때까지만 데이터를 유지하고 싶다면 `req`객체에 데이터를 넣어둔다.
```node.js
// 미들웨어 안에 미들웨어 넣기
app.use((req, res, next) => {
  morgan('dev')(req, res, next);
});

app.use((req, res, next) => {
  if(process.env.NODE_ENV === 'production') {
    morgan('combined')(req, res, next);
  } else {
    morgan('dev')(req, res, next);
  }
});
```
- 위 패턴은 기존 미들웨어의 기능을 확장할 수 있다.
- `morgan`함수는 미들웨어를 반환하고, 이 반환된 미들웨어가 `(req, res, next)`를 인자로 받아 실행한다. 즉 함수가 바로 실행되지 않고 외부함수로부터 인자를 받아 실행된다. 조건에 따라 동적으로 미들웨어를 선택하고 실행할 수 있다. 

📍 **`dotenv`**
```node.js
const dotenv = require('dotenv');
dotenv.config();
```
- .env파일을 읽어서 process.env로 만든다.

📍 **`morgan`**
```node.js
const morgan = require('morgan');
app.use(morgan('dev'));
```
- 요청과 응답에 대한 정보를 콘솔에 기록한다.
- `GET / 500 7.409ms - 50`는 [HTTP 메서드][주소][HTTP 상태 코드][응답 속도] - [응답 바이트]
- dev외에 combined, common, short, tiny 등을 넣을 수 있다. 개발 환경에서는 dev를, 배포 환경에서는 combined

📍 **`cookie-parser`**
```node.js
app.use(cooikeParser(비밀키));

const cooikeParser = require('cookie-parser');
app.use(cooikeParser(process.env.COOKIE_SECRET));
```
- 요청에 동봉된 쿠키를 **해석**해 `req.cookies`객체로 만든다. 따라서 라우트 핸들러에서 쿠키에 접근할 수 있다. 
- 첫 번째 인수인 비밀 키를 넣은 경우, (서명된 쿠키가 있는 경우) 제공한 비밀 키를 통해 해당 쿠키가 **내 서버가 만든 쿠키**임을 검증할 수 있다.
- 서명된 쿠키의 서명을 검증하여, 쿠키가 서버에서 생성된 이후 변조되지 않았는지 확인한다.
- 서명이 검증되면 `req.singedCookies` 객체에 쿠키 값을 저장하고, 서명이 유효하지 않으면 서명된 쿠키의 값을 `false`로 설정하여 무효화한다. 
- **비밀 키를 통해 만들어낸 서명**을 쿠키 값 뒤에 붙인다.
- 서명이 붙으면 쿠키가 `name=gildong.sign`과 같은 모양이 되고, 서명된 쿠키는 `req.cookies` 대신 `req.singedCookies`객체에 들어있다.
- 쿠키를 생성/제거하기 위해서는 `res.cookie`, `res.clearCookie` 메서드를 사용해야 한다. `res.cookie(키, 값, 옵션)`
- 쿠키를 지우려면 키와 값 외에 옵션까지 정확히 일치해야 한다. `expires`나 `maxAge`는 일치할 필요가 없다.
- `signed` 라는 옵션을 `true`로 설정하면 쿠키 뒤에 서명이 붙는다. 내 서버가 쿠키를 만들었다는 것을 검증할 수 있다.
```node.js
res.cookie('name', 'gildong', {
  expires: new Date(Date.now() + 900000),
  httpOnly: true, // 자바스크립트에서 쿠키에 접근할 수 없다. XSS(교차 사이트 스크립팅) 공격을 방지하는 데 도움
  secure: true, // https일 경우에만 쿠키가 전송된다. 
});
res.clearCookie('name', 'gildong', { httpOnly: true, secure: true });

// Path = URL: 쿠키가 전송될 URL을 특정할 수 있다. 기본값은 '/', 모든 URL에서 쿠키를 전송할 수 있다.
// Domain = 도메인명: 쿠키가 전송될 도메인을 특정할 수 있다. 기본값은 현재 도메인
```
- 클라이언트가 쿠키의 내용을 변경하려 하면, 서명이 유효하지 않게 되어 서버가 이를 감지할 수 있다.
- 서버는 클라이언트에서 받은 쿠키 데이터에서 서명 부분을 분리하고, 나머지 데이터를 비밀 키로 서명하여 클라이언트가 보낸 서명과 비교한다.
- 서명이 일치하면, 서버는 쿠키가 변조되지 않았다고 판단하고 요청을 처리한다. 그렇지 않으면 쿠키를 무효로 처리하거나 에러를 반환한다.
```node.js
app.use(cookieParser(secret));

app.get('/set-cookie', (req, res) => {
  const userData = 'someUserData';
  res.cookie('user', userData, { signed: true, httpOnly: true });
  res.send('Signed cookie is set');
});
```
- 위에서 `signed: true`를 지정하면 `res.cookie` 메서드가 내부적으로 `cookie-parser`의 서명 기능을 사용하여 쿠키 값을 서명합니다.

📍 **`express-session`**
```node.js
const session = require('express-session');
app.use(session({
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
    maxAge: 7200000,
  },
  name: 'book-club',
}));
```
- 세션 관리용 미들웨어. 세션은 사용자별로 `req.session`객체 안에 유지된다.
- 인수로 세션에 대한 설정을 받는다.
- `resave`: 요청이 올때 세션에 수정 사항이 생기지 않더라도 세션을 다시 저장할지 설정, `false`라면 세션이 변경되지 않은 경우 세션 저장소에 저장하지 않는다. 
- `saveUninitialized`: 세션에 저장할 내역이 없더라도 처음부터 세션을 생성할지 설정, `false`라면 세션이 새로 생성되고 변경된 내용이 없으면 세션을 저장소에 저장하지 않는다. 
- 세션 관리 시 클라이언트에 쿠키를 보낸다. 쿠키를 서명하기 위해 `secret` 값이 필요하다.
- 이름은 `name` 옵션으로 설정하고 기본 이름은 connect.sid 이다.
- cookie는 세션 쿠키에 대한 설정이다. 일반적인 쿠키 옵션을 설정한다.
- 배포 시에 `store`라는 옵션에 데이터베이스를 연결해 세션을 유지하는 것이 좋다. 보통 레디스가 자주 쓰인다. 현재는 메모리에 세션을 저장하도록 되어 있어 서버를 재시작하면 메모리가 초기화되어 세션이 모두 사라진다.
- `express-session`에서 서명한 쿠키 앞에는 `s:`가 붙고, 이는 `encodeURIComponent`함수가 실행되어 `s%3A`가 된다.
- 세션 식별자가 세션 쿠키의 값이 되고, 보통 고유한 무작위 문자열로 생성된다. 그리고 세션 식별자는 `express-session`에 의해 자동으로 생성되고 관리된다. 
```node.js
req.session.name = 'gildong'; // 세션 등록
req.sessionID; // 현재 세션 아이디 확인
req.session.destroy(); // 세션 모두 제거
```

📍 **`multer`**
- `enctype`이 `multipart/form-data`인 폼을 통해 업로드하는 데이터의 형식을 다룬다.
```node.js
<form action="" method="" enctype="multipart/form-data">
  <input type="file" name="image" />
  <input type="text" name="title" />
  <button type="submit">업로드</button>
</form>
```
```node.js
const path = require('path');
const multer = require('multer');

const upload = multer({
  storage: multer.diskStorage({
    destination(req, file, done) {
      done(null, 'uploads/')
    },
    filename(req, file, done) {
      const ext = path.extname(file.originalname);
      done(null, path.basename(file.originalname, ext) + Date.now() + ext);
    },
  }),
  limits: { fileSize: 5 * 1024 * 1024 },
});  

router.post('/preview', upload.single('image'), (req, res) => {
  const url = `/img/${req.file.filename}`;
  res.json({ url });
});
```
- storage는 어디에 어떤 이름으로 저장할지, req 매개변수에는 요청에 대한 정보가, file 객체에는 업로드한 파일에 대한 정보가
- done(에러, 실제 경로나 파일 이름), req나 file 데이터를 가공해 done으로 넘긴다.
- [파일명+현재시간.확장자],
- limits에는 업로드에 대한 제한 사항을 설정,
- 파일을 하나만 업로드하는 경우 `upload.single('image')`: 파일 업로드 후 `req.file`객체가 생성된다. 인수는 input태그의 name이나 폼 데이터의 키와 일치하게 넣으면 된다. `req.body`에는 파일이 아닌 다른 input 데이터가 들어있다.
```node.js
// 여러 파일을 업로드 하는 경우
<input type="file" name="many" multiple />

router.post('/preview', upload.array('many'), (req, res) => {
  console.log(req.files, req.body);
});
```
```node.js
// 파일을 여러 개 업로드하지만 키가 다른 경우
<input type="file" name="image1" />
<input type="file" name="image2" />

router.post('/preview', upload.fields([{ name: 'image1' }, { name: 'image2' }]), (req, res) => {
  console.log(req.files,image1, req.files,image2);
});
```
```node.js
// 멀티파트지만 파일을 업로드하지 않는 경우
router.post('/preview', upload.none(), (req, res) => {
  console.log(req.body);
});
```

📍 **`Router 객체로 라우팅 분리하기`**
```node.js
// routes/user.jd
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
});

module.exports = router;
```
```node.js
// app.js
const userRouter = require('./routes/user');

app.use('/user', userRouter);
```
```node.js

router.get('/', (req, res, next) => {
  next('route');
}, (req, res, next) => {
  console.log('실행되지 않는다');
  next();
}, (req, res, next) => {
  console.log('실행되지 않는다');
  next();
});

router.get('/', (req, res) => {
  console.log('실행된다');
  res.send('Hello');
});
```
- 라우터에 연결된 나머지 미들웨어들을 건너뛰고 싶을 때 사용한다.

```node.js
router.get('/user/:id', (req, res) => {
  console.log(req.params.id);
  console.log('실행된다');
});

router.get('/user/like', (req, res) => {
  console.log(req.params.id);
  console.log('실행되지 않는다.');
});
```
- 라우트 매개변수 사용 시 일반 라우터보다 뒤에 위치해야 한다. 다양한 라우터를 아우르는 와일드 카드 역할을 하기 때문이다.
```node.js
// 쿼리스트링
// /users/123?limit=5&skip=10
router.get('/users/:id', (req, res) => {
  console.log(
    req.params.id,
    req.query.limit,
    req.query.skip,
  );
});
```
```node.js
// 에러 처리 미들웨어 위에
// 일치하는 라우터가 없을때 404 상태 코드 응답
app.use((req, res, next) => {
  res.status(404).send('Not Fount');
});
```
```node.js
// 주소는 같지만 메서드가 다를때
router.route('/abc')
  .get((req, res) => {})
  .post((req, res) => {})
```

📍 **`req`,`res`객체**
* **`req`**
  * `req.app`: `req.app.get('')`처럼 app 객체에 접근
  * `req.body`
  * `req.cookies`
  * `req.ip`
  * `req.params`
  * `req.query`
  * `req.singedCookies`
  * `req.get(헤더이름)`: 헤더의 값을 가져오고 싶을 때
* **`res`**
  * `res.app`
  * `res.cookie(키, 값, 옵션)`
  * `res.clearCookie(키, 값, 옵션)`
  * `res.end()`: 데이터 없이 응답을 보낸다
  * `res.json(json)`
  * `res.render(뷰, 데이터)`
  * `res.send(데이터)`: 문자열, HTML, 버퍼, 객체, 배열 등
  * `res.sendFile(경로)`
  * `res.set(헤더, 값)`
  * `res.status(코드)`


📍 **`bcrypt`**
```node.js
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12);
const result = await bcrypt.compare(password, exUser.password);
```
