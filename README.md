# 阿里云Tokens怎么获取：百炼免费额度领取步骤、API Token获取方法与付费方案对比

搜“阿里云Tokens怎么获取”的人，通常想办的是两件完全不同的事：要么是想在百炼上调用通义千问这类大模型，缺的是能抵扣调用费用的Token额度；要么是在写代码对接语音识别、网盘这类服务，缺的是用来鉴权的临时Token。两者的获取入口、流程和注意事项完全不同，这篇文章按场景拆开讲，对号入座就行。

> **先给结论**：多数人要的是前者——开通阿里云百炼就能领免费额度，每个模型100万Tokens，所有参与模型累计超过7000万，0成本。👉 [开通阿里云百炼，领取免费Tokens额度](https://www.aliyun.com/product/bailian?userCode=5eqf8rny)。做语音识别、合成开发的，直接看下面的API Token部分。

## 先分清：你要的Token是哪一种

阿里云语境里的“Token”有两个含义，走错入口会白忙半天。

**模型调用额度**：百炼平台上的计费单位。模型每处理一段文字、一张图或一段音频，都会消耗一定数量的Token。免费额度、Token Plan、资源包，买和领的都是这种“用量”。

**API鉴权Token**：智能语音交互、网盘与相册服务等产品发放的临时凭证，调接口时带上它证明权限，一般几小时到一天就过期，过期得重新拿。

判断方法很简单：想跑大模型、搭AI应用，你要的是第一种；在写调接口的代码、报错提示token无效或过期，要的是第二种。

## 百炼免费Tokens领取步骤

领取入口在百炼平台，开通即送每个模型100万Tokens的免费推理额度，参与模型多了之后累计超过7000万。整个流程四步：

1. 打开阿里云百炼页面，登录阿里云账号，点击“免费体验”；
2. 同意服务协议，开通百炼平台；
3. 开通后一般会弹出新人礼包窗口，点击“领取免费额度”；
4. 如果没弹窗，留意页面右上角“新用户登录即享每个模型100万免费Tokens”的横幅，或者主页右下角的图标入口，勾选协议确认开通即可领取。

有的账号额度会自动到账，有的需要手动点一下，两种情况都以页面提示为准。

### 领之前先知道的三个限制

免费额度不等于“7000万随便花”，实际使用有几条硬边界：

- **只抵扣华北2（北京）地域的实时推理调用**。换地域调模型，免费额度不生效。
- 并非所有模型都配了免费额度。文本、图像、音视频场景里只有部分模型参与，具体名单以领取页面展示为准。
- 各模型额度相互独立。某个模型的100万用完了，不会自动从其他模型的额度里扣。

这三条直接影响测试规划：打算认真评估几个模型的话，优先把额度花在消耗最大的调用上。

👉 [前往百炼页面，开通并领取免费额度](https://www.aliyun.com/product/bailian?userCode=5eqf8rny)

## 智能语音交互的API Token怎么获取

三种获取方式，按场景选：控制台领临时Token适合自己测一测，生产环境用SDK自动获取。

控制台领临时Token（仅限测试）：登录智能语音交互控制台，在总览页点击“点击获取临时AccessToken”，当场拿到一个有效期24小时的Token。过期作废，要重新领。

SDK自动获取（生产环境推荐）：Java、C++用nls-sdk-common，Python、Go、Node.js等语言用阿里云公共SDK。把AccessKey ID和AccessKey Secret配置到运行环境的ALIYUN_AK_ID、ALIYUN_AK_SECRET两个环境变量里，SDK会自动调用CreateToken接口获取并刷新Token，过期问题基本不用你操心。

OpenAPI手动请求：所用语言没有现成SDK，或者想自己控制依赖时，直接发带签名的HTTPS请求。请求里Action固定为CreateToken，Version为2019-07-17，签名算法HMAC-SHA1，国内的服务地址是nls-meta.cn-shanghai.aliyuncs.com。响应JSON里的Token.Id就是Token值，Token.ExpireTime是过期时间戳（单位秒），拿这个时间安排缓存和刷新。

### 四个容易踩的坑

- Token都有时效，过期必须重新获取。SDK方案自动处理，手动方案要自己写刷新逻辑。
- 移动端App里不要直接放AccessKey，正确做法是服务端生成Token后下发给App。
- RAM子账号调用报错40020503或“No permission!”，大概率是没授权，给子账号加上AliyunNLSFullAccess权限即可。
- AccessKey Secret和Token都是敏感信息，别提交到公开代码仓库，也别打进日志。

## 网盘、IDaaS这些产品的Token在哪领

除了上面两类，还有几个产品会用到Token，路径各不相同：

- **网盘与相册服务（PDS）**：走标准OAuth 2.0流程。用户授权后拿到授权码，用client_id、client_secret、redirect_uri和这个码调用/v2/oauth/token接口换取access_token，之后可用refresh_token续期。
- **应用身份服务（IDaaS）**：移动端安全认证场景，服务端调用FetchAccessToken接口，传入ApplicationExternalId、UserId等参数后返回access_token。
- **云效对接GitHub/GitLab**：这类Token不在阿里云领。去GitHub或GitLab的个人设置里创建Personal Access Token，勾上repo等所需权限，再填回云效的对接配置。

拿不准自己属于哪种情况，先确认产品名：语音找智能语音交互，大模型找百炼，网盘找PDS，入口完全不同。

## 免费额度用完后，Token怎么买

按量付费是百炼的默认计费方式，用多少扣多少；调用稳定之后，订阅Token Plan或预购资源包能把成本压下来。目前获取Token额度的途径对比如下：

| 获取方式 | 适合谁 | 费用 | 计费方式 | 入口 |
| --- | --- | --- | --- | --- |
| 百炼新人免费额度 | 想先试用的个人 | 0元 | 每个模型100万Tokens | [ 开通领取](https://www.aliyun.com/product/bailian?userCode=5eqf8rny) |
| 按量付费 | 用量波动大、不想预付 | 按实际调用计费 | 用多少扣多少 | [ 开通百炼](https://www.aliyun.com/product/bailian?userCode=5eqf8rny) |
| Token Plan 个人版 | 个人长期调用 | 39元/月起 | 月订阅 | [ 订阅个人版](https://www.aliyun.com/benefit/scene/tokenplan?userCode=5eqf8rny) |
| Token Plan 团队版 | 团队多人使用 | 优惠价150元/月 | 月订阅 | [ 订阅团队版](https://www.aliyun.com/benefit/scene/tokenplan?userCode=5eqf8rny) |
| Coding Plan | 编程、Agent场景 | 以订阅页展示为准 | 月订阅 | [ 查看Coding Plan](https://www.aliyun.com/benefit/scene/codingplan?userCode=5eqf8rny) |
| 资源包 / 节省计划 | 用量大且稳定 | 预购抵扣 | 一次性购买 | [ 查看优惠活动](https://www.aliyun.com/benefit?userCode=5eqf8rny) |

几点补充：

Token Plan覆盖Qwen、DeepSeek、GLM、Wan等多模态模型，支持多Agent并发协作，个人版39元/月起步，团队版活动价150元/月。它和按量付费的区别在于锁成本的方式：订阅制月费固定，用量越大摊得越薄；按量付费灵活，但高峰月份的账单不好预估。

资源包和AI大模型节省计划属于预购模式，适合调用曲线平稳的团队。比较稳的做法是先按量付费跑一两个月，看清自己的月均消耗，再决定订阅还是囤包。

👉 [查看Token Plan个人版与团队版当前价格](https://www.aliyun.com/benefit/scene/tokenplan?userCode=5eqf8rny)

## 常见问题

**免费额度是7000万一次发的吗？**

不是。额度按模型拆分，每个模型100万Tokens免费推理额度，参与模型累计起来超过7000万，而且各模型额度独立不互通。

**语音Token过期了怎么办？**

重新获取即可。走SDK的会自动刷新，代码里不用做额外处理；自己调OpenAPI的，用响应里的ExpireTime提前安排刷新。

**Token Plan和按量付费怎么选？**

判断标准就一条：把最近一两个月的按量账单和订阅价对比，谁低选谁。月消耗长期高于订阅费能覆盖的量，订阅更划算；偶尔用几次，按量付费就行。

## 最后：按你的场景选

- 只想试试通义千问这类大模型：开通百炼领免费额度，一分钱不花，够跑完初步测试；
- 在做语音识别、合成项目：控制台临时Token先跑通流程，上生产换成SDK自动刷新；
- 长期高频调用：按量付费跑一两个月摸清用量，再在Token Plan和资源包之间做选择。

下单前确认两件事：免费额度只抵扣华北2（北京）地域的实时推理调用；Token Plan团队版的150元/月是活动价，订阅前看一眼页面上的当前价格。确认完就可以 👉 [开通阿里云百炼，把免费额度领到手](https://www.aliyun.com/product/bailian?userCode=5eqf8rny)。
