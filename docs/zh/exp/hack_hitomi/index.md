# 自动破解加密

## 调整记录摘要

::: details
基于文档蓝图草稿/原型，分析并细化出蓝图方案至docs\zh\exp\hack_hitomi\archetypes.md；
中途变量为空值的我后期会手动填入你少管；

1. **移除实现细节**：删除所有具体的代码逻辑，如条件判断、日志输出、错误处理等
2. **保留核心接口**：只展示类的主要方法名、参数和返回值
3. **添加方法文档**：在每个方法的 docstring 中提供详细的功能描述和实现指导，这样其他开发者或AI助手可以根据这些描述生成具体的实现代码
4. **突出架构设计**：重点展示模块间的关系和数据流，而不是具体实现

目标是让这个部分成为一个清晰的架构蓝图，其他人可以根据方法文档中的描述来实现具体功能，而不需要在设计文档中看到具体的代码实现。

蓝图方案继续调整：
1.HitomiDecryptor里所有模块的工作职责是不变的，即它仅作为解密的存在！所以update_decrypt_logic和validate_decrypt_result并不是放进HitomiDecryptor去处理！我做这个蓝图的目的就是让蓝图生成更新模块代码，从而把HitomiDecryptor更新解密;
2.数据流和模块交互的解密流程也是错的，更新解密 和 正常解密流程是两回事分清楚;
3.没有批量解密这种自制扩展的方法，原解密HitomiDecryptor接口保持原样;

继续讨论改进方案：
1.mcp是必须的，mcp接收的大模型输入语句大概是”当前图片hash为12345的真实url_new为https://aabb/cc/dd/12345.webp，给我更新加密“这样的语句；
2.mcp接收先HitomiUtils(conf)("12345")排查；
3.与url_new不一致则触发更新解密流程；
4.mcp的验证并不是浏览器环境检验而是直接对比ai输入的真实url,减轻业务复杂度！
5.MCP获取common.js等等都不需要Playwright，都是定值的直接用httpx请求获取响应就行，只需要我在mcp上给common.js等url变量设值即可；

按此应用到docs\zh\exp\hack_hitomi\archetypes.md描述上

蓝图关于JS变化检测的处理过于乐观，gg类的m_cases和b值变化根本不值一提，本来class gg对其就已经动态捕获并处理了；
真正难点在于common.js的各种函数变化，从hash到出真实url的真正调用链，以及与python已实现的Decrypt相关的变动需求；

---

请按照以下要求修改Hitomi解密更新系统的蓝图文档和main.py实现：

**蓝图文档修改要求：**
1. 将蓝图中的代码示例替换为功能描述性文档
2. 使用自然语言描述HitomiDecryptUpdater类的方法职责和设计原则
3. 不要在蓝图中展示具体的Python代码实现细节
4. 重点描述方法的输入、输出和核心逻辑流程

**main.py代码重构要求：**
1. **简化调用链**：将_call_hitomi_decrypt方法的实现简化为最少的步骤，消除不必要的中间方法调用
2. **保留importlib机制**：必须继续使用importlib.import_module()动态导入HitomiUtils模块，这是不可妥协的要求
3. **移除冗余日志**：删除所有无意义的信息打印语句，只保留关键错误日志
4. **简化异常处理**：避免过度的try-catch嵌套，采用简洁直接的错误处理方式
5. **代码可读性**：确保每个方法的逻辑清晰直接，避免"非人类写法"的复杂调用链

**具体实现目标：**
- _call_hitomi_decrypt方法应该极简地完成所有逻辑
- verify_decrypt方法应该极简地完成验证
- 整个HitomiDecryptUpdater类应该保持简洁明了的结构
- 保持importlib动态导入的核心机制不变

:::

## 蓝图草稿/原型
::: detail
章节出图片hash是死的一直是那样，所以设为resp_img_hash;

## mcp调用Playwright获取实时的信息
### 获取加密内容
加密的js模块 commom.js
commom_js_url = ""
### 获取验证正确性的资源
section_url = ""
点开章节页获得图片的真正url

## mcp服务
### 1.跑本地原破解并进行对比
```python
def ori_hack()
    ...
@mcp.tool()
async def compare()
    ...
```
如一致，直接跳过
### 2.非一致，将common.js对比原已经转换成python的代码
进行回归调整python代码调试
终止点为get_img_url(img_hash)与真正url相同
:::
