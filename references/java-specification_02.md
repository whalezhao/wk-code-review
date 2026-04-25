---
trigger: always_on
---
## 4 数据库共识

- 没有唯一编码的数据之间，关联关系通过主键id进行关联
- [建议] 有唯一编码的数据之间，关联关系通过唯一编码进行关联
  > 如，血缘数据涉及重新采集，不存在固定id，应使用code进行关联
- sql脚本中的DDL&DML不可以携带SCHEMA
- Entity使用枚举时，数据库存储类型必须与枚举的字段类型一致  
  > 通过枚举最终存储的是整形，但数据库设置的字符串，mybaits框架用人大金仓时就会报数据转换错误  
- 业务相关的数据，非必要不使用数据库自增id（在描述中说明使用自增id的原因）
- [建议] 推荐使用雪花id、时间序列id等，可根据业务自行扩展  
- 业务表必须有主键字段和以下几个审计字段，关联表可不做要求：
  - `id`: 主键，64位整数
  - `create_time`：时间类型，创建时间
  - `update_time`：时间类型，更新时间
  - `create_by`: 字符串类型，长度32，创建人id
  - `create_name`: 字符串类型，长度64，创建人姓名
  - `update_by`：字符串类型，长度32，更新人id
  - `update_name`: 字符串类型，长度64，更新人姓名
- 严禁把审计字段运用到业务逻辑中，业务逻辑涉及到人员的，都要新增字段存储
- 有租户的情况，需要考虑添加 `tenant_id` 字段
- 有删除的情况，需要考虑添加 `archived` 逻辑删除
- 有并发修改的情况，需要考虑增加 `version` 乐观锁版本，代码中执行更新时要添加version条件
- [建议] 若要充分享受代码生成器的便利，数据库存储枚举的字段要严格按照此模式
  - 字段描述中添加：`[枚举]` 这个前缀，方便代码生成器定位
  - 描述字段枚举的值使用JSON格式：`{"code":"中文描述","code":"中文描述",...}`
  - Java中的枚举名 = 转驼峰(数据库中枚举字段名)Enum
  - 完整案例，如，表示任务状态的数据库字段job_state，字段描述为：`[枚举]{"0":"停止","1":"完成","2":"执行中"}`，对应的Java类是modle模块下enums包中的JobStateEnum
- 业务上具有唯一特性的字段，即使是组合字段，也必须建成唯一索引
- 数据订正（特别是删除或修改记录操作）时，要先 select，确认无误才能执行更新语句，并准备好回滚措施

## 5 网关共识  

- 前端的请求路径应该是: `/<业务网关前缀>/<产品名>/<服务名缩写>/api/<queryPath>`

- yaml路由配置做以下约束:
  - 每个服务都需要有唯一的路由前缀（predicates.path）: `/<产品前缀>/<服务名缩写>/**`
  - filters：使用StripPrefix过滤器来剪切掉 `/<产品前缀>` 
  - 路由ID必须使用服务全名作为前缀，如: app-datakits-resource,  app-datakits-resource-2， app-datakits-resource-备份

```yaml
# 门户中心-前端接口
- id: app-cisdigital-base-foundation
  uri: http://app-cisdigital-base-foundation:8080
  predicates:
    - Path=/base/foundation/api/**
  filters:
    - StripPrefix=1

```

### 5.1 业务网关

功能如下：
- 生成request-id
- 校验平台用户token
- 设置请求header

|  Header名   |                  说明                  |               来源               |
| :---------: | :------------------------------------: | :------------------------------: |
| Request-Id  |               请求链路ID               |             链路追踪             |
|   User-Id   | 用户唯一标识，用于数据库查询、日志记录 |           当前登录用户           |
|  User-Name  |    用户的用户名，用于显示和日志记录    |           当前登录用户           |
|  Org-Code   |  用户当前登录的管理组织，用于业务隔离  |           当前登录用户           |
|  Tenant-Id  |                 租户ID                 |              默认值              |
|   Menu-Id   |   菜单ID，用于控制数据权限，门户前端   |             门户前端             |
|   App-Id    |                 应用ID                 |             前端应用             |
| Super-Admin |      是否为超级管理员（boolean）       |           当前登录用户           |
|     Env     |        运行环境（dev、prod等）         | 运行参数，用于大数据平台资源隔离 |
| Session-Id  |                认证信息                |             认证中心             |


### 5.2 开放网关

功能如下：
- 生成request-id
- 校验开放应用
- 校验接口权限
- 设置请求header(待定)

## 6 接口管理共识

使用smart-doc配合Torna完成接口管理。  
在需要上传接口的maven模块，添加 `smart-doc.json` 配置文件。  
参考 `smart-doc.json` 配置：
```
{
  "projectName": "当前项目名",
  "openUrl": "torna地址",
  "appToken": "torna应用token",
  "pathPrefix": "指定为当前服务前缀",
  "packageFilters": "匹配上包的才上传",

  "outPath": "target/api-docs",
  "isStrict": true,
  "showAuthor": false,
  "inlineEnum": true,
  "tornaDebug": true,
  "replace": true
}
```

执行命令上报接口：  
```
mvnw clean -Dfile.encoding=UTF-8 smart-doc:torna-rest
```

## 7 工程共识

> 对内：指针对内部平台、内部微服务  
> 对外：指通过开放平台的形式，对外部环境提供能力

### 7.1 接口前缀

- controller层每个接口路径需添加服务前缀，服务前缀通过常量管理

