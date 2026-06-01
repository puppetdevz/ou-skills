# Skills Index

这是我的技能仓库的索引，按分类整理。

## 分类目录

### 📊 Code Analysis (代码分析)
- **java-code-review** - Java代码审查工具
  - 位置: `categories/code-analysis/java-code-review/`
  - 描述: 对Java代码进行自动审查和提供建议

### 🛠️ Development (开发工具)
- **template-skill** - 技能模板
  - 位置: `categories/development/template-skill/`
  - 描述: 创建新技能的标准模板和示例

- **git-commit-message** - Git提交消息生成器
  - 位置: `categories/development/git-commit-message/`
  - 描述: 根据暂存区变更和仓库提交风格生成提交消息

- **deployment/claude-deploy-service** - GitHub项目一键部署
  - 位置: `categories/development/deployment/claude-deploy-service/`
  - 描述: 一键部署GitHub项目到Docker并自动集成到Homepage

- **harness** - 长期运行Agent Harness系统
  - 位置: `harness/`
  - 描述: 配置长期运行的Agent Harness系统，支持Node.js/Python/Go项目自动检测

- **release** - 通用发布工作流
  - 位置: `release/`
  - 描述: 自动检测版本文件和更新日志，支持Node.js、Python、Rust等多语言项目发布

## 添加新技能

1. 选择合适的分类目录，如果没有则创建新的
2. 在分类目录下创建技能文件夹（使用短横线命名，如 `my-awesome-skill`）
3. 创建 `SKILL.md` 文件（必需）
4. 添加其他可选文件：`reference.md`、`examples.md`、`scripts/`、`templates/`
5. 更新本索引文件

## 技能模板结构

```txt
my-skill/
├── SKILL.md (required)
├── reference.md (optional documentation)
├── examples.md (optional examples)
├── scripts/
│   └── helper.py (optional utility)
└── templates/
    └── template.txt (optional template)
```

---

最后更新: 2026-02-14
