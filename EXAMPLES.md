# 示例

展示四条原则的实际代码示例。每个示例展示了 LLM 常犯的错误以及如何修正。

---

## 1. 编码前思考

### 示例 1：隐藏假设

**用户请求：** "添加一个导出用户数据的功能"

**❌ LLM 的做法（错误假设）**

```python
def export_users(format='json'):
    """导出所有用户到文件。"""
    users = User.query.all()

    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])

    return f"Exported {len(users)} users"
```

**问题：**
- 假设应该导出所有用户（分页呢？隐私呢？）
- 没有询问就假设了文件位置
- 假设了要包含哪些字段
- 没有检查实际数据结构就假设了 CSV 字段名

**✅ 正确做法（呈现假设）**

```
在实现之前，我需要澄清几点：

1. **范围**：导出所有用户还是过滤后的子集？（涉及隐私问题）
2. **格式**：你提到"导出"——你的意思是：
   - 在浏览器中下载文件？
   - 后台任务配合邮件通知？
   - 返回数据的 API 端点？
3. **字段**：哪些用户字段？（有些可能是敏感信息）
4. **数据量**：通常有多少用户？（影响方案选择）

最简单的方案：添加一个返回分页 JSON 的 API 端点。
基于文件的导出需要更多信息。

你倾向哪种方式？
```

### 示例 2：多种解释

**用户请求：** "让搜索更快"

**❌ LLM 的做法（默默选择）**

```python
# 添加了缓存、数据库索引和异步处理
# 却没有问哪种"更快"才是重要的
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 行优化代码
```

**✅ 正确做法**

```
"让搜索更快"可能意味着：

1. **更快的响应时间**（< 100ms 而不是 500ms）
   - 添加数据库索引
   - 缓存频繁查询
   - 预估工作量：2 小时

2. **处理更多并发搜索**（吞吐量）
   - 添加 async/await
   - 连接池
   - 预估工作量：4 小时

3. **更快的感知速度**（用户体验）
   - 立即显示部分结果
   - 渐进式加载
   - 预估工作量：3 小时

当前搜索对于典型查询大约需要 ~500ms。
对于你的场景，哪个方面最重要？
```

---

## 2. 简洁优先

### 示例 1：过度抽象

**用户请求：** "添加一个计算折扣的函数"

**❌ LLM 的做法（过度工程）**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage

    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount

    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config

    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)

# 使用时需要 30+ 行配置代码来完成一个简单的计算
```

**✅ 正确做法（简洁）**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """计算折扣金额。percent 应为 0-100。"""
    return amount * (percent / 100)

# 使用
discount = calculate_discount(100.0, 10.0)  # 打折 10
```

**何时添加复杂性：** 只有当你真正需要多种折扣类型时。如果这个需求以后才出现，到时候再重构。

### 示例 2：推测性功能

**用户请求：** "将用户偏好设置保存到数据库"

**❌ LLM 的做法（不必要功能）**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()

    def save(self, user_id: int, prefs: dict,
             merge: bool = True,
             validate: bool = True,
             notify: bool = False) -> bool:
        """保存偏好设置，支持可选的合并、验证和通知。"""

        if validate:
            errors = self.validator.validate(prefs)
            if errors:
                raise ValidationError(errors)

        existing = self.get(user_id) if merge else {}

        if merge:
            existing.update(prefs)
            prefs = existing

        self.db.execute(
            "UPDATE users SET preferences = ? WHERE id = ?",
            (json.dumps(prefs), user_id)
        )

        if self.cache:
            self.cache.set(f"prefs:{user_id}", prefs)

        if notify:
            self.notify_preference_change(user_id, prefs)

        return True

    def notify_preference_change(self, user_id: int, prefs: dict):
        # 没人要求的功能又多了 30 行
        pass
```

**✅ 正确做法（只做要求的）**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """将用户偏好设置保存到数据库。"""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

**需要时再添加：** 缓存（当性能有问题时）、验证（当出现脏数据时）、合并（当需求出现时）。

---

## 3. 精准修改

### 示例 1：顺手重构

**用户请求：** "修复空邮箱导致验证器崩溃的 bug"

**❌ LLM 的做法（改得太多）**

```diff
  def validate_user(user_data):
-     # Check email format
-     if not user_data.get('email'):
+     """Validate user data."""
+     email = user_data.get('email', '').strip()
+
+     # Validate email
+     if not email:
          raise ValueError("Email required")
-
-     # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
          raise ValueError("Invalid email")

-     # Check username
-     if not user_data.get('username'):
+     # Validate username
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("Username required")
+     if len(username) < 3:
+         raise ValueError("Username too short")
+     if not username.isalnum():
+         raise ValueError("Username must be alphanumeric")

      return True
```

**问题：**
- 在 bug 修复之外"改进"了邮箱验证
- 添加了没人要求的用户名验证
- 修改了注释
- 添加了文档字符串

**✅ 正确做法（精准修改）**

```diff
  def validate_user(user_data):
      # Check email format
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
          raise ValueError("Email required")

      # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email:
          raise ValueError("Invalid email")

      # Check username
      if not user_data.get('username'):
          raise ValueError("Username required")

      return True