```java
@RequiredArgsConstructor
@RestController
@RequestMapping(ApiConstants.SERVER + "/api/alert")
public class AlertController {

    private final AlertManager alertManager;

    @PostMapping("/checkDorisAvailable")
    public ResVo<Void> checkDorisAvailable() {
        alertManager.checkDorisAvailable();
        return ResVo.ok();
    }

}
```

- 根据接口的使用方不同，controller层每个接口路径需添加前:  
  - /api 前端交互接口，需要在API网关认证QbeeToken
  - /inner-api 服务间内部调用接口，无需认证
  - /open-api 对外开放接口，需要在开放网关认证应用信息

- [建议] 通过/v1，/v2这种前缀来兼容新老接口  

### 7.2 领域模型
> 需求和产品都不明确，所以现阶段无法采用DDD，盲目DDD只会越来越难维护  

- 项目用传统MVC，贫血模式开发  
- 项目里只使用四种POJO类：Entity、Dto、Param、Vo
  > ORM实体：Entity；数据传输对象(Controller层接收)：Param；数据展示对象(Controller层返回)：Vo；
- 除了Entity都必须实现Serializable接口
- POJO类必须加上jsr 303 validation注解
- repository 入参可以是Param或Dto，出参只能是Entity或者Dto
- service 入参只能是Param和Dto，出参只能是Dto或Vo
  > service入参是Param的话，接口只能在controller使用，其他地方不能使用改方法。
  > service入参是Dto的话，只能在controller以下的层级使用
- 前端接口controller 入参只能是Param，出参只能是Vo
- 内部接口innerController 入参只能是Param，出参只能是Dto


### 7.3 Maven模块与代码分层

> maven命名遵守 [命名规范](/elite-forge/code-specification/name/v1/)

生成器提供以下maven模块：
```
- <后端项目名>
  - <service>-model：      pojo模型与mapstruct转换器
  - <service>-client：     需要发布的内部接口，使用Spring6 HttpExchange定义
  - <service>-xxx-starter：需要发布的业务相关的starter
  - <service>-biz：        高内聚的业务模块，里面再根据业务划分不同的java package
  - <service>-boot：       启动模块，存放启动类和yaml配置
  - test-aggregate：   输出聚合测试分析结果
```

代码分层如下：  
```
- <service>-client:
  - xxxClient
  - yyyClient
- <service>-biz：
  - common
    - exception：该模块定义的异常类
    - util：该模块定义的工具类
  - 模块A
    - controller
      - innerapi
        - xxxInnerController implements xxxClient 服务间内部调用的接口，不需要鉴权
      - api
        - xxxController 给前端使用的接口，需要在API网关进行统一鉴权   
      - openapi
        - xxxOpenController 通过开放平台对外提供能力，需要在开放平台网关进行统一鉴权，对业务代码进行二次封装后，保证向下兼容且稳定  
    - service
      - xxxService： 业务服务类，注入repository实现类，直接写实现类不用定义接口 
    - repository       
      - mapper
        - xxxMapper: mybatis mapper
      - xxxRepository: orm服务类，直接写实现类不用定义接口    
- <service>-model: POJO和Map-Struct转换
  - common
    - converter: map-struct的转换类
    - dto: 内部传输类
    - param: controller入参
    - entity: orm实体映射POJO类
    - vo：controller出参
    - enums：枚举类
  - 模块A
    - converter: map-struct的转换类
    - dto: 内部传输类
    - param: controller入参
    - entity: orm实体映射POJO类
    - vo：controller出参
    - enums：枚举类
```


### 7.4 项目文件说明

```
.
├── .editorconfig             编辑器配置
├── .gitignore                git忽略文件配置
├── .gitlab                   gitlab仓库配置
├──├── merge_request_templates  MR默认模版
├── .gitlab-ci.yml            ci远程配置
├── .mvn                      maven wrapper配置 
├── .pre-commit-config.yaml   本地校验配置
├── xxx-biz                   业务代码模块          
├── xxx-boot                  服务启动模块
├── xxx-model                 领域模型模块
├── xxx-client                服务间调用接口模块
├── lombok.config             lombok配置文件
├── Makefile                  封装了常用的项目构建命令
├── mvnw                      maven wrapper命令
├── mvnw.cmd                  maven wrapper命令 for windows
└── readme.md                 项目说明
```

### 7.5 token鉴权  

- 内部服务采用 `零信任` 机制，所有服务间调用都需要简单的进行内部鉴权
- 禁止微服务自己对用户token进行解析和鉴权，统一从网关层鉴权，鉴权信息由网关设置到http header中，微服务通过拦截器获取header中的鉴权信息  
- 服务间调用透传网关的所有header
- 微服务都使用员工ID（employeeId）作为当前用户，数据库也是存储的员工ID

### 7.6 国际化
> 平时开发，只需要写 `<项目名>.properties` 即可  
> KEY的命名遵守 [命名规范](/elite-forge/code-specification/name/v1/)

国际化文件存放在 `biz模块` 下的，`resources/i18n` 目录下:  
- <项目名>.properties
- <项目名>_en_US.properties
- <项目名>_zh_CN.properties

### 7.7 国产化

#### 数据库适配

> 平时开发，必须要适配oceanbase和dm两种数据库

SQL文件存放在biz模块下的，`resources/mapper/数据库` 目录下:
- mysql: 存放mysql数据库的sql xml
- dm: 存放达梦数据库的sql xml 
- kingbase: 存放人大金仓的sql xml
- oceanbase: 存放OB的sql xml

#### 中间件适配

使用框架提供的防腐层，避免直接使用某个中间件的Client，如RedissonClient。

#### 硬件适配

需要在国产化指定的服务器环境中，构建成功，主流程回归测试完毕。  

