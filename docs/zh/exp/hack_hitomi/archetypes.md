# Hitomi自动破解加密系统实施方案

> 本文档作为实施参考，将根据讨论持续更新。
> 本蓝图文档更新的触发点包括且不限于：方案实现方向有误、功能无法实现等；

## 1. 系统概述

Hitomi自动破解加密系统是一个基于Python的Web内容解密工具，通过MCP（Model Context Protocol）接收用户提供的真实URL，自动检测现有解密器失效并更新解密逻辑。系统保持现有HitomiUtils解密器接口不变，通过独立的更新器模块来维护解密算法的时效性。

## 2. 技术栈选择

### 2.1 核心功能库

| 功能模块 | 推荐库 | 备选库 | 说明 |
|---------|-------|-------|------|
| MCP协议 | `mcp` | 自定义实现 | 模型上下文协议支持 |
| HTTP请求 | `httpx` | `requests` | 获取common.js等资源 |
| JavaScript解析 | `re` | `ast` | 简单正则解析JS变化 |
| 代码生成 | `ast` | 字符串模板 | 生成Python更新代码 |
| 日志记录 | `loguru` | `logging` | 执行过程记录和调试 |

### 2.2 项目架构设计

**设计理念**:
- **接口保持**：现有HitomiUtils解密器接口完全不变
- **独立更新**：通过独立更新器检测和修复解密失效
- **MCP协作**：用户直接提供真实URL，无需浏览器验证
- **最小复杂度**：避免过度设计，专注核心功能

**Python版本要求**: Python 3.12+

**项目配置文件 (pyproject.toml)**:
```toml
[project]
name = "hack-hitomi"
version = "0.1.0"
description = "Hitomi自动破解加密系统 - 基于MCP的解密更新工具"
requires-python = ">=3.12"

dependencies = [
    "mcp>=0.1.0",
    "httpx>=0.25.0",
    "loguru>=0.7.0",
    "pyyaml>=6.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-asyncio>=0.21.0",
    "black>=23.0.0",
]

[[tool.uv.index]]
url = "https://mirrors.aliyun.com/pypi/simple"
default = true
```

## 3. 系统架构

### 3.1 核心模块设计

#### 3.1.1 系统架构概览

```text
现有解密器：HitomiUtils (__temp/ai/__init__.py)
├── 接口保持不变：get_img_url(img_hash, hasavif=0, preview=None)
└── 解密逻辑：gg类 + Decrypt类

独立更新器：HitomiDecryptUpdater
├── MCP服务：接收用户输入的真实URL
├── 验证器：对比本地解密结果与真实URL
├── JS获取器：通过httpx获取common.js
└── 代码生成器：生成更新代码修改HitomiUtils
```

#### 3.1.2 HitomiUtils 现有解密器（保持不变）

```python
# 位置：__temp/ai/__init__.py
class HitomiUtils(EroUtils, Req):
    """
    现有Hitomi解密器 - 接口完全保持不变

    核心方法：
    - get_img_url(img_hash, hasavif=0, preview=None) -> str
    - 内部类：gg, Decrypt
    """

    def get_img_url(self, img_hash: str, hasavif=0, preview=None) -> str:
        """
        核心解密接口 - 不允许修改
        """
        pass
        
```

#### 3.1.3 HitomiDecryptUpdater 独立更新器

**核心职责**：
- 通过importlib动态导入HitomiUtils模块
- 直接调用解密方法获取本地结果
- 对比本地结果与用户提供的真实URL
- 检测解密失效时执行更新流程

**关键方法设计**：

**verify_decrypt()**：
- 使用importlib.import_module导入HitomiUtils
- 直接调用get_img_url()获取本地解密结果
- 与真实URL进行字符串对比
- 返回布尔值表示是否匹配

**update_decrypt_logic()**：
- 获取最新common.js内容
- 分析JS调用链变化
- 生成Python更新代码
- 应用更新到HitomiUtils模块

**设计原则**：
- 简化调用链，避免冗余的中间方法
- 最小化日志输出，专注核心逻辑
- 保持代码简洁，避免过度封装

#### 3.1.4 MCP服务工具（精简版）

**正确的 MCP API 使用方式**：
```python
from mcp.server.fastmcp import FastMCP

# 创建 MCP 服务实例
mcp = FastMCP("hitomi-decrypt-updater")

# MCP工具函数 - 使用正确的 FastMCP API
@mcp.tool()
async def update_hitomi_decrypt(img_hash: str, real_url: str) -> str:
    """
    更新Hitomi解密逻辑

    用户输入示例：
    "当前图片hash为12345的真实url_new为https://aabb/cc/dd/12345.webp，给我更新加密"

    实现流程：
    1. 直接导入HitomiUtils模块：importlib.import_module()
    2. 调用HitomiUtils(conf).get_img_url(img_hash)获取本地结果
    3. 与real_url对比，如一致则跳过
    4. 如不一致，获取硬编码的common.js URL
    5. 分析JS变化并生成更新代码
    6. 应用更新到HitomiUtils模块
    """
    updater = HitomiDecryptUpdater()

    # 验证是否需要更新
    if updater.verify_decrypt(img_hash, real_url):
        return "解密结果一致，无需更新"

    # 执行更新
    success = await updater.update_decrypt_logic(img_hash, real_url)
    return "更新成功" if success else "更新失败"

# 注意：不包含以下工具（硬编码，无需MCP）：
# - get_common_js: common.js URL硬编码在代码中
# - get_config_info: 配置信息硬编码，无需查询
# - set_common_js_url: URL硬编码，无需动态设置
# - compare_decrypt_result: 内部逻辑，无需暴露为MCP工具
```

