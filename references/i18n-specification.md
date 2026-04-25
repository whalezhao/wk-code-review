---
trigger: always_on
---
## 1 目的与范围

本规范旨在为软件产品研发提供一套统一的国际化（Internationalization, i18n）与本地化（Localization, l10n）开发标准。目标是确保产品能够在不同语言、地区和文化环境下提供一致、准确且用户友好的体验。  

本规范覆盖从UI设计、前后端开发、数据库设计到测试与维护的全过程。  
所有项目成员，包括产品经理、设计师、开发工程师和测试工程师，均需遵循本规范。

**核心原则：**  
- **防止硬编码**： 所有面向用户的文本、标签、消息等内容资源必须与程序代码分离，存储于独立的资源文件中。

- **编码全球统一**： 系统各层面（文件、数据库、代码）必须统一使用 UTF-8 或 UTF8MB4 编码，以支持全球语言字符。

- **时间世界同步**： 所有时间数据在存储和内部处理时，必须使用世界协调时间（UTC），仅在面向用户展示时转换为本地时区时间。

- **设计预留弹性**： 界面设计必须考虑多语言文本长度差异，预留足够空间，避免布局错乱或文本截断。

- **鲁棒性**：若目标语言缺失，可回退到默认语言。  

## 2 多语言支持规范

### 2.1 语言定义与标识

**标识**：`<语言代码>_<国家或地区代码>`  
**规则**：  
- `语言代码`：采用 `ISO 639-1` 标准，由两个小写字母组成。它代表了具体的语言。
  - zh：中文 (Zhongwen)
  - en：英语 (English)
  - ja：日语 (Japanese)
  - fr：法语 (French)

- `国家或地区代码`：采用 `ISO 3166-1` 标准，由两个大写字母组成。它用于区分同一语言在不同地区的变体。
  - CN：中国 (China)
  - US：美国 (United States)
  - JP：日本 (Japan)
  - FR：法国 (France)

**示例**  
zh_CN（简体中文）、en_US（美式英语）、ja_JP（日语）、fr_FR（法语）    

---

### 2.2 资源文件与命名约定

Java后端多语言资源文件路径：
```
src/main/resources/i18n/
```

**文件命名格式**：见[命名规范](/elite-forge/code-specification/name/v1/)  

示例：  
```
cache_center.properties
cache_center_zh_CN.properties
cache_center_en_US.properties
```

**键名规则**：见[命名规范](/elite-forge/code-specification/name/v1/)  

示例：
```
datakits.gateway.cache.exception.no_semaphore_permits_avaiable=凭证耗尽
```

所有资源文件采用 UTF-8 编码，键名在不同语言版本中保持一致。  

---

### 2.3 文本格式与占位符

所有文本采用 Unicode（UTF-8）编码。  
变量内容使用 `{0}`、`{1}` 占位符，参数顺序与代码保持一致。

示例：
```
base.foundation.login.log.welcome=欢迎回来，{0}！您上次登录时间是 {1}
```

复数或条件语句可使用 ICU MessageFormat：
```
base.foundation.storage.log.item_info=您有 {0, plural, one{# 个物品} other{# 个物品}}
```

---

### 2.4 后端国际化实现



Locale 从 `org.springframework.context.i18n.LocaleContextHolder` 中获得：  
```java
Locale locale = LocaleContextHolder.getLocale();
```

通过 Spring 的 `MessageSource` 获取文本：
```java
messageSource.getMessage("user.login.success", new Object[]{username}, locale);
```

也可通过统一框架的 `cn.cisdigital.elite.forge.infra.commons.util.I18nUtils` 获取文本：
```java
I18nUtils.getMessage("user.login.success");
```

异常国际化必须通过ErrorCode：
```java
throw new XxxException(XxxErrorCode.USER_NOT_ALLOWED, new Object[]{username});
```

---

### 2.5 前端国际化实现

前端使用 `vue-i18n` 插件实现国际化。  
模板文本通过 `$t('key')` 获取：

```vue
<el-button>{{ $t('button.submit') }}</el-button>
```

路由与菜单配置示例：
```js
{
  path: '/user',
  name: 'user',
  meta: { title: 'menu.user' }
}
```

动态数据应由后端根据用户语言返回对应文本。  
前端语言包按模块拆分，可懒加载与缓存更新。

---

### 2.6 数据库多语言设计

