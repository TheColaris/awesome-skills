# 单元测试模式参考

## 测试命名规范

使用**中文描述**，清晰表达测试意图：

```java
// ✅ 正确 - 中文描述
@DisplayName("根据有效ID查询_应返回用户信息")
void 根据有效ID查询_应返回用户信息() { ... }

@DisplayName("用户名为空_应抛出参数校验异常")
void 用户名为空_应抛出参数校验异常() { ... }

@DisplayName("删除已注销用户_应抛出业务异常")
void 删除已注销用户_应抛出业务异常() { ... }

// ❌ 错误
void testGetUser() { ... }
void getUserTest() { ... }
```

---

## 测试范围

| 层级 | 是否单测 | 说明 |
|------|----------|------|
| **Service 层 (public 方法)** | ✅ 必须测试 | 核心业务逻辑 |
| **工具类 Utils** | ✅ 必须测试 | 被广泛复用 |
| **Controller 层** | ❌ 不单独测 | 应保持"瘦" |
| **Repository (简单 CRUD)** | ❌ 不单独测 | 通过 Service 间接覆盖 |
| **Repository (复杂 SQL)** | ⚠️ 集成测试 | 多表 JOIN、动态条件 |
| **VO/DTO** | ❌ 不测试 | 无逻辑纯数据结构 |
| **Private/Package 方法** | ❌ 不直接测 | 通过 public 方法间接覆盖 |

---

## AAA 模式 (Arrange-Act-Assert)

```java
@Test
@DisplayName("计算订单总金额_应返回正确金额")
void 计算订单总金额_应返回正确金额() {
    // Arrange - 准备测试数据和依赖
    List<OrderItem> items = List.of(
        new OrderItem("商品A", 100),
        new OrderItem("商品B", 200)
    );
    OrderService service = new OrderService();
    
    // Act - 执行被测方法
    int result = service.calculateTotal(items);
    
    // Assert - 验证结果
    assertThat(result).isEqualTo(300);
}
```

---

## Mock 策略

**所有外部依赖统一 Mock**：

| 依赖类型 | 处理方式 |
|----------|----------|
| DAO/Repository | Mock |
| 外部 RPC/HTTP | Mock |
| Redis/Cache | Mock |
| 当前时间 | Mock |
| 文件系统 | Mock |

### 基本 Mock 模式

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserServiceImpl userService;
    
    @Test
    @DisplayName("根据有效ID查询_应返回用户信息")
    void 根据有效ID查询_应返回用户信息() {
        // Arrange
        User expected = new User(1L, "张三");
        when(userRepository.findById(1L)).thenReturn(Optional.of(expected));
        
        // Act
        UserVO result = userService.getById(1L);
        
        // Assert
        assertThat(result.getName()).isEqualTo("张三");
    }
}
```

### 验证方法调用

```java
@Test
@DisplayName("保存用户_应调用Repository保存方法")
void 保存用户_应调用Repository保存方法() {
    // Arrange
    CreateUserDTO dto = new CreateUserDTO("张三", 25);
    
    // Act
    userService.create(dto);
    
    // Assert
    verify(userRepository, times(1)).save(any(User.class));
}
```

### 时间 Mock

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private Clock clock;
    
    @Test
    @DisplayName("创建订单_应使用当前时间作为创建时间")
    void 创建订单_应使用当前时间作为创建时间() {
        // Arrange
        Instant fixedInstant = Instant.parse("2024-01-15T10:00:00Z");
        when(clock.instant()).thenReturn(fixedInstant);
        
        // Act
        Order order = orderService.create(new CreateOrderDTO());
        
        // Assert
        assertThat(order.getCreatedAt()).isEqualTo(fixedInstant);
    }
}
```

---

## 异常场景测试

**所有异常分支必须覆盖**：

