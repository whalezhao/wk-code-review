---
trigger: always_on
---
## 1 协作共识

- 使用Torna接口管理平台
- 使用统一的GIT分支管理模型
- 后端及时提供接口，前端确认无误后，再写实现
- 必须自测核心逻辑后，再发到dev环境联调

## 2 开发共识

### 2.1 序列化

- 后端统一使用jackson进行序列化和反序列化，不要fastjson和jackson混着用
  > 后端使用底层框架提供的：cn.cisdigital.datakits.framework.common.util.JsonUtils

#### 2.1.1 时间字段

- 使用标准时间戳(毫秒为单位milliseconds)来交互，前端自己根据业务需求和<u>**时区**</u>进行格式化展示
  > 前端接口（/api/\**）、feign接口（/inner-api/**）使用的ObjectMapper，统一对LocalDateTime、LocalDate等时间类型序列化为了时间戳

#### 2.1.2 枚举字段

- 前后端交互、数据库存储、手动序列化和反序列化，都使用枚举CODE
  > 如，性别枚举SEX：FEMALE(0)、MALE(1)，我们都使用0和1，而不是枚举名。

#### 2.1.3 Long字段

- Long类型使用字符串交互，防止JS精度丢失，如ID字段
  > 1. 前端接口（/api/\**）、feign接口（/inner-api/**）使用的ObjectMapper，统一对Long类型序列化为了String类型
  > 2. 开放接口（/open-api/**）采用不同的ObjectMapper，开放接口不会处理Long为String

#### 2.1.4 字段无数据

- Json中不能忽略没有数据的字段
- 后端使用null值表示“不存在”，前端必须对null数据进行兼容处理，以避免由于null值导致的错误

### 2.2 请求方式

- 仅允许使用GET和POST请求，禁止Restful风格
  > 考虑到政企方面对安全渗透测试的要求以及服务器和网络的限制，为避免潜在的麻烦，选择一刀切的方式。

### 2.3 请求URL
- URL中的单词之间使用kebab-case，例如：/get-unique-data
- 禁止路径参数，例如：/my/api/123/detail，这样的参数对网关侧统一接口鉴权不友好

### 2.4 请求内容
- 每个请求使用独立的Param类，禁止同一个Param在不同接口中复用
- Param类非必要不继承
- Param类禁止包含冗余或无用的字段
- 禁止在GET请求中传递List
- 三个以上的参数，必须使用POST Request Body
- 通过 request body 传递内容时，必须控制大小  

比如nginx、tomcat、spring中涉及请求大小相关的配置：  

```yaml:no-line-numbers title="Servlet"
spring.servlet.multipart.max-file-size=300MB
spring.servlet.multipart.max-request-size=1000MB
```

```yaml:no-line-numbers title="Web容器"
server.tomcat.max-http-post-size=100000000
server.tomcat.max-swallow-size=100000000
```

```nginx:no-line-numbers title="Nginx"
client_max_body_size 100M;  
```

- 分页请求，参数格式统一为：
```json:no-line-numbers title="分页请求必备参数"
{
  current: "当前页"
  size: "显示条数"
  // 其他业务参数
}
```

### 2.5 响应内容
- 每个请求使用独立的VO类，禁止同一个VO在不同接口中复用
- VO类非必要不继承
- VO类禁止包含冗余或无用的字段
- HTTP接口无论正确还是报错，只要到达服务内部，HTTP状态码都是200
- 服务成功和失败用不同的错误码来表示，错误码要符合`服务状态码`规范
  > "0"代表请求成功，"-1"代表系统异常或未知异常。    
  > 其他情况，返回9位错误码，前1～3位为产品唯一标识，4～6位为服务唯一标识，最后7～9位为各项目自定义的数值，不能重复，从1开始递增，高位补0。

- 统一响应体

```json:no-line-numbers title="返回多条数据"
{
  code: "服务状态码"
  message: "响应消息"
  data: [
    // 列表
  ]
}
```
```json:no-line-numbers title="返回单条数据"
{
  code: "服务状态码"
  message: "响应消息"
  data: {
    // 对象
  }
}
```

```json:no-line-numbers title="返回分页数据"
{
  code: "服务状态码"
  message: "响应消息"
  data: {
    current: "当前页"
    size: "显示条数"
    total: "总数"
    records: [
      //列表
    ]
  }
}
```

### 2.6 国际化

- 使用标准的 `Accept-Language` Header切换语言
- `Accept-Language` Header的值，格式为：`<语言代码>-<国家或地区代码>`，注意是短横线分割
  > 如，zh-CN，en-US
