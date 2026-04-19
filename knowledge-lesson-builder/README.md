# 语文知识点学案生成器 · Claude Skill

为初中语文知识点自动生成完整学案（学生版 + 教师版 + 导航页 + Word 文档），内置课程标准、教材和真题材料读取能力。

## ✨ 功能特点

* 📚 自动读取 `references/` 目录下的课标、教材、模卷库、知识点专项材料

* 🧠 按模块归纳核心能力（认知 → 理解 → 分析 → 应用 → 综合）

* 🖨️ 输出美观的 HTML（学生版/教师版）和 Word 文档（.docx）

* 🧭 自动生成导航页 `index.html`，一键切换版本

* 🌐 生成后自动在浏览器中预览

## 📂 项目结构

knowledge-lesson-builder/
├── knowledge-lesson-builder.md # Skill 主文件（Claude 加载此文件）
├── references/ # 参考材料目录（需用户自行填充）
│ ├── 课程标准.pdf # 义务教育语文课程标准
│ ├── 教材/ # 统编语文教材相关单元
│ ├── 模卷库/ # 中考真题/模拟卷
│ └── 知识点/ # 按知识点存放专项材料
│ └── {知识点名称}/
│ └── {知识点名称}学案/ # 自动生成（含 HTML + Word）
├── README.md
└── .gitignore



## 🚀 安装与使用

### 1. 安装 Claude（或兼容客户端）

确保您使用支持 Claude Skills 的客户端（如 Claude Code、Claude.ai with Projects）。

### 2. 将本仓库导入 Claude Skill 目录

```bash
git clone https://github.com/your-username/knowledge-lesson-builder.git
cd knowledge-lesson-builder
```

