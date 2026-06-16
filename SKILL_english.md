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

This ensures the final model matches user intent, not just AI interpretation.

---

## When to Use

✅ **Use this skill when:**
- User requests 3D model creation
- User describes a physical object for 3D
- User needs game assets, digital twins, product visualizations

❌ **Do NOT use when:**
- User only wants 2D images or sketches
- User needs image editing
- User only wants a description (no actual 3D file)

---

## Workflow Overview

```
Step 0: Environment Check & Create Workspace
    ↓
Step 1: Requirements Analysis (Agent 1)
    ↓ Write: 01_specification.json
    ↓ Show to user
    ↓ Wait: User reviews/edits → confirms "done"
    ↓
Step 2: Concept Design (Agent 2)
    ↓ Read: 01_specification.json (user-confirmed)
    ↓ Write: 02_design.md, 03_blocking_script.py
    ↓ Show to user
    ↓ Wait: User reviews/edits → confirms "done"
    ↓
Step 3: Execute Blocking Model
    ↓ Read: 03_blocking_script.py (user-confirmed)
    ↓ Execute in Blender
    ↓ Capture screenshots
    ↓ Wait: User approves → "proceed" or "revise"
    ↓
Step 4: Technical Review (Agent 3)
    ↓ Write: 04_review.md
    ↓ Show to user
    ↓ If needs_revision → back to Step 2
    ↓ If approved → continue
    ↓
Step 5: Asset Building (Agent 4)
    ↓ Read: 01_specification.json (final)
    ↓ Write: 05_production_script.py
    ↓ Show to user
    ↓ Wait: User reviews/edits → confirms "done"
    ↓
Step 6: Execute Production & Export
    ↓ Read: 05_production_script.py (user-confirmed)
    ↓ Execute in Blender
    ↓ Export files
    ↓
Step 7: QA & Documentation (Agent 5)
    ↓ Write: 06_qa_report.md, api_spec.json, integration_guide.md
    ↓ Show to user
    ↓
Step 8: Package & Deliver
    ↓ Create final_package/ with all files
```

---

## Step 0: Environment Check & Create Workspace

**Same as before** - check uv, MCP, Blender connection (see current SKILL.md lines 49-330)

Then create workspace:

```python
import os
from datetime import datetime

# Sanitize model name
model_name = user_description[:30].replace(" ", "_").replace("/", "_")
timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
workspace = os.path.expanduser(f"~/blender_models/{model_name}_{timestamp}")

os.makedirs(workspace, exist_ok=True)
os.chdir(workspace)

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📁 工作目录已创建"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo f"位置: {workspace}"
echo ""
echo "所有中间文件将保存在此目录，方便你随时查看和修改。"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

---

## Step 1: Requirements Analysis

### 1.1 Run Agent

```python
agent_1_prompt = """
n**IMPORTANT: All descriptions, notes, and text fields MUST be in Chinese (简体中文).**
You are a technical requirements analyst for 3D asset production.

USER REQUEST: {user_description}
TARGET PLATFORM: {platform or "generic"}
USE CASE: {use_case or "general"}

Your task:
1. Search online for 3-5 reference images of similar objects
2. Analyze functional requirements (animations, state changes, interactions)
3. Determine necessary components (separate objects/parts)
4. Define technical specifications

Return a structured JSON specification.
"""

spec = agent(
    agent_1_prompt,
    schema=REQUIREMENTS_SCHEMA,  # See references/agent-templates.md
    label="Requirements Analysis"
)
```

### 1.2 Write to File

```python
import json

spec_file = "01_specification.json"
with open(spec_file, 'w', encoding='utf-8') as f:
    json.dump(spec, f, indent=2, ensure_ascii=False)

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📄 Step 1 完成: 需求分析"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "已生成文件: 01_specification.json"
echo ""
```

### 1.3 Show to User

```python
# Display key parts
echo "需求规格摘要："
echo f"  对象名称: {spec['object_name']}"
echo f"  对象类型: {spec['object_type']}"
echo f"  组件数量: {len(spec['components'])}"
echo f"  动画数量: {len(spec.get('animations', []))}"
echo f"  导出格式: {spec['export_format']}"
echo ""
echo "参考图片:"
for i, url in enumerate(spec['reference_urls'], 1):
    echo f"  {i}. {url}"
