---
trigger: always_on
---
## 1 设计目标

统一命名规范，确保在对外交付、产品设计、开发实施等全生命周期文档与资产中保持一致性与可追溯性。  

**唯一性：** 任一名称在门户内定位到唯一产品/服务/资源  

**可读性：** 见名知意；一眼能看出“哪个门户、哪个产品、哪个服务、什么对象/动作”  

**一致性：** 大小写、分隔符、顺序与缩写统一  

命名核心顺序：公司(Company) -> 租户(Tenant) → 产品/子产品(Product) → 域/子域(Domain) → 环境(Env) → 服务(Service) → 业务模块(Module)   

## 2 术语约定

> 全部仅使用 URL 安全字符

| 命名风格             | 分隔符 | 大小写规则                     | 使用场景示例                                                            |
| -------------------- | ------ | ------------------------------ | ----------------------------------------------------------------------- |
| **flatcase**         | 无     | 全部小写                       | 单一简单的命名                                                          |
| **UPPER FLAT CASE**  | 无     | 全部大写                       | 单一简单的命名                                                          |
| **UPPER_SNAKE_CASE** | `_`    | 全部大写                       | 常量名、枚举常量、SQL 关键字 (`DEFAULT_TIMEOUT_MS`, `USER_ROLE_ADMIN`)  |
| **lower_snake_case** | `_`    | 全部小写                       | 数据库表名、列名、文件名 (`user_id`, `ef_auth_users`)                   |
| **kebab-case**       | `-`    | 全部小写                       | 服务名、URL 路径、NPM 包名 (`ef-auth-service`, `/api/v1/ef/auth/users`) |
| **PascalCase**       | 无     | 首字母大写，每个单词首字母大写 | Java 类名、TypeScript 类名 (`UserController`, `LineageGraphService`)    |
| **lowerCamelCase**   | 无     | 首字母小写，其余单词首字母大写 | Java 方法名、变量名 (`createUser`, `mergeRelations`)                    |
| **dot.case**         | `.`    | 全部小写                       | i18n Key、包名 (`ef.auth.login.title`, `com.cisdi.ef.lineage`)          |

## 3 命名规范

### 3.0 公司命名 Company

- **命名风格**: flatcase
- **格式**：`<company>`
- **规则**：
  - 需要统一注册
  - 全局唯一，单个英文单词或缩写，寓意明确 
- **示例**：`cisdigital`

---

### 3.1 波塞冬项目命名 Project
- **命名风格**: kebab-case
- **格式**：`<project>`
- **规则**：
  - 需要统一注册
  - 全局唯一，寓意明确
- **示例**：`minmetals-finance`、`cloud-cisdigital`、`ciip-private`  

---

### 3.2 产品命名 Product
- **命名风格**: kebab-case
- **格式**：`<product>`
- **规则**：
  - 需要统一注册
  - 顶层产品名全局唯一，寓意明确
  - 子产品名在父产品中唯一，寓意明确
- **示例**：`datakits`、`datapulse`、`easy-checker`  

---

### 3.3 领域命名 Domain
- **命名风格**: flatcase
- **格式**：`<domain>`
- **规则**：同产品下唯一，寓意明确
- **示例**：`finance`、`order`、`settlement`、`di`  

---

### 3.4 租户命名 Tenant

- **命名风格**: kebab-case
- **格式**：`<tenant>`
- **规则**： 全局唯一，使用系统生成的编码
- **示例**：`finance`、`order`、`settlement`、`di`  

---

### 3.5 环境 Env

> 用于某些计算服务集群，需要区分开发和生产等环境

- **命名风格**: kebab-case
- **格式**：`<env>`
- **规则**： 全局唯一，使用系统生成的编码
- **示例**：`dev`、`prod`、`xxx-dev`

---

### 3.6 服务命名 Service
> 服务为独立部署和运维的最小单元   

- **命名风格**: kebab-case
- **格式**：
  - 全名：`<serivce>`
  - 缩写：`<service abbr>`
- **规则**：
  - 需要统一注册
  - 同产品下全名唯一，缩写唯一，表示具体服务名，使用最简单的单词或缩写表示
