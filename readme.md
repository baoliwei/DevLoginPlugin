dev-auto-login解决了前端开发环境使用代理时登录的问题。

# 使用场景
例如：后端验证登录是a.com站点，前端启用代理最终启动的站点是b.com，需要登录凭证时获取验证不通过

# 安装

`npm install --save-dev dev-auto-login`

或

`yarn add --dev dev-auto-login`

# 运行流程
## 总体流程
启动浏览器 -->  启动页面 --> hook注册 --> 自动登录。

## hook注册
访问原始站点 --> 运行插件 --> 登录 --> 验证码 --> 获取原始站点登录凭证

## 自动登录
执行hook --> 转移原始站点凭证至最终站点 --> 跳转至最终站点

# 使用
## webpack

```
const DevAutoLogin = require('dev-auto-login');

```

```
new DevAutoLogin(params)

```

## node

```
const DevAutoLogin = require('dev-auto-login');

```

```
const DevAutoLogin = new DevLogin(params)
DevAutoLogin.run()

```
# 配置

构造函数可以接受三个参数，`userOptions`  `browserOptions`  `plugins`

## userOptions
表示了使用者登录原网站时使用的数据，默认参数如下：
```
    cookies:['session'], // 类型：数组。 默认值：['session']    描述：要获取的cookie（登录凭证）名称
    originSiteUrl: '', // 类型：字符串。 默认值：''  描述：源站点url（要获取cookie的url）
    finalSiteUrl: '', // 类型：字符串。 默认值：'' 描述：最终要使用的站点url（要设置cookie的url）
    userName: '', // 类型：字符串。 默认值：'' 描述：登录用户名
    password: '', //  类型：字符串。 默认值：'' 描述：登录密码
    verificationCode: '', //  类型：字符串。 默认值：'1'  描述：登录时的验证码
    closeDefaultLogin: false,  类型：布尔。 默认值：false  描述：是否关闭默认的登录流程。
    closeDefaultVerificationCode: false,  类型：布尔。 默认值：false  描述：是否关闭默认的验证码流程。
    userNameEl: '#username', // 类型：字符串。 默认值：'#username'  描述：用户名输入框dom标识
    passwordEl: '#password', // 类型：字符串。 默认值：'#password'  描述：密码输入框dom标识
    loginEl: '.btn-submit', // 类型：字符串。 默认值：'.btn-submit'  描述：登录按钮dom标识
    verificationCodedEl: '#captcha', // 类型：字符串。 默认值：'#captcha'  描述：验证码输入框dom标识
    verificationCodeBtnEl: '.btn-submit' // 类型：字符串。 默认值：'.btn-submit'  描述：确认验证码按钮dom标识
```
## browserOptions

表示了使用者启动浏览器时使用的数据，默认参数如下：

```
    ignoreHTTPSErrors: true, // 是否在导航期间忽略 HTTPS 错误. 默认是 false。
    headless: false,   // 有无浏览器界面启动
    slowMo: 0,       // 放慢浏览器执行速度，方便测试观察
    defaultViewport: null, // 为每个页面设置一个默认视口大小
    args: [  // 启动 Chromium 的参数
        '--ignore-certificate-errors', // 忽略证书认证出错
        '--ignore-ssl-errors',
        '--disable-gpu',   // 禁用GPU加速      
        '–no-sandbox', // 对通常为沙盒的所有进程类型禁用沙盒。仅用作测试目的的浏览器级别开关。
        '--disable-setuid-sandbox', // 禁用setuid沙盒（仅限Linux）。
        '--remote-debugging-port=9222',  // 浏览器端口
    ],
    dumpio: true  // 是否将浏览器进程标准输出和标准错误输入到 process.stdout 和 process.stderr 中
```
[配置详细参见](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v1.12.0&show=api-puppeteerlaunchoptions)

[启动启动 Chromium 的参数参见](https://peter.sh/experiments/chromium-command-line-switches/)

 > 注意：`executablePath` 必须配置

例如
```
{
    // 可运行 Chromium 或 Chrome 可执行文件的路径
    executablePath: 'C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe'
}
```
## plugins

插件会运行在登录行为之前。

类型：数组。

数组项类型：函数。


* 当数组项参数个数等于0时，代码会被嵌入到浏览器进行运行。
    * 当执行完当前函数需要进行页面跳转时，需要返回：`{nextPage: true}`
    * 当执行完当前函数已拿到登录凭证时，需要返回：`{loginCert: [cookieObject]}`

* 当数组函数参数个数不为0时，代码会被嵌入Node环境进行运行。同时会绑定当前上下文，并且传入[`browser`](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v1.12.0&show=api-class-browser), [`page`](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v1.12.0&show=api-class-page)

# 示例
```
new DevAutoLogin(
    { // userOptions
        closeDefaultLogin: true,
        closeDefaultVerificationCode: true,
        cookies:['session'], // 要获取的cookie名称
        originSiteUrl: 'final url', // 要获取cookie的url
        finalSiteUrl: 'http://localhost:8001/', // 要设置cookie的url
        userName: 'username', // 登录用户名
        password: 'password', // 登录密码
        verificationCode: '1' // 登录时的验证码
    },
    { // browserOptions
    // 可运行 Chromium 或 Chrome 可执行文件的路径
    executablePath: 'C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe'
    },
    [ // plugins
        function () {
            document.querySelector('#username').value = 'username'
            document.querySelector('#password').value = 'password'
            let btn = document.querySelector('.btn-submit')
            btn.click()
            return {nextPage: true}
        },
        function () {
            document.querySelector('#captcha').value = '1'
            // document.querySelector('#password').value = 'aaa111'
            let btn = document.querySelector('.btn-submit')
            btn.click()
            return {nextPage: true}
        }
    ]
)

```