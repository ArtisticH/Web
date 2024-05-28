# auth

## 로컬 로그인

```node.js
// app.js
const passport = require('passport');
const passportConfig = require('./passport'); // ./passport/index.js
passportConfig();

app.use(passport.initialize());
app.use(passport.session());
```
📍 **`passport.initialize()`**
- `passport.initialize()`는 `req`객체에 passport 설정을 심는다.
- `passport.initialize()`은 passport를 초기화한다. 즉 passport를 사용할 수 있는 첫 번째 단계이다.
- 요청의 `req`객체에 passport의 초기 상태를 부여한다.
  
📍 **`passport.session()`**
- `passport.session()`은 **`req.session`에 passport 정보를 저장**한다. `express-session`뒤에 연결해야 한다.
- 예를 들어 `req.session['passport']`는 `{user: 1}`이다.

📍 **`로컬 로그인 구현`**  

**로그인 전**
1. 라우터를 통해 로그인 요청이 들어옴
2. 라우터에서 `passport.authenticate` 메서드 호출
3. 로그인 전략 수행(입력 정보와 데이터베이스 비교)
4. 성공 시 사용자 정보 객체와 함께 `req.login` 호출
5. `req.login` 메서드가 `passport.serializeUser` 호출
6. `req.session`에 사용자 아이디만 저장

**로그인 이후**
1. 요청이 들어옴
2. 라우터에 요청이 도달하기 전에 **`passport.session` 미들웨어가** `passport.deserializeUser` 메서드 호출
3. **`req.session`에 저장된 아이디**로 데이터베이스에서 사용자 조회
4. `req.user`에 저장
5. 라우터에서 `req.user` 사용 가능

📍 **`로그인 접근 권한 제어 미들웨어`**
```node.js
// /routes/middlewares.js
exports.isLoggedIn = (req, res, next) => {
  if(req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send('로그인 필요');
  }
};
// 로그인 하지 않아야만 다음 미들웨어로 이동
exports.isNotLoggedIn = (req, res, next) => {
  if(!req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send('로그인한 상태입니다.');
  }
};
```

📍 **`passportConfig()`**
```node.js
// passport/index.js
const passport = require('passport');
const local = require('./localStrategy');
const kakao = require('./kakaoStrategy');
const Member = require('../models/member');

module.exports = () => {
  passport.serializeUser((user, done) => {
    done(null, user.id);
  });

  passport.deserializeUser((id, done) => {
    Member.findOne({ where: { id }})
      .then(user => {
        done(null, user);
      })
      .catch(err => done(err));
  });

  local();
  kakao();
}
```

📍 **`passport.serializeUser`**
- `serializeUser`는 로그인 시 실행되며, `req.session`에 어떤 데이터를 저장할지 정한다. 로그인 시에만 실행된다.
- 즉 사용자 정보 객체를 세션에 아이디로 저장한다.
- `req.session['passport']`에 `{user: 1}`처럼 저장된다.

📍 **`passport.deserializeUser`**
- `deserializeUser`는 로그인 후 매 요청 시 실행된다.
- `passport.session`미들웨어가 이 메서드를 호출한다.
-  **`req.session`에 저장된 아이디**가 `deserializeUser`의 매개변수가 된다.
- 세션에 불필요한 데이터를 담아두지 않기 위해 `req.session`에 유저의 id를 저장하고 이 유저의 id를 데이터베이스에서 꺼내와 쓴다.
- 그리고 조회한 정보를 `req.user`에 저장한다.

📍 **`localStrategy`**
```node.js
// passport/localStrategy.js
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcrypt');
const Member = require('../models/member');

module.exports = () => {
  passport.use(new LocalStrategy({
    usernameField: 'email',
    passwordField: 'password',
  }, async (email, password, done) => {
    try {
      const exUser = await Member.findOne({ where: { email }});
      if(exUser) {
        const result = await bcrypt.compare(password, exUser.password);
        if(result) {
          done(null, exUser);
        } else {
          done(null, false, { message: '비밀번호가 일치하지 않습니다.' });
        }
      } else {
        done(null, false, { message: '가입되지 않은 회원입니다.' });
      }
    } catch (error) {
      done(error);
    }
  }));
}
```

