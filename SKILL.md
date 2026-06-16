---
name: blender-model-generator
description: >
  Generate production-ready 3D models in Blender from natural language descriptions.
  Use when users want to create 3D assets, models, animations, or request "generate a model",
  "create 3D asset", "make a blender model", or describe physical objects for 3D modeling.
  Orchestrates 5 specialized agents with USER CONFIRMATION at each step.
  Every AI output is saved as a document for user review/editing before proceeding.
  Do NOT use for 2D graphics, image editing, or when user only needs a description/sketch.
allowed-tools: Agent, ToolSearch, Read, Write, Bash, WebSearch
context: default
---

# Blender Model Generator

AI-driven multi-agent system with **user-in-the-loop** workflow.

## Core Principle

**AI generates documents → User reviews/edits → User confirms → AI proceeds**

---

## Step 1: Requirements Analysis

### 1.1 Run Agent

```python
agent_1_prompt = """
你是 3D 资产生产的技术需求分析师。

用户需求: {user_description}
目标平台: {platform or "通用"}
使用场景: {use_case or "通用"}

你的任务:
1. 在线搜索 3-5 张类似物体的参考图片
2. 分析功能需求（动画、状态变化、交互）
3. 确定必要的组件（分离的对象/部件）
4. 定义技术规格

**重要**: 所有描述、注释、文本字段必须使用简体中文。

返回结构化的 JSON 规格。
"""

spec = agent(
    agent_1_prompt,
    schema=REQUIREMENTS_SCHEMA,
    label="需求分析"
)
```

### 1.2 Write & Show

```python
import json

with open("01_需求规格.json", 'w', encoding='utf-8') as f:
    json.dump(spec, f, indent=2, ensure_ascii=False)

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📄 Step 1 完成: 需求分析"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "已生成文件: 01_需求规格.json"
echo ""
echo "需求摘要:"
echo f"  对象名称: {spec['object_name']}"
echo f"  对象类型: {spec['object_type']}"
echo f"  组件数量: {len(spec['components'])}"
echo ""
echo "⏸️  请检查需求规格"
echo "你可以编辑 01_需求规格.json 来修正任何不准确的地方"
echo ""
echo "完成后，输入 'done' 继续，或 'regenerate' 重新生成"
```

---

## Step 2: Concept Design

### 2.1 Run Agent

```python
# 读取用户确认后的规格
with open("01_需求规格.json", 'r', encoding='utf-8') as f:
    confirmed_spec = json.load(f)

agent_2_prompt = """
你是 3D 概念设计师，专注于功能性资产设计。

规格: {json.dumps(confirmed_spec, indent=2, ensure_ascii=False)}

你的任务:
1. 设计层级结构（父子关系）
2. 规划命名规范: {ObjectType}_{ComponentName}
3. 创建 Blender Python 脚本生成阻塞模型（低精度，正确比例）

**重要**: 
- 设计说明必须使用简体中文
- Python 代码注释必须使用简体中文
- 代码本身（变量名、函数名）可以用英文

返回:
1. 层级结构图（文本树）
2. 设计说明（markdown）
3. 完整的 Blender Python 脚本
"""

design = agent(
    agent_2_prompt,
    label="概念设计"
)
```

### 2.2 Write & Show

```python
# 写入设计文档
with open("02_设计文档.md", 'w', encoding='utf-8') as f:
    f.write(f"# 概念设计\n\n")
    f.write(f"## 层级结构\n\n```\n{design.hierarchy}\n```\n\n")
    f.write(f"## 设计说明\n\n{design.explanation}\n")

# 写入脚本
with open("03_阻塞模型脚本.py", 'w', encoding='utf-8') as f:
    f.write(design.blocking_script)

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📄 Step 2 完成: 概念设计"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "已生成文件:"
echo "  - 02_设计文档.md (设计说明)"
echo "  - 03_阻塞模型脚本.py (Python 脚本)"
echo ""
echo "⏸️  请检查设计"
echo "你可以修改脚本中的尺寸、位置等参数"
echo ""
echo "完成后，输入 'done' 继续"
```

---

## Step 3: Execute & Preview

```python
# 读取确认后的脚本
with open("03_阻塞模型脚本.py", 'r', encoding='utf-8') as f:
    confirmed_script = f.read()

echo "正在 Blender 中生成阻塞模型..."
mcp_tool("execute_blender_code", {"code": confirmed_script})

# 截图
for angle in ['front', 'side', 'perspective']:
    screenshot = mcp_tool("screenshot_blender")
    save_screenshot(screenshot, f"预览_{angle}.png")

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "👀 Step 3 完成: 阻塞模型预览"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "请在 Blender 窗口中查看模型"
echo ""
echo "输入 'proceed' 继续精细化"
echo "输入 'revise' 返回 Step 2 修改"
```

---

## Step 4: Technical Review

```python
agent_3_prompt = """
你是 3D 资产生产流程的技术审查员。

规格: {json.dumps(confirmed_spec, indent=2, ensure_ascii=False)}
设计: {read_file("02_设计文档.md")}

审查清单:
1. 层级结构是否合理？
2. 命名是否清晰一致？
3. 组件是否正确分离？
4. 是否支持所需动画？
5. API 需求是否可实现？
6. 平台兼容性如何？

**重要**: 所有审查内容必须使用简体中文。

返回:
- 状态: "通过" 或 "需要修订"
- 问题列表（如有）
- 建议
"""

review = agent(
    agent_3_prompt,
    schema=REVIEW_SCHEMA,
    label="技术审查"
)

# 写入审查报告
with open("04_审查报告.md", 'w', encoding='utf-8') as f:
    f.write(f"# 技术审查\n\n**状态**: {review.status}\n\n")
    # ...

if review.status == "需要修订":
    echo "返回 Step 2 修订..."
    goto_step_2()
```

---

## Step 5-8: Similar Pattern with Chinese

所有文档都使用中文：
- `05_生产脚本.py` - Python 注释用中文
- `06_质检报告.md` - 全中文
- `api_spec.json` - 描述字段用中文
- `integration_guide.md` - 全中文

---

## File Naming

中文文件名：
- `01_需求规格.json`
- `02_设计文档.md`
- `03_阻塞模型脚本.py`
- `04_审查报告.md`
- `05_生产脚本.py`
- `06_质检报告.md`
- `预览_正面.png`
- `预览_侧面.png`
- `预览_透视.png`

---

## ⚠️ Gotchas

- **所有文档内容必须是中文** - 除了代码本身
- **Python 注释必须是中文** - 方便用户理解和修改
- **JSON 的描述字段必须是中文** - description, notes 等
- **文件名可以用中文** - 现代系统都支持
- **代码变量名可以用英文** - 保持编程习惯

---

## References

- [README.md](./README.md)
- [references/agent-templates.md](./references/agent-templates.md)
