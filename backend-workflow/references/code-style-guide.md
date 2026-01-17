# 后端代码规范指南

## Controller 层规范

### 基本结构

```java
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "用户管理", description = "用户相关的 CRUD 操作")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @Operation(summary = "根据ID查询用户", description = "通过用户ID获取用户详细信息")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "查询成功"),
        @ApiResponse(responseCode = "404", description = "用户不存在")
    })
    @GetMapping("/{id}")
    public Result<UserVO> getById(
            @Parameter(description = "用户ID", required = true)
            @PathVariable Long id) {
        return Result.success(userService.getById(id));
    }

    @Operation(summary = "创建用户")
    @PostMapping
    public Result<UserVO> create(
            @Valid @RequestBody CreateUserDTO dto) {
        return Result.success(userService.create(dto));
    }
}
```

### Swagger 注解清单

| 注解 | 位置 | 用途 |
|------|------|------|
| `@Tag` | 类 | 接口分组和描述 |
| `@Operation` | 方法 | 接口说明 |
| `@ApiResponses` | 方法 | 响应状态码说明 |
| `@Parameter` | 参数 | 参数说明 |
| `@Schema` | DTO 字段 | 字段说明 |

---

## Service 层规范

### 基本结构

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;

    @Override
    @Transactional(readOnly = true)
    public UserVO getById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("用户不存在"));
        return userMapper.toVO(user);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public UserVO create(CreateUserDTO dto) {
        // 1. 业务校验
        validateUniqueName(dto.getName());
        
        // 2. 转换实体
        User user = userMapper.toEntity(dto);
        
        // 3. 持久化
        userRepository.save(user);
        
        // 4. 返回结果
        return userMapper.toVO(user);
    }
    
    private void validateUniqueName(String name) {
        if (userRepository.existsByName(name)) {
            throw new BusinessException("用户名已存在");
        }
    }
}
```

### 事务规范

| 场景 | 注解 |
|------|------|
| 只读查询 | `@Transactional(readOnly = true)` |
| 写操作 | `@Transactional(rollbackFor = Exception.class)` |
| 无需事务 | 不加注解 |

---

## DTO/VO 规范

### 请求 DTO

```java
@Data
@Schema(description = "创建用户请求")
public class CreateUserDTO {

    @NotBlank(message = "用户名不能为空")
    @Size(min = 2, max = 50, message = "用户名长度 2-50 字符")
    @Schema(description = "用户名", example = "zhangsan")
    private String name;

    @NotNull(message = "年龄不能为空")
    @Min(value = 0, message = "年龄不能为负数")
    @Max(value = 150, message = "年龄超出范围")
    @Schema(description = "年龄", example = "25")
    private Integer age;
}
```

### 响应 VO

```java
@Data
@Schema(description = "用户信息")
public class UserVO {

    @Schema(description = "用户ID")
    private Long id;

    @Schema(description = "用户名")
    private String name;

    @Schema(description = "创建时间")
    private LocalDateTime createdAt;
}
```

---

## 异常处理规范

### 自定义异常

```java
// 业务异常
public class BusinessException extends RuntimeException {
    private final String code;
    
    public BusinessException(String message) {
        super(message);
        this.code = "BUSINESS_ERROR";
    }
    
    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }
}

// 资源不存在异常
public class NotFoundException extends RuntimeException {
    public NotFoundException(String message) {
        super(message);
    }
}
```

### 全局异常处理

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusinessException(BusinessException e) {
        log.warn("业务异常: {}", e.getMessage());
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(NotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public Result<Void> handleNotFoundException(NotFoundException e) {
        return Result.fail("NOT_FOUND", e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Result<Void> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.joining(", "));
        return Result.fail("VALIDATION_ERROR", message);
    }
}
```

---

## 统一响应格式

```java
@Data
@Schema(description = "统一响应结构")
public class Result<T> {

    @Schema(description = "响应码，0 表示成功")
    private Integer code;

    @Schema(description = "响应消息")
    private String message;

    @Schema(description = "响应数据")
    private T data;

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(0);
        result.setMessage("success");
        result.setData(data);
        return result;
    }

    public static <T> Result<T> fail(String code, String message) {
        Result<T> result = new Result<>();
        result.setCode(-1);
        result.setMessage(message);
        return result;
    }
}
```

---

## 日志规范

```java
// ✅ 正确
log.info("创建用户成功, userId={}, userName={}", user.getId(), user.getName());
log.error("用户查询失败, userId={}", id, e);

// ❌ 错误
log.info("创建用户成功: " + user);  // 字符串拼接
System.out.println("debug: " + data);  // 使用 sysout
```

---

## 数据库规范

### DDL 规范

```sql
-- 表命名：小写下划线
-- 字段命名：小写下划线
-- 必须包含：id, created_at, updated_at
-- 必须添加 COMMENT

CREATE TABLE `user` (
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    `name` VARCHAR(50) NOT NULL DEFAULT '' COMMENT '用户名',
    `status` TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用 1-启用',
    `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `deleted` TINYINT NOT NULL DEFAULT 0 COMMENT '逻辑删除：0-未删除 1-已删除',
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_name` (`name`),
    KEY `idx_status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

### 索引规范

| 类型 | 前缀 | 示例 |
|------|------|------|
| 主键 | - | `PRIMARY KEY` |
| 唯一索引 | `uk_` | `uk_name` |
| 普通索引 | `idx_` | `idx_status` |
| 联合索引 | `idx_` | `idx_status_created_at` |