echo ""
echo "组件列表:"
for comp in spec['components']:
    echo f"  - {comp['name']}: {comp['description']}"
    if comp.get('states'):
        echo f"    状态: {', '.join(comp['states'])}"
echo ""
```

### 1.4 Wait for User Confirmation

```python
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "⏸️  请检查需求规格"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo f"完整规格已保存到: {workspace}/{spec_file}"
echo ""
echo "你可以:"
echo "  1. 查看文件内容"
echo "  2. 修改任何不准确的地方"
echo "  3. 添加遗漏的需求"
echo ""
echo "完成后，输入 'done' 继续，或 'regenerate' 重新生成"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

user_input = wait_for_user_input()

if user_input == "regenerate":
    goto_step_1()
elif user_input == "done":
    echo "✅ 需求规格已确认"
    # Continue to Step 2
```

---

## Step 2: Concept Design

### 2.1 Read Confirmed Specification

```python
# Read user-confirmed (possibly edited) specification
with open("01_specification.json", 'r', encoding='utf-8') as f:
    confirmed_spec = json.load(f)

echo "✅ 已读取确认后的需求规格"
```

### 2.2 Run Agent

```python
agent_2_prompt = """
You are a 3D concept designer.

SPECIFICATION: {json.dumps(confirmed_spec, indent=2)}

Your task:
1. Design hierarchical structure (parent-child relationships)
2. Plan naming convention: {ObjectType}_{ComponentName}
3. Create Blender Python script for BLOCKING MODEL (low-detail, correct proportions)

Return:
1. Hierarchy diagram (text tree)
2. Design explanation (markdown)
3. Complete Blender Python script
"""

design = agent(
    agent_2_prompt,
    label="Concept Design"
)
```

### 2.3 Write to Files

```python
# Write design document
with open("02_design.md", 'w', encoding='utf-8') as f:
    f.write(f"# Concept Design\n\n")
    f.write(f"## Hierarchy\n\n```\n{design.hierarchy}\n```\n\n")
    f.write(f"## Explanation\n\n{design.explanation}\n")

# Write blocking script
with open("03_blocking_script.py", 'w', encoding='utf-8') as f:
    f.write(design.blocking_script)

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📄 Step 2 完成: 概念设计"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "已生成文件:"
echo "  - 02_design.md (设计文档)"
echo "  - 03_blocking_script.py (阻塞模型脚本)"
echo ""
```

### 2.4 Show to User

```python
echo "设计概要:"
echo ""
# Show hierarchy
lines = design.hierarchy.split('\n')
for line in lines[:20]:  # First 20 lines
    echo f"  {line}"
if len(lines) > 20:
    echo f"  ... ({len(lines)-20} more lines)"
echo ""
```

### 2.5 Wait for User Confirmation

```python
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "⏸️  请检查概念设计"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "你可以:"
echo "  1. 查看 02_design.md 了解设计思路"
echo "  2. 查看 03_blocking_script.py 了解实现细节"
echo "  3. 修改脚本中的尺寸、位置等参数"
echo ""
echo "完成后，输入 'done' 继续"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

user_input = wait_for_user_input()

if user_input == "done":
    echo "✅ 设计已确认"
    # Continue to Step 3
```

---

## Step 3: Execute Blocking Model

### 3.1 Read Confirmed Script

```python
# Read user-confirmed (possibly edited) script
with open("03_blocking_script.py", 'r', encoding='utf-8') as f:
    confirmed_script = f.read()

echo "✅ 已读取确认后的阻塞模型脚本"
```

### 3.2 Execute in Blender

```python
echo "正在 Blender 中生成阻塞模型..."