- **示例**：`data-service`、`data-map`、`open-gateway`  

---

### 3.7 后端项目命名
- **命名风格**: kebab-case
- **格式**：
  - 业务服务：app-`<company>-<product>-<service>` 或 app-`<company>-<product>-<domain>-<service>`
  - starter：
    - springboot自动装配：`<company>-<product>[-<domain>]-<service>`-spring-boot-starter
    - springcloud自动装配`<company>-<product>[-<domain>]-<service>`-spring-cloud-starter
  - 脚手架：`<company>-<product>-<service>`-archtype
  - 产品统一依赖管理：`<company>-<product>-dependencies`
- **规则**：  
  - 业务服务以 `app-` 开头
  - `product`：需要体现父子产品
- **示例**：`app-cisdigital-alexadata-common-ai`、`app-cisdigital-alexadata-datakits-di-etl`、`app-cisdigital-alexadata-datakits-di-cdc`、`app-cisdigital-alexadata-datakits-dataservice`、`cisdigital-elite-forge-storage-spring-boot-starter`、`cisdigital-elite-forge-generator-starter-archetype`  

---

### 3.8 后端 Package 命名
- **命名风格**: dot.case  
- **格式**：
  - 贫血开发模式：`cn.<company>.<product>[.<domain>].<service>[.<maven-layer>][.<business>][.<mvc>]`
  - DDD开发模式：待定  
- **规则**：  
  - 全小写、点分隔  
  - `maven-layer` 是一个单独的maven模块，代表功能边界（如 `biz`、`boot`、`client`、`model`），具体见[Java开发规范](/elite-forge/code-specification/java/v1/)  
  - `mvc` 代表controller、service、repository等标准mvc分层结构  
  - `product`：需要体现父子产品 
  - `business`：代表业务模块
- **示例**：
  - `cn.cisdigital.alexadata.datakits.dataservice.biz`：datakits产品下dataservice服务的biz模块  
  - `cn.cisdigital.alexadata.datakits.dataservice.biz.app.controller`：datakits产品下dataservice服务的biz模块，应用管理领域下的接口  

---

### 3.9 Maven 命名

#### Maven模块名
- **命名风格**: kebab-case 
- **格式**：`<service>-<maven-layer>`
- **规则**:
  - `maven-layer`: 符合Java规范中的Maven分层描述
- **示例**：`ai-biz`、`task-client`

#### GroupId
- **命名风格**: dot.case 
- **格式**：
  - 业务服务：`cn.<company>.<product>[.<domain>]`  
  - starter：`cn.<company>.<product>.starter`  
  - 脚手架：`cn.<company>.<product>.archtype`
- **示例**：`cn.cisdigital.datakits.ai`、`cn.cisdigital.infra.build`

#### ArtifactId
- **命名风格**: kebab-case
- **格式**：
  - 业务服务：`<后端项目命名>-<maven-module>`
  - starter：
    - 有多模块时：`<后端项目命名>-<maven-module>`
    - 无多模块时：`<后端项目命名>`
  - 脚手架：`<后端项目命名>`
  - - 产品统一依赖管理：`<后端项目命名>`
- **示例**：`app-cisdigital-datakits-ai-prompt`、`app-cisdigital-datakits-di-task-client`

#### Properties变量名
- **命名风格**: 自定义
- **格式**：`<artifactId>`.version
- **示例**：`<app-cisdigital-datakits-di-task-client.version>1.0.0</app-cisdigital-datakits-di-task-client.version>`

---

### 3.10 前端项目命名
- **命名风格**: kebab-case
- **格式**：fe-`<company>-<product>-<service>` 或 `<company>-<product>-<domain>-<service>`
- **示例**：`fe-cisdigital-lingshu-admin`

---

### 3.11 前端npm包命名
- **命名风格**: kebab-case
- **格式**：@`<scope>`/`<package name>`
- **规则**：
  - npm 官方文档未强制要求kebab-case，但根据官方示例及社区实践，kebab-case 已成为事实上的标准，可读性与一致性更高
  - 包名必须为 URL 安全字符，不允许空格、驼峰或特殊字符
