---
trigger: always_on
---
## 错误码构成

错误码由6位状态码组成：
- 前3位为服务标识，每个产品注意多预留点服务区间，如， 某产品占位001～015
- 中间3位为模块标识
- 最后3位为具体异常数值，同一个模块里不能重复，建议从1开始递增，高位补0

## 错误码管理

后端错误码使用枚举管理，需要实现 `cn.cisdigital.elite.forge.infra.commons.interfaces.enums.ErrorCode`。  
如：  
```
/**
 * 认证相关的错误码
 */
@Getter
@RequiredArgsConstructor
public enum AuthErrorCode implements ErrorCode {

    /**
     * 认证异常
     */
    UNAUTHORIZED("注册表中的错误码", "common.exception.unauthorized"),
    ;

    private final String code;
    private final String messageKey;
}
```