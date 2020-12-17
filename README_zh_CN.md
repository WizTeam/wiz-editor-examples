# wiz-editor-examples

直接从git clone或者下载代码，解压缩到磁盘。

## 运行react 例子

1. 安装和运行

```sh
cd react-demo
npm install
npm start
```

2. 打开浏览器

```
localhost:9000
```

## 运行原生js例子

1. 安装和运行

```sh
cd h5
npm install
npm start
```

2. 打开浏览器

```
localhost:9000
```

## 在自己的项目中wiz-editor使用说明

### 通过npm按章wiz-editor
npm i wiz-editor

### 在项目中使用wiz-editor

```js
import {
  createEditor,
  Editor,
} from 'wiz-editor/client';

// 定义AppID，AppSecret, AppDomain。在自带的测试服务器中，下面三个key不要更改
const AppId = '_LC1xOdRp';
const AppSecret = '714351167e39568ba734339cc6b997b960ed537153b68c1f7d52b1e87c3be24a';
const AppDomain = 'wiz.cn';

// 初始化服务器地址
const WsServerUrl = window.location.protocol !== 'https:'
  ? `ws://${window.location.host}`
  : `wss://${window.location.host}`;

// 定义一个用户。该用户应该是有应用服务器自动获取当前用户身份
// 编辑服务需要提供用户id以及用户的显示名。
const user = {
  userId: `${new Date().valueOf()}`,
  displayName: 'test user',
};

// 设置编辑器选项
const options = {
  serverUrl: WsServerUrl,
  user,
};

// 从应用服务器获取一个AccessToken。应用服务器许需要负责验证用户对文档的访问权限。
// accessToken采用gwt规范，里面应该包含用户的userId，文档的docId，以及编辑应用的AppId。
// 下面是一个演示例子。在正常强况下，AccessToken应该通过用户自己的应用服务器生成。
// 因为在前段使用GWT加密规范的时候，必须在https协议下面的网页才可以使用。为了演示，
// 我们的自带的测试服务器会提供一个虚拟的token生成功能。（启动服务的时候，需要指定--enable-fake-token-api 参数）
// 

async function fakeGetAccessTokenFromServer(userId: string, docId: string): Promise<string> {
  //
  const data = {
    userId,
    docId,
    appId: AppId,
  };

  const fromHexString = (hexString: string) => {
    const parts = hexString.match(/.{1,2}/g);
    assert(parts);
    const arr = parts.map((byte) => parseInt(byte, 16));
    return new Uint8Array(arr);
  };
  //
  const key = fromHexString(AppSecret);

  try {
    const accessToken = await new EncryptJWT(data)
      .setProtectedHeader({ alg: 'dir', enc: 'A256GCM' })
      .setIssuedAt()
      .setIssuer(AppDomain)
      .setExpirationTime('120s')
      .encrypt(key);

    return accessToken;
  } catch (err) {
    const res = await fetch(`http://${window.location.host}/token/${AppId}/${docId}/${userId}`);
    const ret = await res.json();
    return ret.token;
  }
}

// 文档id
const docId = 'my-test-doc-id';

(async function loadDocument() {
  // 验证身份，获取accessToken
  const token = await fakeGetAccessTokenFromServer(user.userId, docId);

  // 生成编辑服务需要的认证信息
  const auth = {
    appId: AppId,
    userId: user.userId,
    docId,
    token,
  };

  // 创建编辑器并加载文档
  const editor = createEditor(document.getElementById('editor') as HTMLElement, options, auth);
})();

```

通过上面的代码，就可以在自己的应用中，创建一个多人实时协同编辑器。
上面的代码，可以通过在h5例子下面，运行npm run simple来查看效果。


