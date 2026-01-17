---
name: backend-workflow
description: 从详细设计文档到后端代码落地的标准化工作流。当用户提供详细设计文档并要求生成后端实现时，应使用此 Skill。此 Skill 根据任务规模自适应流程严格程度，支持灵活的质量门禁，输出包含 DDL、代码、单元测试及质量报告的标准化交付物。
---

# Backend Workflow

此 Skill 定义了从详细设计文档到后端代码落地的标准化工作流程，支持任务分级、灵活门禁和真实验证。

## 适用场景

- 用户提供了详细设计文档（功能需求、接口定义等）
- 需要生成后端服务代码（Java/Python）
- 需要确保代码质量、测试覆盖率和文档完备性

## 核心原则

1. **任务分级**：根据变更规模自适应流程严格程度
2. **灵活门禁**：信息缺失时提示风险，由用户决策是否继续
3. **测试驱动**：代码与测试同步编写，覆盖率目标 ≥ 85%
4. **闭环反馈**：将实现细节反哺设计文档

## 检查点清单

以下步骤需要**暂停等待用户确认**后再继续：

| 检查点 | 位置 | 说明 |
|--------|------|------|
| 🔴 信息缺失 | Step 2 | 存在高风险缺失时暂停 |
| 🟡 DDL 确认 | Step 3.1 | DDL 生成后暂停确认 |
| 🟡 API 确认 | Step 3.2 | API 契约生成后暂停确认 |

---

## Step 0: 任务分级评估

在开始执行前，首先评估任务规模，确定适用的流程严格程度：

| 级别 | 判定条件 | 流程适配 |
|------|----------|----------|
| **S (Simple)** | 单字段/单方法修改、Bug 修复、配置变更 | 简化流程，跳过 Step 4、Step 6 |
| **M (Medium)** | 单模块变更、新增接口、单表 CRUD | 标准流程，按需执行各步骤 |
| **L (Large)** | 跨模块重构、多表关联、核心业务变更 | 完整流程，严格执行所有步骤 |

> [!NOTE]
> 评估后输出：`📊 任务级别: [S/M/L] - [简要理由]`

---

## 阶段一：分析与校验

### Step 1: 深度理解 (Deep Comprehension)

读取并深入理解提供的详细设计文档：

- 提取核心业务意图和用户场景
- 识别技术约束和非功能性需求
- 梳理数据流向和状态变更

### Step 2: 完备性检查 (Soft Gate)

检查文档是否包含开发所需的元数据，按**风险等级**分类：

| 检查项 | 描述 | 风险等级 |
|--------|------|----------|
| 字段定义 | 类型、长度、非空限制、默认值 | 🔴 高风险 |
| 枚举/常量 | 状态码、类型枚举的完整列表 | 🔴 高风险 |
| 错误码 | Error Codes 及对应的用户提示信息 | 🟡 中风险 |
| 外部依赖 | 调用的 API/服务/中间件 | 🟡 中风险 |
| 业务规则 | 校验逻辑、边界条件 | 🟡 中风险 |
| 性能约束 | 并发量、响应时间要求 | 🟢 低风险 |

**决策机制 (Soft Gate):**

- **[存在高风险缺失]**: 输出风险提示，**暂停等待用户确认**：

```markdown
⚠️ **发现信息缺失 (Missing Information Detected)**

以下信息缺失可能影响实现质量：

**🔴 高风险缺失（强烈建议补充）:**
- [ ] [缺失项 1：具体描述] — 风险：[可能导致的问题]

**🟡 中风险缺失（建议补充）:**
- [ ] [缺失项 2：具体描述] — 将采用默认策略：[描述]

**请选择：**
1. 补充上述信息后继续
2. 接受风险，使用默认策略继续（我将在实现时做出合理假设并在交付物中注明）
```

- **[仅低风险缺失或信息完备]**: 直接进入阶段二

---

## 阶段二：设计与契约

### Step 3.1: DDL 设计

分析数据存储需求，产出语法严格正确的 DDL 语句。

#### 建表语句模板

