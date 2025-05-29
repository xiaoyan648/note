# 背景

代码评审在整个研发流程中扮演着至关重要的步骤，是确保代码质量、安全性和可维护性的关键环节。它如同严谨的关卡，能够及时发现代码中潜藏的问题与逻辑漏洞。运用 AI 辅助代码评审已成为各大互联网公司认可且有效的提效方式，减轻开发人员的工作负担，辅助研发工作的顺利推进和高质量完成。

# 文档

- [瓦纳卡（wanaka）](https://x0sgcptncj.feishu.cn/wiki/XKwEw6XHiiITgnkSGNKcQJWYnCb)
    
- [AI CR想法](https://x0sgcptncj.feishu.cn/docx/R9jFdSJ7LoIOYExV8wacTOsYnmh)
    
- 有赞方案：https://mp.weixin.qq.com/s/a5p57iv78IWk90XOc9xBOw
    
- 代码库：
    
    - MVP：https://codeup.aliyun.com/qimao/public/devops/ai-cr
        
    - wanaka：https://codeup.aliyun.com/qimao/public/devops/wanaka
        

# 功能需求

暂时无法在飞书文档外展示此内容

## 行级代码评论

这里参考[有赞技术方案](https://mp.weixin.qq.com/s/a5p57iv78IWk90XOc9xBOw)中的方案思考，以及自身实际的使用体验。

在没有 AI 评审之前，我们就是通过云效（codeup）来对有疑虑的代码行进行评论的。所以这种模式对用户的使用习惯改动最少。

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=ODlkYzZlMTE3ZmQxZmMzOTAzM2I4NTk5M2QwNDIwZjNfTW9mdjlKeUx3eHlEN1dxbm9sT3A4NnJsU2Q4eW05WlZfVG9rZW46WTQ4M2JOUXA0b05NYXp4QmNXOWNmNEkybjNjXzE3NDgwNTEwNzg6MTc0ODA1NDY3OF9WNA)

效果示意：

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=MGUzZmNmZjYwMjViYzgyNGY4YTllY2JkZGM5MDY3NjZfOGFBZ3NPTFFFZUVpOEtnVG5kNVkxbmJlSnluOWhZVTZfVG9rZW46SzI3cWJrbk8zb3h0Tmh4VVE0NmM3cmVWbk1oXzE3NDgwNTEwNzg6MTc0ODA1NDY3OF9WNA)

## 项目偏好

不同的项目有不同的代码评审侧重点，比如 C 端的项目与 B 端的项目就会有较大的差异。第一期先给出一个极简版的偏好设置：

|名称|选项|说明|
|---|---|---|
|评论数量|- 小于 3 个<br>    <br>- 小于 10 个<br>    <br>- 不限制|单选，默认为小于 10 个|
|侧重点|- 是否有性能问题<br>    <br>- 是否存在破坏性改动<br>    <br>- 代码可维护性要求高<br>    <br>- ……|多选，默认全选|
|额外要求|用户自定义|输入框|

功能要求：

1. 方便用户进行填写
    
2. 用户可进行修改
    

## 用户反馈

- 临时方案：调查问卷
    
- 常规方案：通过用户对 AI 评审的反馈来建立机制。（优先级较低）
    

# 设计思路

## 方案一：Workflow 模式

暂时无法在飞书文档外展示此内容

1. ### 获取上下文
    

[AI CR想法](https://x0sgcptncj.feishu.cn/docx/R9jFdSJ7LoIOYExV8wacTOsYnmh) 中将上下文的进行分类：

暂时无法在飞书文档外展示此内容

业务层面的上下文因为目前没有统一的规范所以暂不考虑，后续可以迭代引入。代码层面的上下文考虑使用：

- 代码 diff 内容
    
- 修改文件的代码
    
- diff 的依赖关系：尝试通过大模型的能力，探索一下是否能够实现。（后置到 Agent 步骤中）
    

将上下文进行结构化处理，提供给大模型使用。

2. ### 流程编排
    

项目计划采用字节开源的 Eino 框架，通过组件编排的模式来实现 CR Agent 的开发。

**multiAgent 模式**

- 依赖分析 Agent：职责根据上下文分析出代码依赖关系、判断哪些 diff 块缺失充分条件。
    
- 代码分析 Agent：职责进行代码评审输出评论内容，考虑采用初审 + 复审的两阶段来实施。
    

暂时无法在飞书文档外展示此内容

**调试模式**：中间数据进行持久化存储，方便分析问题、效果评估。

3. ### 结果输出
    

在有项目偏好的情况下，对大模型输出的结果进行强制干预，减少大模型不稳定的因素，降低噪音，提升用户好感度。

## 方案二：Agentic 模式

1. ### 简介
    

在研究 Cline IDE 代码审查实现的时候，发现了 Cline Workflows 中`[pr-review.md](https://github.com/cline/cline/blob/main/.clinerules/workflows/pr-review.md)` 定义的合并请求指导文档。这个 `pr-review.md` 文件定义了一个详细的、结构化的 GitHub Pull Request (PR) 审查流程。它更像是一个供 AI 或开发者遵循的指导性工作流文档，而不是直接在 IDE 中执行的代码。

按照这个思路，我们只需要：

1. 提供推理模型（Reason）
    
2. 提供执行模型（Act）
    
3. 基于云效封装一列操作命令（yunxiaoctl）
    
4. 源码
    
5. 缓存记忆
    

封装成一个服务，在服务器中模拟 IDE 进行 Code Review 即可，预期可达到与 Cursor/Cline 等同等水平的 Code Review 结果。

2. ### ReAct 框架
    

大模型为了完成一个大目标，需要不断地做一些任务。每个任务都会经历思考（Thought）、行动（Action）、观察（Observation）三个阶段。思考，决定了下一步的行动；行动是完成了一个具体的动作；而观察，则是对行动结果进行评估，决定是否要结束这个处理过程。

如果说推理的部分都是大模型可以完成的，但行动要做的事恐怕就不是大模型可以单独完成的，比如搜索网页，这显然就需要有一些其它的方式，帮助大模型完成这个搜索的动作。实际上，这也就是 ReAct 这项技术被称为框架的原因，它需要有一些其它的动作嵌入到这个执行过程中。

```Go
Answer the following questions as best you can. You have access to the following tools:

{tools}
Use the following format:Question: the input question you must answerThought: you should always think about what to doAction: the action to take, should be one of [{tool_names}]Action Input: the input to the actionObservation: the result of the action… (this Thought/Action/Action Input/Observation can repeat N times)　

Thought: I now know the final answer
Final Answer: the final answer to the original input question　

Begin!　

Question: {input}

Thought:{agent_scratchpad}
```

这个提示词模板主要是用来借助一些工具，完成一些具体的任务。你在这里看到和前面类似的思考（Thought）、行动（Action）、观察（Observation）的阶段。重点是在行动里，我们可以使用不同的工具，这些工具就是我们可以嵌入到执行过程中的内容。大模型会结合问题以及工具的特点选择下一步的行动，比如，告诉我们用哪个工具完成任务，然后，就可以执行相应的工具代码。这里的工具代码就是我们本地的代码，显然，它能做的事情就多了，会超出大模型本身的能力范围。如果你能理解这个提示词模板，你就具备了实现一个 Agent 的基础。

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=MTY3M2EwYzM4MjIyYjk3NjhjMTJlNDQ3ZDRlYzZiNzFfdmRKakFlWURrcUZYVUJhcTBBZkZsR0JCNVpHR2hRdHhfVG9rZW46Q21MVGJyUU1Xb1J2Z0x4OTVXa2NOQm5SblpjXzE3NDgwNTEwNzg6MTc0ODA1NDY3OF9WNA)

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=MzNkYjRjZmQ5NDk5NjkzNWRhZjQ5NjM4MWUzYzdkNjJfQmczNHZyd3JjVUF6WnlWWjZha09XZDlEckl3RW5sekZfVG9rZW46UjBFamJDQjF0b2ttV0F4MVFGS2Nwb0phbnZlXzE3NDgwNTEwNzg6MTc0ODA1NDY3OF9WNA)

