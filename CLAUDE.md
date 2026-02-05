# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Nobelium 是一个基于 Notion 和 Next.js 构建的静态博客生成器，部署在 Vercel 上。它将 Notion 数据库作为 CMS，通过 Notion API 获取内容并生成静态博客网站。

## 技术栈

- **前端框架**: Next.js 14 (React 18)
- **样式系统**: Tailwind CSS 3.3.5 + PostCSS
- **内容管理**: Notion API 集成
- **渲染引擎**: react-notion-x (用于渲染 Notion 页面)
- **包管理**: pnpm (支持补丁管理)
- **部署平台**: Vercel
- **容器化**: Docker 支持

## 常用开发命令

```bash
# 开发环境启动
pnpm dev

# 构建生产版本
pnpm build

# 启动生产服务器
pnpm start

# 代码检查
pnpm lint

# 构建后生成站点地图
pnpm postbuild
```

## Docker 开发

```bash
# 构建 Docker 镜像
export NOTION_PAGE_ID=xxx
export IMAGE=nobelium:latest
docker build -t ${IMAGE} --build-arg NOTION_PAGE_ID .

# 运行容器
docker run -d --name nobelium -p 3000:3000 -e NOTION_PAGE_ID=${NOTION_PAGE_ID} nobelium:latest
```

## 核心架构

### 1. 目录结构
- `components/` - React 组件库，包含博客的所有 UI 组件
- `pages/` - Next.js 页面路由，使用文件系统路由
- `lib/` - 核心业务逻辑和工具函数
- `lib/notion/` - Notion API 相关的数据获取逻辑
- `lib/server/` - 服务端配置和 API 封装
- `styles/` - 全局样式和 Notion 特定样式
- `assets/i18n/` - 国际化语言文件

### 2. 数据流架构
1. **数据源**: Notion 数据库作为 CMS
2. **数据获取**: 通过 `lib/notion/` 中的函数从 Notion API 获取数据
3. **静态生成**: Next.js ISR (Incremental Static Regeneration) 生成静态页面
4. **渲染**: 使用 react-notion-x 渲染 Notion 内容块

### 3. 关键模块

#### Notion 数据层 (`lib/notion/`)
- `getAllPosts.js` - 获取所有博客文章
- `getPostBlocks.js` - 获取文章内容块
- `getAllTagsFromPosts.js` - 提取文章标签
- `filterPublishedPosts.js` - 过滤已发布文章
- `getPageProperties.js` - 获取页面属性

#### 配置系统
- `blog.config.js` - 主要配置文件，包含站点信息、主题、分析工具等
- `next.config.js` - Next.js 配置
- `tailwind.config.js` - Tailwind CSS 配置

#### 组件系统
- `components/BlogPost.js` - 博客文章组件
- `components/NotionRenderer.js` - Notion 内容渲染器
- `components/notion-blocks/` - 特定 Notion 块类型组件 (Mermaid, Toggle)

### 4. 环境变量配置
必需的环境变量：
- `NOTION_PAGE_ID` - Notion 数据库页面 ID
- `NOTION_ACCESS_TOKEN` - (可选) Notion 访问令牌

### 5. 补丁管理
项目使用 pnpm 的补丁功能对以下包进行了修改：
- `react-notion-x@6.16.0` - 自定义 Notion 渲染行为
- `notion-utils@6.16.0` - 自定义 Notion 工具函数

### 6. 部署和构建
- 使用 Vercel 进行部署，支持 ISR
- 构建后自动生成站点地图 (`next-sitemap`)
- 支持 Docker 容器化部署

## 开发注意事项

1. **Notion API 限制**: Notion token 有效期为 180 天，需要定期更新
2. **图片处理**: 配置了 Gravatar 域名用于头像显示
3. **国际化**: 支持多语言，配置在 `assets/i18n/` 目录
4. **SEO 优化**: 内置 SEO 配置和 Open Graph 支持
5. **评论系统**: 支持 Gitalk、Utterances、Cusdis 多种评论系统
6. **分析工具**: 支持 Google Analytics 和 Ackee
7. **pnpm 版本管理**: 项目使用 pnpm@9.12.3，通过 packageManager 字段锁定版本以确保 Vercel 部署兼容性

## 常见问题和解决方案

### Vercel 部署错误

#### ERR_PNPM_LOCKFILE_CONFIG_MISMATCH
**问题**: Vercel 部署时出现 "Cannot proceed with the frozen installation. The current 'patchedDependencies' configuration doesn't match the value found in the lockfile"

**原因**:
- Vercel 根据项目创建日期使用 pnpm@9.x，但本地使用了不同版本的 pnpm
- pnpm-lock.yaml 中的 patchedDependencies 配置与 package.json 不匹配

**解决方案**:
1. 在 package.json 中明确指定 pnpm 版本：
   ```json
   {
     "engines": {
       "node": "24.x",
       "pnpm": ">=9.0.0"
     },
     "packageManager": "pnpm@9.12.3"
   }
   ```
2. 使用指定版本重新生成 lockfile：
   ```bash
   rm pnpm-lock.yaml
   pnpm install
   ```
3. 确保 lockfileVersion 为 '9.0' 以匹配 Vercel 的 pnpm@9.x

**预防措施**:
- 团队成员使用相同的 pnpm 版本
- 定期检查 lockfile 与 package.json 的一致性

## 自定义和扩展

- 修改 `blog.config.js` 进行基本配置
- 在 `components/` 中添加或修改组件
- 在 `styles/` 中自定义样式
- 通过 `lib/notion/` 扩展 Notion 数据处理逻辑