result = mcp_tool("execute_blender_code", {
    "code": confirmed_script
})

echo "✅ 阻塞模型已生成"
```

### 3.3 Capture Screenshots

```python
# Camera positions
angles = ['front', 'side', 'top', 'perspective']
screenshots = []

for angle in angles:
    # Position camera
    camera_code = get_camera_code_for_angle(angle)  # See references
    mcp_tool("execute_blender_code", {"code": camera_code})
    
    # Capture
    screenshot = mcp_tool("screenshot_blender")
    screenshot_file = f"preview_{angle}.png"
    save_screenshot(screenshot, screenshot_file)
    screenshots.append(screenshot_file)
    
    echo f"  ✅ 截图: {screenshot_file}"
```

### 3.4 Show Previews & Wait

```python
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "👀 Step 3 完成: 阻塞模型预览"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "已生成预览截图:"
for f in screenshots:
    echo f"  - {f}"
echo ""
echo "同时，你可以在 Blender 窗口中实时查看模型！"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "⏸️  请检查阻塞模型"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "检查要点:"
echo "  1. 整体比例是否正确"
echo "  2. 组件位置是否合理"
echo "  3. 层级关系是否清晰"
echo ""
echo "输入 'proceed' 继续精细化"
echo "输入 'revise' 回到 Step 2 重新设计"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

user_input = wait_for_user_input()

if user_input == "revise":
    echo "📝 返回 Step 2 重新设计"
    goto_step_2()
elif user_input == "proceed":
    echo "✅ 阻塞模型已批准，继续精细化"
    # Continue to Step 4
```

---

## Step 4: Technical Review

### 4.1 Run Agent

```python
agent_3_prompt = """
You are a technical reviewer for 3D asset production.

SPECIFICATION: {json.dumps(confirmed_spec, indent=2)}
DESIGN: {read_file("02_design.md")}
PREVIEW IMAGES: [attached]

Review checklist:
1. Hierarchy: Is structure logical?
2. Naming: Clear and consistent?
3. Modularity: Components properly separated?
4. Animation readiness: Bones/constraints if needed?
5. API feasibility: Can requirements be implemented?
6. Platform compatibility: Any issues?

Return:
- Status: "approved" or "needs_revision"
- Issues list (if any)
- Recommendations
"""

review = agent(
    agent_3_prompt,
    schema=REVIEW_SCHEMA,
    label="Technical Review"
)
```

### 4.2 Write Review & Decide

```python
with open("04_review.md", 'w', encoding='utf-8') as f:
    f.write(f"# Technical Review\n\n")
    f.write(f"**Status**: {review.status}\n\n")
    
    if review.issues:
        f.write(f"## Issues Found\n\n")
        for issue in review.issues:
            f.write(f"### {issue.category} - {issue.severity}\n\n")
            f.write(f"{issue.description}\n\n")
            f.write(f"**Suggestion**: {issue.suggestion}\n\n")

echo "✅ 技术审查完成: 04_review.md"

if review.status == "needs_revision":
    echo "❌ 发现问题，返回 Step 2 修订"
    goto_step_2()
else:
    echo "✅ 审查通过"
```

---

## Step 5-8: Similar Pattern

**Step 5**: Generate production script → Write file → User confirms  
**Step 6**: Read confirmed script → Execute → Export  
**Step 7**: QA → Write documentation  
**Step 8**: Package final_package/  

(Full implementation same as before, see SKILL.md backup)

---

## ⚠️ Gotchas

**User-in-the-loop specific:**
- Always write files before showing
- Always read files before using
- Never skip wait_for_user_input
- Show file paths clearly
- Keep all intermediate files
- Validate user-edited JSON

**Technical:**
- Don't use blender --background
- Split large scripts
- Capture multiple angle screenshots

---

## References

- [README.md](./README.md)
- [references/agent-templates.md](./references/agent-templates.md)
