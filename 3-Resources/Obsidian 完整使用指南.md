[[Obsidian 深度应用指南]]
## 一、核心概念

**Vault（库）** — 本地文件夹，所有笔记以 `.md` 格式存储，你完全拥有数据。

**双向链接** — `[[笔记名]]` 创建链接，是 Obsidian 知识网络的核心。

**Graph View** — 可视化展示笔记间的关联关系。

**Block / Heading 引用** — `[[笔记名#标题]]` 或 `[[笔记名^block-id]]` 精确引用。

---

## 二、基础操作

### 快捷键（必记）

| 操作 | 快捷键 |
|------|--------|
| 快速打开文件 | `Ctrl+O` |
| 命令面板 | `Ctrl+P` |
| 新建笔记 | `Ctrl+N` |
| 全局搜索 | `Ctrl+Shift+F` |
| 切换编辑/预览 | `Ctrl+E` |
| 插入内部链接 | `[[` 触发 |
| 插入标签 | `#` 触发 |
| 折叠/展开侧边栏 | `Ctrl+\` |

### Markdown 增强语法

```markdown
# 标题层级 H1-H6
**粗体** / *斜体* / ~~删除线~~
- [ ] 待办事项
- [x] 已完成

> 引用块

```代码块```

| 表格 | 列1 | 列2 |
|------|-----|-----|

%%注释（不渲染）%%

---（分割线）
```

### Properties（元数据）

```yaml
---
tags: [项目, 工作]
created: 2026-04-20
status: in-progress
aliases: [别名1, 别名2]
---
```

---

## 三、文件夹结构设计

### 推荐：PARA 方法

```text
Vault/
├── 1-Projects/        # 有明确截止日期的项目
│   └── 项目A/
├── 2-Areas/           # 持续维护的领域
│   ├── 工作/
│   ├── 健康/
│   └── 学习/
├── 3-Resources/       # 参考资料、知识库
│   ├── 技术/
│   └── 读书笔记/
├── 4-Archive/         # 已完成/归档内容
├── 0-Inbox/           # 快速捕获入口
└── Templates/         # 模板文件夹
```

### 替代方案：MOC（内容地图）

不依赖深层文件夹，用 MOC 笔记做导航中枢：

```markdown
# 技术知识地图

## 前端
- [[React 核心概念]]
- [[CSS 布局技巧]]

## 后端
- [[Node.js 最佳实践]]
- [[数据库设计]]
```

---

## 四、工作流设计

### 工作流 1：Daily Notes（日记）

```markdown
---
date: {{date}}
tags: [daily]
---

## 今日目标
- [ ] 
- [ ] 

## 工作记录
### 上午

### 下午

## 快速捕获
- 

## 反思
今天完成了什么？
遇到什么问题？
明天重点？
```

建议节奏：
1. 早上写 3 个最重要目标。
2. 白天随时捕获想法。
3. 晚上整理并链接到知识笔记。
4. 写简短复盘。

### 工作流 2：读书/文章笔记（渐进式总结）

```markdown
---
tags: [reading, book]
author: 
source: 
status: reading | done
rating: 
---

## 核心观点

## 金句摘录
> 

## 我的思考

## 行动项
- [ ] 

## 关联笔记
- [[]]
```

建议步骤：摘录原文 -> 加入个人理解 -> 抽象成可复用知识点并建立链接。

### 工作流 3：项目管理

```markdown
---
tags: [project]
status: active
deadline: 2026-05-01
---

## 目标

## 里程碑
- [ ] 阶段1 📅 2026-04-25
- [ ] 阶段2 📅 2026-05-01

## 会议记录
- [[2026-04-20 项目启动会]]

## 相关资源
- [[技术方案]]
- [[竞品分析]]

## 决策日志
| 日期 | 决策 | 原因 |
|------|------|------|
```

### 工作流 4：GTD（捕获-处理-组织-回顾-执行）

```text
捕获 -> 0-Inbox
处理 -> 每天/每周清空 Inbox
组织 -> 分配到 Projects / Areas / Someday
回顾 -> Weekly Review
执行 -> 今日任务清单
```

Weekly Review 模板：

```markdown
## 本周回顾 {{date}}

