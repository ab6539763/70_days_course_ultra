# 原始课表参考(零基础大模型应用开发70天培训课程详细课表)

> 本文件是用户提供的原始技术教学大纲全文,供撰写每天课件时核对技术知识点是否覆盖完整。故事化包装、人物、公司、产品统一以 `00-master-bible.md` 与 `01-beat-sheet.md` 为准,本文件只作技术知识点覆核用。

## 课程总览

| 项目 | 说明 |
|------|------|
| 培训周期 | 70 天(10 周),每天约 6-8 小时学习 |
| 目标学员 | 零编程基础或仅有少量基础的转行者、在校生、产品经理 |
| 培养目标 | 能独立开发 RAG 应用、Agent 应用、微调小模型并部署上线 |
| 技术栈 | Python、LangChain、LlamaIndex、OpenAI/DeepSeek/Qwen API、FastAPI、向量数据库、Docker |
| 考核方式 | 每周小测 + 3 次阶段项目 + 1 个毕业设计 |

## 第一阶段:Python 编程基础(第 1-14 天)

### 第 1 周:Python 入门

Day1 开发环境与第一行代码:课程介绍、大模型行业全景与职业路径分析;安装Python3.10+、VS Code、配置国内镜像源;变量、数据类型(int/float/str/bool)、print与input;实操"个人信息卡片"程序;晚自习配置Git与GitHub。

Day2 运算符与字符串:算术/比较/逻辑运算符、运算优先级;字符串索引、切片、常用方法(split/join/strip/replace/format);f-string格式化;实操文本清洗小工具(去空格、统一大小写、敏感词替换)。

Day3 流程控制:if/elif/else、嵌套条件;while循环、for循环、range、break/continue;实操猜数字游戏、九九乘法表、简易菜单系统;晚自习LeetCode简单题2道。

Day4 核心数据结构(上):列表list的增删改查、切片、排序、列表推导式;元组tuple、集合set的应用场景;实操待办事项管理器(纯命令行版)。

Day5 核心数据结构(下)—字典重点日:字典dict的增删改查、嵌套字典、遍历;JSON格式详解(与API交互的基石);json模块的loads/dumps/load/dump;实操解析一份模拟的API返回JSON,提取指定字段。

