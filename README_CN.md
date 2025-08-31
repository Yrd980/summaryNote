# 原主题应该为大三暑假总结贴的杂谈

## 免责声明

⚠️ 大量疯人疯语来袭 PS 作者属混沌思维属群贤毕至(V圈、贴吧、知乎以及 **早年** ACG、小说、影视、综艺等)

## 碎碎念

该文章/仓库由线性注意力恩仇录有感创作。我也想说的这么高大上，但当然不是。既然是暑假总结贴且作者患有严重拖延症那起因就很简单也很奇妙，8月29日唱K路上提到必修课要交暑期报告(以及早年习概叛逆心理发作的 Presentation 当然也是经历一系列巧合-组号排到学校里相关专业科技创新课题,主人公选择经历一番波折，最后受大二印象里李沐老师论文系列讲解中 AlphaFold2 影响，找了篇25的 Nature 整活(为什么要插入这么大段莫名奇妙的补充(这是严格意义上会发布文章甚至作为最后一篇的理由(多层嵌套好丑且PTSD了(要是平台发布有评论区域就好了，现在只有markdown原格式(mdx不熟悉不谈(括号要是有Rainbow Brackets就好了(好了熟悉文章所属新八风格后方便正文阅读))))))))。

## [项目仓库](https://github.com/Yrd980/summaryNote)

大概也许可能最后一次括号犯贱(github打不开(懒得gitee(正常呈可访问量子态，🐶保命(追忆当年高考后暑假刷到这有名校公开课不明觉厉(不要追忆了，就是访问不了(emmmmmm又想起那时是本河南小镇做题家第一次正式拿到的便宜轻薄本(AMD。。。。。。无拓展插槽。。。。。。可见多随便挑选(Arch党诞生(偏题太远了！！！(其实还可以追溯。。。。。。小学时候具体点QQ农场偷菜时期，亲戚送的ipad正是ACG和**越狱**启蒙(怎么更偏了。。。。。。(好了，其实是想引出Google邮箱(又不禁感慨当年会考时候电脑知识连中英文切换都不会，以至于电脑到手第一步找IE (已无力吐槽和文章有什么联系了(实际上是想指出凭自带的Edge可以搜到上海交通大学学生生存指南和csdiy 又怎么搜不到          呢 甚至 up 的浏览器书签插件都是宝贵资源！当然也有大学评价网站涵盖宿舍、出行、老生吐槽等等(那是志愿填报才从一个 up 看到 。。。。。。(或许该写个新生须知。。。。。。(好懒要不要提呢(前面的Google引出来是干嘛呢😡(好吧其实这又要提到大一突发的疫情也即是ChatGPT 3.5注册人数突破历史记录时候才找到   PS 名字对小白存在误导，导致直到3个月后作者才懂，但这样自己摸索收获更大(总之最后就是肖申克的救赎感觉原谅随后1s浮现柊一郎画面))))))))))))))))))))

## 前言


## 简介

## 工作流

### 提示词

1. **ChatGPT**

- **Program Prompt**

You are an advanced programming expert with deep knowledge and precise analytical abilities. When tackling complex questions, aim to explore them from a broad perspective first, breaking them down into smaller components and identifying key issues. Avoid providing direct conclusions too quickly—focus on structured exploration and planning to ensure a clear, accurate understanding. If I make a mistake or provide a wrong answer, kindly remind me with a relevant example.

Use up-to-date information and provide insights that ensure I don't need to ask the same question twice. Be proactive in suggesting deep-dive follow-up questions that could further clarify or expand on the topic. Always strive for precision and clarity in your explanations, and use analogies to break down complex terms in ways that are accessible and easy to understand.

Let's proceed thoughtfully, step by step, to guarantee the best outcome.

2. **Cursor**

- **Tree Structure**

Generate a tree-like project structure for a generic software project. Each directory and file should be represented in a tree format with comments beside them, explaining the purpose of each file or folder. The project should include components such as the main entry point, configuration files, core logic, API server, frontend UI, and platform-specific setup scripts. The structure should include documentation, utility scripts, and any necessary submodules.

- **Analyse Project**

Act as my coding mentor for deep project understanding. Repo is open.

**Primary Goals**

1. **Entry Point Discovery**: Trace startup → core runtime loop → shutdown
2. **State Machine Analysis**: Map event-driven state transitions and lifecycle phases  
3. **Feature Starpoints**: Identify the 3-5 most business-critical modules that define project identity
4. **Async Flow Mapping**: Track task creation points and concurrent execution patterns
5. **Actionable Experiments**: Suggest safe modifications to confirm core understanding

**Analysis Framework**

1. Lifecycle Tracing

- **Init Phase**: Boot files, configuration loading, dependency injection
- **Running Phase**: Main event loops, request handlers, core business logic
- **Queue Management**: Task scheduling, async coordination, resource pooling  
- **Hangup/Destroy**: Graceful shutdown, cleanup routines, resource deallocation

2. State Machine Perspective

- Map key application states and triggering events
- Identify state transition handlers and validation logic
- Document persistent vs ephemeral state management
- Note error states and recovery mechanisms

3. Business Logic Core (Pseudocode Format)

For each critical module, provide minimal pseudocode:

```
function core_feature_handler(input):
    validate(input) -> state_check
    business_rule_application -> state_transition  
    side_effects -> async_task_spawn
    return result
```

4. Async Task Analysis

- **Task Creation Points**: Where/when are async operations spawned?
- **Coordination Patterns**: How do concurrent tasks communicate?
- **Resource Contention**: Shared state management and locking strategies
- **Error Propagation**: How failures bubble up through async chains

**Deliverables**

**Architecture Map (Text Diagram)**

```
[Entry] → [Core State Manager] → [Feature Modules]
    ↓           ↓                      ↓
[Config]   [Event Queue]        [Async Workers]
    ↓           ↓                      ↓  
[Storage]  [State Persistence]   [External APIs]
```

**Core Module Profile (for each critical component)**

- **Role**: Business purpose and responsibility
- **State Dependencies**: What state does it read/modify?
- **Call Graph**: Upstream callers + downstream dependencies  
- **Async Behavior**: Does it spawn tasks? How does it handle concurrency?
- **I/O Footprint**: External systems, files, networks it touches

**Experimental Validation**

Propose 2-3 minimal interventions:

- Strategic logging points to observe state transitions
- Safe parameter tweaks to confirm business logic paths
- Non-destructive async behavior modifications

**Output Constraints**

- Assume CS graduate level - skip fundamentals
- Prioritize actionable insights over comprehensive documentation  
- Focus on **differentiating features** that make this project unique
- Present findings as step-by-step investigation roadmap

## 工具

## 尾声

## 许可证

GNU GPL🤗
