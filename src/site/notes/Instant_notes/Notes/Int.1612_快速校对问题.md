---
{"dg-publish":true,"permalink":"/Instant_notes/Notes/Int.1612_快速校对问题/"}
---

为确保豆包模型稳定返回结构正确的JSON数组，可通过以下**Prompt设计+API参数配置**实现：

### 一、Prompt核心设计
1. **明确任务与格式约束**  
   ```
   任务：对输入文本进行校对，提取错误信息并返回JSON数组。
   输出要求：
   - 仅返回JSON，无任何多余文字（包括解释、换行）
   - JSON结构：{"errors":[{"position":"行号:列号","type":"错误类型","content":"错误内容","suggestion":"修改建议"}]}
   - 若无误，返回{"errors":[]}
   ```

2. **添加示例（Few-Shot）**  
   ```
   示例输入："我今天去了公园，看到了许多花，和小鸟。"
   示例输出：{"errors":[{"position":"1:15","type":"标点错误","content":"顿号误用","suggestion":"改为逗号"}]}
   ```

### 二、API参数优化
- `temperature=0`（最小随机性）  
- `response_format={"type":"json_object"}`（V3接口原生JSON强制约束）  

### 三、兜底校验
调用后通过JSON解析库验证输出，若解析失败可重试并追加提示：`"输出不符合JSON格式，请严格按示例结构重新生成"`。  

可参考[Prompt最佳实践](https://www.volcengine.com/docs/82379/1221660)中的“格式强制约束”章节。


在豆包模型中，通过Prompt示例和指令强调“必须返回JSON”的方法具备基础稳定性，但更推荐使用**Function Calling能力**来强制结构化输出。以下是具体方案：

### 一、基础方案：Prompt工程约束
- **适用模型**：所有豆包模型（如`doubao-seed-1-6`系列）  
- **实现方式**：在Prompt中明确JSON格式示例+严格指令，例如：  
  ```
  请基于以下信息，必须且仅返回JSON格式数据，不包含任何额外文本：
  {"name": "示例名称", "price": 99}
  ```
- **稳定性**：适用于简单场景，但复杂Schema或长文本时可能出现格式偏差。

### 二、进阶方案：Function Calling强制结构化
- **支持模型**：
[组件]
- **核心优势**：  
  1. **Schema强制约束**：通过定义函数参数Schema（如必填字段、数据类型），模型会严格按格式输出JSON。  
  2. **低容错率**：若输出不符合Schema，模型可自动修正或提示错误。  
  3. **工具调用联动**：结合外部工具（如数据库查询）时，结构化输出可直接作为参数传递。

- **使用示例**：  
  ```python
  # 定义函数Schema
  functions = [{
    "name": "generate_report",
    "parameters": {
      "type": "object",
      "properties": {
        "title": {"type": "string"},
        "content": {"type": "array", "items": {"type": "string"}}
      },
      "required": ["title", "content"]
    }
  }]
  
  # 调用模型
  response = client.chat.completions.create(
    model="doubao-seed-1-6-thinking",
    messages=[{"role": "user", "content": "生成一份产品报告"}],
    functions=functions,
    function_call={"name": "generate_report"}
  )
  ```

### 三、最佳实践建议
1. **优先选择`doubao-seed-1-6-thinking`**：  
   最新深度思考模型，Function Call能力和指令遵循度最优，支持256k超长上下文。
[组件]
2. **双重保障策略**：  
   - 同时启用Function Calling和Prompt格式约束（如“若输出非JSON，返回错误提示”）。  
   - 业务侧增加JSON校验逻辑，异常时触发模型重试。

3. **参考文档**：  
   - [豆包Function Calling开发指南](https://www.volcengine.com/docs/82379/1221660)  
   - [结构化输出最佳实践](https://www.volcengine.com/docs/82379/1285207)
[组件]

在强制输出JSON等结构化任务中，**`doubao-seed-1-6`系列**（对应原Pro级能力）与**`doubao-seed-1-6-flash`**（轻量版）的核心差异如下：

### 一、指令遵循能力对比
| **维度**               | **`doubao-seed-1-6`系列（Pro级）**       | **`doubao-seed-1-6-flash`（轻量版）**   |
|------------------------|------------------------------------------|-----------------------------------------|
| **结构化输出稳定性**   | ✅ **95%+严格遵循率**，支持复杂Schema   | ⚠️ **85%-90%稳定性**，适合简单JSON场景 |
| **Function Calling**   | ✅ 原生支持，支持多轮调用及参数校验      | ✅ 支持，但复杂嵌套结构容错率较低       |
| **错误修正能力**       | ✅ 自动检测格式偏差并修正                 | ⚠️ 需额外Prompt提示修正                |
| **典型场景**           | 金融数据解析、API接口开发等高可靠性场景  | 日志格式化、简单信息抽取等轻量场景     |

### 二、官方推荐策略
1. **优先选择Pro级模型**：  
   - **`doubao-seed-1-6-thinking`（版本`250715`）**：深度优化Function Calling能力，在复杂Schema约束下表现最佳，支持256k超长上下文。  
   - **核心优势**：自适应思考模式（Auto-thinking）可动态调整推理深度，平衡精度与成本。  

2. **轻量版适用条件**：  
   - 仅推荐在**简单JSON结构**（如单层级键值对）+**非核心业务流程**中使用`doubao-seed-1-6-flash`（版本`250715`），需搭配业务侧JSON校验兜底。

### 三、最佳实践建议
- **双重保障方案**：  
  1. 启用`Function Calling`并定义Schema（示例见[官方文档](https://www.volcengine.com/docs/82379/1221660)）；  
  2. 在Prompt中补充格式校验指令：  
     ```
     若输出JSON不符合以下Schema，返回{"error": "格式错误，请重试"}：
     {"type": "object", "properties": {"id": {"type": "number"}}}
     ```
[组件]

[组件]