Day6 函数:函数定义、参数(位置/关键字/默认值/*args/**kwargs)、返回值;作用域、匿名函数lambda、递归入门;实操把前几天的小项目重构为函数式结构。

Day7 第一周复习与周测:知识点串讲、答疑;周测(笔试+上机);综合练习:命令行版"通讯录管理系统"(增删改查+JSON持久化)。

### 第 2 周:Python 进阶

Day8 面向对象编程(上):类与对象、属性与方法、`__init__`构造方法;实例方法、类方法、静态方法;实操定义ChatMessage类(role、content属性)。

Day9 面向对象编程(下):继承、多态、方法重写、super();魔术方法(`__str__`、`__repr__`、`__call__`)、属性装饰器@property;实操设计BaseModel→OpenAIModel/QwenModel的类继承结构。

Day10 模块、包与异常处理:import机制、创建自己的模块和包、`if __name__=="__main__"`;try/except/else/finally、自定义异常、raise;虚拟环境venv/conda、pip与requirements.txt;实操把项目拆分为多文件包结构。

Day11 文件操作与常用标准库:文件读写(txt/csv/json)、with上下文管理器、编码问题(UTF-8);os、pathlib、datetime、random、re正则表达式入门;实操批量文档读取与关键词统计工具。

Day12 网络请求与API调用(关键日):HTTP协议基础(GET/POST、状态码、Header、Body);requests库详解;首次调用大模型API(DeepSeek或通义千问的免费额度);理解API Key、请求体结构、messages格式、解析返回结果;实操命令行AI问答小程序。

Day13 进阶语法与异步入门:装饰器原理与应用、生成器与yield;类型注解typing、asyncio异步编程入门;环境变量管理(python-dotenv);实操给API调用加上重试装饰器与超时控制。

Day14 阶段考核—项目一:全天项目命令行多轮对话AI助手;要求多轮对话记忆、对话历史保存为JSON、异常处理、支持指令(/clear、/save、/exit);晚上代码互评+讲师点评。

## 第二阶段:大模型基础理论与Prompt工程(第15-24天)

### 第3周:大模型原理与API深度使用

Day15 大模型原理科普:NLP发展简史;通俗理解Transformer;预训练、SFT、RLHF三阶段;Token与分词器演示;主流模型盘点;实操用tiktoken计算token数估算API成本。

Day16 大模型API核心参数详解:temperature、top_p、max_tokens、frequency_penalty逐一实验对比;system/user/assistant角色详解;流式输出stream=True(打字机效果);实操参数对比实验报告。

Day17 Prompt工程基础:Prompt设计四要素;Zero-shot/Few-shot/One-shot提示;角色扮演法、分隔符使用、输出格式约束;实操编写10个不同场景的Prompt。

Day18 Prompt工程进阶:思维链CoT;自洽性、思维树ToT概念;Prompt攻击与防御;结构化输出JSON Mode、Function Calling初探;实操构建智能客服意图分类器。

Day19 Function Calling/Tool Use(重点日):Function Calling原理与完整流程图解;定义工具Schema、模型返回工具调用、执行并回传结果;多工具场景;实操手写查天气算数学题AI助手。

Day20 多模态与嵌入模型:多模态模型API使用;Embedding嵌入模型详解、余弦相似度计算;实操相似问题匹配小工具。

Day21 周测+综合练习:周测(Prompt工程+API使用);综合练习多轮对话+工具调用+流式输出整合小项目。

### 第4周:Web开发基础

Day22 前端速成:HTML/CSS快速入门;JavaScript基础、fetch请求;实操静态聊天界面页面。

Day23 FastAPI后端开发(上):FastAPI入门路由/路径参数/查询参数/自动文档;Pydantic数据模型、请求体校验、响应模型;实操把AI对话功能封装成REST API。

Day24 FastAPI后端开发(下)+数据库:SSE流式接口、CORS跨域;SQLite+SQLAlchemy基础;实操前后端联调网页版ChatGPT克隆。

## 第三阶段:LangChain与RAG开发(第25-38天)

### 第5周:LangChain框架

Day25 LangChain入门:架构总览、生态介绍;ChatModel接口、PromptTemplate/ChatPromptTemplate;实操用LangChain重写对话应用。

Day26 LCEL表达式与链:LCEL、管道符、Runnable接口;OutputParser;RunnablePassthrough、RunnableParallel、分支路由;实操"翻译→润色→摘要"三级链。

Day27 Memory记忆机制:对话记忆原理、ChatMessageHistory、RunnableWithMessageHistory;窗口记忆、摘要记忆、数据库持久化记忆;实操多会话记忆管理。

Day28 文档加载与分割:Document Loaders实战;文本分割策略;chunk_size与chunk_overlap调优;实操PDF电子书清洗分割。

Day29 向量数据库:向量数据库原理;Chroma快速上手;Milvus/FAISS/Pinecone对比选型;Retriever接口;实操构建知识库相似度检索测试。

Day30 完整RAG系统搭建(核心日):RAG完整链路;RAG Prompt模板设计、引用来源标注、兜底处理;实操搭建企业知识库问答系统(命令行版)。

Day31 周测+RAG效果调优实验日:周测;调chunk大小、top_k、Embedding模型对比实验。

### 第6周:RAG进阶与项目二

Day32 高级RAG技术(上):查询改写、多路查询;HyDE、上下文压缩;实操对比召回率差异。

Day33 高级RAG技术(下):混合检索(BM25+向量)、RRF融合排序;Rerank重排序模型、父文档检索器;实操混合检索+重排增强RAG流水线。

Day34 RAG评估:评估指标(忠实度、答案相关性、上下文精确率/召回率);Ragas评估框架实战;实操评估报告。

Day35 LlamaIndex框架:核心概念、与LangChain对比选型;快速重建知识库问答;实操对比笔记。

Day36-37 阶段项目二—企业级知识库问答系统:Day36需求分析、架构设计、文档处理与知识库构建、后端API开发;Day37前端界面、多轮对话支持、引用溯源展示、流式输出、联调测试;功能要求多格式文档上传、混合检索、Rerank、答案带引用、会话管理。

Day38 项目答辩与代码评审:项目演示与答辩;代码评审、优秀代码分享、重构建议。

## 第四阶段:Agent智能体开发(第39-50天)

### 第7周:Agent基础

Day39 Agent概念与ReAct范式:感知-思考-行动循环、与Chain的本质区别;ReAct论文精讲;不用框架手写极简ReAct Agent;实操搜索+计算组合任务。

Day40 LangChain Agent与工具:Tool定义(@tool装饰器)、内置工具生态;create_tool_calling_agent、AgentExecutor;实操带搜索/代码执行/文件读写能力的Agent。

Day41 LangGraph入门(重点):State、Node、Edge、条件边、循环;用LangGraph重写ReAct Agent、可视化图结构;实操带人工审批节点的工作流。

Day42 LangGraph进阶:持久化Checkpointer、中断与恢复Human-in-the-loop;子图、并行节点、错误重试策略;实操写作Agent(大纲→并行写作→汇总润色)。

Day43 多Agent系统:Supervisor模式、协作模式、层级模式;LangGraph实现Supervisor多Agent系统;实操研究团队协作。

Day44 MCP与Agent生态:MCP协议详解、MCP Server开发;接入现成MCP服务、Agent记忆系统设计;实操开发自己的MCP Server并接入Agent。

Day45 周测+Dify/Coze低代码平台:周测;Dify平台实战;低代码平台vs代码开发适用边界讨论。

### 第8周:Agent进阶与项目三

Day46 Agent稳定性与工程化:常见失败模式与调试技巧、LangSmith追踪调试;输出校验、护栏、成本与延迟优化;实操给Agent加完整可观测性与容错。

Day47 实用Agent场景专题:Text-to-SQL数据分析Agent;浏览器自动化Agent概念、代码助手Agent;实操自然语言查销售数据SQL Agent。

Day48-49 阶段项目三—多Agent智能办公助手:Day48架构设计、Agent角色划分、工具开发(日程/邮件草拟/网络搜索/文档RAG);Day49 LangGraph编排、人工确认环节、前端界面、联调;功能要求至少3个协作Agent、5个以上工具、支持任务中断恢复。

Day50 项目答辩+阶段复盘。

## 第五阶段:模型微调与部署(第51-60天)

### 第9周:微调与本地部署

Day51 模型微调理论:什么时候需要微调vs RAG vs Prompt;全量微调、LoRA、QLoRA原理;GPU基础知识、显存估算、算力平台租用实操。

Day52 数据集构建:指令微调数据格式(Alpaca/ShareGPT);数据清洗去重、Self-Instruct;实操构建500条垂直领域微调数据集。

Day53 LLaMA-Factory微调实战:安装配置、WebUI使用;对Qwen2.5-7B做LoRA微调、训练参数详解;实操跑通完整微调流程。

Day54 微调评估与模型合并:效果对比评估、过拟合识别处理;LoRA权重合并、模型量化(GPTQ/AWQ/GGUF);实操合并模型并量化导出。

Day55 本地部署与推理服务:Ollama本地部署、OpenAI兼容接口;vLLM高性能推理部署、吞吐量压测对比;实操把微调后模型部署成API服务。

Day56 应用部署工程化:Docker入门(镜像、容器、Dockerfile);Docker Compose编排;云服务器部署上线;实操容器化并部署到公网可访问。

Day57 周测+安全与合规专题:周测;内容审核接入、敏感信息脱敏、备案合规常识、成本监控。

## 第六阶段:毕业设计与就业冲刺(第58-70天)

### 第10周及冲刺期

Day58 毕业设计启动:选题指导(8个备选方向+支持自选);需求文档撰写、技术方案设计、讲师一对一评审方案。

Day59-64 毕业设计开发(6天):Day59项目骨架搭建、核心数据/知识库准备;Day60核心功能开发(RAG/Agent主链路);Day61核心功能开发(工具、记忆、多轮交互);Day62前端界面+后端API联调;Day63测试、评估报告、性能与成本优化;Day64 Docker部署上线、录制演示视频、撰写项目文档。

Day65 毕业答辩:每人15分钟演示+10分钟答辩,企业评委参与打分。

Day66 简历与作品集打磨:技术简历撰写方法、STAR法则;GitHub仓库整理、README规范、个人作品集页面。

Day67 面试专题(一)—技术八股:高频面试题精讲(Transformer/注意力机制、RAG优化、Agent设计);Python与工程问题、模拟笔试。

Day68 面试专题(二)—项目深挖与系统设计:如何应对项目被问穿、系统设计题;两两模拟面试+讲师点评。

Day69 模拟面试日:一对一全真模拟面试(技术面+HR面),逐人反馈改进清单。

Day70 结业日:行业前沿分享(多模态Agent、具身智能趋势)、持续学习路线图;结业典礼、优秀学员分享、就业资源对接。

## 附:配套学习资源

| 类别 | 内容 |
|------|------|
| 每日安排 | 上午9:00-12:00授课/下午14:00-17:30实操/晚19:00-21:00自习答疑 |
| 算力支持 | 前50天用API,51-57天租用云GPU |
| 代码管理 | 全程Git提交 |
| 辅助资料 | 每日课后习题、每周思维导图、高频面试题库200题 |
| 能力认证 | 3个阶段项目+1个毕业设计 |