- **示例**：`@lingshu/dynamic-table`

---

### 3.12 方法命名
- **命名风格**: lowerCamelCase
- **格式**：`<action><Object>`  
- **规则**：  
  - 动宾结构
  - 避免缩写和无意义的动词  
  - 获取可以为空的单个对象的方法用 `get`或`query` 做前缀，并返回Optional
  - 获取不能为空的单个对象的方法用 `getNoneNull` 做前缀
  - 获取多个对象的方法用 `list` 做前缀
  - 分页查询的方法用 `page` 做前缀
  - 批量操作数据的方法用 `batch` 做前缀 
- **示例**：
  - `Optional<User> getUserById(String userId)`
  - `User getNonNullUserById(String userId)`
  - `List<Menu> listMenus(QueryConditionParam param)`
  - `IPage<Menu> pageMenus(QueryConditionParam param)`
  - `void batchInsertUsers(List<Users> users`

---

### 3.13 URL 命名
- **命名风格**: kebab-case
- **格式**：
  - 给前端页面使用的接口：`<auth-gateway prefix>/<product>/<service>/api[/vN]/xxx`  
  - 给服务间调用使用的接口：`<product>/<service>/inner-api[/vN]/xxx`  
  - 给开放平台使用的接口：`<open-gateway prefix>/<product>/<service>/open-api[/vN]/xxx`  
- **规则**：  
  - 禁止restful的path parameter
- **示例**：  
  - 列表：`GET /api/v1/datakits/auth/users`  
  - 单体：`GET /api/v1/datakits/auth/users/{userId}`  
  - 动作：`POST /api/v1/datakits/lineage/relations:merge`

---

### 3.14 缓存 KEY 命名
- **命名风格**: 自定义
- **格式**： 前缀：`[<tenant>:]<product>[:domain]:<service>:<identifier>:<dataType>`
- **规则**：  
  - `:` 分隔层级
  - 避免包含空格和大写字母
  - `dataType` 表示value的数据类型
- **示例**：  
  - 用户详情缓存：`10000:datakits:gateway:user_detail:12345:string`
  - 分布式锁：`lock:datakits:lineage:task:67890`

---

### 3.15 MQ 命名
- **命名风格**: lower_snake_case
- **格式**：
  - Topic：`[<tenant>_]<product>[_<domain>]_<service>_<identifier>`
  - Consumer Group：`[<tenant>_]<product>[_<domain>]_<service>_<identifier>_consumser`
- **示例**：  
  - Topic：`datakits_lineage_update_event`  
  - Consumer Group：`datakits_lineage_update_event_consumer`

---

### 3.16 数据库命名

- **命名风格**: 根据数据库类型，选择lower_snake_case或UPPER_SNAKE_CASE
- **格式**：  
  - Schema：`<product>[_<service>]`  
  - 表：`<service缩写>_<table name>`
  - 多对多关系表： `<service缩写>_<主要表table name>_<从表table name>_relation`
  - 视图：`<service缩写>_v_<view name>`
  - 物化视图：`<service缩写>_mv_<materialize view name>`
  - 唯一约束：`<service缩写>_uk_<table name>_<cols>`
  - 普通索引：`<service缩写>_idx_<table name>_<cols>` 
- **规则**：  
  - 布尔类型强制以下方式来命名
    - `has_xxx` 如，has_permission，拥有/存在型属性，表示是否具有某种资源、权限、元素
    - `can_xxx` 如，can_execute，能力型属性，表示是否可以执行某个动作
    - `xxxable` 如，editable / cacheable，能力/特性型属性，强调“可被…”
    - `must_xxx` 如，must_change_password，约束型属性，表示是否必须满足某条件

- **示例**：  
  - `hr_users` 、`v_one_day_summary`、`hr_users_org_relation`

---

### 3.17 国际化命名

#### 文件命名  

- **命名风格**: lower_snake_case
- **格式**：`<service>_<语言代码>_<国家或地区代码>`
- **示例**：  
  - `resource_zh_CN.properties`
  - `framework_en_US.properties`