📍 **로그인 전 요청**
```node.js
req.user: undefined
req.session[passport]: undefined
req.session[cookie] {
  path: '/',
  _expires: 2024-05-21T11:39:36.005Z,
  originalMaxAge: 7178695,
  httpOnly: true,
  secure: false
}
req.signedCookies: [Object: null prototype] {
  BOOKCLUB: 'aUAA_lSXxT4Nn2ANUbG7KWQdxxu6Uzl5'
}
req.sessionID: Iyy6Gqd0mMNUQv4a1iy34ho0hka2iVDh

req.user: undefined
req.session[passport]: undefined
req.session[cookie] {
  path: '/',
  _expires: 2024-05-21T11:39:36.005Z,
  originalMaxAge: 7158923,
  httpOnly: true,
  secure: false
}
req.signedCookies: [Object: null prototype] {
  BOOKCLUB: 'aUAA_lSXxT4Nn2ANUbG7KWQdxxu6Uzl5'
}
req.sessionID: FiSi80u_kZ5938GmmJjAEWkz2xCNe3O5
```
- `req.sessionID`값은 요청마다 새롭고
- `req.signedCookies`은 값이 같다. 그리고 이전 로그인 값인 것 같다.
- 이 둘이 다르다.


📍 **로그인 후**
```node.js
req.session[passport]: { user: 1 }
req.session[cookie] {
  path: '/',
  _expires: 2024-05-21T11:39:36.006Z,
  originalMaxAge: 7119048,
  httpOnly: true,
  secure: false
}
req.signedCookies: [Object: null prototype] {
  BOOKCLUB: 'N9BrSeKjsPVgjNpd3EPUP6_mW-fgOtJR'
}
req.sessionID: N9BrSeKjsPVgjNpd3EPUP6_mW-fgOtJR

req.session[passport]: { user: 1 }
req.session[cookie] {
  path: '/',
  _expires: 2024-05-21T11:40:02.909Z,
  originalMaxAge: 7119048,
  httpOnly: true,
  secure: false
}
req.signedCookies: [Object: null prototype] {
  BOOKCLUB: 'N9BrSeKjsPVgjNpd3EPUP6_mW-fgOtJR'
}
req.sessionID: N9BrSeKjsPVgjNpd3EPUP6_mW-fgOtJR
```
- `req.session[passport]`값이 생기고
- `req.sessionID`와 `req.signedCookies`의 값이 같다(동기화)

📍 **로그아웃 후**
```node.js
req.user: undefined
req.session[passport]: undefined
req.session[cookie] {
  path: '/',
  _expires: 2024-05-21T11:39:36.005Z,
  originalMaxAge: 7066897,
  httpOnly: true,
  secure: false
}
req.signedCookies: [Object: null prototype] {
  BOOKCLUB: 'N9BrSeKjsPVgjNpd3EPUP6_mW-fgOtJR'
}
req.sessionID: gHnZEcvfzU6uhTNWSMwwUB0Mg8RbJmZk

req.user: undefined
req.session[passport]: undefined
req.session[cookie] {
  path: '/',
  _expires: 2024-05-21T11:39:36.005Z,
  originalMaxAge: 7052727,
  httpOnly: true,
  secure: false
}
req.signedCookies: [Object: null prototype] {
  BOOKCLUB: 'N9BrSeKjsPVgjNpd3EPUP6_mW-fgOtJR'
}
req.sessionID: 3ggdkdImLQrRQJgzxpEQcoGY_N-ACC2x
```
- `req.session[passport]`값이 사라지고
- `req.sessionID`와 `req.signedCookies`의 값이 달라진다.(분리), `req.signedCookies`는 이전 로그인때의 값이고, `req.sessionID`는 다시 매번 새로운 값
