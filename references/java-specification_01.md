---
trigger: always_on
---
## 1 概述
### 1.1 目的

- 统一团队编码风格，降低维护和代码审查成本
- 减少低级错误，提升代码健壮性
- 通过自动化工具（spotless,jacoco,sonar等）实现高效协作

### 1.2 实施流程

- IDE安装 `SonarLint` 插件，编写代码实时检查
- 通过 `pre-commit` ，触发本地规范校验
- 通过 `gitlab-ci`，触发远程规范校验、代码质量校验
- 通过 `Gitlab MR`，进行代码走查，防止迭代失控  

## 2 风格共识

### 2.1 文件风格

文件风格由 `.editorconfig` 确保:  
- 缩进
  - java: 4个空格
  - yaml: 2个空格
  - properties: 2个空格
  - xml: 2个空格
  - json: 2个空格
  - factories: 2个空格
- 去掉多余的前后空格
- 文件最后一行为空行
- 每一行最大长度：150
- 换行符: LF
- 文件编码：UTF-8

### 2.2 代码风格

采用自定义的 `eclipse jdt format` 统一格式化Java代码，通过 `spotless-maven-plugin` 确保代码能够有效格式化。  

### 2.3 Import风格
 
****vscode 配置****
```json
"java.sources.organizeImports.starThreshold": 5,
"java.sources.organizeImports.staticStarThreshold": 3,
"java.completion.importOrder": [
    "",
    "javax",
    "java",
    "#"
],
```

****IDEA 配置****
![idae import1](/images/code-specification/java/idea-import1.png)  
![idae import2](/images/code-specification/java/idea-import2.png)  
![idea import3](/images/code-specification/java/idea-import3.png)

## 3 编码共识

### 3.1 代码注释

- 标识可以为空和不能为空使用 `JSpecify` 的注解，如：`@NonNull`、`@Nullable`、 `@NullMarked`
- 类、类属性、类方法的注释（特别注意枚举、POJO类、接口、抽象类）必须使用 Javadoc 规范，使用`/** 内容 */` 格式
- JavaDoc只允许使用标准标签，和smartdoc的特殊标签
- 合理通过html标签，保证换行、代码链接等格式
- 禁用 `@author` JavaDoc
- 使用 `@since` 标识新版本添加的内容，值为增加次功能的迭代版本

```java title="标准类注释"
/**
 * 国际化资源加载策略
 *
 * <p>
 * 支持读取jar包及项目内的国际化资源加载器
 * </p>
 *
 * @since 1.0.0
 */
@Slf4j
public class ProjectResourceBundleMessageSource extends ResourceBundleMessageSource {}
```

```java title="标准方法注释"
/**
  * 检查给定的布尔条件，如果不满足条件则抛出业务异常{@link BusinessException}
  *
  * @param condition
  *            条件
  * @param errorCode
  *            错误码枚举 {@link cn.cisdigital.datakits.framework.model.interfaces.ErrorCode}
  * @param msgFormats
  *            消息模板
  * @since 1.0.0
  */
public void checkArgument(boolean condition, ErrorCode errorCode, Object... msgFormats) {}
```

### 3.2 POJO类

- Controller的请求响应统一使用 `cn.cisdigital.elite.forge.infra.commons.model.vo.ResVo<T>` 对象，如，`ResVo<UserVo>`
- 分页Param参数类，直接继承Mybaits-Plus的 `com.baomidou.mybatisplus.extension.plugins.pagination.Page<T>` 类，避免额外转换
- 类名使用 UpperCamelCase 风格，**领域后缀也准守*,如，UserDto而不是UserDTO
- 必须实现Serializable，并定义`serialVersionUID`
- 当类的结构发生变化，更新 `serialVersionUID` 
- 简单POJO类上使用 `@Accessors(chain = true)` 注解
- 复杂POJO类上使用 `@Builder` 注解
- Validation注解的message须写国际化KEY
- POJO 类中的任何布尔类型(基础类型)的变量，都不要加is前缀，否则部分框架解析会引起序列化错误
- 没特殊需求时，字段必须使用包装数据类型
- 每个成员都要有javadoc注释

```java title="标准写法"
@Data
@NoArgsConstructor
@AllArgsConstructor
@Accessors(chain = true)
public class InstantQueryParam implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 波塞冬项目英文名
     */
    @NotBlank(message = "resource.meter.valid.project_not_blank")
    private String project;
    
    /**
     * 查询超时时间
     */
    private String timeout;
}
```

### 3.3 枚举类

- 类名带上 `Enum` 后缀
- 给前端接口和Entity类使用的枚举类，必须 `implements BizEnum`
- `messageKey` 字段使用 `国际化KEY`
- 变量全为final类型
- 成员名称需要全大写
- 单词间用下划线隔开
- 每个成员都要有javadoc注释，特别简单意思明确的可以不写
- 使用@Getter和@RequiredArgsConstructor注解
 