```sql
CREATE TABLE `table_name` (
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    `field_name` VARCHAR(255) NOT NULL DEFAULT '' COMMENT '字段说明',
    `status` TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用 1-启用',
    `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `deleted` TINYINT NOT NULL DEFAULT 0 COMMENT '逻辑删除：0-未删除 1-已删除',
    PRIMARY KEY (`id`),
    KEY `idx_field_name` (`field_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='表说明';
```

#### 变更语句模板

```sql
-- 新增字段
ALTER TABLE `table_name` 
ADD COLUMN `new_field` VARCHAR(100) NOT NULL DEFAULT '' COMMENT '新字段说明' AFTER `existing_field`;

-- 修改字段
ALTER TABLE `table_name` 
MODIFY COLUMN `field_name` VARCHAR(200) NOT NULL DEFAULT '' COMMENT '修改后的说明';

-- 新增索引
ALTER TABLE `table_name` 
ADD INDEX `idx_new_field` (`new_field`);

-- 新增唯一索引
ALTER TABLE `table_name` 
ADD UNIQUE INDEX `uk_unique_field` (`unique_field`);
```

> [!IMPORTANT]
> **严禁使用物理外键**。所有关联约束必须在 Service 层通过业务逻辑实现。

#### 🟡 检查点：DDL 确认

DDL 生成后，**暂停等待用户确认**：

```markdown
📋 **DDL 脚本已生成，请确认：**

[DDL 内容]

**请确认：**
1. ✅ 确认无误，继续生成 API 契约
2. ❌ 需要修改（请说明修改点）
```

---

### Step 3.2: API 契约定义

定义或更新 API 接口文档，使用以下简化模板：

#### API 契约模板

```markdown
### [接口名称]

| 属性 | 值 |
|------|-----|
| Method | POST / GET / PUT / DELETE |
| Path | `/api/v1/xxx` |
| 描述 | [功能描述] |

#### Request

| 参数名 | 类型 | 必填 | 描述 | 示例 |
|--------|------|------|------|------|
| name | String | 是 | 名称 | "张三" |

#### Response

| 参数名 | 类型 | 描述 |
|--------|------|------|
| id | Long | 主键ID |
| name | String | 名称 |

#### 错误码

| 错误码 | 描述 |
|--------|------|
| 10001 | 参数校验失败 |

#### Sample

**Request:**
​```json
{ "name": "张三" }
​```

**Response:**
​```json
{ "code": 0, "data": { "id": 1, "name": "张三" } }
​```
```

#### 🟡 检查点：API 契约确认

API 契约生成后，**暂停等待用户确认**：

```markdown
📋 **API 契约已生成，请确认：**

[API 契约内容]

**请确认：**
1. ✅ 确认无误，继续开发
2. ❌ 需要修改（请说明修改点）
```

---

## 阶段三：防御性准备

### Step 4: 旧代码影响面分析

> [!NOTE]
> **S 级任务可跳过此步骤**

识别本次变更可能影响的现有文件或类：

1. **扫描范围**：
   - 被修改的文件
   - 调用被修改类/方法的上游代码
   - 被修改类/方法依赖的下游服务

2. **测试覆盖检查**：
   - 检查受影响的旧逻辑是否有单元测试
   - **L 级任务**：如果缺失测试覆盖，编写**基准测试 (Snapshot Tests)** 锁定当前行为
   - **M 级任务**：记录缺失测试的文件，在交付物中提示风险

---

## 阶段四：开发与 TDD

### Step 5: 代码与测试实现

#### 后端代码规范

**Controller 层**：
- 使用面向对象风格
- 包含完整的 Swagger 注解
- 使用统一响应格式

**Service 层**：
- 业务逻辑集中处理
- 实现字段级别校验
- 处理所有关联约束

#### 代码实现完成后：输出修改报告

代码实现完成后，**输出修改报告**（不暂停）：

```markdown
## 📝 代码修改报告

### 修改文件清单

| 文件路径 | 操作 | 修改内容摘要 |
|----------|------|--------------|
| `XxxController.java` | 新增 | 新增 createXxx 接口 |
| `XxxService.java` | 修改 | 新增 validateXxx 方法 |
| `XxxRepository.java` | 新增 | 新增 findByXxx 方法 |

### 核心修改点

#### 1. XxxController.java
- 新增 `POST /api/v1/xxx` 接口
- 参数校验：name 非空，长度 1-100

#### 2. XxxServiceImpl.java
- 新增 `createXxx(CreateXxxDTO)` 方法
- 业务规则：名称不可重复
- 异常处理：抛出 BusinessException

#### 3. XxxRepository.java
- 新增 `existsByName(String)` 方法
```

#### 单元测试规范

##### 测试范围

| 层级 | 是否单测 | 说明 |
|------|----------|------|
| **Service 层 (public 方法)** | ✅ 必须测试 | 核心业务逻辑 |
| **工具类 Utils** | ✅ 必须测试 | 被广泛复用，出 bug 影响面大 |
| **Controller 层** | ❌ 不单独测 | 应保持"瘦"，无业务逻辑 |
| **Repository (简单 CRUD)** | ❌ 不单独测 | 通过 Service 测试间接覆盖 |
| **Repository (复杂 SQL)** | ⚠️ 集成测试 | 多表 JOIN、动态条件需集成测试验证 |
| **VO/DTO** | ❌ 不测试 | 无逻辑纯数据结构 |
| **Private/Package 方法** | ❌ 不直接测 | 通过 public 方法间接覆盖 |

##### 测试规范

- **框架**：Java 使用 JUnit + Mockito；Python 使用 Pytest
- **模式**：AAA (Arrange-Act-Assert)
- **命名**：**中文描述**，清晰表达测试意图
- **异常分支**：**全部覆盖**，每个异常路径都需要测试用例

##### Mock 策略

**所有外部依赖统一 Mock**，确保只测纯业务逻辑：

| 依赖类型 | 处理方式 |
|----------|----------|
| DAO/Repository | Mock |
| 外部 RPC/HTTP | Mock |
| Redis/Cache | Mock |
| 当前时间 | Mock |
| 文件系统 | Mock |

##### 测试示例

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
        Long userId = 1L;
        User expectedUser = new User(userId, "张三");
        when(userRepository.findById(userId)).thenReturn(Optional.of(expectedUser));
        
        // Act
        UserVO result = userService.getById(userId);
        
        // Assert
        assertThat(result.getName()).isEqualTo("张三");
    }

    @Test
    @DisplayName("查询不存在的用户_应抛出NotFoundException")
    void 查询不存在的用户_应抛出NotFoundException() {
        // Arrange
        when(userRepository.findById(anyLong())).thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(NotFoundException.class, () -> userService.getById(999L));
    }
    
    @Test
    @DisplayName("查询已注销用户_应抛出BusinessException")
    void 查询已注销用户_应抛出BusinessException() {
        // Arrange
        User deletedUser = new User(1L, "张三");
        deletedUser.setDeleted(true);
        when(userRepository.findById(1L)).thenReturn(Optional.of(deletedUser));
        
        // Act & Assert
        assertThrows(BusinessException.class, () -> userService.getById(1L));
    }
}
```

### Step 6: 覆盖率审计

> [!NOTE]
> **S 级任务可跳过此步骤**

#### 覆盖率目标

| 指标 | 目标 | 说明 |
|------|------|------|
| **public 方法覆盖率** | ≥ 85% | 核心指标 |
| **异常分支覆盖** | 100% | 所有异常路径必须覆盖 |

#### 未达标处理

如果覆盖率 < 85%，执行以下流程：

1. **分析原因**：识别未覆盖的分支
2. **记录说明**：在质量报告中注明原因
3. **向下传递**：将未覆盖项作为技术债务记录

```markdown
### 未覆盖分支说明

