# wiz-editor 用户认证

wiz-editor编辑服务端没有保存任何用户信息。所以一个文档可以由哪些用户访问或者编辑，需要企业的业务服务来确定。

## 获得应用AppId和AppSecret

企业的应用应该首先向wiz-editor服务申请一个AppId和AppSecret。私有部署的wiz-editor服务，则可以在服务端的配置文件里面找到。

## 验证用户身份并确认用户是否可以访问文档

首先，企业应用需要根据当前用户的身份，文档Id，来确认用户对文档的权限。
然后，通过AppId，AppSecret以及用户Id，生成一个JWT规范的access token，并将这个access token返回给前端。
前端在创建编辑器的时候，将AppId，用户Id以及生成的access token，作为参数传递给编辑器。
编辑器将会把这些数据，提交给编辑服务。编辑服务会根据用户的AppId，用户Id以及access token来进行验证。如果通过，则可以正常编辑，
否则将无法编辑。

## 演示代码 （NodeJS）

### 安装jose

```sh
npm i jose
```

### 验证用户身份，生成access token

```ts

import EncryptJWT from 'jose/jwt/encrypt';

const AppId = 'xxx-xxx-xxx';
  // AppSecret 应该通过环境变量/配置文件获取
const AppSecret = 'xxx';
const AppDomain = 'wiz.cn';

async function fakeGetAccessTokenFromServer(userId: string, docId: string): Promise<string> {
  //
  // userId应该通过当前用户的业务token（cookie等）获取，得到当前用户的身份
  // docId可以作为Api参数获取
  //
  // JWT 自定义数据
  const data = {
    userId,
    docId,
    appId: AppId,
  };

  const key = Buffer.from(AppSecret);

  // 生成JWT规范的token，并返回给前端。
  const accessToken = await new EncryptJWT(data)
    .setProtectedHeader({ alg: 'dir', enc: 'A256GCM' })
    .setIssuedAt()
    .setIssuer(AppDomain)
    .setExpirationTime('120s')
    .encrypt(key);

  return accessToken;
}
```

企业应用的前端，需要调用该Api，获取到用户的accessToken，然后再创建编辑器。


## 启动服务

```js
const { startServer } = require('wiz-editor/server');
const path = require('path');

// 参考node_modules/wiz-editor/config/server.json 文件
const options = {
  enableFakeTokenApi: true,
  serveStatic: true,
  staticDir: path.resolve('./dist'),
};

console.log(options);

startServer(options);
```

### 启动服务选项

```js
{
    port: 9000, // 服务端口
    serveStatic: true, // 是否发布静态网页服务
    staticDir: "/public/www", // 如果发布静态网页，则在此设置路径
    allowCORS: false, // 是否允许跨域。如果允许跨域，则网页需要使用https。如果无法使用https（例如开发环境），可以使用代理
    storage: {
        webhook : {
            enable: false,    // 是否启用webhook
            latestVersionDelay: 30, // 停止文档编辑操作多长时间后，发送最新版本给webhook地址
            latestVersionURL: "http://localhost" // webhook地址
        },
    }
}
```

#### webhook推送的内容

服务器会在每次文档在设定的时间间隔未修改时，将文档转为文本格式，并Post到指定URL，Post过去的数据结构为：

```json
{
  "appId": "应用Id",
  "docId": "文档Id",
  "data": "文本数据"
}
```
