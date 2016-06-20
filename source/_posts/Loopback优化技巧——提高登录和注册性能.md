---
title: Loopback优化技巧——提高登录和注册性能
date: 2016-06-19 19:43:12
tags: loopback
---

初次使用loopback框架进行开发时就发现登陆和注册的速度慢得令人发指，有一天终于受不了了，决定找出问题所在。

loopback的用户模型一般都继承自官方封装的User模型，少爷的项目当然也不例外，我们打开${workspaceRoot}/node_modules/loopback/common/models/user.js，发现login方法如下：

```javascript
  User.login = function(credentials, include, fn) {
    var self = this;
    if (typeof include === 'function') {
      fn = include;
      include = undefined;
    }

    fn = fn || utils.createPromiseCallback();

    include = (include || '');
    if (Array.isArray(include)) {
      include = include.map(function(val) {
        return val.toLowerCase();
      });
    } else {
      include = include.toLowerCase();
    }

    var realmDelimiter;
    // Check if realm is required
    var realmRequired = !!(self.settings.realmRequired ||
      self.settings.realmDelimiter);
    if (realmRequired) {
      realmDelimiter = self.settings.realmDelimiter;
    }
    var query = self.normalizeCredentials(credentials, realmRequired,
      realmDelimiter);

    if (realmRequired && !query.realm) {
      var err1 = new Error('realm is required');
      err1.statusCode = 400;
      err1.code = 'REALM_REQUIRED';
      fn(err1);
      return fn.promise;
    }
    if (!query.email && !query.username) {
      var err2 = new Error('username or email is required');
      err2.statusCode = 400;
      err2.code = 'USERNAME_EMAIL_REQUIRED';
      fn(err2);
      return fn.promise;
    }

    self.findOne({where: query}, function(err, user) {
      var defaultError = new Error('login failed');
      defaultError.statusCode = 401;
      defaultError.code = 'LOGIN_FAILED';

      function tokenHandler(err, token) {
        if (err) return fn(err);
        if (Array.isArray(include) ? include.indexOf('user') !== -1 : include === 'user') {
          // NOTE(bajtos) We can't set token.user here:
          //  1. token.user already exists, it's a function injected by
          //     "AccessToken belongsTo User" relation
          //  2. ModelBaseClass.toJSON() ignores own properties, thus
          //     the value won't be included in the HTTP response
          // See also loopback#161 and loopback#162
          token.__data.user = user;
        }
        fn(err, token);
      }

      if (err) {
        debug('An error is reported from User.findOne: %j', err);
        fn(defaultError);
      } else if (user) {
        user.hasPassword(credentials.password, function(err, isMatch) {
          if (err) {
            debug('An error is reported from User.hasPassword: %j', err);
            fn(defaultError);
          } else if (isMatch) {
            if (self.settings.emailVerificationRequired && !user.emailVerified) {
              // Fail to log in if email verification is not done yet
              debug('User email has not been verified');
              err = new Error('login failed as the email has not been verified');
              err.statusCode = 401;
              err.code = 'LOGIN_FAILED_EMAIL_NOT_VERIFIED';
              fn(err);
            } else {
              if (user.createAccessToken.length === 2) {
                user.createAccessToken(credentials.ttl, tokenHandler);
              } else {
                user.createAccessToken(credentials.ttl, credentials, tokenHandler);
              }
            }
          } else {
            debug('The password is invalid for user %s', query.email || query.username);
            fn(defaultError);
          }
        });
      } else {
        debug('No matching record is found for user %s', query.email || query.username);
        fn(defaultError);
      }
    });
    return fn.promise;
  };
```

我们来分析下这段代码，当我们调用self.findOne()之前，基本是判断参数是否存在、整合参数，看不出什么问题，我们接着往下追，当我们查找出用户之后，发现调用了hasPassword并传入了用户输入的密码，hasPassword方法代码如下：

```javascript
  User.prototype.hasPassword = function(plain, fn) {
    fn = fn || utils.createPromiseCallback();
    if (this.password && plain) {
      bcrypt.compare(plain, this.password, function(err, isMatch) {
        if (err) return fn(err);
        fn(null, isMatch);
      });
    } else {
      fn(null, false);
    }
    return fn.promise;
  };
```

这是User模型instance的方法，将用户输入的密码和模型实例中的密码进行对比（这种校验的方式和PHP的password_verify，以及ruby on rails一致，是一种哈希加密算法，一个字符串每次创建的哈希密码都不一致，但是每个哈希密码都能和原字符串校验为真，是一种不可逆的加密算法）。光看这代码我们也不能发现什么问题，但是下边就要返回accessToken啦，问题会不会出现在校验这儿呢？抱着怀疑的心态，我跳转到bcrypt定义的代码块：

```javascript
var bcrypt;
try {
  // Try the native module first
  bcrypt = require('bcrypt');
  // Browserify returns an empty object
  if (bcrypt && typeof bcrypt.compare !== 'function') {
    bcrypt = require('bcryptjs');
  }
} catch (err) {
  // Fall back to pure JS impl
  bcrypt = require('bcryptjs');
}
```

我揉了揉眼，去查看了下loopback的package.json：

```json
{
  "dependencies": {
    "async": "^0.9.0",
    "bcryptjs": "^2.1.0",
    "body-parser": "^1.12.0",
    "canonical-json": "0.0.4",
    "continuation-local-storage": "^3.1.3",
    "cookie-parser": "^1.3.4",
    "debug": "^2.1.2",
    "depd": "^1.0.0",
    "ejs": "^2.3.1",
    "errorhandler": "^1.3.4",
    "express": "^4.12.2",
    "inflection": "^1.6.0",
    "loopback-connector-remote": "^1.0.3",
    "loopback-phase": "^1.2.0",
    "nodemailer": "^1.3.1",
    "nodemailer-stub-transport": "^0.1.5",
    "serve-favicon": "^2.2.0",
    "stable": "^0.1.5",
    "strong-remoting": "^2.21.0",
    "uid2": "0.0.3",
    "underscore.string": "^3.0.3"
  }
}
```

果然只有bcryptjs而没有bcrypt，这时候我们只需要在${workspaceRoot}执行：

```shell
cnpm install bcrypt --save
```

再次访问登录接口，发现速度已经飞起有木有！！！

> 其实官网文档有关于这个的介绍，只是藏得比较深，平时注意不到（其实这些都是借口啦，主要是博主英文战5渣），附上官方文档传送门[[Managing user](https://docs.strongloop.com/display/APIC/Managing+users)]