| 未覆盖分支 | 原因 | 风险评估 | 建议后续处理 |
|------------|------|----------|--------------|
| `validateXxx()` 第3个分支 | 依赖外部状态难以构造 | 低 | 集成测试覆盖 |
```

### Step 7: 回归验证（真实执行）

执行实际测试命令验证代码正确性：

**Java 项目**：
```bash
# 执行单元测试
mvn test -Dtest=XxxServiceTest

# 执行全量测试（L 级任务）
mvn test
```

**Python 项目**：
```bash
# 执行单元测试
pytest tests/test_xxx.py -v

# 执行全量测试（L 级任务）
pytest tests/ -v
```

> [!IMPORTANT]
> 必须执行实际测试命令，记录测试结果（通过/失败），如有失败需修复后重新验证。

---

## 阶段五：交付与闭环

### Step 8: 产出交付物

根据任务级别输出交付内容：

| 交付物 | S 级 | M 级 | L 级 |
|--------|------|------|------|
| 代码修改报告 | ✅ | ✅ | ✅ |
| 代码实现 | ✅ | ✅ | ✅ |
| DDL 脚本 | 按需 | ✅ | ✅ |
| 单元测试 | 按需 | ✅ | ✅ |
| 质量报告 | ❌ | ✅ | ✅ |
| 数据流图 | ❌ | 按需 | ✅ |
| 设计修订建议 | ❌ | 按需 | ✅ |

#### 标准交付格式

```markdown
## 1. 代码修改报告
[修改文件清单和核心修改点]

---

## 2. 代码实现

### 2.1 Controller
[代码内容]

### 2.2 Service
[代码内容]

### 2.3 Repository/DAO
[代码内容]

---

## 3. DDL 脚本
[完整的 SQL 脚本]

---

## 4. 单元测试
[完整的测试代码]

---

## 5. 质量报告

| 指标 | 数值 |
|------|------|
| 测试执行命令 | `mvn test -Dtest=XxxTest` |
| 测试通过状态 | ✅ 全部通过 / ⚠️ N 个失败 |
| 新增测试用例数 | N 个 |
| public 方法覆盖率 | XX% |

### 未覆盖分支说明
| 未覆盖分支 | 原因 | 风险评估 |
|------------|------|----------|
| ... | ... | ... |

---

## 6. 数据流图
[Mermaid.js 时序图或流程图]

---

## 7. 设计修订建议

### 已采用的默认策略
| 场景 | 决策 | 建议补充至设计文档 |
|------|------|---------------------|
| [场景] | [处理方式] | ✅ 建议 |

### 待确认事项
1. [问题描述]
```

### Step 9: 设计反哺 (Feedback Loop)

识别编码过程中做出的、原文档未明确的实现细节：

- 默认值处理策略
- 特定异常的兜底策略
- 边界条件的处理方式
- 隐含的业务规则

输出**设计文档修订建议 (Design Doc Amendment Proposal)**。

---

## 快速通道 (Fast Track)

对于紧急修复或极小改动，可启用快速通道：

**触发条件**：
- 用户明确要求"快速修复"
- 改动范围 < 10 行代码
- 不涉及数据库变更
- 不涉及新增接口

**快速通道流程**：
1. Step 1: 理解需求
2. Step 5: 实现代码（含基础测试）
3. Step 7: 验证测试通过
4. Step 8: 仅输出代码实现

---

## 参考资源

- `references/api-doc-template.md` - API 文档模板
- `references/test-patterns.md` - 单元测试模式参考
- `references/code-style-guide.md` - 代码规范指南
