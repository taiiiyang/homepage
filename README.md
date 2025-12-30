# Taiiiyang's Homepage

基于 Hexo 构建的个人技术博客，使用简洁优雅的 Shiro 主题。

## 在线访问

🌐 [https://homepage-lake-theta.vercel.app](https://homepage-lake-theta.vercel.app)

## 技术栈

- **静态站点生成器**: [Hexo](https://hexo.io/) 8.1.1
- **主题**: [Shiro](https://github.com/Acris/hexo-theme-shiro)
- **包管理器**: pnpm
- **部署平台**: [Vercel](https://vercel.com/)
- **版本控制**: Git + GitHub

## 功能特性

- ✨ 简洁优雅的日式设计风格
- 📝 Markdown 写作支持
- 🖼️ 本地图片资源管理
- 🏷️ 文章分类和标签系统
- 📱 响应式设计，支持移动端
- 🚀 自动化部署（推送到 GitHub 自动触发 Vercel 部署）
- 🌏 中文界面

## 本地开发

### 环境要求

- Node.js 14+
- pnpm

### 安装依赖

```bash
pnpm install
```

### 本地预览

```bash
# 启动本地服务器（带热重载）
pnpm run server

# 访问 http://localhost:4000
```

### 生成静态文件

```bash
# 清理缓存
pnpm run clean

# 生成静态文件到 public 目录
pnpm run build
```

## 创建新文章

```bash
# 创建新文章
hexo new post "文章标题"

# 文章将生成在 source/_posts/ 目录下
# 同时会创建同名的资源文件夹用于存放图片
```

### 文章格式

```markdown
---
title: 文章标题
date: 2025-06-03 12:00:00
tags:
  - 标签1
  - 标签2
categories:
  - 分类名
---

文章摘要内容...

<!-- more -->

文章正文内容...

## 插入图片

{% asset_img 图片文件名.png 图片描述 %}
```

## 项目结构

```
homepage/
├── _config.yml              # Hexo 主配置文件
├── _config.shiro.yml        # Shiro 主题配置
├── package.json             # 项目依赖
├── scaffolds/               # 文章模板
├── source/                  # 源文件目录
│   └── _posts/             # 博客文章
│       ├── article-name.md
│       └── article-name/   # 文章图片资源
├── themes/                  # 主题目录
│   └── shiro/              # Shiro 主题
└── public/                  # 生成的静态文件（不提交到 Git）
```

## 部署

项目已配置自动部署到 Vercel：

1. 推送代码到 GitHub 的 `main` 分支
2. Vercel 自动检测变更并触发构建
3. 构建成功后自动部署到生产环境

手动部署命令：

```bash
# 使用 Vercel CLI 部署
vercel --prod
```

## 博客内容

当前博客包含以下技术文章：

- **Canvas 文本截断的实现与优化** - Canvas API、算法优化、二分查找
- **深入理解 V8 的垃圾回收机制** - JavaScript 引擎、内存管理、性能优化

## 配置说明

### 主题配置

主题配置文件位于 `_config.shiro.yml`，主要配置项：

- 导航菜单
- 站点图标
- RSS 订阅
- 评论系统
- 网站统计

### Hexo 配置

主站点配置文件 `_config.yml`，主要配置项：

- 网站标题、作者等基本信息
- URL 和永久链接格式
- 文章资源文件夹设置
- 语言和时区

## 致谢

- [Hexo](https://hexo.io/) - 快速、简洁且高效的博客框架
- [Shiro Theme](https://github.com/Acris/hexo-theme-shiro) - 简洁优雅的 Hexo 主题
- [Vercel](https://vercel.com/) - 现代化的部署平台

## License

MIT