3. ### CR 流程图
    

暂时无法在飞书文档外展示此内容

```Markdown
这个 `pr-review.md` 文件定义了一个详细的、结构化的 `GitHub Pull Request (PR)` 审查流程。它更像是一个供 AI 或开发者遵循的 指导性工作流文档，而不是直接在 IDE 中执行的代码。以下是对其主要内容的分析：

**核心目标与受众：**
- **目标: ** 提供一个标准化的、分步的 PR 审查指南，确保审查的全面性和一致性。
- **受众: ** 看上去这个文档是为 AI 助手（比如 cline 自身）设计的，指导它如何利用 GitHub CLI (gh) 和其他工具 (如 <read_file>, <search_files>, <ask_followup_question>) 来辅助或半自动地完成 PR 审查。当然，人类开发者也可以参考这个流程。

**流程步骤分解：**
文件将 PR 审查过程分为了 6 个主要步骤：
1. **Gather PR Information (收集 PR 信息):**
    - 使用 `gh pr view <PR-number> --json title,body,comments` 获取 PR 的标题、描述和评论。
    - 使用 `gh pr diff <PR-number>` 获取 PR 的完整 diff。
    - 这确保了审查者（AI 或人）对 PR 的基本情况有一个初步了解。
2. **Understand the Context (理解上下文):**
    - 使用 `gh pr view <PR-number> --json files` 识别被修改的文件。
    - 使用 <read_file> 工具读取原始文件内容 (通常是 main 分支上的版本)，以便理解变更的背景。
    - 使用 <search_files> 工具在特定目录或文件中搜索相关代码片段，进一步理解上下文。
    - 这一步强调了理解代码变更所处的环境和原始状态的重要性。
3. **Analyze the Changes (分析变更):**
    - 这是审查的核心步骤，要求对每个修改的文件理解：
        - 修改了什么 (What)
        - 为什么修改 (Why - 基于 PR 描述)
        - 如何影响代码库 (How)
        - 潜在的副作用 (Potential side effects)
    - 关注点包括：代码质量、潜在 bug、性能影响、安全问题、测试覆盖率。
    - 这是一个偏向人工分析或需要 AI 进行深度代码理解的阶段。
4. **Ask for User Confirmation (请求用户确认):**
    - 在 AI 形成初步审查意见后，使用 <ask_followup_question> 工具向用户展示其评估和理由，并询问用户是否同意该建议 (例如，批准 PR 或请求修改)。
    - 这引入了人工监督和决策的环节，AI 提供建议，用户做最终决定。
5. **Ask if User Wants a Comment Drafted (询问用户是否需要草拟评论):**
    - 在用户决定了操作 (批准/拒绝) 之后，再次使用 <ask_followup_question> 询问用户是否需要 AI 帮助草拟一个 PR 评论。
    - 这为用户提供了便利，AI 可以根据分析结果生成结构化的评论。
6. **Make a Decision (做出决定):**
    - 根据用户的最终决定，执行相应的 GitHub 操作：
        - Approve (批准): 使用 `gh pr review <PR-number> --approve --body "..." 或通过 cat << EOF | ... --body-file`
        - 提交多行评论并批准。
        - Request Changes (请求修改): 使用 `gh pr review <PR-number> --request-changes --body "..."` 或类似的多行评论方式。
    - 这里详细列出了使用 GitHub CLI 执行这些操作的具体命令。

**关键工具和技术：**
- **GitHub CLI (gh):** 这是执行 PR 相关操作的核心工具，如查看 PR 信息、获取 diff、批准、请求修改等。文档开头明确指出 AI 有权限使用 gh。
- **XML 风格的工具调用:**
    - <read_file>: 读取文件内容。
    - <search_files>: 在文件中搜索内容。
    - <ask_followup_question>: 向用户提问并获取反馈。这些标签表明 cline 内部有一套用于与用户或外部系统交互的工具集。
- **结构化输出**: 无论是 AI 的分析结果、向用户提出的问题，还是最终的 PR 评论，都强调了结构化和清晰性。

**示例 (example_review_process)：**
文件提供了一个非常具体的审查 PR #3627 的例子，这个例子贯穿了上述所有步骤，展示了如何实际应用这个工作流。这使得流程更容易理解和执行。

**通用命令 (common_gh_commands)：**
最后一部分列出了一些常用的 gh 命令，作为快速参考。

**总结与意义：**
`pr-review.md` 文件为 cline 项目定义了一个半自动化的、AI 辅助的代码审查工作流。它旨在：
- **提升审查效率:** 通过自动化信息收集、上下文理解的初步阶段，并辅助生成评论。
- **保证审查质量:** 提供一个结构化的流程，确保关键方面 (如代码质量、测试、潜在问题) 得到考虑。
- **增强人机协作:** AI 进行分析和初步判断，但最终决策权和关键评估仍掌握在人类用户手中。AI 扮演的是一个强大的助手角色。
- **标准化操作:** 为团队提供一个统一的 PR 审查方法论。

这个文件本身并不是可执行代码，而是一个**元指令或蓝图**，指导 AI 如何利用可用的工具 (如 gh 命令和内部工具) 来参与和协助代码审查过程。它体现了将复杂任务分解为可管理步骤，并通过工具调用来实现自动化的思想。这种方式使得 AI 能够更深入地集成到开发者的工作流程中，特别是在代码审查这种需要细致分析和判断的环节。
```

4. ### 预期效果
    

|   |   |   |
|---|---|---|
||||
||||

5. ### 规划 Todo
    

- 完成 MVP Demo
    

# 参考

1. [腾讯 AICR : 智能化代码评审技术探索与应用实践(上) - 粤海科技君 - 博客园](https://www.cnblogs.com/txycsig/p/18570302)
    
2. [腾讯 AICR : 智能化代码评审技术探索与应用实践(下) - 粤海科技君 - 博客园](https://www.cnblogs.com/txycsig/p/18571875)
    
3. https://mp.weixin.qq.com/s/a5p57iv78IWk90XOc9xBOw
    
4. [提高代码审查准确率的方法.md](https://x0sgcptncj.feishu.cn/wiki/Xl0jwZVoqiLVPskDPBAcrCHmnkh)
    
5. [cline ide 代码审查实现总结.md](https://x0sgcptncj.feishu.cn/wiki/E8hEwoghsiYTM7ko2sucReESnDe)