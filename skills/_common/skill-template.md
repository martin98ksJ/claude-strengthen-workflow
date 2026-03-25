# Skill 模板

新建 skill 时复制此模板到 `~/.claude/skills/<新名称>/SKILL.md`。

```yaml
---
name: skill-name
description: 一句话（20-40字）说明这个 skill 解决什么问题
version: "1.0"
---
```

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md

## 输入

- 需要什么上下文（代码文件/设计文档/错误日志/Git diff 等）

## 职责（必须做）

- [ ] 动作 1：具体描述
- [ ] 动作 2：具体描述
- [ ] 动作 3：具体描述
- [ ] 动作 4：具体描述
- [ ] 动作 5：具体描述
（5-8 条，多了说明职责不单一，应拆分）

## 边界（不做什么）

- 不涉及 XXX（明确排除项 1）
- 不涉及 XXX（明确排除项 2）
- 不涉及 XXX（明确排除项 3）
（3-5 条，防止 skill 膨胀成全能助手）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。

---

## 扩展指南

1. 复制本模板到 ~/.claude/skills/<新名称>/SKILL.md
2. 改 frontmatter 的 name/description
3. 填写 输入/职责/边界
4. 保持 ≤ 100 行
5. 边界里引用相关 skill（如"通用后端规范见 backend-conventions"）