```java title="枚举标准写法"
/**
 * 环境枚举
 *
 * @since 1.0.0
 */
@Getter
@RequiredArgsConstructor
public enum EnvironmentEnum implements BizEnum {

    /**
     * 1 生产环境
     */
    PROD(1, "datakits.framework.model.env.prod"),
    /**
     * 2 开发环境
     */
    DEV(2, "datakits.framework.model.env.dev"),
    ;

    private final int code;
    private final String key;
}
```

### 3.4 工具类

- 造轮子前，先在开发群里问一下有没有类似的，或者开源类似的能力，**不要重复造轮子**  
- 禁用hutool，Apache 、Google 、Spring 发布的工具包已经够用  
- 判断元素是否存在，统一用 `apache commons` :  
  - 判断对象是否存在，可以 object == null、object != null 或 Objects.isNull(object)、Objects.noneNull(object) 或 Optional
  - 判断集合是否存在，统一用apache commons collection4工具类：CollectionUtils.isEmpty(collection)、MapUtils.isEmpty(map)
  - 判断数组是否存在，统一用apache commons lang3工具类：ArrayUtils.isEmpty(array)、ArrayUtils.isNotEmpty(array)
  - 判断String是否存在，统一用apache commons lang3工具类：StringUtils.isBlank(str)、StringUtils.isNotBlank(str)  
- 二维元组使用：`org.apache.commons.lang3.tuple.Pair<L, R>`
- 时间工具：`cn.cisdigital.elite.forge.infra.commons.util.TimeUtils`  
- Spring上下文工具：`cn.cisdigital.elite.forge.infra.commons.util.SpringUtils`
- 序列化工具：`cn.cisdigital.elite.forge.infra.commons.util.JsonUtils`  
- 国际化工具：`cn.cisdigital.elite.forge.infra.commons.util.I18nUtils`  
- 上下文信息: `cn.cisdigital.elite.forge.infra.commons.context.ContextStore`

### 3.5 控制语句

-  非特殊情况下，禁止在循环和递归里调用feign接口以及对单个元素的查询、新增、删除操作，必须封装批量处理接口  
-  禁止复杂嵌套，如果条件不满足，应该立即中止，避免无意义的缩进

```java
public void complexCheck(UserDto userDto) {
    if(userDto == null) {
        return;
    }
    // 后续逻辑
}

```

- 将复杂逻辑判断的结果赋值给一个有意义的布尔变量名，以提高可读性  

```java
boolean invalidProjectName = StringUtils.isNotBlank(projectNamePrefix)
  && StringUtils.isNotBlank(property.getProjectName())
  && !property.getProjectName().startsWith(projectNamePrefix);
if (invalidProjectName) {
    throw new RuntimeException("项目名错误！后端项目名必须以" + projectNamePrefix + "开头");
}
```

### 3.6 日志打印

- 使用 `logback` 作为日志框架
- 通过 `@Slf4j` lombok注解和 `log` 对象来打印日志
- 传统部署方式下，INFO和ERROR分别输入到不同的日志文件；容器化部署方式下，全部日志只打印到控制台
- 禁止出现e.printStackTrace()、System.out相关的日志打印
- 禁止吞并原始异常信息
- 核心业务流程的日志，必须带有唯一识别的关键字，方便直接过滤出来排查问题
- 希望某个特定模块的日志只写入一个独立的日志文件，而不希望它与主日志文件混在一起时，配置 `additivity="false"`

```xml
<logger name="com.example.moduleA" level="DEBUG" additivity="false">
  <appender-ref ref="MODULE_A_FILE" />
</logger>
```

- 涉及外部组件的日志必须打印必要的上下文信息

```java title="体现MQ的详情"
public static fianl String MQ_LOG_TEMPLATE = "[{}] 接收到mq消息, broker={}, topic={}, group={}, message={}";
// 消息队列
log.debug(MQ_LOG_TEMPLATE, "数据质量扫描结果", nameServer, topic, group, message);
```

- 对trace/debug/info级别的日志输出，不允许字符串拼接，必须使用条件输出形式或者使用占位符的方式，避免性能损耗

```java title="日志打印"
logger.debug("Processing trade with id: {} and symbol : {} ", id, symbol);
logger.error("ProcessInstance exec error, id: {} ", id, exception);
```

- **沉默是金**，非必要的日志都使用 `debug` 级别，出问题时通过 `actuator`动态调整日志级别  
> 写之前请思考：这些日志真的有人看吗？看到这条日志能不能给问题排查带来好处？

```bash
curl --location --request POST 'http://<host>/actuator/loggers/cn.cisdigital.datakits.di.task.biz.repository.mapper' \
--header 'Content-Type: application/json' \
--data '{
    "configuredLevel": "DEBUG"
}'
```


### 3.7 并发处理

- 线程安全的类，标记 `javax.annotation.concurrent.ThreadSafe` 注解，没标记的类都当作线程不安全
- 线程资源必须通过线程池提供，不允许直接使用 Executors 去创建（容易OOM），而是通过 ThreadPoolExecutor 的方式
- 需要在线程之间传递上下文时，使用 `TransmittableThreadLocal` 和 `TtlExecutors`
- 创建线程池时，指定有意义的线程名称
- 创建线程池时，需要提供 `Micrometer监控指标`
- 单例类必须无状态，其中的方法也都要线程安全
- 使用 `DateTimeFormatter` 而不是 `SimpleDateFormat`  