```

**只修改了：** 修复空邮箱处理的具体几行。

### 示例 2：风格偏移

**用户请求：** "给上传函数添加日志"

**❌ LLM 的做法（全部重新格式化）**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """Upload file to destination with logging."""
+     logger.info(f"Uploading {file_path} to {destination}")
+
      try:
-         with open(file_path, 'rb') as f:
-             data = f.read()
-
-         response = requests.post(destination, files={'file': data})
-
-         if response.status_code == 200:
-             return True
-         else:
-             return False
+         with open(file_path, "rb") as f:
+             data = f.read()
+         response = requests.post(destination, files={"file": data})
+         success = response.status_code == 200
+
+         if success:
+             logger.info(f"Upload successful: {file_path}")
+         else:
+             logger.error(f"Upload failed: {file_path}, status={response.status_code}")
+
+         return success
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f"Upload error for {file_path}: {e}")
          return False
```

**问题：**
- 改了引号风格（'' → ""）
- 添加了没人要求的类型提示
- 添加了文档字符串
- 重新格式化了空格
- 改变了布尔返回逻辑

**✅ 正确做法（匹配现有风格）**

```diff
+ import logging
+
+ logger = logging.getLogger(__name__)
+
  def upload_file(file_path, destination):
+     logger.info(f'Starting upload: {file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()

          response = requests.post(destination, files={'file': data})

          if response.status_code == 200:
+             logger.info(f'Upload successful: {file_path}')
              return True
          else:
+             logger.error(f'Upload failed: {file_path}, status={response.status_code}')
              return False
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f'Upload error: {file_path}')
          return False
```

**保持了：** 单引号、无类型提示、现有布尔模式、空格风格。

---

## 4. 目标驱动执行

### 示例 1：模糊 vs. 可验证

**用户请求：** "修复认证系统"

**❌ LLM 的做法（模糊方式）**

```
我会修复认证系统，通过以下步骤：
1. 审查代码
2. 识别问题
3. 进行改进
4. 测试改动

[没有明确的成功标准就开始改代码了]
```

**✅ 正确做法（可验证目标）**

```
要"修复认证"，我需要定义成功标准。
我们具体要解决什么问题？

如果问题是"用户修改密码后仍然保持登录状态"：

计划：
1. 编写测试：修改密码 → 验证旧会话失效
   验证：测试失败（重现 bug）

2. 实现：密码修改时使会话失效
   验证：测试通过

3. 检查边界情况：多个活跃会话、并发修改
   验证：额外测试通过

4. 验证无回归：现有认证测试仍然通过
   验证：完整测试套件通过

当前认证的测试覆盖率：[显示覆盖率]
你遇到的具体认证问题是什么？
```

### 示例 2：带验证的多步骤任务

**用户请求：** "给 API 添加限流"

**❌ LLM 的做法（一次性全做）**

```python
# 在一个 300 行的提交中实现了完整的限流，包括 Redis、
# 多种策略、配置系统和监控
# 没有明确的验证步骤
```

**✅ 正确做法（增量加验证）**

```
限流实现计划：

1. 添加基本的内存限流（单个端点）
   验证：
   - 测试：100 个请求 → 前 10 个成功，其余返回 429
   - 手动：curl 端点 11 次，看到限流错误

2. 提取为中间件（应用到所有端点）
   验证：
   - 测试：限流对 /users 和 /posts 都生效
   - 现有端点测试仍然通过

3. 添加 Redis 后端（用于多服务器）
   验证：
   - 测试：应用重启后限流仍然存在
   - 测试：两个应用实例共享限流计数器

4. 添加配置（每个端点不同的限流速率）
   验证：
   - 测试：/search 允许 10次/分钟，/users 允许 100次/分钟
   - 配置文件正确解析

每一步都是独立可验证且可部署的。
从第 1 步开始？
```

### 示例 3：测试先行验证

**用户请求：** "当有重复分数时排序会出错"

**❌ LLM 的做法（未重现就修复）**

```python
# 没有确认 bug 就直接改了排序逻辑
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**✅ 正确做法（先重现）**

```python
# 1. 首先，编写一个能重现问题的测试
def test_sort_with_duplicate_scores():
    """测试当多个项目有相同分数时的排序。"""
    scores = [
        {'name': 'Alice', 'score': 100},
        {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]

    result = sort_scores(scores)

    # Bug：重复分数的排序顺序不确定
    # 多次运行此测试，结果应该一致
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# 验证：运行测试 10 次 → 因排序不一致而失败

# 2. 现在用稳定排序修复
def sort_scores(scores):
    """按分数降序排序，同分按名字升序。"""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# 验证：测试稳定通过
```

---

## 反模式总结

| 原则 | 反模式 | 修正 |
|-----------|-------------|-----|
| 编码前思考 | 默默假设文件格式、字段、范围 | 明确列出假设，寻求澄清 |
| 简洁优先 | 用策略模式做单个折扣计算 | 一个函数就够了，直到复杂性真正被需要 |
| 精准修改 | 修 bug 时顺手改引号风格、加类型提示 | 只修改与报告问题相关的代码行 |
| 目标驱动 | "我会审查并改进代码" | "为 bug X 编写测试 → 让它通过 → 验证无回归" |

## 核心洞察

那些"过度复杂"的例子并不是明显错误的——它们遵循了设计模式和最佳实践。问题在于**时机**：它们在复杂性被需要之前就添加了，这会导致：

- 代码更难理解
- 引入更多 bug
- 实现耗时更长
- 更难测试

而"简洁"版本则是：
- 更容易理解
- 实现更快
- 更容易测试
- 可以在复杂性真正被需要时再重构

**好的代码是简单地解决今天问题的代码，而不是过早地解决明天问题的代码。**
