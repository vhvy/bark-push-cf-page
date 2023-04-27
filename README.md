## BARK-PUSH-CF-PAGE


灵感来自：https://day.app/2018/06/bark-server-document/ ，看了作者的文章后我发现原来实现调用苹果的推送接口还是挺简单的，我不想维护 Docker ，于是打算找个免费的平台来实现 Bark Push Server。

本来打算使用 Vercel 的 Edge Function，因为它有香港的节点，速度比较快；但是写完之后才发现 Edge Function 内置的 fetch 不支持 http2，网上搜了一圈发现 Cloudflare 的 Worker 似乎是支持 http2 的。经过测试之后的确是支持 http2 的，但是本地测试环境不支持。

Bark 支持自定义推送图标，但 Worker 不能返回图片，而 Cloudflare Pages 支持集成 Worker，于是乎就用 Cloudflare Pages 完成了这个小工具。

### 如何使用

1. 克隆本项目，然后在 Cloudflare Pages 中通过`连接到Git`创建项目。

2. 创建过程中的构建设置：

    + 框架：         `None`
    + 构建命令：     `npm i`
    + 构建输出目录： `/public`

![1](/examples/1.png)

3. 环境变量：

    + AUTH_KEY_ID： 见灵感来源文章
    + TEAM_ID:      同上
    + DEVICE_TOKEN： 从 iOS Bark App 中获取
    + VERIFY_TOKEN:  自行设置，调用接口时携带在 header 中鉴权用
    + PRIVATE_KEY： 填写 https://github.com/Finb/bark-server/releases 中`AuthKey_LH4T9V5U4R_5U8LBRXG3A.p8` 的内容。

![2](/examples/2.png)

4. 在 Cloudflare Worker KV 中创建一个 KV。

5. Cloudflare Pages 项目设置/函数中绑定刚才创建的KV，变量名称为`PUSH_KV`。

![3](/examples/3.png)

6. Cloudflare Pages 项目部署/所有部署，重新发起部署，因为刚才绑定的 KV 需要重新部署才能生效。

![4](/examples/4.png)

7. 如何调用：
    + 请求方法: GET
    + 请求路径: /api/push
    + 携带Url参数(不带*的为可选)
      + *name: App名称
      + *body: 通知正文
      + title: 副标题
      + icon: 自定义图标网络地址

![5](/examples/5.png)
![6](/examples/6.png)

8. 如何增加内置图标

将项目克隆到本地，将`png`格式的图片放到项目`/public/assets/`文件夹下，并将图片名称加入`functions/api/push`中的`iconNameList`数组，然后推送到`Github`，`Cloudflare Pages`会自动触发部署。