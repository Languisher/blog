---
title: "基于 Mermaid 的 Markdown 基本插图"
description: ""
date: 2024-08-23
category: [""]
tags: ["Git"]
mermaid: true
---

## 图表一览

|图表名称|图表用途|文档链接|
|---|---|---|
|[Flowchart](#flowchart)|流程图、分支图|[Flowcharts - Basic Syntax](https://mermaid.js.org/syntax/flowchart.html)|
|[State](#state)|状态转移（流程）图|[Flowcharts - Basic Syntax](https://mermaid.js.org/syntax/flowchart.html)|
|[Sequence](#sequence)|强调物体间通信关系|[Sequence diagrams](https://mermaid.js.org/syntax/sequenceDiagram.html)|
|[Class](#class)|类属性及继承关系|[Class diagrams](https://mermaid.js.org/syntax/classDiagram.html)|
|[Gitgraph](#gitgraph)|版本历史记录、线路图|[Gitgraph Diagrams](https://mermaid.js.org/syntax/gitgraph.html)|

- Statechart 相较于传统流程图支持专属于状态的特殊的操作，例如 Fork-join.


### Flowchart

```mermaid
flowchart LR
  subgraph TOP
    direction TB
    subgraph B1
        direction RL
        i1 -->f1
    end
    subgraph B2
        direction BT
        i2 -->f2
    end
  end
  A --> TOP --> B
  B1 --> B2
```

### State

```mermaid
   stateDiagram-v2
    state fork_state <<fork>>
      [*] --> fork_state
      fork_state --> State2
      fork_state --> State3

      state join_state <<join>>
      State2 --> join_state
      State3 --> join_state
      join_state --> State4
      State4 --> [*]
```


### Sequence

```mermaid
sequenceDiagram
    Alice->John: Hello John, how are you?
    Note over Alice,John: A typical interaction<br/>But now in two lines
```

### Class

```mermaid
---
title: Animal example
---
classDiagram
    note "From Duck till Zebra"
    Animal <|-- Duck
    note for Duck "can fly\ncan swim\ncan dive\ncan help in debugging"
    Animal <|-- Fish
    Animal <|-- Zebra
    Animal : +int age
    Animal : +String gender
    Animal: +isMammal()
    Animal: +mate()
    class Duck{
        +String beakColor
        +swim()
        +quack()
    }
    class Fish{
        -int sizeInFeet
        -canEat()
    }
    class Zebra{
        +bool is_wild
        +run()
    }
```

### Gitgraph

```mermaid
---
title: Example Git diagram
---
gitGraph LR:
    commit
    commit
    branch develop
    commit
    commit
    checkout main
    commit
    commit
    merge develop
    commit
    commit
```