```java
@Test
@DisplayName("用户ID为空_应抛出参数校验异常")
void 用户ID为空_应抛出参数校验异常() {
    // Act & Assert
    assertThrows(IllegalArgumentException.class, () -> {
        userService.getById(null);
    });
}

@Test
@DisplayName("查询不存在的用户_应抛出NotFoundException")
void 查询不存在的用户_应抛出NotFoundException() {
    // Arrange
    when(userRepository.findById(anyLong())).thenReturn(Optional.empty());
    
    // Act & Assert
    assertThrows(NotFoundException.class, () -> {
        userService.getById(999L);
    });
}

@Test
@DisplayName("查询已注销用户_应抛出BusinessException")
void 查询已注销用户_应抛出BusinessException() {
    // Arrange
    User deletedUser = new User(1L, "张三");
    deletedUser.setDeleted(true);
    when(userRepository.findById(1L)).thenReturn(Optional.of(deletedUser));
    
    // Act & Assert
    BusinessException ex = assertThrows(BusinessException.class, () -> {
        userService.getById(1L);
    });
    assertThat(ex.getMessage()).contains("已注销");
}
```

---

## 测试分类

### 1. 正向用例 (Happy Path)

测试正常输入下的预期行为：

```java
@Test
@DisplayName("创建用户_有效参数_应成功创建并返回用户信息")
void 创建用户_有效参数_应成功创建并返回用户信息() {
    // 正常创建用户
}
```

### 2. 边界用例 (Boundary Cases)

测试边界条件：

```java
@Test
@DisplayName("商品列表为空_应返回总金额为0")
void 商品列表为空_应返回总金额为0() {
    // 空列表处理
}

@Test
@DisplayName("用户名长度为100_应成功创建")
void 用户名长度为100_应成功创建() {
    // 最大长度边界
}
```

### 3. 异常用例 (Error Cases)

测试所有异常情况：

```java
@Test
@DisplayName("用户名为空_应抛出参数校验异常")
void 用户名为空_应抛出参数校验异常() {
    // 空值校验
}

@Test
@DisplayName("用户ID不存在_应抛出NotFoundException")
void 用户ID不存在_应抛出NotFoundException() {
    // 资源不存在
}
```

---

## 常用断言 (AssertJ)

```java
// 基础断言
assertThat(result).isEqualTo(expected);
assertThat(result).isNotNull();
assertThat(result).isNull();

// 集合断言
assertThat(list).hasSize(3);
assertThat(list).contains("a", "b");
assertThat(list).isEmpty();

// 异常断言
assertThatThrownBy(() -> service.method())
    .isInstanceOf(RuntimeException.class)
    .hasMessageContaining("错误信息");

// 对象属性断言
assertThat(user)
    .extracting("name", "age")
    .containsExactly("张三", 25);
```

---

## Python (Pytest) 模式

```python
import pytest
from unittest.mock import Mock, patch

class TestUserService:
    
    @pytest.fixture
    def user_repository(self):
        return Mock()
    
    @pytest.fixture
    def user_service(self, user_repository):
        return UserService(user_repository)
    
    def test_根据有效ID查询_应返回用户信息(self, user_service, user_repository):
        # Arrange
        expected = User(id=1, name="张三")
        user_repository.find_by_id.return_value = expected
        
        # Act
        result = user_service.get_by_id(1)
        
        # Assert
        assert result == expected
    
    def test_查询不存在的用户_应抛出异常(self, user_service, user_repository):
        # Arrange
        user_repository.find_by_id.return_value = None
        
        # Act & Assert
        with pytest.raises(NotFoundException):
            user_service.get_by_id(999)
```

---

## 覆盖率目标

| 指标 | 目标 |
|------|------|
| **public 方法覆盖率** | ≥ 85% |
| **异常分支覆盖** | 100% |

### 未达标处理

如果覆盖率 < 85%，需在质量报告中说明原因：

```markdown
| 未覆盖分支 | 原因 | 风险评估 |
|------------|------|----------|
| `validateXxx()` 第3分支 | 依赖外部状态难以构造 | 低风险，建议集成测试覆盖 |
```
