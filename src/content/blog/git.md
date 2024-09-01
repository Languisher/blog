---
title: "Git 基础操作"
description: ""
date: 2024-08-23
category: [""]
tags: ["Git"]
mermaid: true
---

## 不同分支之间关系

```mermaid
gitGraph
    commit
    commit
    branch develop
    commit
    branch v2
    commit
    checkout main
    commit
    checkout develop
    commit
    checkout main
    merge v2
    merge develop
```

## 不同存储地点之间关系

![](https://miro.medium.com/v2/resize:fit:1400/1*4MIIRk5lvcbQcKnx9ktz4Q.png)

## 版本控制流程图

![](https://shockerli.net/post/git-flow-guide/media/Git%E5%B7%A5%E4%BD%9C%E6%B5%81%E6%A8%A1%E5%9E%8B.png)

## Reference
- [Git 版本控制 常用篇](https://medium.com/daai/git-%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E4%B8%80%E4%B8%8B%E5%90%A7-6e26fc432b16)
- [Git 工作流与规范](https://shockerli.net/post/git-flow-guide/)