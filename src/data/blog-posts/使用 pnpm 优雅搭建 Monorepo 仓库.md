---
title: 使用 pnpm 优雅搭建 Monorepo 仓库
tags:
  - Monorepo
  - pnpm
publishDate: "2025-07-07"
slug: "使用-pnpm-优雅搭建-monorepo-仓库"
description: 本文介绍了如何使用 pnpm 搭建 Monorepo 仓库，包括配置、依赖管理等。
date: "2025-07-07"
---

## 前言

我们在日常开发中，经常会遇到多项目管理，如果是单项目，大家都很好理解和操作，如果项目有相互关联关系，并且涉及多个项目管理，大家就可以采用更好的管理方式`Monorepo` 

## 简单介绍 Monorepo

单一仓库（Monorepo）架构方式，可以简单概括为：**利用单一仓库来管理多个项目包（packages)的一种策略或手段**；与其相对的是多仓库（Multirepo）架构

Monorepo 目录中除了会有公共的`package.json`依赖以外，在每个`sub-package`子包下面，也会有其特有的`package.json`依赖。

**兄弟模块之间可以通过模块** `package.json` **定义的** `name` **相互引用，保证模块之间的独立性**

## Monorepo 使用场景

在通常情况下，我们新开一个项目会先在 Github 上面创建一个新仓库，然后在本地创建这个项目，和远程仓库进行关联，基本上是一个仓库对应一个项目。

 * ***复用代码和配置困难**
 
一旦项目多起来，就会遇到一些更复杂的情况。比如一些独立的 h5 活动页面，这些页面往往是不相关的，不方便部署到一起，需要独立部署到不同域名。

除了这些，还有项目有很多共同之处，比如相同的 ts 配置，相同的 eslint 和 prettier 处理，以及相同的多语言文案配置等...

* ***资源浪费**

同时，每次有一个新的页面就去创建一个项目，这些项目也会过于分散，不便管理。另外相同的包也会占用空间， node_module 究竟会变成多大呢

 * ***调试麻烦**
 
如果你想在本地项目进行调试，但这个项目依赖了另一个项目，那么你只能用 `npm` `link` 的方式将它 `link` 到需要调试的项目里面。

一旦 `link` 的项目多了，手动去管理这些 `link` 操作就容易心累。

如果你因为这些问题感到头疼，那 Monorepo 就是你寻找的良药。

**这些情况要谨慎使用 Monorepo**
- **代码量过大**：需要考虑构建性能、代码可维护性
- **权限管理复杂**：需细化目录权限控制
- **团队独立性高**：若子团队高度自治，多仓库可能更灵活

## 轻量化 Monorepo 方案

*  Lerna
* yarn/pnpm
* ...

### pnpm + workspace

本文采用 pnpm+ workspace 方案实现，简单易用，且可以借助  pnpm来管理磁盘上依赖，减少依赖安装，并且规避 **幽灵依赖** 问题 ，更多内容有兴趣可于 `pnpm ` 官网了解 `pnpm` 原理，在此不做过多说明。

### 方案实践

#### 如何配置 Monorepo

使用 pnpm 配置 Monorepo 十分简单方便，仅需要在项目根目录创建一个 pnpm-workspace.yaml 文件，即可支持 Monorepo。

```
packages:
  # 选择 packages 目录下的所有首层子目录的包
  - 'packages/*'
  # 选择 components 目录下所有层级的包
  - 'components/**'
  # 排除所有包含 test 的包
  - '!**/test/**'
```

目录空间根据需求定义，没有严格要求。

经典的Monorepo结构:
```json
demo-monorepo/
├── pnpm-workspace.yaml
├── package.json
├── packages/
│   ├── shared/          # 共享库
│   │   ├── package.json
│   │   └── src/
│   ├── ui-components/   # UI 组件库
│   │   ├── package.json
│   │   └── src/
├── apps/
│   ├── web-app/         # 前端应用
│   │   ├── package.json
│   │   └── src/
│   ├── mobile-app/      # 移动应用
│   │   ├── package.json
│   │   └── src/
├── scripts/             # 共享脚本
├── docs/                # 文档
└── .gitignore
```

#### 安装全局依赖

```bash
/demo-monorepo/ 文件夹下
pnpm add <package-name> -w
# or
pnpm add <package-name> --workspace-root
```

#### 给指定 workspace（工作空间） 安装依赖
> --filter 为 package.json name

```bash
pnpm add <package-name> --filter <workspace-name>
# example
pnpm add axios --filter docs
```
#### 更新依赖

更新根目录依赖，看执行路径

```bash
pnpm update <package-name> [-w]
```

更新指定 workspace 依赖

```bash
pnpm update <package-name> --filter <workspace-name>
# or
pnpm update axios --filter docs
```

#### 卸载依赖

```bash
pnpm uninstall <package-name> [-w]
# or
pnpm uninstall <package-name> --filter <workspace-name>
# or
pnpm uninstall axios --filter docs
```

### 在 workspace 中使用 Catalogs

“_Catalogs_” 是一个 [工作空间功能](https://pnpm.io/zh/workspaces)，可将依赖项版本定义为可复用常量。

#### 定义目录

目录在 `pnpm-workspace.yaml` 文件中定义。 有两种方法来定义目录。

1. 使用 (单数) `catalog` 字段创建名为 `default` 的目录。
2. 使用 (复数) `catalogs` 字段创建任意命名的目录。


#### 默认 Catalog

顶层 `catalog` 字段允许用户定义一个名为 `default` 的目录。

pnpm-workspace.yaml

```bash
catalog: 
	react: ^18.2.0
	react-dom: ^18.2.0
```

这些版本范围可以通过 `catalog:default` 引用。 仅有默认目录时，也可以使用特殊的 `catalog:` 简写。 将 `catalog:` 视为可扩展为 `catalog:default` 的简写。

#### 具名目录

可以在 `catalogs` 键下配置具有名称任意选择的多个`catalog`。

pnpm-workspace.yaml

```bash
catalog:
  react: ^16.14.0
  react-dom: ^16.14.0

catalogs:
  # 可以通过 "catalog:react17" 引用
  react17:
    react: ^17.0.2
    react-dom: ^17.0.2

  # 可以通过 "catalog:react18" 引用
  react18:
    react: ^18.2.0
    react-dom: ^18.2.0
```

##### 优势

在工作空间（即 monorepo 或多包 repo）中，许多包使用相同的依赖项是很常见的。 在编写 `package.json` 文件时，目录可以减少重复，并在此过程中提供一些好处：

- **维护唯一版本** - 我们通常希望在工作空间中共同的依赖项版本一致。 Catalog 让工作区内共同依赖项的版本更容易维护。 重复的依赖关系可能会在运行时冲突并导致错误。 当使用打包器时，不同版本的重复依赖项也会增大项目体积。
- **易于更新** — 升级或者更新依赖项版本时，只需编辑 `pnpm-workspace.yaml` 中的目录，而不需要更改所有用到该依赖项的 `package.json` 文件。 这样可以节省时间 — 只需更改一行，而不是多行。
- **减少合并冲突** — 由于在升级依赖项时不需要编辑 `package.json`文件，所以这些依赖项版本更新时就不会发生 git 冲突。
- 
> https://pnpm.io/catalogs
