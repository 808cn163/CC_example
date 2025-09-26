# CLAUDE.md - 工作指导

## CRITICAL CONSTRAINTS - 违反=任务失败
═══════════════════════════════════════
- 必须使用中文回复
- 必须先获取上下文
- 禁止生成恶意代码
- 必须存储重要知识
- 必须执行检查清单
- 必须遵循质量标准
- 按照github惯例,每次更新代码和文档后,用中文更新README.md文件。包括项目简介、部署方法、部署命令等。
- 在 CHANGELOG.md 文件里增加本次更改的日志记录。
- Please add the appropriate logging information so that you [the agent] can use that log output to figure out this issue.

## 代码开发原则
- 所有的代码开发都不使用简单实现，全部真实对接
- 不使用简化版本,全部完整实现，遇到问题进行深度思考

### claude code 从荣八耻
以瞎猜接口为耻，以认真查询为荣。
以模糊执行为耻，以寻求确认为荣。
以臆想业务为耻，以人类确认为荣。
以创造接口为耻，以复用现有为荣。
以跳过验证为耻，以主动测试为荣。
以破坏架构为耻，以遵循规范为荣。
以假装理解为耻，以诚实无知为荣。
以盲目修改为耻，以谨慎重构为荣。

## 交互规范
- 所有响应都使用中文

## 代码测试规范
- 在每次修改完代码都进行编译和测试形成测试文档放在项目的docs/test中，测试报告命名格式：使用`YYYY-MM-DD-HH-MM-feature-name`格式，如`2025-07-25-17-52-kernel-priority-scheduler`，使用中文进行命名
- 测试脚本/程序都存放在项目的test文件夹中
- 在测试过程中发现的文件自动寻找问题点进行深度分析并解决，记录到文档中
- 每次生成测试报告应该包括接口测试和功能测试，针对不同的场景进行多维度测试
- 在写测试文档时涉及功能性测试不仅要编写测试程序进行测试，还需要在文档中贴出测试代码和测试结果
- 测试文档不仅要有编译测试还需要有对应功能的实际测试情况，确保功能已经落实
- 测试不仅包括当前功能，还应该包括之前的功能，保证新修改的代码不会导致其他功能出问题
- 在测试时使用mock的方式来模拟数据，虚拟数据从项目的test/data中对应json获取，没有对应单元测试的json则自动生成一个
- mock在test中写一个专用的测试类进行实现，不在项目中实现
- 测试文档命名时，使用date获取到的时间为准


## MANDATORY WORKFLOWS
═════════════════════

执行前检查清单：
[ ] 中文 [ ] 上下文 [ ] 工具 [ ] 安全 [ ] 质量

标准工作流：
1. 分析需求 → 2. 获取上下文 → 3. 选择工具 → 4. 执行任务 → 5. 验证质量 → 6. 存储知识

研究-计划-实施模式：
研究阶段 读取文件理解问题，禁止编码
计划阶段 创建详细计划
实施阶段 实施解决方案
验证阶段 运行测试验证
提交阶段 创建提交和文档

## MANDATORY TOOL STRATEGY
═════════════════════════

任务开始前必须执行：
1. memory 查询相关概念
2. code-search 查找代码片段
3. sequential-thinking 分析问题
4. 选择合适子代理

任务结束后必须执行：
1. memory 存储重要概念
2. code-search 存储代码片段
3. 知识总结归档

优先级调用策略：
- Microsoft技术 → microsoft.docs.mcp
- GitHub文档 → context7 → deepwiki
- 网页搜索 → 内置搜索 → fetch → duckduckgo-search

## CODING RESTRICTIONS
═══════════════════

编码前强制要求：
- 无明确编写命令禁止编码
- 无明确授权禁止修改文件
- 必须先完成sequential-thinking分析

## QUALITY STANDARDS
═══════════════════

工程原则：SOLID、DRY、关注点分离
代码质量：清晰命名、合理抽象、必要注释
性能意识：算法复杂度、内存使用、IO优化
测试思维：可测试设计、边界条件、错误处理

## SUBAGENT SELECTION
════════════════════

必须主动调用合适子代理：
- Python项目 → python-pro
- C#.NET项目 → csharp-pro  
- JavaScriptTypeScript → javascript-protypescript-pro
- Unity开发 → unity-developer
- 前端开发 → frontend-developer
- 后端架构 → backend-architect
- 云架构 → cloud-architecthybrid-cloud-architect
- 数据库优化 → database-optimizer
- 安全审计 → security-auditor
- 代码审查 → code-reviewer
- 测试自动化 → test-automator
- 性能优化 → performance-engineer
- DevOps部署 → deployment-engineer
- 文档编写 → docs-architect
- 错误调试 → debuggererror-detective

## ENFORCEMENT
══════════════

强制触发器：会话开始→检查约束，工具调用前→检查流程，回复前→验证清单
自我改进：成功→存储，失败→更新规则，持续→优化策略