#### 国际化KEY命名  

- **命名风格**: dot.case + lower_snake_case
- **格式**：`<product>[.<domain>].<service>.<module>.<type>.<desc>`
- **规则**： 
  - `type`
    - exception：用于抛出异常
    - valid：用于validation的message
    - log：用于日志打印
    - enum：用于枚举描述
  - `module`: 业务模块英文名
  - `desc`: 英文描述
- **示例**：  
  - `datakits.framework.cache.exception.no_semaphore_permits_avaiable=凭证耗尽`
  - `datakits.framework.common.log.log_request_error=请求日志打印异常`

---

### 3.18 ETCD 命名

- **命名风格**：路径分段使用 kebab-case，路径使用 `/` 分隔

- **格式**：  
  - 配置键：`[/<tenant>]/<product>[/<domain>][/<env>]/<service>/config[/<module>]/<identifier>`
  - 服务注册：`[/<tenant>]/<product>[/<domain>][/<env>]/<service>/registry/<instanceId>`
- **规则**：  
  - 配置使用`/config`前缀
  - 注册使用`/registry`前缀
- **示例**：  

应用配置：`/alexadata/datakits/dataservice/config/pool-size → "64"`  

特性开关：`/alexadata/ai/config/prompt/cache-enabled → true`  

服务注册：`/cap/cache-center/registry → 10.0.0.12:8080`

---

### 3.19 FeignClient命名

- **命名风格**：
  - 接口类名：PascalCase
  - @FeignClient(name)：kebab-case
  - @FeignClient(contextId)：kebab-case

- **格式**：
  ```java
  @FeignClient(
    name = "datakits-dataservice",
    contextId = "datakits-dataservice-task-client",
    path = "/datakits/dataservice/inner-api/v1",
    fallbackFactory = xxxFallbackFactory.class
  )
  ```

- **规则**：
  - @FeignClient(name)，保证与服务名一致
  - 同一个服务允许有多个FeignClient，但@FeignClient(contextId)上下文必须不同，把类名转换为kebab-case
  - 避免在 name 中加入环境信息（由配置中心或注册中心区分

- **示例**： 
  ```java
  @FeignClient(
      name = "datakits-dataservice",
      contextId = "datakits-dataservice-task-client",
      path = "/datakits/dataservice/inner-api/v1",
      fallbackFactory = DataServiceFallbackFactory.class
  )
  public interface DataserviceTaskClient {
      @GetMapping("/tasks")
      PageResponse<TaskDto> pageTasks(@RequestParam Map<String, Object> query);

      @PostMapping("/tasks")
      IdResponse createTask(@RequestBody CreateTaskCmd cmd);
  }
  ```

---

### 3.20 Elastic Search 命名

---

### 3.21 图数据库命名

---

### 3.22 Nacos命名


---

### K8S命名

#### configmap

#### secret

#### service account


---

## 4 缩写规范

非必要不缩写，缩写使用以下共识的缩写：  

| 全称                 | 缩写   | 备注说明                   |
| -------------------- | ------ | -------------------------- |
| Application          | app    | 常用于服务、项目名前缀     |
| Configuration        | config | 常用于配置文件、配置类     |
| Information          | info   | 常用于信息类、信息接口     |
| Internationalization | i18n   | 数字表示中间省略的字母数量 |
| Authentication       | auth   | 常用于登录鉴权             |
| Administration       | admin  | 常用于后台管理系统         |
| Function             | func   | 常用于函数式编程场景       |
| Database             | db     | 常用于数据库相关           |
| Transaction          | tx     | 常用于事务处理             |
| Number               | num    | 常用于计数字段             |
| Object               | obj    | 常用于通用对象命名         |
| Identifier           | id     | 常用于唯一标识字段         |
| Request              | req    | 常用于请求参数类           |
| Response             | resp   | 常用于响应结果类           |
| Parameter            | param  | 常用于参数类               |
| Message              | msg    | 常用于消息实体或字段       |
| Index                | idx    | 常用于索引或下标           |