### 3.2 数据流和模块交互

#### 3.2.1 正常解密流程（无需更新）

```text
用户调用 → HitomiUtils.get_img_url(img_hash) → 返回解密URL
```

#### 3.2.2 更新解密流程（mcp检测触发）

```text
1. 用户输入 → MCP: "当前图片hash为12345的真实url_new为https://aabb/cc/dd/12345.webp，给我更新加密"
2. MCP解析 → 提取img_hash="12345", real_url="https://aabb/cc/dd/12345.webp"
3. 验证解密 → HitomiUtils.get_img_url("12345") vs real_url
4. 如不一致 → 获取最新common.js (httpx请求)
5. 分析变化 → 对比新旧JS代码
6. 生成更新 → 修改HitomiUtils解密逻辑
7. 验证更新 → 再次调用get_img_url()确认修复
```

### 3.3 系统使用流程

#### 3.3.1 正常使用流程

```python
# 直接使用现有解密器，无需任何修改
from __temp.ai import HitomiUtils

hitomi = HitomiUtils(conf)
decrypted_url = hitomi.get_img_url("example_hash")
```

#### 3.3.2 解密失效时的MCP更新流程

```python
# 用户通过MCP输入
# "当前图片hash为12345的真实url_new为https://aabb/cc/dd/12345.webp，给我更新加密"

# MCP工具自动处理
result = await update_hitomi_decrypt(
    img_hash="12345",
    real_url="https://aabb/cc/dd/12345.webp"
)

# 更新后继续正常使用
hitomi = HitomiUtils(conf)
decrypted_url = hitomi.get_img_url("12345")  # 现在应该返回正确URL
```

#### 3.3.3 MCP服务配置

```python
# MCP服务启动时设置URL变量
mcp_config = {
    "common_js_url": "https://ltn.gold-usergeneratedcontent.net/common.js",
    "domain": "gold-usergeneratedcontent.net"
}
```

## 4. 实现难点与解决方案

### 4.1 JS变化检测（重新评估复杂度）

**问题**：如何准确检测common.js的关键变化

**现实情况**：
- gg类的m_cases和b值变化已被现有代码动态处理（class gg.__init__()），不需要管gg.js
- 真正难点在于common.js中可能出现的新函数和调用链变化
- 从hash到真实URL的整个调用逻辑可能发生根本性改变

**真正的技术挑战**：

- **函数调用链分析**：检测JS中新增或修改的函数调用顺序
- **算法逻辑变化**：识别影响subdomain_from_url、full_path_from_hash等核心方法的变化
- **语义理解**：不仅仅是参数提取，需要理解JS逻辑的语义变化
- **Python映射**：将JS的新逻辑准确转换为Python的Decrypt类实现

### 4.2 代码生成和应用

**问题**：如何安全地更新现有HitomiUtils代码

**解决方案**：

- 生成最小化的更新代码片段
- 使用AST操作确保代码正确性
- 提供回滚机制防止更新失败

### 4.3 验证机制

**问题**：如何确保更新后的解密逻辑正确

**解决方案**：

- 使用用户提供的真实URL进行验证
- 多个测试用例确保稳定性
- 记录更新历史便于调试

## 5. 可行性评估

### 5.1 技术可行性: ✅ 高

- 现有HitomiUtils代码结构清晰，易于分析
- MCP协议提供良好的AI协作支持
- httpx足以处理简单的HTTP请求需求

### 5.2 开发难度: ⚠️ 中等偏高（重新评估）

- **JS语义分析复杂**：需要理解函数调用链的语义变化，不仅仅是参数提取
- **调用链检测困难**：从hash到URL的完整调用路径可能发生根本性改变
- **Python映射挑战**：将JS的新逻辑准确转换为Decrypt类的Python实现
- **动态更新风险**：修改核心解密逻辑存在破坏现有功能的风险

### 5.3 代码量估计（重新评估）

- HitomiDecryptUpdater: ~200行
- MCP工具函数: ~150行
- JS语义分析器: ~300行（复杂度被低估）
- 调用链检测器: ~200行
- 代码生成器: ~250行
- 测试用例: ~400行
- **总计: ~1500行**（vs 之前低估的600行）

### 5.4 开发周期估计（重新评估）

- 基础框架搭建: 2天
- JS语义分析和调用链检测: 5-7天（核心难点）
- Python代码生成和映射: 3-4天
- MCP服务集成: 2天
- 测试和调试: 4-6天
- **总计: 16-21天**（vs 之前低估的4-6天）

## 6. 初版功能范围

### 6.1 核心功能

- MCP接收用户输入的真实URL
- 验证现有解密器是否失效
- 获取最新common.js内容
- 分析JS变化并生成更新代码
- 应用更新到HitomiUtils解密器

## 7. 总结

### 7.1 关键设计原则

- **接口不变**：HitomiUtils解密器接口完全保持原样
- **独立更新**：通过独立的更新器处理解密失效
- **用户驱动**：用户直接提供真实URL，无需自动验证
- **最小复杂度**：避免过度设计，专注核心功能

### 7.2 实施优先级

1. **第一阶段**：实现基础MCP工具和验证逻辑
2. **第二阶段**：完善JS分析和代码生成
3. **第三阶段**：添加错误处理和日志记录
4. **第四阶段**：优化和测试

### 7.3 成功标准

- 用户输入真实URL后，系统能自动修复解密失效
- HitomiUtils.get_img_url()返回正确的解密URL
- 整个更新过程对用户透明，无需手动干预
- 代码量控制在1500行以内，开发周期不超过21天