### 完成了什么
- 

### 未完成 & 原因
- 

### 下周重点
- [ ] 
- [ ] 

### 需要归档的笔记
- 
```

---

## 五、插件建议

### 核心插件（官方）

- Daily Notes
- Templates
- Backlinks
- Outline
- Tags
- Canvas
- Bases

### 社区插件（按优先级）

1. **Templater**：高级模板和自动化。
2. **Dataview**：把笔记当数据库查询。
3. **Tasks**：聚合任务管理。
4. **Calendar / Periodic Notes**：日周月节奏。
5. **QuickAdd**：快速捕获入口。
6. **Excalidraw**：视觉化思考。

---

## 六、Dataview 最佳实践示例

### 1) 进行中项目

```dataview
TABLE deadline, status
FROM "1-Projects"
WHERE status = "active"
SORT deadline ASC
```

### 2) 最近 7 天新增笔记

```dataview
LIST
WHERE file.ctime >= date(today) - dur(7 days)
SORT file.ctime DESC
```

### 3) 未完成任务聚合

```dataview
TASK
WHERE !completed
FROM "1-Projects"
GROUP BY file.link
```

---

## 七、标签与命名规范

### 标签分层建议

- `#状态/进行中`
- `#状态/已完成`
- `#类型/会议记录`
- `#类型/读书笔记`
- `#领域/技术`
- `#领域/产品`

控制核心标签数量（20-30 个）优先，避免碎片化标签泛滥。

### 命名建议

- 日记：`YYYY-MM-DD`
- 会议：`YYYY-MM-DD 会议主题`
- 项目：`项目名 - 文档类型`
- 知识笔记：短而具体，可直接复用为链接名

---

## 八、同步与备份策略

| 方案 | 优点 | 注意点 |
|------|------|--------|
| Obsidian Sync | 官方支持、端对端加密、版本历史 | 付费 |
| iCloud / OneDrive | 易用 | 偶发冲突 |
| Git + GitHub | 完整历史、可审计 | 需要 Git 基础 |
| Syncthing | 免费、P2P | 需要自行维护 |

开发者建议：用 `obsidian-git` 定时提交，最少保留一份异地备份。

---

## 九、常见误区与最佳实践

1. **只收藏不整理**：每周至少一次回顾与归档。
2. **过度追求系统完美**：先跑起来，再迭代。
3. **只用文件夹不用链接**：知识网络价值会下降。
4. **笔记太长太杂**：优先原子化拆分。
5. **插件装太多**：从刚需插件开始，按痛点扩展。

实践原则：
- 先捕获，后整理。
- 先可用，后精致。
- 先连接，后分类。
- 以复盘驱动系统进化。

---

## 十、30 天落地路线图

### 第 1 周：基础可用
- 开启 Daily Notes / Templates。
- 只做捕获与简单链接。

### 第 2 周：结构成型
- 建立 PARA 或 MOC。
- 统一命名与标签。

### 第 3 周：流程自动化
- 引入 Templater、Tasks、Calendar。
- 建立周回顾模板。

### 第 4 周：查询与优化
- 引入 Dataview 看板。
- 清理冗余标签与过期笔记。

目标不是“配置最强”，而是“持续稳定产出”。

---

## 参考来源

- [Obsidian Help](https://obsidian.md/help/)
- [Obsidian 教程：从入门到开发者最佳实践](https://www.cnblogs.com/gyc567/p/19559303)
- [我的Obsidian工作流｜插件及配置推荐](https://forum-zh.obsidian.md/t/topic/24139)
- [Obsidian Tips for 2026](https://www.geeky-gadgets.com/obsidian-tips-tricks-2026/)
- [Example Workflows in Obsidian](https://forum.obsidian.md/t/example-workflows-in-obsidian/1093)
- [My complete Obsidian workflow to manage my life](https://forum.obsidian.md/t/my-complete-obsidian-workflow-to-manage-my-life/64522)
- [Getting Things Done workflow in obsidian](https://forum.obsidian.md/t/getting-things-done-workflow-in-obsidian/100145)