采用“主表 + 子表”模式，国际化表使用 `_i18n` 作为后缀：

```sql
CREATE TABLE product (
  id BIGINT PRIMARY KEY,
  code VARCHAR(64),
  create_time TIMESTAMP
);

CREATE TABLE product_i18n (
  id BIGINT PRIMARY KEY,
  product_id BIGINT,
  lang VARCHAR(10),
  name VARCHAR(200),
  description TEXT,
  UNIQUE(product_id, lang)
);
```

查询示例：
```sql
SELECT p.*, pi.name, pi.description
FROM product p
LEFT JOIN product_i18n pi
  ON p.id = pi.product_id AND pi.lang = 'en_US';
```

若目标语言缺失，系统回退至默认语言（如 zh_CN）。  

---

### 2.7 UI/UX 多语言要求

1. 界面元素应支持不同语言文本长度（英文一般比中文长 30–50%）。  
2. 控件使用弹性布局或自动换行，避免截断。  
3. 图片与图标中不得嵌入文字。  
4. 支持 RTL（Right-to-Left）语言布局，如阿拉伯语、希伯来语。  
5. 字体需覆盖主要语言字符集与 Emoji。  

---

## 3 多时区支持规范

### 3.1 时间存储与类型

所有时间统一存储为 UTC 格式：
```
2024-05-20T12:00:00Z
```

非特殊情况，代码中都使用不带时区的时间类型，禁止使用带时区的类型：
- LocalDate
- LocalTime
- LocalDateTime
- Instant

JavaScript 使用 `Date` 或 `Intl` API。

---

### 3.2 时间转换与展示

服务器端统一以 UTC 为基准，展示层根据用户时区转换。  
用户时区配置示例：
```
sys_user.timezone = 'Asia/Shanghai'
```

前端时间展示：
```js
moment(utcTime).tz(userTimezone).format('LLL')
dayjs("2013-11-18 11:55").tz("America/Toronto")
```

格式示例：
- 中国：`YYYY-MM-DD HH:mm:ss`  
- 美国：`MM/DD/YYYY hh:mm a`

输入时间示例：  
在北京时间（UTC+8）输入 `2024-05-20 20:00:00`，转换为 `2024-05-20 12:00:00Z` 存储。

---

### 3.3 跨时区任务与日志

1. 日期范围查询前，应将本地时间转换为 UTC。  
   例如：`Asia/Shanghai` 的今日为 `2024-05-20` → UTC `[2024-05-19T16:00:00Z, 2024-05-20T16:00:00Z)`。  
2. 定时任务统一以 UTC 时间执行，避免夏令时影响。  
3. 日志记录统一使用 UTC 时间，并标注 “UTC”：  
   ```
   [2024-05-20T12:00:00Z UTC] Task executed successfully
   ```
4. 日志 MDC 中包含 `locale` 与 `timezone` 字段，便于跨区调试。

---

## 4 工程化与翻译管理

### 4.1 术语治理与安全合规

1. 定义语言白名单，拒绝非法 locale 输入。  
2. 外部语言包纳入版本控制并进行签名校验。  
3. 主数据字典更新时同步维护多语言描述。  
4. 所有术语需通过术语表统一定义，禁止随意翻译。  

---

## 5 OceanBase 数据库补充规范

### 5.1 字符集与排序规则

字符集统一使用 UTF8MB4（基于 Unicode 11.0）。  
推荐排序规则：

- 通用：`UTF8MB4_GENERAL_CI`  
- 精确语言排序：`UTF8MB4_UNICODE_CI`

示例：
```sql
CREATE TENANT IF NOT EXISTS i18n_tenant
  CHARSET='UTF8MB4' COLLATE='UTF8MB4_GENERAL_CI';

CREATE TABLE product (
  id BIGINT PRIMARY KEY,
  ...
) CHARSET=UTF8MB4 COLLATE=UTF8MB4_GENERAL_CI;
```

---

### 5.2 字段与索引设计

短文本使用 `VARCHAR2(N)`，长文本使用 `TEXT` 或 `LONGTEXT`。  
多语言子表需建立唯一索引 `(main_id, lang_code)`。  
可根据查询模式创建单列索引或全文索引：

```sql
CREATE FULLTEXT INDEX ft_idx_product_name_en
ON product_i18n(product_name) WHERE lang_code='en_US';
```

---

### 5.3 多时区数据处理

UTC 时间使用 `TIMESTAMP` 存储。  