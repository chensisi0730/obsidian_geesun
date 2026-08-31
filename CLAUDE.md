# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

我是自动化毕业的工程师，拥有基本 RL 知识和 [SO-ARM101](https://github.com/TheRobotStudio/SO-ARM100) 主从机械臂硬件，有RTX 2080 Ti 22GB之前使用过 pybullet 仿真，现在想开始学习isaac Lab仿真和训练机器人自主遵循指令抓取物品。目前跑通 Leader 遥操作，

# my-wiki-llm — Claude Code Schema

@AGENTS.md

> **本文件仅包含 Claude Code 专属的补充指令。**
>
> 共享 wiki Schema 全部定义在 `AGENTS.md` 中，通过 `@` 导入自动加载。
> **绝不将 AGENTS.md 已有的内容复制到本文件。**
> 需要修改共享规则时，编辑 AGENTS.md，不要在此文件中重新定义。
>
> ingest / lint / query 操作修改的是 wiki 页面，不是 Schema 文件。

## 可用技能

本 wiki 项目常用以下内置技能：


| 技能                      | 用途                                                        |
| ------------------------- | ----------------------------------------------------------- |
| `obsidian-markdown`       | 创建/编辑 Obsidian 标记（wikilinks、callouts、frontmatter） |
| `obsidian-cli`            | Obsidian vault 操作（搜索笔记、管理属性、运行命令）         |
| `excalidraw-diagram`      | 生成 Excalidraw 架构/流程图表                               |
| `mermaid-visualizer`      | 内联 Mermaid 图表（序列图、状态图）                         |
| `json-canvas`             | 创建 JSON Canvas 知识关系地图                               |
| `obsidian-canvas-creator` | Obsidian Canvas 可视化布局                                  |
| `defuddle`                | 提取网页内容为干净 markdown（替代 WebFetch 用于网页文章）   |

## 工作流速查

```
摄取文章/资料     → 先检查 raw/收件箱/ 和raw/Clippings，再按 ingest 流程处理
提问/查询          → 查 Wiki 目录.md → 读 wiki 页面 → 综合回答
健康检查           → lint 检查（孤页/死链/过时/矛盾/标签）
URL 文章           → defuddle 提取 → 存 raw/ → 标准 ingest
图表               → Excalidraw > JSON Canvas > Mermaid（不用 ASCII 画）
```

## 注意

- `raw/sortspec.md` 和 `wiki/sortspec.md` 是 Obsidian 文件排序配置，不需手动编辑。
- `.obsidian/` 目录为 Obsidian 配置，不直接修改。
- 创建页面时始终先读取对应 `templates/<type>.md` 模板。
