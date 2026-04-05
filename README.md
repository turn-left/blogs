# no name blog

个人技术文档站点，记录后端（Java、Python）和前端（Vue）开发相关的笔记与实践。

## 快速开始

### 本地预览

使用 Docsify CLI 本地预览：

```bash
npx docsify serve ./docs
```

或使用任意静态文件服务器打开 `docs/index.html`。

### 内容结构

```
2026/docs/
├── _coverpage.md    # 封面
├── _navbar.md       # 导航栏
├── _sidebar.md      # 侧边栏
└── index.html       # 入口文件
```

## 写作规范

- 文件名使用小写字母和连字符：如 `java-basics.md`
- 中文文档使用中文标点，代码示例语言不限
- 内部链接使用相对路径：`[Java 基础](backend/java.md)`
- 详细规范见 [AGENTS.md](AGENTS.md)

## 目录结构

- `backend/` - 后端技术（Java、Python等）
- `frontend/` - 前端技术（Vue等）
- 其他分类见侧边栏