### 3.8 Mybatis-Plus

- 禁止用lambdaQuery进行多次的单表查询，复杂查询必须使用xml编写sql，一次性查出需要的数据，避免性能问题
- xml中，没有特殊需求不要用resultMap映射，直接用 `resultType` 映射
- `@TableField` 中，必须**开启全局格式化，且字段名不能包裹引用符**。如：@TableField("`meta_id`")改为@TableField(value = "meta_id", keepGlobalFormat = true)
- 在mapper文件夹下面，创建与数据库同名的文件夹(如mysql，dm)，然后适配所有文件夹中的sql
- 禁用 `@Select`、`@Insert`、`@Update` 注解
- wrapper必须用函数式接口，禁止用String传字段参数
- xml中，如果`<if>`等标签有用到枚举，必须用ognl表达式，不能直接写数字  

```
wt.state = ${@cn.cisdigital.datakits.framework.workflow.abs.constant.TaskInstanceStateEnum@DOING.getCode}
```

### 3.9 事务

- 事务注解加在 public 方法
- 非必要，不用分布式事务
- 分布式事务使用seata实现，只允许使用TCC事务模式，长事务用SAGA模式，尽最大可能避免长事务
- 事务只围住最小必要的数据库读写代码，耗时等无关操作必须从事务脱离
- 非必要优先使用乐观锁（version字段）

### 3.10 Spring

- 禁用 `context-path`，防止融合部署时，url冲突 
- 服务间调用通过 `@HttpExchange` 
- Bean注入统一用 `@RequiredArgsConstructor` 完成构造器注入
- 无特殊情况不要用原生设计模式，要与Spring的IOC和依赖注入结合
- 禁止在controller层写逻辑代码
- 禁止使用 `@Value` 注解读取yaml配置参数，必须写 `Properties` 类
- 有try块放到了事务代码中，catch异常后，如果需要回滚事务，一定要注意手动回滚事务
- 禁止以覆盖的方式，替换底层框架相关的Configuration。只允许通过扩展类或者yaml配置方式修改底层框架行为

```java title="注入标准写法"
@Slf4j
@Service
@RequiredArgsConstructor
public class A {
    private final B b;
}
```

```java title="Spring策略模式"
/**
  * 告警适配器管理器
  * 
  * @since 1.0.0
  */
@Slf4j
@Component
public class AlertAdaptorManager {

    private static final String ALET_LOG_TEMPLATE = "[ ALERT ] 收到告警消息, provider={}, converter={}, handler={}";

    private final Map<String, AlertAdaptor<?>> handlers;

    public AlertAdaptorManager(List<AlertAdaptor<?>> handlers) {
        this.handlers = handlers.stream()
                .collect(Collectors.toMap(AlertAdaptor::name, Function.identity()));
    }

    /**
     * 告警处理统一入口
     * 
     * @param provider  告警消息生产者
     * @param converter 告警消息转换器名字
     * @param handler   告警消息处理器名字
     * @param msg       告警消息
     * @since 1.0.0
     */
    @SuppressWarnings("unchecked")
    public <T> void handle(String provider, String converter, String handler, Map<String, Object> msg) {
        log.info(ALET_LOG_TEMPLATE, provider, converter, handler);
        AlertAdaptor<T> adaptor = (AlertAdaptor<T>) handlers.get(provider);
        if (adaptor == null) {
            throw new AlertException(AlertErrorCode.ALERT_ADAPTOR_NOT_FOUND);
        }
        adaptor.adapt(converter, handler, adaptor.parseMessage(msg));
    }
}
```

### 3.10 Maven

- 使用JDK21驱动Maven
- 使用语义化版本 `主版本号.次版本号.修订版本号`，开发中的版本添加 `-SNAPSHOT` 后缀，正式版不带任何后缀
- 必须使用项目中的maven wrapper
- 引入外部jar包，必须导入公司的外部依赖统一管理，不允许覆盖依赖version
- 引入的业务模块，必须在根pom中的dependencyManagement管理依赖版本
- 正式版本必须排除所有SNAPSHOT依赖
- 使用 `flatten` 插件，统一同一个项目中，多模块的版本
- 禁止私自处理依赖冲突，请联系技术委员会处理

### 3.11 泛型类型

严格按照以下规范使用泛型类型：  
- T: Type（JAVA 类）通用泛型类型，通常作为第一个泛型类型
- S: 通用泛型类型，如果需要使用多个泛型类型，可以将S作为第二个泛型类型
- U: 通用泛型类型，如果需要使用多个泛型类型，可以将U作为第三个泛型类型
- V: 通用泛型类型，如果需要使用多个泛型类型，可以将V作为第四个泛型类型
- E: 集合元素泛型类型，主要用于定义集合泛型类型
- K: 映射-键泛型类型，主要用于定义映射泛型类型
- V: 映射-值泛型类型，主要用于定义映射泛型类型
- N: Number数值泛型类型，主要用于定义数值类型的泛型类型
- ?: 表示不确定的JAVA类型