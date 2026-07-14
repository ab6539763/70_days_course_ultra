# 第12天 · 网络请求与API调用(关键日)

> **周次/Sprint**:Sprint 0 · 新人内训(Day1-14)—— 阶段项目一:命令行多轮对话AI助手(预计Day14交付)
> **星期**:周五(第二周第五天)
> **学员**:陈铭、苏梦、韩露、张凡(导师:王振宇)
> **内训任务卡**:PYXT-ONB-D12(蓬远科技培训中心 · 新人训练营 · Day12)
> **今日关键词**:HTTP协议基础 / 请求方法与状态码 / `requests`库 / GET与POST / `headers`与`params`与`json`参数 / DeepSeek API首调 / 通义千问API首调 / API Key管理 / `messages`消息格式 / `choices[0].message.content`解析 / 401鉴权失败 / 429限流 / 超时排查 / 命令行AI问答小程序`ask_ai.py`

---

## 【旁白】

从Day1到Day11,陈铭写下的每一行代码,不管是通讯录管理系统里的字典操作,还是`cangqiong_core`包里精心设计的异常继承体系,骨子里都有一个共同的特点——它们运行的整个世界,都装在陈铭自己电脑那块屏幕的方寸之间。数据是自己造的,输入是自己编的,连"模型调用失败"这种听起来很唬人的场景,过去十一天里也只是`random.random() < 0.1`这样一句概率判断,用来假装"这里模拟了一次限流"。今天,这一切要翻篇了。今天陈铭要做的事情,说出来平平无奇——发一个HTTP请求,等一个HTTP响应——但这一次,请求真的会离开他的笔记本电脑,穿过公司的网络、穿过运营商的骨干网、抵达深圳或者杭州某个机房里真实运行着的大模型服务,然后带着一段真正由大模型生成的文字,原路返回。这是全书第一次,陈铭写的代码"离开了自己的电脑",去跟这个真实世界打了一次照面。

这件事的分量,不只是技术上的——虽然技术上确实也是一个不小的跨越:HTTP协议、`requests`库、API鉴权、JSON请求体的构造,这些概念此前陈铭或多或少听说过名字,但从未真正动手摸过。更重要的是心理上的分量。入职培训的前十一天,陈铭偶尔会有一种说不清楚的"悬浮感"——学的东西看起来都指向"AI应用开发"这个目标,可写出来的东西始终是`ChatMessage`类、异常继承体系、批量文档处理工具这类听起来更像"通用编程能力"的东西,离"AI"这两个字总感觉还差着一层看不见的窗户纸。今天这层窗户纸被捅破了。当陈铭的终端里第一次打印出一行不是他自己写死、而是DeepSeek模型真实生成的文字时,那种感觉,他后来在成长笔记里写下一句话:"原来这十一天不是绕远路,是在搭一架梯子,而今天,我终于爬上去,摸到了梯子顶端那扇窗户的窗框。"

老王对这一天同样格外看重,这份看重甚至体现在了一些不常见的细节里——比如他破例在下午申请了公司的正式会议室"起航"用了两个小时,只为了让四个新人在第一次调用真实API成功的那一刻,能有一个相对"正式"的仪式感,而不是各自蹲在工位上悄无声息地划过这个里程碑。产品经理林悦那天下午路过会议室,听见里面一阵压抑不住的低声惊呼,好奇地探头进来看了一眼投影上滚动打出的那段AI回复,笑着说了一句"这就是我们苍穹以后天天要干的事啊,你们今天摸到的,就是最核心的那根线"。这句话说得轻描淡写,但对陈铭来说,却像是被人当面确认了一件一直悬在心里、不敢完全相信的事——他现在写的代码,和公司真正在做的产品,确确实实是同一条技术脉络上的东西,不是两件事,是一件事的不同阶段。

技术上还有一层不那么显眼、却贯穿全书的因果关系值得提前点破:今天写出来的`ask_ai.py`,注定是一个"脆弱"的版本——它裸调用网络请求,没有任何超时重试、没有任何限流应对,一旦网络稍有波动,或者请求发得稍微密集了一点,它就会毫不留情地崩掉,抛出陈铭从未见过的报错。这不是今天没写好,恰恰相反,这是故意留出来的一道缺口——明天(Day13),老王会拿着这个还带着"体温"的脆弱版本反问陈铭一句话:"网络请求这种事,出错是常态,不出错才是运气好,你打算怎么办?"由此引出装饰器与重试机制这条全新的技术主线。今天的不完美,是明天故事的起点,这也是整整七十天里反复出现的叙事节奏——每一天的"交付物",都同时是下一天的"问题清单"。

老王在晚上给培训群留的那句评语——"恭喜,你们已经比80%的转行者摸到真东西了"——听起来像一句随口的鼓励,但如果拆开来看,其实分量不轻。老王后来私下跟陈铭多聊了一句:很多转行学AI应用开发的人,花了大量时间在"看教程""收藏文章""跟着敲一遍demo代码"上,却从来没有真正独立地、从零开始地调通过一次生产级的大模型API调用——遇到401报错不知道从哪查起,遇到429限流以为是自己代码写错了,遇到超时干脆放弃换个"更简单"的教程接着看。而陈铭他们今天做的事情,是真正把"HTTP协议→构造请求→处理鉴权→解析真实返回→排查真实报错"这一整条链路,踏踏实实走了一遍,哪怕走得磕磕绊绊、哪怕韩露的请求体格式写错了、哪怕苏梦对着一个`401`报错懵了十分钟——这份"磕磕绊绊但真的走完了"的经历,恰恰是很多"看起来学了很久"的人始终没有真正拥有过的东西。这也是为什么,这一天会被称为"关键日"——它不是知识点密度最高的一天,却是整整十一天铺垫之后,第一次让"AI应用开发"这几个字,从一个抽象的职业方向,变成了陈铭亲手摸到过、亲眼看到过、亲手排查过报错的具体经验。

---

## 晨会纪要 / 今日目标

**时间**:上午8:50,二层小会议室"起航"
**出席**:王振宇(老王)、陈铭、苏梦、韩露、张凡

老王进门时脚步比平时快一点,手里没有拿电脑,只端着一杯咖啡,一屁股坐下就开口:"先说个事——今天下午,不管你们几点调通API,不管调通的是DeepSeek还是通义千问,第一时间发到培训群里,不用等收工。这是咱们训练营从第一天到现在,第一次真正连上大模型的一天,值得记一下。"

四个人对视了一眼,张凡先开口:"老王,听起来今天是个大日子啊。"

"算是。"老王把咖啡放下,"先看看昨天的收尾。"

**昨天进展(Day11)**:
- 四人的`doc_toolkit`批量文档处理工具全部跑通,`file_readers.py`的编码兜底逻辑普遍写得不错,韩露的csv报告修复了`newline=""`的问题。
- 苏梦昨晚回去按照老王的点评,把裸的`except Exception`改成了明确捕获`DecodeFailedError`等具体异常类型,今天早上提交的代码老王已经过了一眼,"改得对"。
- 陈铭想清楚了"提取规则可配置化"的思路,虽然还没有实现,但老王表示"想清楚比现在就写更重要,先记下来,不着急动手"。
- 张凡的正则表达式经过昨天点评已经收紧,不再把"1-2-3"这种明显不是日期的字符串误判为日期。

老王喝了一口咖啡,神情忽然认真起来:"接下来我要说一件事,这件事是今天一整天的起点。你们这十一天写的所有代码,有没有发现一个共同点?"

韩露想了想:"数据都是我们自己造的?"

"对,而且不只是数据。"老王在白板上写下几个词:模拟调用、模拟延迟、模拟报错概率。"你们的`OpenAIModel.call()`、`QwenModel.call()`,里面那些`time.sleep`加`random`判断,模拟的是'调用一个大模型API大概是什么感觉',但它终究是假的——它没有真的发出一次网络请求,没有真的经过一次DNS解析、TCP连接、TLS加密握手,也没有真的收到过一个来自远方服务器、由GPU算出来的字符串。"他停顿了一下,"今天,这些'模拟'的部分,要被替换成真的了。"

陈铭听到这句话,手心莫名有点冒汗——那种感觉很微妙,像是刚学完游泳的所有动作要领,教练突然说"好,今天下水"。

老王把白板翻到新的一页,写下今天的三段式安排:"今天信息量不算特别大,但每一步都'来真的',我建议你们操作的时候比平时更谨慎一点——尤其是涉及到API Key的部分,这是你们第一次真正持有一个'能花公司/个人真实额度的凭证',处理方式要认真,细节我等下午会专门强调。"

**今日目标**:

1. 上午9:00-12:00:HTTP协议基础(请求方法GET/POST、状态码分类、Header与Body的作用、URL结构)、`requests`库入门与详解(`get`/`post`方法、`params`/`headers`/`json`/`data`参数、`Response`对象的常用属性与方法)。
2. 下午14:00-17:30:DeepSeek API与通义千问(DashScope)API首次调用——API Key的申请与保存、请求体结构、`messages`消息格式、响应结构解析、真实调用DeepSeek `deepseek-chat`模型与Qwen `qwen-plus`模型拿到第一条真实回复;学习排查`401`鉴权失败、`429`限流、超时等真实错误。
3. 晚自习19:00-21:00:综合实战——开发命令行AI问答小程序`ask_ai.py`,支持连续追问(多轮对话),支持在DeepSeek与通义千问之间切换。

**风险点**:
- API Key安全是今天反复强调的重点,老王打算明确要求"密钥不写进代码文件本身",哪怕今天还没有系统学`.env`(那是Day13的内容),也要用最朴素的方式——`os.environ`手动设置或者用一个不会被提交进Git的本地文件——先把这个规矩立住。
- 网络环境可能存在不稳定因素(公司网络、部分学员用的移动热点),老王已经在群里提前提醒大家,如果长时间连不上,先检查网络本身,不要一上来就怀疑代码写错了。
- 401、429、超时这几类错误,新手很容易把它们混为一谈,统统归结为"网不好"或者"代码有bug",老王计划用真实复现的方式,让四个人亲眼看到这几类报错在报错文本、报错原因、排查思路上的具体差异。
- 晚自习的`ask_ai.py`工作量不小,但老王已经把它拆成了清晰的小步骤(先能问一次拿到答案,再支持连续追问,再支持切换模型),不要求今天就做到完美,重点是"跑通一条完整链路"。

---

## 需求文档:内部工具任务书 —— 命令行AI问答小程序`ask_ai.py`

> 本任务书由技术导师王振宇发放,格式沿用公司内部PRD模板。这是新人训练营第一份"真正对接外部生产系统"的任务书——此前所有任务书对接的都是本地文件或者内存里的数据结构,而今天这份任务书,第一次要求学员的代码去调用一个部署在公司之外、由第三方厂商运营的真实在线服务。

### 背景

老王在晨会上补充了这份任务书的业务背景:"苍穹平台最核心的能力之一,就是把用户的一句话,转发给某个大模型,再把大模型的回复,包装好返回给用户。这个动作,平台里每天要执行成千上万次。你们今天要写的这个命令行小工具,`ask_ai.py`,功能上极其简化——只是一个能在终端里跟AI对话的小程序,但它内部真正在发生的事情,和苍穹平台'对话引擎层'里正在跑的核心逻辑,本质上是同一件事的最小版本。这不是一句鼓励式的比喻,是真的——十几天之后(Day15开始),苍穹项目正式立项,大家会看到,对话引擎层要做的第一件事,依然是'构造请求→调用大模型API→解析返回→展示给用户',区别只是那时候要考虑更多的并发、更复杂的会话管理、更完善的容错。今天,你们先把这条最核心的链路,用最简单的方式跑通一次。"

具体的业务背景设定为:蓬远智能内部技术团队希望验证一个最小可用的"命令行AI问答工具",用于内部日常的快速提问场景(比如工程师想临时问一个技术问题、想让AI帮忙润色一段文字),同时作为验证"苍穹模型接入层"设计思路是否可行的一次早期原型探索。这个工具需要支持DeepSeek和通义千问两家模型服务商,原因是苍穹平台的一贯原则——不绑定单一供应商,任何一个模型接入层的组件,都要考虑"如果换一家供应商,代码改动量有多大"这个问题。

### 用户故事

- 作为一名内部工程师,我希望能在终端里直接输入问题,快速得到AI的回复,而不需要打开浏览器切换到某个网页。
- 作为一名需要连续提问的使用者,我希望这个工具支持"连续追问"——我问完第一个问题、得到回复之后,可以接着问第二个问题,并且AI能"记住"我们之前聊过什么,而不是每次都从零开始一次孤立的对话。
- 作为一名对成本敏感的团队成员,我希望在切换到不同的大模型服务商(DeepSeek或通义千问)时,不需要改动核心逻辑代码,只需要修改一个配置项或者启动参数。
- 作为一名刚接触API开发的新人,我希望这个工具在遇到网络问题、鉴权失败、限流等常见错误时,能给出清晰易懂的错误提示,而不是甩出一堆看不懂的原始报错堆栈然后直接崩溃退出。
- 作为一名重视信息安全的技术负责人,我希望这个工具在任何情况下都不会把API Key硬编码进代码文件,也不会把它打印到终端日志里。
- 作为技术导师,我希望通过这次练习,验证四位培训生是否已经具备"独立完成一次从零到一的真实API对接"这项对AI应用开发工程师而言最基础、也最核心的能力。

### 功能列表

| 编号 | 功能 | 说明 | 优先级 |
|---|---|---|---|
| F1 | 单轮问答 | 用户输入一个问题,程序调用大模型API,打印出回复内容 | P0(必须实现) |
| F2 | 多轮连续追问 | 程序在内存中维护一份对话历史(`messages`列表),每次新提问时,把历史一起带上,支持AI"记住"上下文 | P0 |
| F3 | 双模型服务商支持 | 支持通过启动参数或者交互指令,在DeepSeek(`deepseek-chat`)与通义千问(`qwen-plus`)之间切换 | P0 |
| F4 | API Key安全加载 | API Key只能通过环境变量`DEEPSEEK_API_KEY`/`QWEN_API_KEY`读取,代码中不允许出现任何硬编码密钥字符串 | P0 |
| F5 | 常见错误友好提示 | 针对401鉴权失败、429限流、超时、网络连接失败等场景,给出明确、可操作的中文提示,而不是让原始异常直接冒出来终止程序 | P0 |
| F6 | 退出与帮助指令 | 支持输入`/exit`或`/quit`退出程序,输入`/help`查看帮助,输入`/clear`清空对话历史重新开始 | P1(尽量实现) |
| F7 | 响应耗时统计 | 每次得到回复后,打印本次请求耗费的时间(秒),方便直观感受网络调用的开销 | P1 |
| F8 | 对话历史落盘 | 程序退出时,可以选择将本次对话历史保存为一份带时间戳的json文件,方便回溯 | P2(有时间再做) |

### 验收标准

1. 程序能够成功调用DeepSeek或通义千问API,拿到并正确打印出真实的AI回复内容,不能是模拟数据。
2. 支持至少3轮连续追问,且AI的回复能够体现出"记得"之前对话内容(比如追问"我刚才问了什么")。
3. API Key必须通过环境变量加载,代码仓库(包括Git提交历史)中不能出现任何真实密钥字符串。
4. 故意制造401(改错Key)、429(如可复现)、超时(设置极短超时时间)三类错误场景,程序均能给出清晰的中文错误提示,而不是直接抛出未处理异常导致进程崩溃。
5. 代码需要通过`black`格式化检查,函数需要有清晰的文档字符串说明参数与返回值。
6. 支持通过启动参数(如`--provider deepseek`或`--provider qwen`)指定使用哪一家模型服务商。

---

## 架构设计图:苍穹模型接入层中,今天这次调用处在什么位置

老王在讲需求之前,先把苍穹平台的整体分层图(第一天入职时就见过的那张海报级架构图)重新投到白板上,专门圈出了"模型接入层"这一块,说:"你们今天写的东西,虽然是个孤零零的命令行小工具,但它对应的位置,就是这一块。我希望你们脑子里始终有这张图,不要把今天的练习当成一次孤立的、'为了完成作业而完成作业'的demo。"

```mermaid
graph TD
    subgraph PLATFORM["苍穹企业级智能体中台(远期完整形态, 陆续搭建中)"]
        UI["苍穹控制台(Web前端)<br/>对话工作台 / 知识库管理 / Agent编排 / 模型管理"]
        API["苍穹开放API(FastAPI)<br/>/chat /knowledge /agent /finetune /admin"]

        subgraph CORE["核心能力层"]
            DIALOG["对话引擎层<br/>多轮记忆 / 流式输出"]
            RAG["RAG检索引擎层<br/>(尚未开始, 第三阶段搭建)"]
            AGENT["Agent编排层<br/>(尚未开始, 第四阶段搭建)"]

            subgraph MODELACCESS["模型接入层(今天真正动手的地方)"]
                ROUTER["模型路由/选择逻辑<br/>决定这次请求发给谁"]
                HTTPCLIENT["HTTP调用客户端<br/>requests.post(...)"]
                DSADAPT["DeepSeek适配逻辑<br/>call_deepseek_api.py"]
                QWADAPT["通义千问适配逻辑<br/>call_qwen_api.py"]
                ERRHANDLE["错误处理与分类<br/>401/429/超时 -> 自定义异常"]
            end
        end

        INFRA["基础设施层<br/>向量数据库 / 关系数据库 / 对象存储 / Docker"]
    end

    subgraph EXTERNAL["外部世界(公司之外, 真实存在的第三方服务)"]
        DSAPI["DeepSeek开放平台<br/>api.deepseek.com"]
        QWAPI["阿里云DashScope<br/>OpenAI兼容接口"]
    end

    UI --> API
    API --> DIALOG
    DIALOG --> ROUTER
    ROUTER -->|"provider=deepseek"| DSADAPT
    ROUTER -->|"provider=qwen"| QWADAPT
    DSADAPT --> HTTPCLIENT
    QWADAPT --> HTTPCLIENT
    HTTPCLIENT -->|"真实的HTTP网络请求"| DSAPI
    HTTPCLIENT -->|"真实的HTTP网络请求"| QWAPI
    DSAPI -->|"HTTP响应(JSON)"| HTTPCLIENT
    QWAPI -->|"HTTP响应(JSON)"| HTTPCLIENT
    HTTPCLIENT --> ERRHANDLE
    ERRHANDLE --> DIALOG
    RAG -.->|"第三阶段接入"| DIALOG
    AGENT -.->|"第四阶段接入"| DIALOG
    CORE --> INFRA

    style MODELACCESS fill:#fff3cd,stroke:#d4a017,stroke-width:2px
    style EXTERNAL fill:#e8f4ea,stroke:#2e7d32,stroke-width:2px
```

老王指着图里高亮的两块——黄色的"模型接入层"和绿色的"外部世界"——解释说:"你们今天的`ask_ai.py`,现在还是一个孤立的脚本,没有对话引擎层、没有API层、没有前端,但它内部真正在跑的逻辑,和图里`模型接入层`这个方框里的东西,是一模一样的三件事:第一,决定这次请求该发给DeepSeek还是通义千问(路由选择);第二,真正发出一次HTTP请求,穿过图里那条'真实的HTTP网络请求'的箭头,到达外部世界;第三,拿到响应之后,要先经过错误处理与分类,再把结果往上层传。图里画的这条从`HTTPCLIENT`穿到`DSAPI`/`QWAPI`再穿回来的箭头,今天你们要亲手让它'真的动起来'一次,而不是像之前十一天那样,只在图纸上画一画。"

---

## 流程图:一次API调用的完整请求生命周期

为了让大家在动手写代码之前,先在脑子里过一遍"点下回车之后,到底发生了什么",老王画了这张更细粒度的时序图,专门标出一次真实API调用从"用户按下回车"到"终端打印出回复"之间,请求经历的每一步。

```mermaid
sequenceDiagram
    participant U as 用户(陈铭)
    participant APP as ask_ai.py 程序
    participant DNS as DNS解析
    participant NET as 网络传输层(TCP/TLS)
    participant SRV as DeepSeek/通义千问 API服务器
    participant LLM as 大模型推理引擎

    U->>APP: 输入问题, 按下回车
    APP->>APP: 把新问题追加进messages历史列表
    APP->>APP: 构造请求体(model/messages/temperature等)
    APP->>APP: 设置headers(Authorization携带API Key)
    APP->>DNS: 解析目标域名(api.deepseek.com / dashscope...)
    DNS-->>APP: 返回目标服务器IP地址
    APP->>NET: 建立TCP连接, 完成TLS握手(HTTPS加密通道)
    NET->>SRV: 发送HTTP POST请求(headers+body)
    SRV->>SRV: 鉴权校验(检查Authorization中的API Key是否有效)

    alt 鉴权失败
        SRV-->>APP: 返回401状态码 + 错误信息JSON
        APP->>U: 打印"API Key无效或已过期"的清晰提示
    else 触发限流
        SRV-->>APP: 返回429状态码 + retry建议
        APP->>U: 打印"请求过于频繁, 请稍后重试"提示
    else 鉴权通过, 正常处理
        SRV->>LLM: 把messages转发给大模型推理引擎
        LLM->>LLM: 逐token生成回复内容(推理计算)
        LLM-->>SRV: 生成完成, 返回完整回复文本
        SRV-->>APP: 返回200状态码 + JSON响应体(choices/usage等)
        APP->>APP: 解析response.json(), 取出choices[0].message.content
        APP->>APP: 把AI的回复也追加进messages历史列表
        APP->>U: 打印AI回复内容 + 本次耗时
    end

    Note over U,LLM: 一次来回, 请求经过了本地代码构造、DNS解析、<br/>TCP/TLS网络传输、服务端鉴权、模型推理、原路返回解析<br/>这六个环节, 任何一个环节都可能失败, 这正是Day13要解决的问题
```

老王讲这张图的时候特别强调了两点:"第一,你们平时说'调个API'听起来就是一行代码的事,但实际上背后是这么多个环节串联起来的一整条链路,`requests`库帮你把DNS解析、TCP连接、TLS加密这些底层细节都封装好了,你写代码的时候看不见它们,但它们是真实发生的,而且真实地会消耗时间、真实地可能出错。第二,注意图里那个`alt`分支——鉴权失败、限流、正常处理,这是三条完全不同的路径,对应着完全不同的处理方式,这也是为什么下午我会专门带你们分别复现这几种情况,而不是笼统地讲一句'调API可能会出错'就完事。"

---

## 示意图:一次HTTP请求与响应的结构

在正式写代码之前,老王要求大家先搞清楚一次HTTP请求和响应,到底"长什么样子"——不是抽象的概念,而是具体到"有哪几个部分,每个部分里装的是什么"。

```mermaid
flowchart TB
    subgraph REQUEST["HTTP请求(客户端 -> 服务器)"]
        RL["请求行: POST /v1/chat/completions HTTP/1.1"]
        RH["请求头 Headers:<br/>Content-Type: application/json<br/>Authorization: Bearer sk-xxxxxxxx"]
        RB["请求体 Body(JSON):<br/>model, messages, temperature, max_tokens 等参数"]
        RL --> RH --> RB
    end

    subgraph RESPONSE["HTTP响应(服务器 -> 客户端)"]
        SL["状态行: HTTP/1.1 200 OK"]
        SH["响应头 Headers:<br/>Content-Type: application/json<br/>x-ratelimit-remaining: 剩余配额信息"]
        SB["响应体 Body(JSON):<br/>id, choices[0].message.content,<br/>usage(消耗的token数量) 等字段"]
        SL --> SH --> SB
    end

    REQUEST -->|"经过网络传输"| RESPONSE
```

老王补了一句形象的比喻:"你可以把一次HTTP请求想象成寄一份快递——请求行就是快递单上写的'寄给谁、办什么事'(方法+路径);请求头就是快递单上的备注信息,比如'易碎品轻拿轻放'、这里的`Authorization`就相当于'身份证明,证明这份快递确实是你本人寄的,有权限寄这个东西';请求体才是真正装在箱子里的货物,也就是你真正想问的问题、想传的参数。响应也是一样的结构,只是方向反过来——状态行告诉你'这件事办得顺不顺利'(状态码),响应头是附加信息,响应体才是真正的'货'——也就是大模型生成的回复内容。"

韩露听完这个比喻,追问了一句:"那`Content-Type`算是快递单上的什么?"老王想了想:"算是'包装说明'——告诉收件人'这个箱子里装的是什么形态的东西, 用什么方式拆开才对'。你写`application/json`, 就是告诉服务器'我这个箱子里装的是JSON格式的文字, 麻烦你按JSON的规则解析', 如果你的Body其实是表单格式, 但Header里却写了`application/json`, 服务器很可能会按JSON的规则去解析一份根本不是JSON的内容, 直接解析失败, 报出格式错误——这也是为什么我一直强调, 用`requests`的`json=`参数而不是自己手动拼`data=`加`json.dumps()`, 是因为前者能保证'箱子里装的东西'和'包装说明上写的'始终是对得上的, 不会出现这种'货不对板'的错误。"这段追问被老王记进了当天的"值得留下来的课堂提问"清单,他事后跟陈铭提过一句,"问出'这个字段到底对应现实里的什么东西'这种问题,说明理解正在往深处走,不是停留在'记住怎么写代码'这个层面。"

---

## 课堂笔记

### 上午:HTTP协议基础与`requests`库详解

上午9点整,老王没有直接打开IDE,而是先在白板上画了一个简化的网络示意图——一台笔记本电脑,中间画了一条波浪线代表"互联网",另一端画了一个服务器机柜图标,标注"api.deepseek.com"。他说:"在讲任何代码之前,先把这几个词的意思钉死——请求(Request)、响应(Response)、状态码(Status Code)、请求头(Header)、请求体(Body)。这几个词,你们以后每天工作里都会用到,今天必须一次搞清楚,不留模糊地带。"

**HTTP是什么**。老王的解释比较朴素:"HTTP,超文本传输协议,你可以先不管这个名字里'超文本'具体指什么,先记住一句更实用的话——HTTP是一套'客户端怎么问服务器要东西、服务器怎么把东西还给客户端'的约定好的规则。你打开浏览器访问任何一个网页,背后都是你的浏览器(客户端)按照HTTP协议的规则,给某台服务器发了一个请求,服务器按照同样的规则,把网页内容打包成一个响应发回来。我们今天要做的事情,本质上和打开一个网页没有任何区别——只是我们不用浏览器,改用Python代码充当'客户端'这个角色,而对方服务器返回的不是一个网页,是一段JSON格式的文字,里面装着AI生成的回复。"

苏梦举手问:"那HTTPS和HTTP是什么关系?我看DeepSeek的接口地址是`https://`开头的。"

"HTTPS就是加了一层加密的HTTP。"老王回答,"普通的HTTP,数据在网络上是'裸奔'的,理论上如果有人在你和服务器之间的某个网络节点上'偷听',是能看到你传输的具体内容的——这在传输API Key这种敏感信息的场景下是绝对不能接受的。HTTPS在HTTP的基础上加了一层TLS加密,你可以理解成'把要寄的东西先上了一把锁,再交给快递员,只有收件人有钥匙能打开',即使中途被人截获,拿到的也是一堆加密后的乱码,解不开。现在几乎所有正式的API服务,包括我们今天要用的DeepSeek和通义千问,都要求必须用HTTPS,不允许用不加密的HTTP,这是行业里最基本的安全底线。"

**请求方法(HTTP Method)**。老王重点讲了两种最常用的方法——GET和POST,并强调二者的核心区别不在于"能不能带参数",而在于语义和用法习惯:

"GET,语义上代表'我要获取一份资源',典型场景是打开一个网页、查询一份数据。GET请求的参数,通常放在URL的查询字符串里(也就是`?key=value&key2=value2`这种形式),这带来一个直接的后果——参数会明明白白地出现在URL里,长度有限制(不同浏览器/服务器限制不同,但通常几KB级别),而且不适合传输敏感信息,因为URL经常会被浏览器历史记录、服务器访问日志之类的地方记录下来。

POST,语义上代表'我要提交一份数据、让服务器处理这份数据并产生某种变化',典型场景是提交一个表单、上传一段内容、发起一次操作。POST请求的数据放在请求体(Body)里,不会出现在URL上,长度限制通常也远比GET宽松得多。我们今天要调用的大模型API,几乎清一色用的是POST——原因很直观,你要问AI的问题,可能是一大段文字,还要带上一堆参数(模型名称、历史对话、温度参数等等),这些内容显然不适合塞进URL里,而且这些'鉴权信息+具体问题'这类内容,理应通过请求体和请求头传输,而不是暴露在URL上。"

张凡问得比较细:"那状态码呢,我看过一些文档里提到200、404、500这些数字,是怎么分类的?"

老王在白板上写下分类表,这是他准备好的内容:

- **1xx(信息性)**:表示请求已被接收,正在处理,实际开发中很少直接打交道,今天不展开。
- **2xx(成功)**:最常见的是`200 OK`,表示请求成功并且服务器正常返回了内容;还有`201 Created`(资源创建成功)等。今天调用API成功,看到的就是`200`。
- **3xx(重定向)**:表示请求的资源被转移到了别的地方,浏览器/客户端需要跟着跳转,今天暂时不会遇到。
- **4xx(客户端错误)**:表示问题出在请求这一方——最常见的是`400 Bad Request`(请求格式本身有问题,比如JSON格式错误)、`401 Unauthorized`(未通过鉴权,通常是API Key错误或缺失)、`403 Forbidden`(鉴权通过了但没有权限访问这个资源)、`404 Not Found`(请求的地址不存在,比如URL写错了)、`429 Too Many Requests`(触发限流,请求太频繁)。今天下午会真实遇到401和429。
- **5xx(服务器错误)**:表示问题出在服务器这一方,比如`500 Internal Server Error`(服务器内部出了未知错误)、`502 Bad Gateway`、`503 Service Unavailable`(服务暂时不可用,常见于服务器过载或维护)、`504 Gateway Timeout`(网关等待上游服务响应超时)。这类错误通常不是我们代码的问题,遇到了应该重试或者联系服务商,而不是死磕自己的代码。

老王说:"这几个分类,不需要你们把每个具体数字都背下来,但要记住这个分类的思路——看到4字头,先怀疑自己这边的请求有没有问题(参数、鉴权、格式);看到5字头,先怀疑对方服务器那边是不是暂时出了问题,大概率不是你代码写错了。这个判断思路,能帮你在排查问题的时候,第一时间把范围缩小一半。"

**请求头(Headers)与请求体(Body)**。老王强调这两者经常被新手搞混,他给出一个简单的区分方式:"Headers是'元信息',是描述这次请求'附加信息'的部分,不是这次请求真正想传达的核心内容——比如`Content-Type`告诉对方'我这次body里装的是什么格式的数据'(比如`application/json`表示是JSON格式);`Authorization`告诉对方'我是谁,我有没有权限做这件事';`User-Agent`告诉对方'我是用什么工具/浏览器发起这次请求的'。Body才是真正的、核心的、你这次请求想要传达的数据本身。今天调用大模型API,Headers里最重要的就是`Authorization`(带着API Key,证明你的身份和额度)和`Content-Type`(告诉对方body是JSON格式),Body里装的才是真正的问题内容——你的`messages`、模型名称这些。"

**`requests`库的安装与最基础用法**。讲完协议层面的概念,老王终于打开了IDE,带着大家安装第三方库:

"Python标准库里其实也有一个能发HTTP请求的模块,叫`urllib`,但写起来相当繁琐,业界几乎所有人写Python的HTTP请求代码都会选`requests`这个第三方库——它把很多底层细节都封装好了,写起来非常直观。先装库。"

老王让大家在各自的虚拟环境里执行:

```bash
pip install requests
```

然后现场演示了`requests`库最基础的GET请求用法,用的是一个公开的、专门用于教学和测试HTTP请求的免费站点`httpbin.org`(这个网站会原样把你发过去的请求信息回显给你,非常适合用来观察"我到底发了什么、对方收到了什么"):

```python
import requests

response = requests.get("https://httpbin.org/get")
print(response.status_code)
print(response.text)
```

陈铭跟着敲完这几行,运行之后看到终端里打印出`200`,以及一大段JSON格式的文字,里面包含了他这次请求的各种信息——他的请求头、他的"来源IP"(经过公司网络出口的IP)、请求的URL等等。陈铭愣了一下:"这个网站是怎么知道我发了什么的?"

老王笑了:"因为`httpbin.org`这个网站,本身的设计目的就是'你发给我什么,我就原样告诉你我收到了什么',专门给学网络请求的人用的,不用担心它是恶意网站,这是一个业界很多教程都会用的公开测试服务。你现在明白了——你的请求,不是凭空消失,是真的经过网络,被对方服务器实实在在地收到了,对方服务器还看得到你请求里带的各种信息。"

接着老王讲`requests.get()`的常见参数用法,尤其是`params`参数(用于构造GET请求的查询字符串):

```python
import requests

params = {"name": "chenming", "role": "trainee"}
response = requests.get("https://httpbin.org/get", params=params)
print(response.url)
# 打印结果类似:https://httpbin.org/get?name=chenming&role=trainee
```

"注意看,"老王说,"你不需要自己手动拼接`?name=chenming&role=trainee`这段字符串,`requests`帮你自动把字典转成了正确格式的查询字符串,还会自动处理特殊字符的转义(比如空格、中文这种URL里不能直接出现的字符)。如果你自己手写字符串拼接,迟早会在某个包含特殊字符的参数上栽跟头,这是`requests`库替你省掉的第一个坑。"

紧接着讲`headers`参数——如何在请求中附带自定义的请求头:

```python
import requests

headers = {"User-Agent": "cangqiong-training-bot/1.0"}
response = requests.get("https://httpbin.org/headers", params={"debug": "true"}, headers=headers)
print(response.json())
```

老王特别提到`response.json()`这个方法:"如果你确定对方返回的内容是JSON格式,不用自己手动`import json`再`json.loads(response.text)`,直接调`response.json()`,`requests`帮你一步做完解析,返回的就是Python的字典或列表。这个方法你们下午调大模型API的时候会用到很多次,记住它。"

然后是POST请求,老王强调POST最重要的是搞清楚`data`参数和`json`参数的区别,这是他专门设计的一个容易踩坑的对比案例:

```python
import requests

# 用data参数:requests会把字典编码成表单格式(application/x-www-form-urlencoded)
response1 = requests.post(
    "https://httpbin.org/post",
    data={"question": "什么是HTTP协议"},
)
print(response1.json()["headers"]["Content-Type"])
# 输出: application/x-www-form-urlencoded

# 用json参数:requests会自动把字典序列化为JSON字符串,并自动设置正确的Content-Type
response2 = requests.post(
    "https://httpbin.org/post",
    json={"question": "什么是HTTP协议"},
)
print(response2.json()["headers"]["Content-Type"])
# 输出: application/json
```

老王敲重点:"这个区别,是今天下午你们调大模型API最容易踩的坑之一——几乎所有大模型API,包括DeepSeek和通义千问,都要求请求体是JSON格式,`Content-Type`必须是`application/json`。如果你手误用了`data=`而不是`json=`,`requests`会把你的字典编码成表单格式而不是JSON格式,服务器那边收到的数据结构完全不对,大概率会返回一个`400 Bad Request`,报错信息可能还不会直接告诉你'你用错参数了',而是说'请求体格式不合法'这种比较抽象的话,排查起来会让新手摸不着头脑。所以我的建议是——只要是调大模型API这类要求JSON请求体的接口,永远用`json=`参数,不要用`data=`,这是个可以直接当规矩记下来的结论,不需要每次都重新判断。"

韩露在这里追问了一句:"如果我自己手动`json.dumps()`把字典转成字符串,再用`data=`参数传进去,是不是也可以?"

"技术上可以,效果和用`json=`参数基本等价,但你需要自己手动设置`headers={"Content-Type": "application/json"}`,因为`requests`不会帮你自动加这个头。"老王回答,"这就是为什么我建议直接用`json=`参数——它一步做了两件事(序列化+设置正确的Content-Type),用`data=`加手动`json.dumps()`则需要你自己记得做全,多一步就多一个可能忘掉的地方。工程上的选择,往往就是在'哪种写法让人更不容易犯错'之间做取舍。"

**`Response`对象的常用属性**。老王最后梳理了一遍`requests.Response`对象上最常用的几个属性和方法,列成一张小表让大家记在笔记本上:

- `response.status_code`:整数类型的HTTP状态码,比如`200`、`401`、`429`。
- `response.text`:响应体的原始文本内容(字符串)。
- `response.json()`:把响应体当作JSON解析,返回Python字典/列表;如果响应体本身不是合法JSON,这个方法会抛出异常。
- `response.headers`:响应头,一个类似字典的对象,可以用`response.headers.get("Content-Type")`这样的方式取值。
- `response.ok`:布尔值,状态码在200-399之间时为`True`,否则为`False`,可以用来做一个粗粒度的"这次请求算不算成功"的快速判断。
- `response.raise_for_status()`:如果状态码是4xx或5xx,主动抛出一个`requests.exceptions.HTTPError`异常;如果是2xx,什么都不做。这个方法在很多场景下能省掉手写`if response.status_code >= 400`判断的麻烦。

老王补充说:"`raise_for_status()`这个方法,我个人挺推荐用,它能让你的代码更'声明式'——你不需要每次手写状态码判断,直接调用它,配合`try/except`,出错自然会走到异常处理分支。不过要注意,它只对4xx/5xx敏感,如果服务器返回了200,但响应体本身的JSON结构不符合你的预期(比如缺了某个关键字段),`raise_for_status()`是发现不了的,这种情况需要你自己额外校验响应体结构,这个我们下午会碰到实际案例。"

上午临近12点的时候,老王给大家留了一个小任务作为过渡:"中午吃饭前,把今天要用到的两份API Key先申请好——DeepSeek开放平台的Key,以及阿里云DashScope的通义千问Key。申请流程我发在群里了,都是免费注册加实名认证,新用户通常都会送一点免费额度,足够我们今天和接下来练习用的。申请的时候一定注意——Key拿到手之后,先复制粘贴到一个本地的、不会被提交进Git的文本文件里存好,千万不要直接粘贴到聊天窗口或者截图发到群里,这是我们下午要重点强调的安全规矩,先养成习惯。"

### 下午:DeepSeek/通义千问API详解 —— 从申请Key到解析真实回复

下午2点,四个人陆陆续续吃完饭回到工位,老王先在群里确认了一遍每个人的API Key是否申请成功。张凡的DeepSeek账号实名认证还在审核中,老王让他先跟着看,用陈铭的账号(经陈铭同意,且用完这次演示后陈铭会记得去后台看一下自己的调用记录)现场做演示,晚点张凡自己的账号通过审核后再补跑一遍。

**第一步:确认API Key,理解鉴权方式**。老王先讲清楚这两家服务商的鉴权方式,虽然细节有差异,但本质思路相同:

"DeepSeek和通义千问,包括几乎所有主流大模型API,都采用同一种鉴权方式——Bearer Token。你申请到的那一长串以`sk-`开头的字符串,就是你的Token(令牌)。每次发请求,你要把它放进请求头的`Authorization`字段,格式固定是`Bearer <你的Key>`,注意`Bearer`和Key之间有一个空格,这是HTTP鉴权的一种标准约定,不是DeepSeek或者通义千问自己发明的格式。服务器收到请求后,第一步就是检查这个字段——如果没有这个字段,或者Key不对、过期、被禁用,直接返回`401 Unauthorized`,后面的事情都不会再往下走。"

老王在白板上写下这行示意:

```
Authorization: Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

"这个Key,你可以理解成一张'能直接花钱的信用卡'——虽然大部分情况下额度不算特别大,但一旦泄露,别人可以拿着它调用API消耗你的额度,严重的话产生真实的费用。所以我要求的规矩非常明确:第一,永远不要把Key写死在代码文件里;第二,永远不要把Key提交到Git仓库(哪怕是私有仓库);第三,永远不要把Key发到聊天工具、截图、或者任何可能被别人看到的地方。今天我们还没系统学`.env`文件管理(那是明天的内容),所以今天先用最朴素但足够有效的方式——通过操作系统的环境变量来加载Key,代码里永远只写`os.environ.get("DEEPSEEK_API_KEY")`这样的读取逻辑,不写真实的字符串。"

老王让大家在各自的终端里,按照操作系统的方式设置环境变量(macOS/Linux在`~/.zshrc`或`~/.bashrc`里加一行`export DEEPSEEK_API_KEY="你的真实Key"`,Windows在系统环境变量设置里添加),然后重新打开一个终端窗口,用`echo $DEEPSEEK_API_KEY`(Windows是`echo %DEEPSEEK_API_KEY%`)验证是否设置成功。

**第二步:理解请求体结构与`messages`格式**。老王打开DeepSeek官方API文档,投影到大屏幕上,带大家一起读:

"你们注意看这份文档给的示例请求体,长这个样子:"

```json
{
  "model": "deepseek-chat",
  "messages": [
    {"role": "system", "content": "你是一个专业的助手。"},
    {"role": "user", "content": "你好,请介绍一下你自己。"}
  ],
  "temperature": 0.7,
  "max_tokens": 1024
}
```

"这个结构,你们眼熟吗?"老王故意问了一句。

苏梦反应很快:"这不是`ChatMessage`吗!我们Day8写的那个类,`role`加`content`两个字段,一模一样!"

"对,而且不是巧合。"老王点头,"你们Day8写`ChatMessage`类的时候,`role`限定在`"system"`/`"user"`/`"assistant"`这三个值,这不是我随手定的规则,是照着几乎所有主流大模型厂商约定的消息格式设计的——`system`角色的消息,用来设定AI的'人设'或行为规范,通常放在对话最开头,只出现一次;`user`角色代表用户说的话;`assistant`角色代表AI自己之前说过的话(在多轮对话里,把AI上一轮的回复重新作为`assistant`消息放回`messages`列表,才能让AI'记住'上下文,因为大模型本身是没有'记忆'的,它每次收到的,都是完整的一份`messages`列表,靠这份列表里的历史记录,'装出'记得之前聊过什么的样子)。"

这句话让陈铭愣了一下:"等等,所以AI并不是真的'记得'我们聊过什么,是我们每次把之前聊过的内容原样再发一遍过去?"

"完全正确。"老王说,"这是理解大模型对话API最关键的一个认知转折点——从用户的角度看,好像AI'记住'了你们聊过的一切,体验上是连续的;但从技术实现的角度看,每一次请求都是完全独立、无状态的,大模型服务器不会帮你保存'上次跟这个用户聊了什么',你每次发过去的`messages`列表,必须自己在本地维护、自己拼接完整,发多少内容过去,AI就'知道'多少内容。这也是为什么等一下你们写`ask_ai.py`的时候,'维护一份不断增长的messages列表'会是整个程序最核心的一段逻辑,不比调用API本身简单。"

老王接着讲每个字段的含义:

"`model`字段,指定要用哪个具体模型,DeepSeek这边我们用`deepseek-chat`(它的通用对话模型),通义千问这边等下会看到用`qwen-plus`。`temperature`,取值通常在0到2之间(不同厂商范围可能略有差异),数值越低,AI的回复越'保守''确定',同样的问题多问几次,答案会比较接近;数值越高,回复越'有创造性''发散',但也更容易出现不太靠谱的内容。`max_tokens`,限制这次回复最多生成多少个token(token不完全等同于'字',但可以粗略理解为文字长度的一种计量单位),用来控制回复长度和调用成本。"

**第三步:实际调用DeepSeek API**。老王现场编写代码,逐行讲解:

```python
import os
import requests

api_key = os.environ.get("DEEPSEEK_API_KEY")
if not api_key:
    raise RuntimeError("未检测到DEEPSEEK_API_KEY环境变量,请先设置后再运行")

url = "https://api.deepseek.com/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {api_key}",
}
payload = {
    "model": "deepseek-chat",
    "messages": [
        {"role": "system", "content": "你是蓬远智能内部的技术助手。"},
        {"role": "user", "content": "用一句话解释一下什么是HTTP协议。"},
    ],
    "temperature": 0.7,
    "max_tokens": 512,
}

response = requests.post(url, headers=headers, json=payload, timeout=30)
response.raise_for_status()
result = response.json()
reply = result["choices"][0]["message"]["content"]
print(reply)
```

陈铭跟着敲完这段代码,手指悬在回车键上方停了两秒——这大概是他这十一天以来第一次对"运行代码"这件事产生了一种类似"发送一封重要邮件之前"的紧张感。按下回车,终端先是短暂地卡了一下(大约一两秒钟,这正是网络请求真实的延迟),然后一行文字打印出来:

"HTTP协议是一种客户端与服务器之间约定好的规则,用于在网络上传输和交换数据,是互联网通信最基础的协议之一。"

陈铭盯着这行字看了几秒,忽然笑了出来,转头跟旁边的苏梦说:"这是真的,这不是我写的,是它自己'想'出来的。"苏梦这时候也刚好跑通了自己那份代码,两人对着屏幕又确认了一遍,确实是完全不同的措辞——同一个问题,DeepSeek给他俩的回复用词不完全一样,这个细节让"这确实是模型实时生成的,不是提前写好的固定文案"这件事变得格外真实可信。

老王让大家在群里发一下自己拿到的第一条真实回复截图,群里瞬间热闹起来。他等大家分享完,补了一句:"恭喜,你们已经比80%的转行者摸到真东西了。"这句话发出去之后,陈铭对着屏幕愣了几秒,把这句话截图保存了下来。

**第四步:解析返回结构,理解`choices`和`usage`**。老王让大家打印完整的`result`看一看,而不只是取出最终的回复文字:

```python
import json

print(json.dumps(result, indent=2, ensure_ascii=False))
```

打印出来的完整结构大致是这样:

```json
{
  "id": "chatcmpl-xxxxxxxxxxxxxxxxxxxxx",
  "object": "chat.completion",
  "created": 1752300000,
  "model": "deepseek-chat",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "HTTP协议是一种客户端与服务器之间约定好的规则……"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 28,
    "completion_tokens": 34,
    "total_tokens": 62
  }
}
```

老王逐字段讲解:"`choices`是一个列表,理论上大模型API支持`n`参数,一次请求生成多个候选回复,`choices`里就会有多个元素,但绝大部分场景下(包括我们今天)只会有一个,所以固定取`choices[0]`。`message`字段的结构,和你们发过去的`messages`列表里每一条消息的结构完全一致——同样是`role`加`content`,只不过这次`role`固定是`assistant`,代表这是AI说的话。`finish_reason`表示这次生成'为什么结束'——`stop`表示模型自然结束了这次回答(说完了该说的话);还可能是`length`,表示还没说完,但达到了你设置的`max_tokens`上限,被强行截断了;这个字段值得留意,如果你发现AI的回复经常被莫名其妙地截断到一句话说到一半,大概率是`max_tokens`设置得太小了,这是`finish_reason`能帮你快速定位的一个真实场景。`usage`字段记录了这次请求消耗的token数量,分为输入部分(`prompt_tokens`,你发过去的所有内容,包括历史对话)、输出部分(`completion_tokens`,AI生成的回复)、总计(`total_tokens`)——这个字段直接关系到调用成本,大模型API通常是按token计费的,`usage`是你监控和控制成本最直接的数据来源。"

**第五步:切换到通义千问,验证"一套逻辑,两个厂商"**。老王接着讲通义千问的接入方式,特意强调这是选用了阿里云DashScope提供的"OpenAI兼容接口",目的是让大家直观感受"不同厂商但接口形态高度相似"这件事:

"通义千问的接口,阿里云专门提供了一种'OpenAI兼容模式',意思是——你不需要学一套全新的、和DeepSeek长得完全不一样的接口规范,只需要把请求地址换成DashScope的兼容接口地址,把`model`字段换成`qwen-plus`,其他的请求体结构、`messages`格式、返回值结构,基本可以直接复用刚才DeepSeek那一套代码,几乎不用改。这不是巧合,而是行业里事实上形成的一种'标准'——因为OpenAI最早把这套`messages`格式和响应结构定义出来之后,后续绝大多数厂商为了方便开发者迁移,都主动兼容了这套接口形态,你们以后接入任何新的大模型服务商,大概率也会发现这个规律。"

```python
import os
import requests

api_key = os.environ.get("QWEN_API_KEY")
if not api_key:
    raise RuntimeError("未检测到QWEN_API_KEY环境变量,请先设置后再运行")

url = "https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {api_key}",
}
payload = {
    "model": "qwen-plus",
    "messages": [
        {"role": "system", "content": "你是蓬远智能内部的技术助手。"},
        {"role": "user", "content": "用一句话解释一下什么是HTTP协议。"},
    ],
    "temperature": 0.7,
    "max_tokens": 512,
}

response = requests.post(url, headers=headers, json=payload, timeout=30)
response.raise_for_status()
result = response.json()
reply = result["choices"][0]["message"]["content"]
print(reply)
```

韩露对比完两份代码,发现除了`url`、环境变量名、`model`字段值这三处不一样,其他几乎一字不差,她感慨了一句:"这也太像了,感觉换个厂商跟换个函数参数没什么区别。"

老王点头:"这正是我希望你们今天能体会到的一个工程认知——设计良好的抽象,能让'切换供应商'这件事,变成改几个配置项,而不是重写一整套代码。这也是为什么苍穹平台的模型接入层,当初的架构设计,一定要抽出一个统一的基类(还记得Day9的`BaseModel`吗),让`OpenAIModel`、`DeepSeekModel`、`QwenModel`这些具体实现,只需要各自填上不同的URL、不同的Key、不同的model名称,核心调用逻辑完全复用。你们今天亲手验证了一次这个设计思路为什么是对的。"

**第六步:真实错误复现——401鉴权失败**。讲完正常流程,老王开始"故意搞破坏",这是他事先计划好的教学环节:"接下来我要让你们看几种真实的报错,光听我说没有用,得自己眼睛看到、亲手排查一次才真正记得住。"

他让大家把环境变量`DEEPSEEK_API_KEY`临时改成一个明显错误的字符串(比如在末尾随手加几个字符),重新运行刚才那段调用代码。终端里报出:

```
requests.exceptions.HTTPError: 401 Client Error: Unauthorized for url: https://api.deepseek.com/v1/chat/completions
```

如果去掉`raise_for_status()`,直接打印`response.status_code`和`response.text`,能看到更具体的信息,类似:

```
状态码: 401
响应内容: {"error": {"message": "Authentication Fails, Your api key: ****Key is invalid", "type": "authentication_error"}}
```

老王讲解排查思路:"看到401,第一反应永远是——检查Key。具体检查这几件事:第一,环境变量有没有设置成功(用`echo`或者在代码里`print(api_key)`前几位字符确认,注意千万不要把完整Key打印到日志或者截图里,只打印前几位用于确认身份,比如`api_key[:8] + '...'`这样的写法,这是一个值得记住的安全小技巧);第二,Key有没有多余的空格或者换行符(有些人从网页复制的时候容易带上意外的空白字符);第三,Key是否已经过期或者被平台禁用(比如账户欠费、触发了平台的风控);第四,请求头里`Authorization`的格式是否正确,尤其是`Bearer`和Key之间那个空格有没有写对,漏了这个空格,同样会导致401。"

苏梦这时候正好遇到一个真实的问题,她一开始没有理解错误信息里"多余空格"的意思,把Key粘贴的时候不小心多复制了一个换行符,请求头变成了`Authorization: Bearer sk-xxxx\n`,导致连续报了十分钟401,自己怎么看代码逻辑都觉得没问题。老王过去看了一眼,让她用`repr(api_key)`打印出来,一下就看出了字符串末尾藏着一个`\n`。"这种'看起来一样,实际上藏着看不见的字符'的坑,`repr()`是最好的排查工具,它会把不可见字符也显示出来,这个技巧记一下,以后调试字符串问题经常用得上。"

**第七步:真实错误复现——429限流**。429不像401那样可以简单地"故意改错"来复现,老王的做法是让大家用一个简单的循环,在极短时间内连续发起多次请求,尝试触发限流(新注册账号通常有比较严格的并发限制或者速率限制):

```python
import time

for i in range(20):
    response = requests.post(url, headers=headers, json=payload, timeout=30)
    print(f"第{i+1}次请求, 状态码: {response.status_code}")
    if response.status_code == 429:
        print("触发限流! 响应内容:", response.text)
        break
```

张凡的账号真的触发了一次429,响应内容大致是:

```json
{"error": {"message": "Rate limit reached for requests", "type": "rate_limit_error"}}
```

老王借这个真实案例讲解:"429和401不一样,401是'你没有资格进门',429是'你有资格,但你敲门太快太频繁了,先等等'。看到429,不应该立刻重试(立刻重试大概率又会撞上同一个限流窗口),更合理的做法是等待一段时间之后再重试(有些API会在响应头里带`Retry-After`字段,明确告诉你该等多久;没有这个字段的话,通常等几秒到几十秒是比较安全的经验值)。这个'等一等再重试'的逻辑,听起来简单,但手动做很烦,明天(Day13)我们会用装饰器把这套逻辑自动化,今天先知道'429该怎么正确应对'这个原则就够了。"

**第八步:真实错误复现——超时**。老王让大家把`requests.post`的`timeout`参数临时改成一个极小的值(比如`timeout=0.01`,也就是10毫秒),正常情况下,大模型生成一次回复的时间通常在1秒到十几秒之间(取决于问题复杂度和回复长度),0.01秒几乎不可能等到任何服务器的响应:

```python
response = requests.post(url, headers=headers, json=payload, timeout=0.01)
```

运行后,终端报出:

```
requests.exceptions.ReadTimeout: HTTPSConnectionPool(host='api.deepseek.com', port=443): Read timed out. (read timeout=0.01)
```

老王解释:"这个错误的意思是——你的请求已经成功发出去了,但在你规定的时间(这里是0.01秒)内,没有等到服务器的响应,`requests`库主动放弃了等待,抛出了`ReadTimeout`异常。注意这和`ConnectTimeout`是不一样的概念——`ConnectTimeout`是连TCP连接都没建立起来(比如网络完全不通、目标地址根本连不上);`ReadTimeout`是连接已经建立、请求已经发出去了,但等回复等太久。你们平时设置`timeout`参数的时候,给一个合理的值很重要——设置得太短,正常的慢请求也会被误判为超时;设置得太长,一旦服务器真的卡住了,你的程序也会跟着卡住很久才能反应过来。今天我建议大家统一设置成30秒左右,这是一个对大模型这类响应耗时相对较长的API而言比较合理的经验值。"

老王最后总结这三类错误的应对思路,写在白板上,让大家抄下来:

"401——检查Key本身(是否设置、是否正确、有无多余空格);429——不要立即重试,等待后重试,长期方案是限流+重试策略;超时——检查网络本身是否通畅,检查`timeout`设置是否合理,长期方案是重试机制+合理的超时时间设置。这三类问题,今天先学会'识别+手动应对',明天学完装饰器,会把'等待后重试'这类重复逻辑自动化,不需要每次都手动写`try/except`再等一等再手动跑第二遍。"

### 晚自习:命令行AI问答小程序`ask_ai.py`完整实现

晚上7点,四个人回到工位开始晚自习,老王把今晚的目标写在群里置顶:"今晚一件事——把下午学的东西整合成一个能连续对话的命令行小程序`ask_ai.py`。我把功能拆成几个清晰的层级,你们不用一口气写完整版,按顺序来。"

**第一层:最简单的单轮问答**。老王先带大家写一个最基础的版本,只做"问一次、答一次",不涉及多轮记忆:

```python
import os
import sys
import requests


def call_deepseek(question: str) -> str:
    api_key = os.environ.get("DEEPSEEK_API_KEY")
    if not api_key:
        raise RuntimeError("未检测到DEEPSEEK_API_KEY环境变量")

    url = "https://api.deepseek.com/v1/chat/completions"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}",
    }
    payload = {
        "model": "deepseek-chat",
        "messages": [{"role": "user", "content": question}],
        "temperature": 0.7,
    }
    response = requests.post(url, headers=headers, json=payload, timeout=30)
    response.raise_for_status()
    result = response.json()
    return result["choices"][0]["message"]["content"]


def main():
    print("最简单版ask_ai, 输入exit退出")
    while True:
        question = input("你: ").strip()
        if question.lower() in ("exit", "quit"):
            break
        answer = call_deepseek(question)
        print(f"AI: {answer}")


if __name__ == "__main__":
    main()
```

大家跑通这个最简版本之后,老王让每个人都亲自体验一下"AI是不是真的没有记忆"这件事——先问一句"我叫陈铭",AI礼貌地回应了一句;紧接着问"我刚才说我叫什么?",AI给出的回答大多是"抱歉,我没有获取到您之前提到的名字信息"之类的话。陈铭看到这个结果,回想起下午老王讲的"每次请求都是独立无状态的"这句话,一下子有了非常具体的体感,不再是一句抽象的理论。

**第二层:加上多轮对话记忆**。老王引导大家意识到问题所在——"AI没有记忆,但你可以把历史对话,当作这次请求的一部分,一起发过去,让它'看起来'记得。"于是把`messages`从"每次临时构造一条"改成"维护成一个持续增长的列表":

```python
def build_conversation():
    """初始化一份对话历史, 包含一条system消息设定AI的角色。"""
    return [
        {
            "role": "system",
            "content": "你是蓬远智能内部培训助手, 说话简洁清晰, 优先给出可操作的回答。",
        }
    ]


def ask_with_history(conversation: list, question: str, api_key: str) -> str:
    """把新问题加入历史, 调用API, 并把AI的回复也加入历史, 返回本次回复内容。"""
    conversation.append({"role": "user", "content": question})

    url = "https://api.deepseek.com/v1/chat/completions"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}",
    }
    payload = {
        "model": "deepseek-chat",
        "messages": conversation,
        "temperature": 0.7,
    }
    response = requests.post(url, headers=headers, json=payload, timeout=30)
    response.raise_for_status()
    result = response.json()
    reply = result["choices"][0]["message"]["content"]

    conversation.append({"role": "assistant", "content": reply})
    return reply
```

苏梦在实现这一步的时候,犯了后来在Day13晨会纪要里被反复提到的那个错误——她把`conversation.append({"role": "user", "content": question})`误写成了直接把`question`本身赋值给`messages`字段(也就是`payload["messages"] = question`,一个字符串而不是一个列表),运行后拿到`400 Bad Request`。她排查了将近一个小时,反复检查API Key、检查URL、检查网络,都没有发现问题,最后在老王的提示下打印`payload`的完整内容才发现——`messages`字段本该是一个列表,却被她写成了一整个字符串。老王后来点评这个错误时说:"这种错误,报错信息本身通常不会直接告诉你'你把列表写成字符串了',它可能只会说'请求体格式不合法'这种比较笼统的话,排查思路应该是——出现`400`,先把你实际发出去的`payload`完整打印出来,对照官方文档的示例结构,一个字段一个字段地核对类型是否正确,而不是凭感觉瞎猜。"

**第三层:支持双模型服务商切换**。老王要求把DeepSeek和通义千问的调用逻辑,统一封装成一致的接口,方便随时切换:

```python
PROVIDER_CONFIGS = {
    "deepseek": {
        "url": "https://api.deepseek.com/v1/chat/completions",
        "env_key": "DEEPSEEK_API_KEY",
        "model": "deepseek-chat",
    },
    "qwen": {
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions",
        "env_key": "QWEN_API_KEY",
        "model": "qwen-plus",
    },
}


def call_model_api(provider: str, conversation: list) -> dict:
    """统一的模型调用入口, 根据provider选择对应厂商的配置, 返回完整的响应字典。"""
    if provider not in PROVIDER_CONFIGS:
        raise ValueError(f"不支持的provider: {provider}, 可选值: {list(PROVIDER_CONFIGS)}")

    config = PROVIDER_CONFIGS[provider]
    api_key = os.environ.get(config["env_key"])
    if not api_key:
        raise RuntimeError(f"未检测到环境变量{config['env_key']}, 请先设置后再运行")

    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}",
    }
    payload = {
        "model": config["model"],
        "messages": conversation,
        "temperature": 0.7,
    }
    response = requests.post(config["url"], headers=headers, json=payload, timeout=30)
    response.raise_for_status()
    return response.json()
```

老王看到这段代码,评价说:"这个`PROVIDER_CONFIGS`字典,是今天最值得留下来的一个设计——它把'不同厂商之间的差异',收敛成了几条配置数据,而不是写成一堆`if provider == "deepseek": ... elif provider == "qwen": ...`的分支逻辑。以后如果要接入第三个厂商,你只需要在字典里加一条新配置,不需要改动`call_model_api`函数本身的逻辑,这是一种很朴素但很实用的'开闭原则'体现——对扩展开放,对修改关闭。"

**第四层:错误处理与用户友好提示**。老王要求给`call_model_api`包一层错误处理,把`requests`抛出的各种原始异常,转换成对用户更友好的提示:

```python
def call_model_api_safely(provider: str, conversation: list) -> str:
    """在call_model_api基础上包装友好的错误提示, 返回AI的回复文本, 出错时返回错误说明字符串。"""
    try:
        result = call_model_api(provider, conversation)
        return result["choices"][0]["message"]["content"]
    except requests.exceptions.Timeout:
        return "[错误] 请求超时, 可能是网络较慢或服务暂时繁忙, 请稍后重试。"
    except requests.exceptions.ConnectionError:
        return "[错误] 网络连接失败, 请检查本机网络是否正常。"
    except requests.exceptions.HTTPError as exc:
        status = exc.response.status_code if exc.response is not None else None
        if status == 401:
            return "[错误] API Key鉴权失败, 请检查对应环境变量是否正确设置。"
        if status == 429:
            return "[错误] 请求过于频繁, 触发了限流, 请稍等片刻再试。"
        return f"[错误] 服务器返回了错误状态码: {status}"
    except (KeyError, IndexError):
        return "[错误] 返回结果结构异常, 无法解析出回复内容, 请检查API返回格式。"
```

**第五层:完整的命令行交互与指令支持**。最后把上面这些拼装成一个完整的、支持`/exit`、`/help`、`/clear`等指令的交互程序,这部分的完整代码在下面的"代码实战"章节中给出。老王强调这一层没有新的API调用知识,纯粹是把已经学过的字符串处理、循环、条件判断组织起来,重点是"代码结构要清晰,每个函数只干一件事"。

晚上9点,四个人陆续跑通了完整的`ask_ai.py`,每个人都用它连续问了好几轮问题,验证多轮记忆确实生效——问"我叫陈铭"然后追问"我刚才说我叫什么",AI这次能正确回答"你说你叫陈铭"。陈铭盯着这行输出,心里那种"这是真的"的感觉,比下午第一次拿到回复的时候更踏实了一层——因为这一次,不只是"调通了一个接口",而是自己一步步搭出了一个"看起来会持续记住对话"的完整体验,虽然知道这份"记忆"背后的真相只是messages列表在不断变长,但这份"知道真相之后依然觉得有意思"的感受,反而让他对这件事的理解更深了一层。

---

## 代码实战

> 以下代码构成今天完整的产出物:HTTP基础与`requests`库练习脚本、DeepSeek与通义千问API的独立调用示例、错误处理与自定义异常演示、以及晚自习综合实战产出的`ask_ai.py`命令行AI问答小程序。所有代码均可在本地Python 3.11环境下直接运行,调用大模型API相关的脚本运行前需要先设置好环境变量`DEEPSEEK_API_KEY`和`QWEN_API_KEY`。建议按`http_basics_demo.py → requests_usage_demo.py → api_exceptions.py → call_deepseek_api.py → call_qwen_api.py → error_handling_demo.py → ask_ai.py`的顺序阅读与运行。

### 文件一:`http_basics_demo.py`(HTTP协议基础练习)

```python
"""
文件名: http_basics_demo.py
说明:
    上午HTTP协议基础教学的配套练习脚本, 使用公开的httpbin.org测试站点,
    演示GET/POST请求方法、状态码分类、Headers与Body的作用、
    以及requests库最基础的用法, 不涉及任何真实的大模型API调用。

    httpbin.org是一个专门用于HTTP请求教学与调试的公开免费服务,
    它的设计目的就是"你发给我什么, 我原样回显给你", 非常适合
    用来直观观察一次HTTP请求/响应的真实结构。

    运行前提: 本机能够正常访问外网(httpbin.org)。
"""

from __future__ import annotations

import json
import time

import requests

BASE_URL = "https://httpbin.org"
TIMEOUT_SECONDS = 15


def demo_basic_get() -> None:
    """演示最基础的GET请求, 观察状态码与响应体结构。"""
    print("=" * 60)
    print("演示1: 最基础的GET请求")
    print("=" * 60)

    response = requests.get(f"{BASE_URL}/get", timeout=TIMEOUT_SECONDS)
    print(f"请求URL: {response.url}")
    print(f"状态码: {response.status_code}")
    print(f"是否成功(response.ok): {response.ok}")
    print("响应体(节选前300字符):")
    print(response.text[:300])
    print()


def demo_get_with_params() -> None:
    """演示GET请求携带查询参数, 观察requests如何自动拼接URL。"""
    print("=" * 60)
    print("演示2: GET请求携带params参数")
    print("=" * 60)

    params = {
        "trainee": "chenming",
        "day": "12",
        "topic": "网络请求与API调用",
    }
    response = requests.get(f"{BASE_URL}/get", params=params, timeout=TIMEOUT_SECONDS)
    print(f"最终拼接出的URL: {response.url}")

    data = response.json()
    print("服务器实际收到的查询参数(args字段):")
    print(json.dumps(data.get("args", {}), indent=2, ensure_ascii=False))
    print()


def demo_get_with_headers() -> None:
    """演示GET请求携带自定义请求头, 观察headers在Body中的回显。"""
    print("=" * 60)
    print("演示3: GET请求携带自定义headers")
    print("=" * 60)

    headers = {
        "User-Agent": "cangqiong-training-bot/1.0",
        "X-Trainee-Name": "chenming",
    }
    response = requests.get(f"{BASE_URL}/headers", headers=headers, timeout=TIMEOUT_SECONDS)
    data = response.json()
    print("服务器实际收到的请求头(headers字段, 节选):")
    server_headers = data.get("headers", {})
    for key in ("User-Agent", "X-Trainee-Name", "Host"):
        if key in server_headers:
            print(f"  {key}: {server_headers[key]}")
    print()


def demo_post_form_vs_json() -> None:
    """
    演示POST请求两种常见参数写法的核心差异:
        data=  -> 编码为表单格式(application/x-www-form-urlencoded)
        json=  -> 编码为JSON格式(application/json), requests自动设置正确的Content-Type
    这是新手在对接大模型API时最容易踩的一个坑, 单独拿出来对比演示。
    """
    print("=" * 60)
    print("演示4: POST请求 data参数 与 json参数 的区别")
    print("=" * 60)

    payload = {"question": "什么是HTTP协议"}

    response_form = requests.post(f"{BASE_URL}/post", data=payload, timeout=TIMEOUT_SECONDS)
    form_result = response_form.json()
    print("用data=发送时, 服务器收到的Content-Type:")
    print(f"  {form_result['headers'].get('Content-Type')}")
    print("用data=发送时, 服务器解析出的form字段:")
    print(f"  {form_result.get('form')}")
    print("用data=发送时, 服务器解析出的json字段(预期为None, 因为不是JSON):")
    print(f"  {form_result.get('json')}")
    print()

    response_json = requests.post(f"{BASE_URL}/post", json=payload, timeout=TIMEOUT_SECONDS)
    json_result = response_json.json()
    print("用json=发送时, 服务器收到的Content-Type:")
    print(f"  {json_result['headers'].get('Content-Type')}")
    print("用json=发送时, 服务器解析出的json字段:")
    print(f"  {json_result.get('json')}")
    print()


def demo_status_codes() -> None:
    """
    演示httpbin.org提供的一个专门用于返回指定状态码的接口,
    直观感受不同状态码分类下, response.ok与raise_for_status()的表现差异。
    """
    print("=" * 60)
    print("演示5: 不同状态码的表现")
    print("=" * 60)

    codes_to_try = [200, 201, 301, 400, 401, 404, 429, 500, 503]
    for code in codes_to_try:
        try:
            response = requests.get(
                f"{BASE_URL}/status/{code}",
                timeout=TIMEOUT_SECONDS,
                allow_redirects=False,
            )
            ok_text = "成功(2xx-3xx)" if response.ok else "存在问题(4xx/5xx)"
            print(f"状态码{code}: response.ok={response.ok} -> {ok_text}")
        except requests.exceptions.RequestException as exc:
            print(f"状态码{code}请求时出现异常: {exc}")

    print()
    print("使用raise_for_status()的效果演示:")
    response_404 = requests.get(f"{BASE_URL}/status/404", timeout=TIMEOUT_SECONDS)
    try:
        response_404.raise_for_status()
        print("未抛出异常(不应该走到这里)")
    except requests.exceptions.HTTPError as exc:
        print(f"raise_for_status()按预期抛出了异常: {exc}")
    print()


def demo_response_delay_and_timeout() -> None:
    """
    演示httpbin.org的延迟接口(/delay/n), 用于配合理解timeout参数的作用。
    这里故意设置一个比服务器延迟更短的timeout, 让ReadTimeout真实发生。
    """
    print("=" * 60)
    print("演示6: 响应延迟与timeout的关系")
    print("=" * 60)

    delay_seconds = 3
    print(f"请求一个会延迟{delay_seconds}秒才响应的接口, 设置timeout=1秒, 预期会超时……")
    start = time.time()
    try:
        requests.get(f"{BASE_URL}/delay/{delay_seconds}", timeout=1)
        print("未超时(不符合预期, 可能是网络环境特殊情况)")
    except requests.exceptions.ReadTimeout:
        elapsed = time.time() - start
        print(f"符合预期: 在约{elapsed:.2f}秒后触发了ReadTimeout异常")

    print(f"再次请求同一个延迟接口, 设置timeout=10秒, 预期能正常等到响应……")
    start = time.time()
    response = requests.get(f"{BASE_URL}/delay/{delay_seconds}", timeout=10)
    elapsed = time.time() - start
    print(f"成功获得响应, 状态码{response.status_code}, 实际耗时约{elapsed:.2f}秒")
    print()


def main() -> None:
    """按顺序运行全部演示函数, 每个函数彼此独立, 互不依赖。"""
    demo_basic_get()
    demo_get_with_params()
    demo_get_with_headers()
    demo_post_form_vs_json()
    demo_status_codes()
    demo_response_delay_and_timeout()
    print("全部HTTP基础演示运行完毕。")


if __name__ == "__main__":
    main()
```

### 文件二:`api_exceptions.py`(模型API异常体系, 承接Day10)

```python
"""
文件名: api_exceptions.py
说明:
    这套异常体系直接承接Day10在cangqiong_core.exceptions里设计的异常继承结构
    (CangqiongError -> ConfigurationError / ModelAPIError -> RateLimitError /
    ModelTimeoutError / ModelResponseParseError), 今天把它原样搬到独立的
    今日实战项目里, 区别在于: Day10这些异常抛出的场景是"模拟"的(random判断),
    而从今天起, 这些异常真正对应requests库抛出的、来自真实网络请求的原始异常,
    这是"同一套设计, 从教学模拟走向真实生产环境"的一次直接印证。
"""

from __future__ import annotations

from typing import Optional


class CangqiongError(Exception):
    """苍穹核心库所有自定义异常的基类, 定义与Day10完全一致。"""

    def __init__(self, message: str, detail: Optional[str] = None) -> None:
        super().__init__(message)
        self.message = message
        self.detail = detail

    def __str__(self) -> str:
        if self.detail:
            return f"{self.message}(详情: {self.detail})"
        return self.message


class ConfigurationError(CangqiongError):
    """
    配置错误: 今天最常见的触发场景是——对应环境变量(DEEPSEEK_API_KEY/
    QWEN_API_KEY)没有被正确设置, 在真正发起网络请求之前就应该被发现并终止。
    """

    def __init__(self, message: str, missing_key: Optional[str] = None) -> None:
        super().__init__(message)
        self.missing_key = missing_key


class ModelAPIError(CangqiongError):
    """模型API调用错误的基类, 今天开始真正对应requests库的原始异常与真实状态码。"""

    def __init__(
        self,
        message: str,
        provider: Optional[str] = None,
        status_code: Optional[int] = None,
    ) -> None:
        super().__init__(message)
        self.provider = provider
        self.status_code = status_code


class AuthenticationError(ModelAPIError):
    """
    鉴权失败: 对应服务器真实返回的401状态码。

    典型场景(今天亲手复现过的): API Key未设置、Key本身错误、Key包含
    多余的空白字符、Key已过期或被禁用。
    """

    def __init__(self, message: str, provider: Optional[str] = None) -> None:
        super().__init__(message, provider=provider, status_code=401)


class RateLimitError(ModelAPIError):
    """
    触发限流: 对应服务器真实返回的429状态码。

    调用方推荐的处理策略: 等待retry_after指定的秒数(如果服务器提供)
    之后再重试, 而不是立即重试。
    """

    def __init__(
        self,
        message: str,
        retry_after: float = 3.0,
        provider: Optional[str] = None,
    ) -> None:
        super().__init__(message, provider=provider, status_code=429)
        self.retry_after = retry_after


class ModelTimeoutError(ModelAPIError):
    """
    请求超时: 对应requests.exceptions.Timeout(包含ConnectTimeout与ReadTimeout)。
    """

    def __init__(
        self,
        message: str,
        timeout_seconds: float,
        provider: Optional[str] = None,
    ) -> None:
        super().__init__(message, provider=provider, status_code=504)
        self.timeout_seconds = timeout_seconds


class ModelConnectionError(ModelAPIError):
    """网络连接失败: 对应requests.exceptions.ConnectionError, 通常是本机网络问题。"""

    def __init__(self, message: str, provider: Optional[str] = None) -> None:
        super().__init__(message, provider=provider, status_code=None)


class ModelResponseParseError(ModelAPIError):
    """
    响应解析错误: 状态码是200, 但响应体结构不符合预期
    (比如缺少choices字段、choices为空列表、JSON解析失败等)。
    """

    def __init__(
        self,
        message: str,
        raw_response: Optional[str] = None,
        provider: Optional[str] = None,
    ) -> None:
        super().__init__(message, provider=provider, status_code=200)
        self.raw_response = raw_response
```

### 文件三:`requests_usage_demo.py`(`requests`库详解)

```python
"""
文件名: requests_usage_demo.py
说明:
    进一步梳理requests库的常用方法与Response对象属性, 补充上午课堂笔记里
    提到但未展开的一些细节, 包括: Session的用法、超时与重试的手动实现、
    响应体的多种读取方式、以及自定义请求头的常见场景。
"""

from __future__ import annotations

import time
from typing import Any, Optional

import requests

BASE_URL = "https://httpbin.org"


def demo_response_object_attributes() -> None:
    """系统梳理一次response对象上最常用的属性与方法。"""
    print("=" * 60)
    print("演示1: Response对象的常用属性一览")
    print("=" * 60)

    response = requests.get(f"{BASE_URL}/get", params={"lesson": "day12"}, timeout=15)

    print(f"response.status_code (整数状态码): {response.status_code}")
    print(f"response.ok (布尔值, 是否2xx-3xx): {response.ok}")
    print(f"response.url (最终请求的完整URL): {response.url}")
    print(f"response.headers['Content-Type']: {response.headers.get('Content-Type')}")
    print(f"response.encoding (推断出的文本编码): {response.encoding}")
    print(f"response.elapsed (本次请求耗时): {response.elapsed}")
    print(f"len(response.content) (原始字节长度): {len(response.content)}")
    print(f"len(response.text) (解码后的文本长度): {len(response.text)}")

    data = response.json()
    print(f"response.json() 解析出的类型: {type(data)}")
    print()


def demo_session_for_connection_reuse() -> None:
    """
    演示requests.Session的用法。

    当需要连续发起多次请求(比如多轮对话不断调用同一个API域名)时,
    使用Session对象可以复用底层的TCP连接(HTTP Keep-Alive), 避免每次
    请求都重新建立TCP连接、重新完成TLS握手, 能明显降低总耗时, 这也是
    ask_ai.py在连续追问场景下值得考虑的一个优化点(教学阶段先了解原理,
    今天的ask_ai.py简化实现仍使用普通requests.post, 留作作业里的思考)。
    """
    print("=" * 60)
    print("演示2: Session连接复用与普通方式的耗时对比")
    print("=" * 60)

    request_count = 5

    start = time.time()
    for _ in range(request_count):
        requests.get(f"{BASE_URL}/get", timeout=15)
    plain_elapsed = time.time() - start
    print(f"不使用Session, 连续{request_count}次请求总耗时: {plain_elapsed:.3f}秒")

    start = time.time()
    with requests.Session() as session:
        for _ in range(request_count):
            session.get(f"{BASE_URL}/get", timeout=15)
    session_elapsed = time.time() - start
    print(f"使用Session, 连续{request_count}次请求总耗时: {session_elapsed:.3f}秒")
    print("(实际差异会受网络环境影响, 网络状况不稳定时差异可能不明显,"
          " 但在稳定的高频调用场景下, Session通常能带来可观的性能提升)")
    print()


def demo_custom_headers_common_scenarios() -> None:
    """演示几种真实工作中常见的自定义请求头场景。"""
    print("=" * 60)
    print("演示3: 常见的自定义请求头场景")
    print("=" * 60)

    scenarios = [
        {
            "name": "声明客户端身份",
            "headers": {"User-Agent": "cangqiong-ask-ai/0.1 (training)"},
        },
        {
            "name": "声明期望的响应语言",
            "headers": {"Accept-Language": "zh-CN,zh;q=0.9"},
        },
        {
            "name": "声明这是一次内部追踪请求(自定义头, 类似链路追踪ID)",
            "headers": {"X-Request-Id": "day12-demo-0001"},
        },
    ]

    for scenario in scenarios:
        response = requests.get(
            f"{BASE_URL}/headers", headers=scenario["headers"], timeout=15
        )
        echoed = response.json().get("headers", {})
        print(f"场景: {scenario['name']}")
        for key in scenario["headers"]:
            print(f"  发送的{key} -> 服务器回显: {echoed.get(key)}")
    print()


def manual_retry_without_decorator(
    url: str,
    max_attempts: int = 3,
    wait_seconds: float = 1.0,
    **kwargs: Any,
) -> Optional[requests.Response]:
    """
    手写一个"笨办法"的重试逻辑, 用循环加try/except实现, 不使用装饰器。

    这个函数故意写得比较啰嗦, 目的是让大家先体会"手动重试"到底要写多少
    重复代码, 为明天(Day13)学习用装饰器封装这套逻辑做铺垫对比。

    :param url: 请求地址
    :param max_attempts: 最大尝试次数
    :param wait_seconds: 每次失败后等待的秒数
    :param kwargs: 传递给requests.get的其他参数
    :return: 成功时返回Response对象, 全部尝试失败后返回None
    """
    for attempt in range(1, max_attempts + 1):
        try:
            response = requests.get(url, timeout=15, **kwargs)
            response.raise_for_status()
            print(f"第{attempt}次尝试成功")
            return response
        except requests.exceptions.RequestException as exc:
            print(f"第{attempt}次尝试失败: {exc}")
            if attempt < max_attempts:
                print(f"等待{wait_seconds}秒后重试……")
                time.sleep(wait_seconds)
    print(f"已达到最大尝试次数{max_attempts}, 放弃请求")
    return None


def demo_manual_retry() -> None:
    """演示手写重试逻辑, 用一个故意会随机失败的接口来触发重试。"""
    print("=" * 60)
    print("演示4: 手写重试逻辑(为Day13装饰器重试做对比铺垫)")
    print("=" * 60)

    # /status/500 会稳定返回500, 用来模拟"每次都失败, 最终放弃"的情况
    manual_retry_without_decorator(f"{BASE_URL}/status/500", max_attempts=3, wait_seconds=1.0)
    print()


def demo_parse_retry_after_header() -> None:
    """
    演示如何读取并解析响应头中的Retry-After字段。

    很多支持限流的API(包括部分大模型服务商)在返回429状态码时,
    会在响应头里附带一个Retry-After字段, 明确告诉调用方应该等待多久
    再重试, 这个字段的值可能是纯数字(秒数), 也可能是一个具体的HTTP日期
    字符串, 这里只演示最常见的纯数字场景, 并对解析失败的情况做兜底处理。
    """
    print("=" * 60)
    print("演示5: 解析Retry-After响应头")
    print("=" * 60)

    # httpbin.org没有直接提供带Retry-After的429接口, 这里用response-headers
    # 接口人为构造一个带Retry-After头的响应, 用于演示解析逻辑本身
    response = requests.get(
        f"{BASE_URL}/response-headers",
        params={"Retry-After": "5", "X-Simulated-Status": "429"},
        timeout=15,
    )
    raw_value = response.headers.get("Retry-After")
    print(f"原始Retry-After响应头的值: {raw_value!r}")

    wait_seconds = parse_retry_after(raw_value, default=3.0)
    print(f"解析后建议等待的秒数: {wait_seconds}")
    print()


def parse_retry_after(raw_value: Optional[str], default: float = 3.0) -> float:
    """
    尝试把Retry-After响应头的原始字符串解析为等待秒数, 解析失败时返回默认值。

    :param raw_value: 响应头中Retry-After字段的原始字符串, 可能为None
    :param default: 无法解析时的兜底等待秒数
    :return: 建议等待的秒数
    """
    if not raw_value:
        return default
    try:
        return float(raw_value)
    except ValueError:
        # 有些服务商返回的是HTTP日期格式而不是纯数字秒数,
        # 教学阶段简化处理, 遇到无法直接转成数字的情况, 统一走默认值,
        # 生产环境中可以进一步用email.utils.parsedate_to_datetime解析日期格式
        return default


def demo_json_vs_text_response() -> None:
    """
    演示response.text与response.json()两种读取响应体方式的区别与适用场景,
    以及当响应体不是合法JSON时, response.json()会如何报错。
    """
    print("=" * 60)
    print("演示6: response.text 与 response.json() 的区别")
    print("=" * 60)

    json_response = requests.get(f"{BASE_URL}/get", timeout=15)
    print(f"response.text的类型: {type(json_response.text)}, 只是原始字符串")
    print(f"response.json()的类型: {type(json_response.json())}, 已经是Python字典")

    html_response = requests.get(f"{BASE_URL}/html", timeout=15)
    print("\n请求一个返回HTML内容(非JSON)的接口, 尝试调用response.json():")
    try:
        html_response.json()
        print("未抛出异常(不符合预期)")
    except requests.exceptions.JSONDecodeError as exc:
        print(f"符合预期, 捕获到JSONDecodeError: {type(exc).__name__}")
        print("说明: 遇到这种情况, 应该先用response.text查看原始内容,"
              " 确认对方到底返回了什么, 再决定下一步怎么处理,"
              " 而不是想当然地直接调用.json()")
    print()


def main() -> None:
    demo_response_object_attributes()
    demo_session_for_connection_reuse()
    demo_custom_headers_common_scenarios()
    demo_manual_retry()
    demo_parse_retry_after_header()
    demo_json_vs_text_response()
    print("requests库详解演示运行完毕。")


if __name__ == "__main__":
    main()
```

### 文件四:`call_deepseek_api.py`(DeepSeek API完整调用示例)

```python
"""
文件名: call_deepseek_api.py
说明:
    今天下午第一次真实调用大模型API的完整可运行示例, 使用DeepSeek开放平台的
    deepseek-chat模型。运行前必须先设置环境变量DEEPSEEK_API_KEY。

    获取API Key的方式(教学说明, 具体流程以DeepSeek官方平台实际界面为准):
        1. 访问DeepSeek开放平台官网, 完成注册与实名认证。
        2. 进入"API Keys"管理页面, 创建一个新的Key(创建后只会完整显示一次,
           务必当场复制保存好, 关闭页面后无法再次查看完整Key, 只能重新创建)。
        3. 新注册账号通常会有一定的免费额度, 足够教学阶段使用。
        4. 将Key设置为本机环境变量, 不要写入任何代码文件:
               macOS/Linux: 在 ~/.zshrc 或 ~/.bashrc 中加入一行
                   export DEEPSEEK_API_KEY="你的真实Key"
               然后执行 source ~/.zshrc (或重新打开终端)
               Windows: "系统属性 -> 环境变量" 中添加用户变量DEEPSEEK_API_KEY

    安全提醒: 任何时候都不要把真实Key打印到日志、截图或提交进Git仓库,
    本文件所有打印Key的地方都做了脱敏处理(只显示前若干位)。
"""

from __future__ import annotations

import json
import os
import time
from typing import Optional

import requests

DEEPSEEK_API_URL = "https://api.deepseek.com/v1/chat/completions"
DEEPSEEK_MODEL_NAME = "deepseek-chat"
DEFAULT_TIMEOUT_SECONDS = 30


def load_api_key() -> str:
    """
    从环境变量DEEPSEEK_API_KEY中读取API Key。

    :raises RuntimeError: 环境变量未设置时抛出, 提示应该如何设置
    :return: API Key字符串
    """
    api_key = os.environ.get("DEEPSEEK_API_KEY")
    if not api_key:
        raise RuntimeError(
            "未检测到环境变量DEEPSEEK_API_KEY, 请先设置后再运行本脚本。"
            "参考本文件顶部说明设置方式。"
        )
    return api_key


def mask_key(api_key: str) -> str:
    """
    对Key做脱敏处理, 只保留前8位与后4位, 中间用星号替代,
    用于日志打印和调试, 避免完整Key出现在任何输出中。
    """
    if len(api_key) <= 12:
        return "*" * len(api_key)
    return f"{api_key[:8]}{'*' * 8}{api_key[-4:]}"


def build_headers(api_key: str) -> dict:
    """构造DeepSeek API所需的请求头, Authorization采用Bearer Token方式。"""
    return {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}",
    }


def build_payload(
    messages: list,
    temperature: float = 0.7,
    max_tokens: int = 1024,
) -> dict:
    """
    构造DeepSeek chat completions接口所需的请求体。

    :param messages: 对话历史列表, 每个元素形如{"role": ..., "content": ...}
    :param temperature: 生成的随机性程度, 越高越有创造性, 越低越确定保守
    :param max_tokens: 本次回复最多生成的token数量
    :return: 请求体字典
    """
    return {
        "model": DEEPSEEK_MODEL_NAME,
        "messages": messages,
        "temperature": temperature,
        "max_tokens": max_tokens,
    }


def call_deepseek_chat(
    messages: list,
    temperature: float = 0.7,
    max_tokens: int = 1024,
    timeout: int = DEFAULT_TIMEOUT_SECONDS,
) -> dict:
    """
    真正发起一次DeepSeek chat completions请求, 返回完整的响应字典(未做二次封装)。

    :param messages: 对话历史列表
    :param temperature: 生成随机性参数
    :param max_tokens: 最大生成token数
    :param timeout: 请求超时时间(秒)
    :raises requests.exceptions.HTTPError: 状态码为4xx/5xx时抛出
    :raises requests.exceptions.Timeout: 请求超时时抛出
    :raises requests.exceptions.ConnectionError: 网络连接失败时抛出
    :return: DeepSeek服务器返回的完整JSON响应(已解析为字典)
    """
    api_key = load_api_key()
    headers = build_headers(api_key)
    payload = build_payload(messages, temperature=temperature, max_tokens=max_tokens)

    response = requests.post(
        DEEPSEEK_API_URL, headers=headers, json=payload, timeout=timeout
    )
    response.raise_for_status()
    return response.json()


def extract_reply_content(api_response: dict) -> str:
    """
    从DeepSeek返回的完整响应字典中, 提取出AI真正生成的回复文本。

    :param api_response: call_deepseek_chat返回的完整响应字典
    :raises KeyError: 缺少choices/message/content等预期字段时抛出
    :raises IndexError: choices列表为空时抛出
    :return: AI回复的纯文本内容
    """
    return api_response["choices"][0]["message"]["content"]


def print_usage_info(api_response: dict) -> None:
    """打印本次调用消耗的token信息, 便于直观感受调用成本。"""
    usage = api_response.get("usage", {})
    if not usage:
        print("(本次响应未包含usage字段)")
        return
    print(
        f"本次消耗token: 输入{usage.get('prompt_tokens')} + "
        f"输出{usage.get('completion_tokens')} = "
        f"总计{usage.get('total_tokens')}"
    )


def demo_single_turn() -> None:
    """演示最基础的单轮问答, 这是今天下午第一次真正调用大模型API的核心场景。"""
    print("=" * 60)
    print("演示1: DeepSeek单轮问答")
    print("=" * 60)

    api_key = load_api_key()
    print(f"当前使用的API Key(已脱敏): {mask_key(api_key)}")

    messages = [
        {"role": "system", "content": "你是蓬远智能内部的技术助手, 回答简洁清晰。"},
        {"role": "user", "content": "用一句话解释一下什么是HTTP协议。"},
    ]

    start_time = time.time()
    result = call_deepseek_chat(messages)
    elapsed = time.time() - start_time

    reply = extract_reply_content(result)
    print(f"AI回复: {reply}")
    print(f"本次请求耗时: {elapsed:.2f}秒")
    print_usage_info(result)
    print()


def demo_multi_turn() -> None:
    """演示多轮对话——把AI上一轮的回复重新放回messages, 让它'记住'上下文。"""
    print("=" * 60)
    print("演示2: DeepSeek多轮对话(验证上下文记忆)")
    print("=" * 60)

    conversation = [
        {"role": "system", "content": "你是蓬远智能内部的技术助手, 回答简洁清晰。"},
    ]

    questions = [
        "我叫陈铭, 是一名转行做AI应用开发的培训生。",
        "我刚才说我叫什么, 我的职业背景是什么?",
    ]

    for question in questions:
        conversation.append({"role": "user", "content": question})
        result = call_deepseek_chat(conversation)
        reply = extract_reply_content(result)
        conversation.append({"role": "assistant", "content": reply})

        print(f"你: {question}")
        print(f"AI: {reply}")
        print()


def demo_full_response_structure() -> None:
    """打印一次完整的原始响应结构, 帮助理解choices/usage等字段的含义。"""
    print("=" * 60)
    print("演示3: 完整响应结构展示")
    print("=" * 60)

    messages = [{"role": "user", "content": "简单介绍一下你自己。"}]
    result = call_deepseek_chat(messages, max_tokens=200)
    print(json.dumps(result, indent=2, ensure_ascii=False))
    print()


def main() -> None:
    try:
        demo_single_turn()
        demo_multi_turn()
        demo_full_response_structure()
    except RuntimeError as exc:
        print(f"[配置错误] {exc}")
    except requests.exceptions.HTTPError as exc:
        print(f"[HTTP错误] {exc}")
    except requests.exceptions.RequestException as exc:
        print(f"[网络异常] {exc}")


if __name__ == "__main__":
    main()
```

### 文件五:`call_qwen_api.py`(通义千问API完整调用示例)

```python
"""
文件名: call_qwen_api.py
说明:
    通义千问(qwen-plus)的完整调用示例, 通过阿里云DashScope提供的
    "OpenAI兼容模式"接口调用, 请求体结构与DeepSeek几乎一致, 印证课堂笔记里
    "同一套逻辑, 两个厂商"的结论。运行前必须先设置环境变量QWEN_API_KEY。

    获取API Key的方式(教学说明, 具体流程以阿里云DashScope实际界面为准):
        1. 注册并登录阿里云账号, 开通DashScope大模型服务。
        2. 进入DashScope控制台的"API-KEY管理"页面, 创建新的API Key。
        3. 新用户通常会获得一定的免费试用额度。
        4. 将Key设置为本机环境变量QWEN_API_KEY, 方式与DEEPSEEK_API_KEY相同。
"""

from __future__ import annotations

import json
import os
import time

import requests

QWEN_API_URL = "https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions"
QWEN_MODEL_NAME = "qwen-plus"
DEFAULT_TIMEOUT_SECONDS = 30


def load_api_key() -> str:
    """从环境变量QWEN_API_KEY中读取API Key, 未设置时抛出明确的RuntimeError。"""
    api_key = os.environ.get("QWEN_API_KEY")
    if not api_key:
        raise RuntimeError(
            "未检测到环境变量QWEN_API_KEY, 请先设置后再运行本脚本。"
        )
    return api_key


def mask_key(api_key: str) -> str:
    """Key脱敏处理, 逻辑与call_deepseek_api.py中的mask_key保持一致。"""
    if len(api_key) <= 12:
        return "*" * len(api_key)
    return f"{api_key[:8]}{'*' * 8}{api_key[-4:]}"


def build_headers(api_key: str) -> dict:
    """构造通义千问(DashScope兼容模式)所需的请求头, 与DeepSeek完全一致的Bearer格式。"""
    return {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}",
    }


def build_payload(
    messages: list,
    temperature: float = 0.7,
    max_tokens: int = 1024,
) -> dict:
    """
    构造通义千问请求体, 字段结构与DeepSeek一致, 唯一区别是model字段的取值。
    这正是课堂笔记里强调的"OpenAI兼容接口"带来的好处——迁移成本极低。
    """
    return {
        "model": QWEN_MODEL_NAME,
        "messages": messages,
        "temperature": temperature,
        "max_tokens": max_tokens,
    }


def call_qwen_chat(
    messages: list,
    temperature: float = 0.7,
    max_tokens: int = 1024,
    timeout: int = DEFAULT_TIMEOUT_SECONDS,
) -> dict:
    """
    发起一次通义千问chat completions请求, 返回完整响应字典。

    :param messages: 对话历史列表
    :param temperature: 生成随机性参数
    :param max_tokens: 最大生成token数
    :param timeout: 请求超时时间(秒)
    :return: 通义千问服务器返回的完整JSON响应
    """
    api_key = load_api_key()
    headers = build_headers(api_key)
    payload = build_payload(messages, temperature=temperature, max_tokens=max_tokens)

    response = requests.post(QWEN_API_URL, headers=headers, json=payload, timeout=timeout)
    response.raise_for_status()
    return response.json()


def extract_reply_content(api_response: dict) -> str:
    """提取通义千问回复内容, 字段路径与DeepSeek完全一致, choices[0].message.content。"""
    return api_response["choices"][0]["message"]["content"]


def demo_single_turn() -> None:
    """演示通义千问单轮问答, 与DeepSeek示例问同一个问题, 方便对比措辞差异。"""
    print("=" * 60)
    print("演示1: 通义千问(qwen-plus)单轮问答")
    print("=" * 60)

    api_key = load_api_key()
    print(f"当前使用的API Key(已脱敏): {mask_key(api_key)}")

    messages = [
        {"role": "system", "content": "你是蓬远智能内部的技术助手, 回答简洁清晰。"},
        {"role": "user", "content": "用一句话解释一下什么是HTTP协议。"},
    ]

    start_time = time.time()
    result = call_qwen_chat(messages)
    elapsed = time.time() - start_time

    reply = extract_reply_content(result)
    print(f"AI回复: {reply}")
    print(f"本次请求耗时: {elapsed:.2f}秒")
    print()


def demo_compare_with_deepseek_style() -> None:
    """
    打印通义千问的完整响应结构, 对照call_deepseek_api.py中打印的DeepSeek响应结构,
    直观验证两家厂商的响应体字段(id/object/choices/usage)高度一致。
    """
    print("=" * 60)
    print("演示2: 通义千问完整响应结构(用于对照DeepSeek)")
    print("=" * 60)

    messages = [{"role": "user", "content": "简单介绍一下你自己。"}]
    result = call_qwen_chat(messages, max_tokens=200)
    print(json.dumps(result, indent=2, ensure_ascii=False))
    print()


def demo_multi_turn() -> None:
    """演示通义千问多轮对话, 验证上下文记忆同样依赖messages列表的累积。"""
    print("=" * 60)
    print("演示3: 通义千问多轮对话")
    print("=" * 60)

    conversation = [
        {"role": "system", "content": "你是蓬远智能内部的技术助手, 回答简洁清晰。"},
    ]

    questions = [
        "我今天刚学会调用大模型API, 心情有点激动。",
        "我刚才说了什么心情?",
    ]

    for question in questions:
        conversation.append({"role": "user", "content": question})
        result = call_qwen_chat(conversation)
        reply = extract_reply_content(result)
        conversation.append({"role": "assistant", "content": reply})

        print(f"你: {question}")
        print(f"AI: {reply}")
        print()


def main() -> None:
    try:
        demo_single_turn()
        demo_compare_with_deepseek_style()
        demo_multi_turn()
    except RuntimeError as exc:
        print(f"[配置错误] {exc}")
    except requests.exceptions.HTTPError as exc:
        print(f"[HTTP错误] {exc}")
    except requests.exceptions.RequestException as exc:
        print(f"[网络异常] {exc}")


if __name__ == "__main__":
    main()
```

### 文件六:`error_handling_demo.py`(401/429/超时真实错误复现与统一异常转换)

```python
"""
文件名: error_handling_demo.py
说明:
    下午"真实错误复现"环节的配套脚本, 完整复现401鉴权失败、429限流、
    超时(ReadTimeout)、网络连接失败、响应结构异常这五类真实场景,
    并演示如何把requests库抛出的原始异常, 统一转换为api_exceptions.py
    中定义的、与Day10 cangqiong_core异常体系一脉相承的自定义异常。

    运行本文件中"故意制造401"的演示时, 会临时使用一个明显错误的Key,
    不会影响你在环境变量中设置的真实DEEPSEEK_API_KEY。
"""

from __future__ import annotations

import os
import time

import requests

from api_exceptions import (
    AuthenticationError,
    ConfigurationError,
    ModelConnectionError,
    ModelResponseParseError,
    ModelTimeoutError,
    RateLimitError,
)

DEEPSEEK_API_URL = "https://api.deepseek.com/v1/chat/completions"


def call_raw(api_key: str, messages: list, timeout: float = 30) -> requests.Response:
    """
    最"裸"的一次调用, 不做任何异常转换, 只负责发出请求并返回原始Response对象,
    留给上层函数决定如何解读结果, 便于本文件反复演示不同的错误处理策略。
    """
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}",
    }
    payload = {
        "model": "deepseek-chat",
        "messages": messages,
        "temperature": 0.7,
        "max_tokens": 200,
    }
    return requests.post(DEEPSEEK_API_URL, headers=headers, json=payload, timeout=timeout)


def call_deepseek_with_typed_errors(
    api_key: str,
    messages: list,
    timeout: float = 30,
) -> str:
    """
    调用DeepSeek API, 并把requests库可能抛出的各类原始异常,
    统一转换为api_exceptions.py中定义的、语义清晰的自定义异常。

    这是"错误处理"这件事在真实工程代码里应该有的样子——调用方
    (比如ask_ai.py)只需要捕获这几个自定义异常类型, 就能针对性地
    处理不同场景, 不需要关心requests库内部具体抛出的是哪个原始类型。

    :raises ConfigurationError: 传入的api_key为空
    :raises AuthenticationError: 服务器返回401
    :raises RateLimitError: 服务器返回429
    :raises ModelTimeoutError: 请求超时(连接超时或读取超时)
    :raises ModelConnectionError: 网络连接失败(DNS解析失败/无法建立连接等)
    :raises ModelResponseParseError: 状态码正常但响应结构不符合预期
    :return: AI回复的文本内容
    """
    if not api_key:
        raise ConfigurationError(
            "API Key为空, 无法发起请求", missing_key="DEEPSEEK_API_KEY"
        )

    try:
        response = call_raw(api_key, messages, timeout=timeout)
    except requests.exceptions.Timeout as exc:
        raise ModelTimeoutError(
            f"请求超时(超过{timeout}秒未收到响应)", timeout_seconds=timeout, provider="deepseek"
        ) from exc
    except requests.exceptions.ConnectionError as exc:
        raise ModelConnectionError(
            "网络连接失败, 请检查本机网络是否正常", provider="deepseek"
        ) from exc
    except requests.exceptions.RequestException as exc:
        raise ModelConnectionError(f"请求发生未分类的网络异常: {exc}", provider="deepseek") from exc

    if response.status_code == 401:
        raise AuthenticationError(
            "API Key鉴权失败, 请检查DEEPSEEK_API_KEY是否正确、是否包含多余空白字符",
            provider="deepseek",
        )
    if response.status_code == 429:
        retry_after = float(response.headers.get("Retry-After", 3))
        raise RateLimitError(
            "触发限流, 请求过于频繁", retry_after=retry_after, provider="deepseek"
        )
    if response.status_code >= 400:
        raise ModelResponseParseError(
            f"服务器返回了未特殊处理的错误状态码{response.status_code}",
            raw_response=response.text,
            provider="deepseek",
        )

    try:
        result = response.json()
    except ValueError as exc:
        raise ModelResponseParseError(
            "响应体不是合法的JSON格式", raw_response=response.text, provider="deepseek"
        ) from exc

    try:
        return result["choices"][0]["message"]["content"]
    except (KeyError, IndexError, TypeError) as exc:
        raise ModelResponseParseError(
            "响应JSON结构缺少预期的choices/message/content字段",
            raw_response=str(result),
            provider="deepseek",
        ) from exc


def demo_authentication_error() -> None:
    """故意使用一个明显错误的Key, 复现真实的401鉴权失败场景。"""
    print("=" * 60)
    print("演示1: 复现401鉴权失败(故意使用错误的Key)")
    print("=" * 60)

    fake_key = "sk-this-is-an-intentionally-invalid-key-000000"
    messages = [{"role": "user", "content": "你好"}]

    try:
        call_deepseek_with_typed_errors(fake_key, messages)
    except AuthenticationError as exc:
        print(f"捕获到AuthenticationError: {exc}")
        print(f"  provider={exc.provider}, status_code={exc.status_code}")
    print()


def demo_missing_key() -> None:
    """演示未设置API Key(传入空字符串)时的配置错误处理。"""
    print("=" * 60)
    print("演示2: 复现ConfigurationError(Key为空)")
    print("=" * 60)

    messages = [{"role": "user", "content": "你好"}]
    try:
        call_deepseek_with_typed_errors("", messages)
    except ConfigurationError as exc:
        print(f"捕获到ConfigurationError: {exc}")
        print(f"  missing_key={exc.missing_key}")
    print()


def demo_timeout_error() -> None:
    """故意设置极短的超时时间, 复现真实的ReadTimeout场景。"""
    print("=" * 60)
    print("演示3: 复现请求超时(设置timeout=0.01秒)")
    print("=" * 60)

    api_key = os.environ.get("DEEPSEEK_API_KEY", "")
    if not api_key:
        print("(跳过本演示: 未设置真实的DEEPSEEK_API_KEY环境变量)")
        return

    messages = [{"role": "user", "content": "请详细介绍HTTP协议的历史"}]
    try:
        call_deepseek_with_typed_errors(api_key, messages, timeout=0.01)
    except ModelTimeoutError as exc:
        print(f"捕获到ModelTimeoutError: {exc}")
        print(f"  timeout_seconds={exc.timeout_seconds}")
    print()


def demo_rate_limit_simulation() -> None:
    """
    通过短时间内连续发起多次请求, 尝试触发真实的429限流。
    注意: 是否能真实触发429, 取决于账号当前的限流配置, 不保证每次都能复现,
    这里额外加了一层保护, 即使触发不了429, 也不会导致脚本异常终止。
    """
    print("=" * 60)
    print("演示4: 尝试复现429限流(连续快速请求)")
    print("=" * 60)

    api_key = os.environ.get("DEEPSEEK_API_KEY", "")
    if not api_key:
        print("(跳过本演示: 未设置真实的DEEPSEEK_API_KEY环境变量)")
        return

    messages = [{"role": "user", "content": "1+1等于几"}]
    triggered = False
    for attempt in range(1, 11):
        try:
            call_deepseek_with_typed_errors(api_key, messages, timeout=10)
            print(f"第{attempt}次请求成功")
        except RateLimitError as exc:
            print(f"第{attempt}次请求触发限流: {exc}, 建议等待{exc.retry_after}秒后重试")
            triggered = True
            break
        except (ModelTimeoutError, ModelConnectionError) as exc:
            print(f"第{attempt}次请求出现网络问题(非限流): {exc}")
            break

    if not triggered:
        print("本次未能复现429(账号限流阈值较宽松, 属于正常情况)")
    print()


def demo_response_parse_error() -> None:
    """
    模拟"状态码正常但响应结构不符合预期"的场景——用一个人为构造的、
    缺少choices字段的假响应, 验证ModelResponseParseError能被正确抛出。
    这个场景在真实网络请求中较难主动复现, 因此用直接调用底层解析逻辑的方式演示。
    """
    print("=" * 60)
    print("演示5: 复现ModelResponseParseError(响应结构异常)")
    print("=" * 60)

    fake_malformed_result = {"id": "fake-id", "object": "chat.completion"}
    try:
        fake_malformed_result["choices"][0]["message"]["content"]
    except (KeyError, IndexError, TypeError) as exc:
        wrapped = ModelResponseParseError(
            "响应JSON结构缺少预期的choices/message/content字段",
            raw_response=str(fake_malformed_result),
            provider="deepseek",
        )
        print(f"捕获到ModelResponseParseError: {wrapped}")
        print(f"  raw_response(用于排查): {wrapped.raw_response}")
    print()


def main() -> None:
    demo_missing_key()
    demo_authentication_error()
    demo_timeout_error()
    demo_rate_limit_simulation()
    demo_response_parse_error()
    print("全部错误处理演示运行完毕。")


if __name__ == "__main__":
    main()
```

### 文件七:`ask_ai.py`(今日最终交付物 —— 命令行AI问答小程序)

```python
"""
文件名: ask_ai.py
说明:
    Day12晚自习的最终交付物——一个支持连续追问的命令行AI问答小程序。

    功能覆盖今天的需求文档F1-F8:
        F1 单轮问答
        F2 多轮连续追问(维护messages历史列表)
        F3 支持DeepSeek/通义千问双服务商切换(启动参数 --provider)
        F4 API Key只通过环境变量加载, 代码中不出现任何硬编码密钥
        F5 401/429/超时/网络异常的中文友好提示
        F6 /exit /quit /help /clear 交互指令
        F7 每次回复打印本次请求耗时
        F8 退出时可选择将对话历史保存为带时间戳的json文件

    用法:
        python ask_ai.py --provider deepseek
        python ask_ai.py --provider qwen

    运行前提: 已设置好对应的环境变量DEEPSEEK_API_KEY或QWEN_API_KEY。
"""

from __future__ import annotations

import argparse
import json
import os
import sys
import time
from datetime import datetime
from pathlib import Path
from typing import Optional

import requests

from api_exceptions import (
    AuthenticationError,
    CangqiongError,
    ConfigurationError,
    ModelConnectionError,
    ModelResponseParseError,
    ModelTimeoutError,
    RateLimitError,
)

PROVIDER_CONFIGS = {
    "deepseek": {
        "display_name": "DeepSeek",
        "url": "https://api.deepseek.com/v1/chat/completions",
        "env_key": "DEEPSEEK_API_KEY",
        "model": "deepseek-chat",
    },
    "qwen": {
        "display_name": "通义千问",
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions",
        "env_key": "QWEN_API_KEY",
        "model": "qwen-plus",
    },
}

DEFAULT_SYSTEM_PROMPT = (
    "你是蓬远智能内部培训助手, 面向刚转行学习AI应用开发的培训生, "
    "说话简洁清晰, 优先给出可操作、可理解的回答, 避免使用过多晦涩术语。"
)

DEFAULT_TIMEOUT_SECONDS = 30
HISTORY_SAVE_DIR = Path(__file__).resolve().parent / "conversation_history"

HELP_TEXT = """
可用指令:
  /help          查看本帮助信息
  /clear         清空当前对话历史, 重新开始一轮新对话
  /provider      查看当前使用的模型服务商
  /switch <名称> 切换模型服务商, 例如: /switch qwen 或 /switch deepseek
  /save          将当前对话历史保存为json文件
  /exit 或 /quit 退出程序(退出前会询问是否保存对话历史)

直接输入你的问题并回车, 即可向AI提问, 支持连续追问。
""".strip()


def load_api_key(provider: str) -> str:
    """
    根据provider从对应的环境变量中加载API Key。

    :param provider: "deepseek" 或 "qwen"
    :raises ConfigurationError: 对应环境变量未设置时抛出
    :return: API Key字符串
    """
    config = PROVIDER_CONFIGS[provider]
    api_key = os.environ.get(config["env_key"])
    if not api_key:
        raise ConfigurationError(
            f"未检测到环境变量{config['env_key']}, 无法使用{config['display_name']}, "
            f"请先设置该环境变量后再运行本程序。",
            missing_key=config["env_key"],
        )
    return api_key


def build_initial_conversation() -> list:
    """构造一份新对话的初始messages列表, 包含一条system消息设定AI的人设。"""
    return [{"role": "system", "content": DEFAULT_SYSTEM_PROMPT}]


def call_model_api(
    provider: str,
    conversation: list,
    timeout: int = DEFAULT_TIMEOUT_SECONDS,
) -> str:
    """
    统一的模型调用入口, 根据provider选择对应厂商配置发起请求,
    并把requests库可能抛出的各类异常统一转换为自定义异常类型,
    调用方(交互主循环)只需要捕获CangqiongError这一个基类即可兜底处理。

    :param provider: "deepseek" 或 "qwen"
    :param conversation: 完整的对话历史列表(包含本次新问题)
    :param timeout: 请求超时时间(秒)
    :raises ConfigurationError: API Key未设置
    :raises AuthenticationError: 服务器返回401
    :raises RateLimitError: 服务器返回429
    :raises ModelTimeoutError: 请求超时
    :raises ModelConnectionError: 网络连接失败
    :raises ModelResponseParseError: 响应结构不符合预期
    :return: AI回复的文本内容
    """
    config = PROVIDER_CONFIGS[provider]
    api_key = load_api_key(provider)

    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}",
    }
    payload = {
        "model": config["model"],
        "messages": conversation,
        "temperature": 0.7,
        "max_tokens": 1024,
    }

    try:
        response = requests.post(config["url"], headers=headers, json=payload, timeout=timeout)
    except requests.exceptions.Timeout as exc:
        raise ModelTimeoutError(
            f"请求超时(超过{timeout}秒未收到响应), 请检查网络状况后重试",
            timeout_seconds=timeout,
            provider=provider,
        ) from exc
    except requests.exceptions.ConnectionError as exc:
        raise ModelConnectionError(
            "网络连接失败, 请检查本机网络是否正常", provider=provider
        ) from exc
    except requests.exceptions.RequestException as exc:
        raise ModelConnectionError(f"发生未分类的网络异常: {exc}", provider=provider) from exc

    if response.status_code == 401:
        raise AuthenticationError(
            f"{config['display_name']}鉴权失败, 请检查{config['env_key']}是否正确设置,"
            f"是否包含多余的空格或换行符",
            provider=provider,
        )
    if response.status_code == 429:
        retry_after = float(response.headers.get("Retry-After", 3))
        raise RateLimitError(
            f"{config['display_name']}触发限流, 请求过于频繁",
            retry_after=retry_after,
            provider=provider,
        )
    if response.status_code >= 400:
        raise ModelResponseParseError(
            f"服务器返回错误状态码{response.status_code}",
            raw_response=response.text,
            provider=provider,
        )

    try:
        result = response.json()
        return result["choices"][0]["message"]["content"]
    except (ValueError, KeyError, IndexError, TypeError) as exc:
        raise ModelResponseParseError(
            "响应结构不符合预期, 无法解析出回复内容",
            raw_response=response.text,
            provider=provider,
        ) from exc


def format_error_hint(exc: CangqiongError) -> str:
    """把自定义异常转换成一段面向终端用户的、清晰易懂的中文提示。"""
    if isinstance(exc, ConfigurationError):
        return f"[配置错误] {exc.message} (缺失项: {exc.missing_key})"
    if isinstance(exc, AuthenticationError):
        return f"[鉴权失败, 401] {exc.message}"
    if isinstance(exc, RateLimitError):
        return f"[触发限流, 429] {exc.message}, 建议等待约{exc.retry_after:.0f}秒后重试"
    if isinstance(exc, ModelTimeoutError):
        return f"[请求超时] {exc.message}(设置的超时时间: {exc.timeout_seconds}秒)"
    if isinstance(exc, ModelConnectionError):
        return f"[网络连接失败] {exc.message}"
    if isinstance(exc, ModelResponseParseError):
        return f"[响应解析失败] {exc.message}"
    return f"[未分类错误] {exc}"


def save_conversation_history(conversation: list, provider: str) -> Path:
    """
    将当前对话历史保存为一份带时间戳的json文件, 存放于conversation_history/目录下。

    :param conversation: 完整的messages历史列表
    :param provider: 当前使用的模型服务商标识
    :return: 保存后的文件路径
    """
    HISTORY_SAVE_DIR.mkdir(parents=True, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    file_path = HISTORY_SAVE_DIR / f"conversation_{provider}_{timestamp}.json"

    payload = {
        "provider": provider,
        "saved_at": datetime.now().isoformat(),
        "messages": conversation,
    }
    file_path.write_text(
        json.dumps(payload, indent=2, ensure_ascii=False), encoding="utf-8"
    )
    return file_path


def print_welcome_banner(provider: str) -> None:
    """打印程序启动时的欢迎信息, 提示当前使用的模型服务商与基本操作方式。"""
    config = PROVIDER_CONFIGS[provider]
    print("=" * 60)
    print("苍穹训练营 · 命令行AI问答小程序 ask_ai.py")
    print(f"当前模型服务商: {config['display_name']}({config['model']})")
    print("输入 /help 查看可用指令, 直接输入问题并回车即可开始对话。")
    print("=" * 60)


def handle_switch_command(current_provider: str, target: str) -> str:
    """
    处理/switch指令, 校验目标服务商名称是否合法, 并检查对应的API Key是否已配置。

    :param current_provider: 当前使用的provider
    :param target: 用户输入的目标provider名称
    :return: 切换后的provider(如果切换失败, 返回原provider)
    """
    target = target.strip().lower()
    if target not in PROVIDER_CONFIGS:
        print(f"[提示] 不支持的服务商'{target}', 可选值: {list(PROVIDER_CONFIGS)}")
        return current_provider

    try:
        load_api_key(target)
    except ConfigurationError as exc:
        print(f"[提示] 无法切换到{target}: {exc.message}")
        return current_provider

    print(f"[提示] 已切换到 {PROVIDER_CONFIGS[target]['display_name']}")
    return target


def run_interactive_loop(initial_provider: str) -> None:
    """
    程序的主交互循环, 处理用户输入(普通问题或以/开头的指令),
    维护对话历史, 调用模型API, 打印回复与耗时, 直到用户退出。

    :param initial_provider: 启动时指定的初始模型服务商
    """
    provider = initial_provider
    conversation = build_initial_conversation()
    print_welcome_banner(provider)

    while True:
        try:
            user_input = input("\n你: ").strip()
        except (EOFError, KeyboardInterrupt):
            print("\n检测到中断信号, 正在退出……")
            break

        if not user_input:
            continue

        if user_input in ("/exit", "/quit"):
            handle_exit(conversation, provider)
            break

        if user_input == "/help":
            print(HELP_TEXT)
            continue

        if user_input == "/clear":
            conversation = build_initial_conversation()
            print("[提示] 对话历史已清空, 开始新一轮对话。")
            continue

        if user_input == "/provider":
            print(f"[提示] 当前服务商: {PROVIDER_CONFIGS[provider]['display_name']}")
            continue

        if user_input.startswith("/switch"):
            parts = user_input.split()
            if len(parts) != 2:
                print("[提示] 用法: /switch deepseek 或 /switch qwen")
                continue
            provider = handle_switch_command(provider, parts[1])
            continue

        if user_input == "/save":
            saved_path = save_conversation_history(conversation, provider)
            print(f"[提示] 对话历史已保存至: {saved_path}")
            continue

        conversation.append({"role": "user", "content": user_input})
        start_time = time.time()
        try:
            reply = call_model_api(provider, conversation)
        except CangqiongError as exc:
            print(format_error_hint(exc))
            # 请求失败时, 把刚追加的这条用户消息从历史中移除,
            # 避免下一轮请求把一次"失败的提问"也当作已发生的对话历史带上
            conversation.pop()
            continue

        elapsed = time.time() - start_time
        conversation.append({"role": "assistant", "content": reply})
        print(f"AI: {reply}")
        print(f"(本次耗时 {elapsed:.2f} 秒, 当前对话共{len(conversation)}条消息)")


def handle_exit(conversation: list, provider: str) -> None:
    """
    处理退出前的收尾逻辑: 如果对话历史中有真实的问答内容(不只是初始system消息),
    询问用户是否需要保存对话历史。
    """
    has_real_conversation = len(conversation) > 1
    if not has_real_conversation:
        print("再见!")
        return

    try:
        choice = input("是否将本次对话历史保存为json文件? (y/n): ").strip().lower()
    except (EOFError, KeyboardInterrupt):
        choice = "n"

    if choice == "y":
        saved_path = save_conversation_history(conversation, provider)
        print(f"对话历史已保存至: {saved_path}")
    print("再见!")


def parse_arguments() -> argparse.Namespace:
    """解析命令行启动参数, 目前只支持--provider指定初始模型服务商。"""
    parser = argparse.ArgumentParser(description="苍穹训练营命令行AI问答小程序")
    parser.add_argument(
        "--provider",
        choices=list(PROVIDER_CONFIGS),
        default="deepseek",
        help="指定初始使用的模型服务商, 默认为deepseek",
    )
    return parser.parse_args()


def main() -> None:
    args = parse_arguments()

    try:
        load_api_key(args.provider)
    except ConfigurationError as exc:
        print(f"[启动失败] {exc}")
        print("请先设置好对应的环境变量后再重新运行本程序。")
        sys.exit(1)

    run_interactive_loop(args.provider)


if __name__ == "__main__":
    main()
```

### 文件八:`README.md`(项目说明与运行手册)

````markdown
# day12_api_lab —— 网络请求与API调用实战项目

本项目是蓬远科技新人训练营Day12的完整代码产出, 涵盖HTTP协议基础练习、
requests库详解、DeepSeek/通义千问API首次调用、错误处理体系,
以及最终的命令行AI问答小程序ask_ai.py。

## 目录结构

```
day12_api_lab/
├── http_basics_demo.py       # HTTP协议基础(GET/POST/状态码/headers)
├── requests_usage_demo.py    # requests库详解(Session/重试/常见headers场景)
├── api_exceptions.py         # 模型API异常体系(承接Day10 cangqiong_core)
├── call_deepseek_api.py      # DeepSeek API完整调用示例
├── call_qwen_api.py          # 通义千问API完整调用示例
├── error_handling_demo.py    # 401/429/超时真实错误复现
├── ask_ai.py                 # 今日最终交付物: 命令行AI问答小程序
├── conversation_history/     # 运行时自动生成, 存放保存的对话历史json
└── README.md                 # 本说明文件
```

## 环境要求

- Python 3.11及以上
- 第三方依赖: `requests`

安装依赖:

```bash
pip install requests
```

## 运行前必须完成的配置

本项目所有涉及真实API调用的脚本, 都需要通过环境变量加载API Key,
代码中永远不会出现任何真实密钥字符串。请先完成以下配置:

1. 前往DeepSeek开放平台申请API Key, 设置环境变量`DEEPSEEK_API_KEY`。
2. 前往阿里云DashScope控制台申请通义千问API Key, 设置环境变量`QWEN_API_KEY`。

macOS/Linux示例(写入`~/.zshrc`或`~/.bashrc`后执行`source`使其生效):

```bash
export DEEPSEEK_API_KEY="你的真实DeepSeek Key"
export QWEN_API_KEY="你的真实通义千问 Key"
```

Windows示例(在"系统属性 -> 环境变量"中添加用户变量, 或在PowerShell中临时设置):

```powershell
$env:DEEPSEEK_API_KEY="你的真实DeepSeek Key"
$env:QWEN_API_KEY="你的真实通义千问 Key"
```

## 各脚本运行方式与用途

| 脚本 | 是否需要API Key | 用途说明 |
|---|---|---|
| `http_basics_demo.py` | 不需要 | 演示HTTP协议基础概念, 使用公开的httpbin.org测试站点 |
| `requests_usage_demo.py` | 不需要 | 演示requests库的进阶用法(Session/手写重试等) |
| `api_exceptions.py` | 不需要 | 异常类定义, 不直接运行, 被其他脚本import |
| `call_deepseek_api.py` | 需要DEEPSEEK_API_KEY | DeepSeek单轮/多轮对话完整调用示例 |
| `call_qwen_api.py` | 需要QWEN_API_KEY | 通义千问单轮/多轮对话完整调用示例 |
| `error_handling_demo.py` | 需要DEEPSEEK_API_KEY(部分演示可跳过) | 复现401/429/超时等真实错误 |
| `ask_ai.py` | 需要对应服务商的Key | 最终交付物, 命令行AI问答小程序 |

运行示例:

```bash
python http_basics_demo.py
python requests_usage_demo.py
python call_deepseek_api.py
python call_qwen_api.py
python error_handling_demo.py
python ask_ai.py --provider deepseek
python ask_ai.py --provider qwen
```

### 文件九:`test_api_exceptions.py`(异常体系单元测试)

> 补充说明:上面的README.md把今天产出的七个核心文件走了一遍运行手册,但陈铭在周末整理代码的时候意识到,api_exceptions.py这套异常体系,虽然被ask_ai.py和error_handling_demo.py广泛使用,却从未被专门测试过——`CangqiongError.__str__`在有detail和没有detail两种情况下是否都拼接正确、`RateLimitError`的默认`retry_after`是否符合预期、五个具体子类是否都能被`ModelAPIError`和`CangqiongError`正确捕获,这些问题过去都只是靠"跑一下demo脚本, 眼睛看一下输出对不对"这种不严谨的方式验证的。以下补充的这几份代码,把"异常体系本身对不对""网络层错误处理逻辑在各种边界情况下对不对""重试策略的决策对不对""对话历史分析工具对不对"这四类问题,分别用正式的单元测试与配套的工具模块补齐,进一步把Day12这条"HTTP协议->requests库->大模型API->错误处理"的链路夯实。

```python
"""
文件名: test_api_exceptions.py
说明:
    针对api_exceptions.py中定义的整套异常体系补充的单元测试。

    背景:
        陈铭在今天下午写完api_exceptions.py之后,只是在ask_ai.py和
        error_handling_demo.py里"顺带"用到了这些异常类,从未针对异常类
        本身写过一行专门的测试——比如CangqiongError的__str__方法在有
        detail和没有detail两种情况下是否都能正确拼接文案、RateLimitError
        的retry_after默认值是否符合预期、ModelAPIError子类之间的继承关系
        是否严格符合设计(比如AuthenticationError必须能被ModelAPIError
        捕获到, 也必须能被最顶层的CangqiongError捕获到)。这些"异常类本身
        对不对"的问题,过去都是靠"跑一下demo脚本, 眼睛看一下报错文案对
        不对"这种不严谨的方式验证的, 今天补上一套正式的单元测试。

    运行方式:
        python -m unittest test_api_exceptions.py -v
"""

from __future__ import annotations

import unittest

from api_exceptions import (
    AuthenticationError,
    CangqiongError,
    ConfigurationError,
    ModelAPIError,
    ModelConnectionError,
    ModelResponseParseError,
    ModelTimeoutError,
    RateLimitError,
)


class TestCangqiongErrorBaseClass(unittest.TestCase):
    """测试异常体系最顶层基类CangqiongError的基础行为。"""

    def test_str_without_detail_returns_message_only(self):
        """没有传入detail参数时, __str__应该只返回message本身。"""
        exc = CangqiongError("发生了一个错误")
        self.assertEqual(str(exc), "发生了一个错误")

    def test_str_with_detail_appends_detail_in_parentheses(self):
        """传入detail参数时, __str__应该把detail以"(详情: xxx)"的格式追加在后面。"""
        exc = CangqiongError("发生了一个错误", detail="这是补充说明")
        self.assertEqual(str(exc), "发生了一个错误(详情: 这是补充说明)")

    def test_message_and_detail_are_accessible_as_attributes(self):
        """message和detail都应该能作为实例属性被外部代码读取。"""
        exc = CangqiongError("消息文本", detail="详情文本")
        self.assertEqual(exc.message, "消息文本")
        self.assertEqual(exc.detail, "详情文本")

    def test_is_a_real_exception_subclass(self):
        """CangqiongError必须是Python内置Exception的子类, 才能被try/except正常捕获。"""
        self.assertTrue(issubclass(CangqiongError, Exception))
        with self.assertRaises(CangqiongError):
            raise CangqiongError("测试抛出")


class TestConfigurationError(unittest.TestCase):
    """测试ConfigurationError, 今天最常见的触发场景是环境变量未设置。"""

    def test_missing_key_attribute_recorded_correctly(self):
        exc = ConfigurationError("未检测到环境变量", missing_key="DEEPSEEK_API_KEY")
        self.assertEqual(exc.missing_key, "DEEPSEEK_API_KEY")

    def test_missing_key_defaults_to_none_when_not_provided(self):
        """如果调用方没有传missing_key, 不应该报错, 而是应该有一个安全的默认值None。"""
        exc = ConfigurationError("配置有问题")
        self.assertIsNone(exc.missing_key)

    def test_is_subclass_of_cangqiong_error(self):
        self.assertTrue(issubclass(ConfigurationError, CangqiongError))


class TestModelAPIErrorAndSubclasses(unittest.TestCase):
    """
    测试ModelAPIError这个中间层基类, 以及它派生出的
    AuthenticationError/RateLimitError/ModelTimeoutError/
    ModelConnectionError/ModelResponseParseError五个具体子类。
    """

    def test_model_api_error_records_provider_and_status_code(self):
        exc = ModelAPIError("调用失败", provider="deepseek", status_code=500)
        self.assertEqual(exc.provider, "deepseek")
        self.assertEqual(exc.status_code, 500)

    def test_model_api_error_provider_and_status_code_default_to_none(self):
        exc = ModelAPIError("调用失败")
        self.assertIsNone(exc.provider)
        self.assertIsNone(exc.status_code)

    def test_authentication_error_status_code_is_always_401(self):
        """
        AuthenticationError代表401场景, 它的status_code不需要外部传入,
        应该在内部被硬编码为401, 因为401这个状态码本身就是"401鉴权失败"
        这个异常类型的定义特征, 不应该由调用方随意指定成别的值。
        """
        exc = AuthenticationError("鉴权失败", provider="qwen")
        self.assertEqual(exc.status_code, 401)
        self.assertEqual(exc.provider, "qwen")

    def test_authentication_error_is_a_model_api_error_and_cangqiong_error(self):
        """
        验证继承链条完整: AuthenticationError -> ModelAPIError -> CangqiongError。
        这个测试的意义在于, 调用方代码如果写了except ModelAPIError,
        应该能捕获到AuthenticationError这个更具体的子类实例, 反之如果写了
        except AuthenticationError, 则不应该捕获到其他不相关的ModelAPIError子类。
        """
        exc = AuthenticationError("鉴权失败")
        self.assertIsInstance(exc, ModelAPIError)
        self.assertIsInstance(exc, CangqiongError)

    def test_rate_limit_error_default_retry_after_is_three_seconds(self):
        """RateLimitError如果调用方没有指定retry_after, 应该有一个合理的默认等待时间。"""
        exc = RateLimitError("触发限流")
        self.assertEqual(exc.retry_after, 3.0)
        self.assertEqual(exc.status_code, 429)

    def test_rate_limit_error_custom_retry_after_is_respected(self):
        exc = RateLimitError("触发限流", retry_after=12.5, provider="deepseek")
        self.assertEqual(exc.retry_after, 12.5)

    def test_model_timeout_error_records_timeout_seconds(self):
        exc = ModelTimeoutError("请求超时", timeout_seconds=0.01, provider="deepseek")
        self.assertEqual(exc.timeout_seconds, 0.01)
        # 超时场景约定用504(网关超时)作为一种内部统一的语义标记,
        # 即使requests库本身在纯客户端超时时并不会真正返回一个具体的HTTP状态码
        self.assertEqual(exc.status_code, 504)

    def test_model_connection_error_status_code_is_none(self):
        """
        ModelConnectionError对应的是"连TCP连接都没建立起来"的场景,
        这种情况下根本没有从服务器收到任何带状态码的响应, 因此status_code
        理应保持为None, 而不是被强行赋予一个虚构的数值。
        """
        exc = ModelConnectionError("网络连接失败")
        self.assertIsNone(exc.status_code)

    def test_model_response_parse_error_records_raw_response(self):
        """
        ModelResponseParseError专门用于"状态码是200,但响应体结构不对"的场景,
        因此它应该固定status_code为200, 并且能记录原始响应文本便于排查。
        """
        raw = '{"id": "abc", "object": "chat.completion"}'
        exc = ModelResponseParseError("响应结构异常", raw_response=raw, provider="qwen")
        self.assertEqual(exc.status_code, 200)
        self.assertEqual(exc.raw_response, raw)

    def test_all_five_subclasses_are_catchable_as_model_api_error(self):
        """
        用一个循环, 系统性地验证所有具体子类, 都能被更上层的ModelAPIError
        和CangqiongError正确捕获, 避免未来有人新增子类时忘记继承正确的基类。
        """
        subclasses_and_kwargs = [
            (AuthenticationError, {"message": "x"}),
            (RateLimitError, {"message": "x"}),
            (ModelTimeoutError, {"message": "x", "timeout_seconds": 1.0}),
            (ModelConnectionError, {"message": "x"}),
            (ModelResponseParseError, {"message": "x"}),
        ]
        for cls, kwargs in subclasses_and_kwargs:
            with self.subTest(cls=cls.__name__):
                instance = cls(**kwargs)
                self.assertIsInstance(instance, ModelAPIError)
                self.assertIsInstance(instance, CangqiongError)


class TestExceptionChaining(unittest.TestCase):
    """
    测试异常链(exception chaining, 即raise ... from exc)在整个体系里是否
    被正确保留, 这对排查真实问题时"追根溯源"非常重要——如果没有正确保留
    __cause__, 排查者只能看到最外层的自定义异常文案, 却看不到最初到底是
    requests库内部具体抛出的哪一种原始异常。
    """

    def test_raise_from_preserves_original_cause(self):
        original = ValueError("这是原始的底层错误")
        try:
            try:
                raise original
            except ValueError as exc:
                raise ModelResponseParseError("包装后的错误") from exc
        except ModelResponseParseError as wrapped:
            self.assertIs(wrapped.__cause__, original)

    def test_error_without_explicit_chaining_has_no_cause(self):
        exc = ConfigurationError("独立抛出, 没有链式来源")
        self.assertIsNone(exc.__cause__)


class TestMaskKeyLogicIndependentReimplementation(unittest.TestCase):
    """
    call_deepseek_api.py和call_qwen_api.py里各自实现了一份逻辑完全相同的
    mask_key函数(脱敏处理API Key), 为了不在测试文件里重复import两份几乎
    一样的实现,这里重新按同样的规则实现一份纯函数版本, 用它验证
    "脱敏规则本身"在各种边界长度下是否表现正确——这套用例可以同时
    用来校验两份原始实现是否遵循了同样的行为约定。
    """

    @staticmethod
    def mask_key(api_key: str) -> str:
        if len(api_key) <= 12:
            return "*" * len(api_key)
        return f"{api_key[:8]}{'*' * 8}{api_key[-4:]}"

    def test_short_key_is_fully_masked(self):
        self.assertEqual(self.mask_key("short-key"), "*" * len("short-key"))

    def test_key_of_exactly_twelve_chars_is_fully_masked(self):
        key = "a" * 12
        self.assertEqual(self.mask_key(key), "*" * 12)

    def test_key_of_thirteen_chars_shows_prefix_and_suffix(self):
        key = "a" * 13
        masked = self.mask_key(key)
        self.assertTrue(masked.startswith("a" * 8))
        self.assertTrue(masked.endswith("a" * 4))
        self.assertIn("*" * 8, masked)

    def test_long_realistic_key_masks_middle_section(self):
        key = "sk-1234567890abcdefghijklmnopqrstuvwxyz"
        masked = self.mask_key(key)
        self.assertEqual(masked[:8], key[:8])
        self.assertEqual(masked[-4:], key[-4:])
        self.assertNotIn(key[8:-4], masked)

    def test_empty_key_returns_empty_string(self):
        self.assertEqual(self.mask_key(""), "")


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

### 文件十:`test_error_handling_with_mock.py`(基于Mock的网络错误场景全覆盖测试)

> 说明:error_handling_demo.py里的五个演示函数全部依赖真实网络请求,这类测试只能人工运行、人工观察,没办法放进持续集成(CI)环境反复自动运行。这里用Python标准库`unittest.mock`,把`requests.post`替换成一个完全在内存中运行的假函数,在不联网、不消耗任何真实调用额度的情况下,完整验证`call_deepseek_with_typed_errors`在面对401/429/超时/连接失败/响应结构异常等场景时,是否都能正确转换成对应的自定义异常类型。这是往后长期需要对接第三方API的AI应用开发工作中,一项非常重要的测试能力。

```python
"""
文件名: test_error_handling_with_mock.py
说明:
    error_handling_demo.py里的五个演示函数, 全部依赖真实的网络请求
    (需要真实的DEEPSEEK_API_KEY, 需要真实触发401/429/超时), 这类测试
    只能靠人工运行、人工观察输出, 没办法放进持续集成(CI)环境里自动化
    反复运行——CI环境通常没有网络权限, 也不应该在每次代码提交时都真的
    去消耗一次真实的API调用额度。

    今天补充的这套测试, 用Python标准库unittest.mock, 把requests.post
    这个真正发起网络请求的函数"替换"成一个完全在内存里运行的假函数,
    让它按照测试代码指定的方式"伪造"出各种响应(200/401/429/超时异常/
    连接异常), 从而在完全不联网、不消耗任何真实调用额度的情况下,
    完整地验证call_deepseek_with_typed_errors这个函数在面对
    "服务器真实返回401""服务器真实返回429""网络层面直接超时"
    "网络层面直接连接失败""响应体格式异常"这五大类场景时,
    是否都能正确地转换成对应的自定义异常类型。

    这是一种非常重要的工程能力——"如何在不依赖真实外部服务的情况下,
    测试那些原本依赖外部服务的代码逻辑", Mock技术是解决这个问题最
    常用的手段之一, 在往后需要频繁对接第三方API的AI应用开发工作中,
    这套思路会反复被用到。

    运行方式:
        python -m unittest test_error_handling_with_mock.py -v
"""

from __future__ import annotations

import json
import unittest
from unittest.mock import MagicMock, patch

import requests

from api_exceptions import (
    AuthenticationError,
    ConfigurationError,
    ModelConnectionError,
    ModelResponseParseError,
    ModelTimeoutError,
    RateLimitError,
)
from error_handling_demo import call_deepseek_with_typed_errors


def make_fake_response(
    status_code: int,
    json_body: dict | None = None,
    headers: dict | None = None,
    text: str = "",
) -> MagicMock:
    """
    构造一个"看起来像"requests.Response对象的假对象, 用于Mock场景。

    真实的requests.Response是一个C扩展支撑的复杂对象, 完整模拟它的
    所有行为成本很高, 也没有必要——我们只需要模拟被测代码里实际会
    用到的那几个属性/方法: status_code、headers、.json()、.text。

    :param status_code: 要伪造的HTTP状态码
    :param json_body: .json()方法应该返回的字典, 为None时.json()会抛出异常
    :param headers: 伪造的响应头字典, 默认为空字典
    :param text: .text属性应该返回的原始文本
    :return: 一个配置好的MagicMock对象, 可以当作Response使用
    """
    fake = MagicMock(spec=requests.Response)
    fake.status_code = status_code
    fake.headers = headers or {}
    fake.text = text or (json.dumps(json_body, ensure_ascii=False) if json_body else "")
    if json_body is not None:
        fake.json.return_value = json_body
    else:
        fake.json.side_effect = ValueError("响应体不是合法JSON")
    return fake


class TestSuccessfulCallReturnsContent(unittest.TestCase):
    """验证正常成功场景(状态码200, 结构完整)下, 能正确提取出回复文本。"""

    @patch("error_handling_demo.requests.post")
    def test_normal_200_response_returns_reply_content(self, mock_post):
        mock_post.return_value = make_fake_response(
            200,
            json_body={
                "id": "chatcmpl-fake-001",
                "choices": [{"message": {"role": "assistant", "content": "这是伪造的AI回复"}}],
                "usage": {"prompt_tokens": 10, "completion_tokens": 5, "total_tokens": 15},
            },
        )
        result = call_deepseek_with_typed_errors("sk-fake-key", [{"role": "user", "content": "你好"}])
        self.assertEqual(result, "这是伪造的AI回复")
        mock_post.assert_called_once()

    @patch("error_handling_demo.requests.post")
    def test_post_is_called_with_correct_bearer_header(self, mock_post):
        """
        验证无论响应内容如何, 实际发出的请求头里, Authorization字段
        的格式必须严格是"Bearer <key>", 这是DeepSeek等厂商鉴权协议的硬性要求,
        一旦这个格式被不小心改动(比如少了空格), 会导致所有请求都收到401。
        """
        mock_post.return_value = make_fake_response(
            200, json_body={"choices": [{"message": {"content": "ok"}}]}
        )
        call_deepseek_with_typed_errors("sk-my-real-looking-key", [{"role": "user", "content": "hi"}])

        _, kwargs = mock_post.call_args
        self.assertEqual(kwargs["headers"]["Authorization"], "Bearer sk-my-real-looking-key")
        self.assertEqual(kwargs["headers"]["Content-Type"], "application/json")


class TestConfigurationErrorOnEmptyKey(unittest.TestCase):
    """验证空Key场景在发起任何网络请求之前就应该被拦截。"""

    @patch("error_handling_demo.requests.post")
    def test_empty_key_raises_configuration_error_without_calling_network(self, mock_post):
        with self.assertRaises(ConfigurationError):
            call_deepseek_with_typed_errors("", [{"role": "user", "content": "你好"}])
        # 关键校验点: 既然Key为空这件事在本地就能判断出来, 根本不应该
        # 浪费一次真实的网络往返, mock_post理应完全没有被调用过
        mock_post.assert_not_called()


class TestAuthenticationErrorFromMockedResponse(unittest.TestCase):
    """验证服务器真实返回401时, 能被正确转换为AuthenticationError。"""

    @patch("error_handling_demo.requests.post")
    def test_status_401_raises_authentication_error(self, mock_post):
        mock_post.return_value = make_fake_response(
            401,
            json_body={"error": {"message": "Authentication Fails", "type": "authentication_error"}},
        )
        with self.assertRaises(AuthenticationError) as ctx:
            call_deepseek_with_typed_errors("sk-wrong-key", [{"role": "user", "content": "你好"}])
        self.assertEqual(ctx.exception.status_code, 401)
        self.assertEqual(ctx.exception.provider, "deepseek")


class TestRateLimitErrorFromMockedResponse(unittest.TestCase):
    """验证429限流场景, 包括Retry-After响应头存在与不存在两种情况。"""

    @patch("error_handling_demo.requests.post")
    def test_status_429_with_retry_after_header_is_parsed(self, mock_post):
        mock_post.return_value = make_fake_response(
            429,
            json_body={"error": {"message": "Rate limit reached"}},
            headers={"Retry-After": "7"},
        )
        with self.assertRaises(RateLimitError) as ctx:
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])
        self.assertEqual(ctx.exception.retry_after, 7.0)

    @patch("error_handling_demo.requests.post")
    def test_status_429_without_retry_after_header_uses_default(self, mock_post):
        """
        如果服务器返回429但没有附带Retry-After响应头(有些厂商确实不提供这个头),
        代码里的兜底逻辑应该使用一个合理的默认等待秒数, 而不应该直接报错崩溃。
        """
        mock_post.return_value = make_fake_response(
            429, json_body={"error": {"message": "Rate limit reached"}}, headers={}
        )
        with self.assertRaises(RateLimitError) as ctx:
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])
        self.assertEqual(ctx.exception.retry_after, 3.0)


class TestTimeoutAndConnectionErrorFromRawExceptions(unittest.TestCase):
    """
    验证requests.post本身直接抛出异常(而不是返回一个带状态码的Response)的场景,
    这类场景对应"请求根本没有从对方那里收到任何回应"的情况。
    """

    @patch("error_handling_demo.requests.post")
    def test_requests_timeout_is_converted_to_model_timeout_error(self, mock_post):
        mock_post.side_effect = requests.exceptions.ReadTimeout("Read timed out")
        with self.assertRaises(ModelTimeoutError) as ctx:
            call_deepseek_with_typed_errors(
                "sk-any-key", [{"role": "user", "content": "你好"}], timeout=0.01
            )
        self.assertEqual(ctx.exception.timeout_seconds, 0.01)

    @patch("error_handling_demo.requests.post")
    def test_connect_timeout_is_also_treated_as_timeout_error(self, mock_post):
        """ConnectTimeout是Timeout的子类, 同样应该被归类为ModelTimeoutError。"""
        mock_post.side_effect = requests.exceptions.ConnectTimeout("Connection timed out")
        with self.assertRaises(ModelTimeoutError):
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])

    @patch("error_handling_demo.requests.post")
    def test_connection_error_is_converted_to_model_connection_error(self, mock_post):
        mock_post.side_effect = requests.exceptions.ConnectionError("Failed to establish a new connection")
        with self.assertRaises(ModelConnectionError):
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])

    @patch("error_handling_demo.requests.post")
    def test_generic_request_exception_falls_back_to_connection_error(self, mock_post):
        """
        对于requests.exceptions.RequestException这个大家族里, 除了Timeout和
        ConnectionError之外的其他未特殊分类的异常(比如TooManyRedirects),
        代码里有一个兜底分支, 统一转换成ModelConnectionError, 这里验证
        这个兜底分支确实生效, 不会让一个未预料到的异常直接冒泡到调用方。
        """
        mock_post.side_effect = requests.exceptions.TooManyRedirects("Exceeded 30 redirects")
        with self.assertRaises(ModelConnectionError):
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])


class TestResponseParseErrorFromMalformedBody(unittest.TestCase):
    """验证状态码正常但响应体结构异常时, 能被正确转换为ModelResponseParseError。"""

    @patch("error_handling_demo.requests.post")
    def test_non_json_body_raises_parse_error(self, mock_post):
        mock_post.return_value = make_fake_response(200, json_body=None, text="<html>not json</html>")
        with self.assertRaises(ModelResponseParseError):
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])

    @patch("error_handling_demo.requests.post")
    def test_missing_choices_field_raises_parse_error(self, mock_post):
        mock_post.return_value = make_fake_response(200, json_body={"id": "abc", "object": "chat.completion"})
        with self.assertRaises(ModelResponseParseError):
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])

    @patch("error_handling_demo.requests.post")
    def test_empty_choices_list_raises_parse_error(self, mock_post):
        """choices字段存在, 但是是一个空列表, 取choices[0]会触发IndexError, 应该被转换。"""
        mock_post.return_value = make_fake_response(200, json_body={"choices": []})
        with self.assertRaises(ModelResponseParseError):
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])

    @patch("error_handling_demo.requests.post")
    def test_choices_present_but_message_missing_content_raises_parse_error(self, mock_post):
        mock_post.return_value = make_fake_response(
            200, json_body={"choices": [{"message": {"role": "assistant"}}]}
        )
        with self.assertRaises(ModelResponseParseError):
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])


class TestUnclassifiedHttpErrorStatusCodes(unittest.TestCase):
    """
    验证除401/429之外的其他4xx/5xx状态码(比如400/500/503),
    会走"通用错误"分支, 被转换为ModelResponseParseError并保留原始响应文本,
    而不是被错误地误判为401或429。
    """

    @patch("error_handling_demo.requests.post")
    def test_status_400_is_not_misclassified_as_auth_or_rate_limit(self, mock_post):
        mock_post.return_value = make_fake_response(
            400, json_body={"error": {"message": "Bad Request"}}, text='{"error": "bad request"}'
        )
        with self.assertRaises(ModelResponseParseError) as ctx:
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])
        self.assertIn("400", str(ctx.exception))

    @patch("error_handling_demo.requests.post")
    def test_status_500_preserves_raw_response_text(self, mock_post):
        mock_post.return_value = make_fake_response(
            500, json_body={"error": "internal"}, text='{"error": "internal server error"}'
        )
        with self.assertRaises(ModelResponseParseError) as ctx:
            call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])
        self.assertEqual(ctx.exception.raw_response, '{"error": "internal server error"}')


class TestMultipleSequentialCallsWithDifferentOutcomes(unittest.TestCase):
    """
    验证mock_post.side_effect支持传入一个"结果列表", 依次模拟连续多次调用
    分别得到不同的结果——这模拟了real_handling_demo.py里
    demo_rate_limit_simulation()那种"连续请求, 前几次成功, 某一次突然
    触发限流"的真实场景, 用完全不联网的方式复现同样的测试意图。
    """

    @patch("error_handling_demo.requests.post")
    def test_third_call_triggers_rate_limit_after_two_successes(self, mock_post):
        success_response = make_fake_response(200, json_body={"choices": [{"message": {"content": "ok"}}]})
        rate_limited_response = make_fake_response(
            429, json_body={"error": {"message": "限流"}}, headers={"Retry-After": "4"}
        )
        mock_post.side_effect = [success_response, success_response, rate_limited_response]

        results = []
        errors = []
        for _ in range(3):
            try:
                results.append(
                    call_deepseek_with_typed_errors("sk-any-key", [{"role": "user", "content": "你好"}])
                )
            except RateLimitError as exc:
                errors.append(exc)

        self.assertEqual(len(results), 2)
        self.assertEqual(len(errors), 1)
        self.assertEqual(errors[0].retry_after, 4.0)
        self.assertEqual(mock_post.call_count, 3)


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

### 文件十一:`retry_strategy_lab.py`(按错误类型区分的手写重试策略模块)

> 说明:这份代码承接requests_usage_demo.py里`manual_retry_without_decorator`这个"笨办法"手写重试函数,以及课后作业第6题里给出的分析框架——"401不重试、429等待后重试、超时立即或短暂等待后重试"——把这套分析框架落地成一套真正可以运行、可以针对不同异常类型采用不同处理策略的重试工具模块。依然没有使用装饰器(那是Day13的内容),全部用显式的循环加函数调用实现,目的是让"重试策略"这个概念本身先被理解透彻,明天学装饰器时才能真正体会到装饰器帮忙省掉了多少重复的样板代码。

```python
"""
文件名: retry_strategy_lab.py
说明:
    这份代码承接requests_usage_demo.py里manual_retry_without_decorator
    这个"笨办法"手写重试函数, 以及课后作业第6题里给出的思考框架——
    "401不重试、429等待后重试、超时立即或短暂等待后重试"——把这套
    分析框架, 落地成一套真正可以运行、可以按错误类型区分处理策略的
    重试工具模块。

    需要特别说明: 这里依然没有使用装饰器(那是Day13的内容), 全部通过
    显式的循环加函数调用实现, 目的是让"重试策略"这个概念本身先被理解
    透彻, 明天学装饰器的时候, 才能真正体会到装饰器帮我们省掉了多少
    重复的样板代码(boilerplate), 而不是本末倒置地先学一个"看起来很酷"
    的语法糖, 却不理解它到底解决了什么问题。

    本模块提供的核心能力:
        1. RetryOutcome: 记录一次完整重试过程的结果(成功与否、
           尝试次数、每次尝试的耗时与结果)。
        2. RetryStrategy: 一个可配置的重试策略描述, 区分"是否应该重试"
           "重试前等待多久"两个维度, 且能针对不同异常类型给出不同策略。
        3. execute_with_retry_strategy: 真正执行"调用函数 -> 判断是否
           需要重试 -> 等待 -> 再次调用"这套循环逻辑的核心函数。
        4. 一组预置的策略工厂函数, 分别对应401/429/超时/连接失败四类场景,
           直接对应课后作业第6题的分析结论。
"""

from __future__ import annotations

import time
from dataclasses import dataclass, field
from typing import Callable, List, Optional, Type

from api_exceptions import (
    AuthenticationError,
    ModelConnectionError,
    ModelTimeoutError,
    RateLimitError,
)


@dataclass
class AttemptRecord:
    """记录一次单独尝试的结果, 用于事后复盘一次完整重试流程发生了什么。"""

    attempt_number: int
    succeeded: bool
    elapsed_seconds: float
    error_type: Optional[str] = None
    error_message: Optional[str] = None
    waited_before_seconds: float = 0.0


@dataclass
class RetryOutcome:
    """一次完整"可能包含多轮重试"的调用过程的最终结果汇总。"""

    succeeded: bool
    total_attempts: int
    total_elapsed_seconds: float
    attempts: List[AttemptRecord] = field(default_factory=list)
    final_result: object = None
    final_error: Optional[Exception] = None

    def summary_text(self) -> str:
        """生成一段用于打印到终端或写入日志的中文摘要文本。"""
        status = "成功" if self.succeeded else "最终失败"
        lines = [
            f"重试流程{status}, 共尝试{self.total_attempts}次, "
            f"总耗时约{self.total_elapsed_seconds:.2f}秒",
        ]
        for record in self.attempts:
            outcome_text = "成功" if record.succeeded else f"失败({record.error_type}: {record.error_message})"
            wait_note = f", 等待{record.waited_before_seconds:.1f}秒后发起本次尝试" if record.waited_before_seconds else ""
            lines.append(
                f"  第{record.attempt_number}次尝试{wait_note}: {outcome_text}, "
                f"耗时{record.elapsed_seconds:.2f}秒"
            )
        return "\n".join(lines)


@dataclass
class RetryStrategy:
    """
    描述"面对某一类具体异常, 应该怎么重试"的策略。

    :param should_retry: 遇到这类异常时是否应该重试
    :param max_attempts: 最大尝试次数(包含第一次), 例如3表示最多尝试3次
    :param base_wait_seconds: 基础等待秒数
    :param use_exponential_backoff: 是否采用指数退避(每次等待时间翻倍)
    :param respect_retry_after: 是否优先使用异常对象自带的retry_after建议值
        (目前只有RateLimitError携带这个信息)
    """

    should_retry: bool
    max_attempts: int = 1
    base_wait_seconds: float = 0.0
    use_exponential_backoff: bool = False
    respect_retry_after: bool = False

    def compute_wait_seconds(self, attempt_index: int, exc: Optional[Exception]) -> float:
        """
        计算"第attempt_index次失败之后, 下一次重试前应该等待多少秒"。

        :param attempt_index: 从1开始计数的、刚刚失败的这次尝试的序号
        :param exc: 刚刚捕获到的异常实例, 用于读取retry_after等动态信息
        :return: 建议等待的秒数
        """
        if self.respect_retry_after and isinstance(exc, RateLimitError):
            return exc.retry_after
        if not self.use_exponential_backoff:
            return self.base_wait_seconds
        # 指数退避: 第1次失败后等base, 第2次等base*2, 第3次等base*4, 以此类推
        return self.base_wait_seconds * (2 ** (attempt_index - 1))


def strategy_for_authentication_error() -> RetryStrategy:
    """
    401鉴权失败对应的策略: 不重试。

    理由(对应课后作业第6题的分析): 401的根本原因是凭证本身有问题,
    不改变凭证的情况下, 重试多少次结果都会是同一个401, 是纯粹的无意义
    重复劳动, 甚至可能因为反复无效请求触发额外的账户风控。
    """
    return RetryStrategy(should_retry=False, max_attempts=1)


def strategy_for_rate_limit_error() -> RetryStrategy:
    """
    429限流对应的策略: 应该重试, 但必须等待, 且优先尊重服务器建议的
    Retry-After时间, 没有这个信息时才退化为固定等待时间。
    """
    return RetryStrategy(
        should_retry=True,
        max_attempts=4,
        base_wait_seconds=3.0,
        respect_retry_after=True,
    )


def strategy_for_timeout_error() -> RetryStrategy:
    """
    超时对应的策略: 应该重试, 且大多数情况下可以短暂等待后立即重试,
    因为超时往往源于网络的临时抖动, 采用指数退避避免"连续超时的网络问题
    还没缓解就立刻又发起下一次请求"这种无效的快速重试。
    """
    return RetryStrategy(
        should_retry=True,
        max_attempts=3,
        base_wait_seconds=1.0,
        use_exponential_backoff=True,
    )


def strategy_for_connection_error() -> RetryStrategy:
    """
    网络连接失败对应的策略: 与超时类似, 应该重试, 但由于"完全连不上"
    往往比"连上了但响应慢"更严重, 采用相对更长的基础等待时间。
    """
    return RetryStrategy(
        should_retry=True,
        max_attempts=3,
        base_wait_seconds=2.0,
        use_exponential_backoff=True,
    )


DEFAULT_STRATEGY_MAP: dict = {
    AuthenticationError: strategy_for_authentication_error(),
    RateLimitError: strategy_for_rate_limit_error(),
    ModelTimeoutError: strategy_for_timeout_error(),
    ModelConnectionError: strategy_for_connection_error(),
}


def resolve_strategy_for_exception(
    exc: Exception, strategy_map: Optional[dict] = None
) -> RetryStrategy:
    """
    根据异常的具体类型, 从策略映射表中查找对应的重试策略。

    查找规则: 优先精确匹配异常的确切类型; 如果没有精确匹配,
    再按照isinstance逐一尝试映射表中的每个键(兼容异常子类的场景);
    如果依然找不到任何匹配, 返回一个"不重试"的保守兜底策略,
    避免遇到未知异常类型时程序陷入不受控制的无限重试。

    :param exc: 捕获到的异常实例
    :param strategy_map: 异常类型到RetryStrategy的映射, 默认使用
        DEFAULT_STRATEGY_MAP
    :return: 匹配到的重试策略
    """
    strategy_map = strategy_map if strategy_map is not None else DEFAULT_STRATEGY_MAP

    exact_match = strategy_map.get(type(exc))
    if exact_match is not None:
        return exact_match

    for exc_type, strategy in strategy_map.items():
        if isinstance(exc, exc_type):
            return strategy

    return RetryStrategy(should_retry=False, max_attempts=1)


def execute_with_retry_strategy(
    func: Callable[[], object],
    strategy_map: Optional[dict] = None,
    sleep_func: Callable[[float], None] = time.sleep,
    time_func: Callable[[], float] = time.time,
) -> RetryOutcome:
    """
    执行func(), 如果抛出异常, 根据异常类型查表决定是否重试、重试前等待多久,
    直到成功、或者达到该异常类型允许的最大尝试次数、或者遇到一个"不应该
    重试"的异常类型为止。

    :param func: 一个不接受参数的可调用对象, 内部包含实际的业务逻辑
        (比如一次真实的API调用), 调用者需要自行用闭包或functools.partial
        把具体参数绑定好
    :param strategy_map: 自定义的策略映射表, 默认使用DEFAULT_STRATEGY_MAP
    :param sleep_func: 用于等待的函数, 测试时可以替换为不真正休眠的假函数,
        避免单元测试因为真实sleep而变慢
    :param time_func: 用于计时的函数, 同样便于测试时替换为可控的假实现
    :return: 一份完整记录了整个重试过程的RetryOutcome
    """
    attempts: List[AttemptRecord] = []
    attempt_number = 0
    waited_before = 0.0
    process_start = time_func()

    while True:
        attempt_number += 1
        attempt_start = time_func()
        try:
            result = func()
        except Exception as exc:  # noqa: BLE001 - 这里故意捕获所有异常, 交给策略判断如何处理
            elapsed = time_func() - attempt_start
            attempts.append(
                AttemptRecord(
                    attempt_number=attempt_number,
                    succeeded=False,
                    elapsed_seconds=elapsed,
                    error_type=type(exc).__name__,
                    error_message=str(exc),
                    waited_before_seconds=waited_before,
                )
            )
            strategy = resolve_strategy_for_exception(exc, strategy_map)

            should_continue = strategy.should_retry and attempt_number < strategy.max_attempts
            if not should_continue:
                total_elapsed = time_func() - process_start
                return RetryOutcome(
                    succeeded=False,
                    total_attempts=attempt_number,
                    total_elapsed_seconds=total_elapsed,
                    attempts=attempts,
                    final_error=exc,
                )

            waited_before = strategy.compute_wait_seconds(attempt_number, exc)
            if waited_before > 0:
                sleep_func(waited_before)
            continue
        else:
            elapsed = time_func() - attempt_start
            attempts.append(
                AttemptRecord(
                    attempt_number=attempt_number,
                    succeeded=True,
                    elapsed_seconds=elapsed,
                    waited_before_seconds=waited_before,
                )
            )
            total_elapsed = time_func() - process_start
            return RetryOutcome(
                succeeded=True,
                total_attempts=attempt_number,
                total_elapsed_seconds=total_elapsed,
                attempts=attempts,
                final_result=result,
            )


def demo_authentication_error_never_retries() -> None:
    """演示401场景: 无论max_attempts配置得多大, should_retry=False意味着只会尝试一次。"""
    print("=" * 60)
    print("演示1: 401鉴权失败, 策略应为完全不重试")
    print("=" * 60)

    call_count = 0

    def always_fail_with_401():
        nonlocal call_count
        call_count += 1
        raise AuthenticationError("模拟的401鉴权失败", provider="deepseek")

    outcome = execute_with_retry_strategy(always_fail_with_401, sleep_func=lambda seconds: None)
    print(outcome.summary_text())
    print(f"实际调用次数: {call_count}(预期为1, 因为401不应该重试)")
    print()


def demo_rate_limit_error_retries_with_retry_after() -> None:
    """演示429场景: 前两次失败(带Retry-After), 第三次成功, 验证重试后能拿到最终结果。"""
    print("=" * 60)
    print("演示2: 429限流, 前两次失败, 第三次成功")
    print("=" * 60)

    call_count = 0
    waited_amounts: List[float] = []

    def sleep_recorder(seconds: float) -> None:
        waited_amounts.append(seconds)

    def fail_twice_then_succeed():
        nonlocal call_count
        call_count += 1
        if call_count <= 2:
            raise RateLimitError("模拟的429限流", retry_after=2.0, provider="deepseek")
        return "第三次终于成功拿到的AI回复"

    outcome = execute_with_retry_strategy(fail_twice_then_succeed, sleep_func=sleep_recorder)
    print(outcome.summary_text())
    print(f"最终结果: {outcome.final_result}")
    print(f"记录到的等待秒数序列: {waited_amounts}(预期两次都是2.0, 因为尊重了Retry-After建议值)")
    print()


def demo_timeout_error_uses_exponential_backoff() -> None:
    """演示超时场景下指数退避的等待秒数序列: 1秒, 2秒, ……"""
    print("=" * 60)
    print("演示3: 超时场景, 指数退避等待秒数验证")
    print("=" * 60)

    call_count = 0
    waited_amounts: List[float] = []

    def sleep_recorder(seconds: float) -> None:
        waited_amounts.append(seconds)

    def always_timeout():
        nonlocal call_count
        call_count += 1
        raise ModelTimeoutError("模拟的持续超时", timeout_seconds=0.01, provider="deepseek")

    outcome = execute_with_retry_strategy(always_timeout, sleep_func=sleep_recorder)
    print(outcome.summary_text())
    print(f"记录到的等待秒数序列: {waited_amounts}(预期为[1.0, 2.0], 体现指数退避)")
    print()


def demo_unknown_exception_type_falls_back_to_no_retry() -> None:
    """演示遇到策略表里完全没有登记的未知异常类型时, 兜底策略是不重试, 避免失控。"""
    print("=" * 60)
    print("演示4: 未登记的未知异常类型, 兜底为不重试")
    print("=" * 60)

    call_count = 0

    def raise_unexpected_value_error():
        nonlocal call_count
        call_count += 1
        raise ValueError("这是一个策略表里完全没有登记过的异常类型")

    outcome = execute_with_retry_strategy(raise_unexpected_value_error, sleep_func=lambda seconds: None)
    print(outcome.summary_text())
    print(f"实际调用次数: {call_count}(预期为1, 未知异常走保守的不重试兜底策略)")
    print()


def main() -> None:
    demo_authentication_error_never_retries()
    demo_rate_limit_error_retries_with_retry_after()
    demo_timeout_error_uses_exponential_backoff()
    demo_unknown_exception_type_falls_back_to_no_retry()
    print("全部重试策略演示运行完毕。")


if __name__ == "__main__":
    main()
```

### 文件十二:`test_retry_strategy_lab.py`(重试策略模块单元测试)

> 说明:针对retry_strategy_lab.py这套手写重试策略模块的单元测试,重点验证"重试逻辑的决策是否正确",完全不需要真实发起网络请求,只需要构造一个"会按照测试代码指定的次数失败、之后再成功(或永远失败)"的假函数传给`execute_with_retry_strategy`,观察其行为是否符合预期。所有测试都传入一个只负责记录"被要求等待了多少秒"而不会真正休眠的假sleep函数,确保整套测试套件运行飞快。

```python
"""
文件名: test_retry_strategy_lab.py
说明:
    针对retry_strategy_lab.py这套手写重试策略模块的单元测试。

    测试的重点不是"网络请求本身", 而是"重试逻辑的决策是否正确"——
    这类测试完全不需要真实发起网络请求, 只需要构造一个"会按照测试代码
    指定的次数失败, 之后再成功(或者永远失败)"的假函数, 传给
    execute_with_retry_strategy, 观察它的行为是否符合预期。

    为了让测试运行得飞快(不会因为真的sleep几秒而拖慢整个测试套件),
    所有测试都会传入一个"假的sleep函数", 它只负责记录"被要求等待了
    多少秒", 而不会真的让测试暂停。

    运行方式:
        python -m unittest test_retry_strategy_lab.py -v
"""

from __future__ import annotations

import unittest
from typing import List

from api_exceptions import (
    AuthenticationError,
    ModelConnectionError,
    ModelTimeoutError,
    RateLimitError,
)
from retry_strategy_lab import (
    RetryStrategy,
    execute_with_retry_strategy,
    resolve_strategy_for_exception,
    strategy_for_authentication_error,
    strategy_for_connection_error,
    strategy_for_rate_limit_error,
    strategy_for_timeout_error,
)


class FakeClock:
    """
    一个完全可控的假时钟, 每次调用都按固定步长递增, 用于让"耗时统计"
    这类依赖time.time()的逻辑在测试环境下也能得到确定性的、可预测的数值,
    避免测试断言依赖真实的系统时间, 那样的测试会因为机器性能波动而不稳定。
    """

    def __init__(self, step: float = 0.1) -> None:
        self._current = 0.0
        self._step = step

    def __call__(self) -> float:
        self._current += self._step
        return self._current


class TestRetryStrategyBasicFields(unittest.TestCase):
    """测试RetryStrategy数据类本身的字段与compute_wait_seconds计算逻辑。"""

    def test_fixed_wait_without_backoff_stays_constant(self):
        strategy = RetryStrategy(should_retry=True, max_attempts=5, base_wait_seconds=2.0)
        self.assertEqual(strategy.compute_wait_seconds(1, None), 2.0)
        self.assertEqual(strategy.compute_wait_seconds(3, None), 2.0)

    def test_exponential_backoff_doubles_each_time(self):
        strategy = RetryStrategy(
            should_retry=True, max_attempts=5, base_wait_seconds=1.0, use_exponential_backoff=True
        )
        self.assertEqual(strategy.compute_wait_seconds(1, None), 1.0)
        self.assertEqual(strategy.compute_wait_seconds(2, None), 2.0)
        self.assertEqual(strategy.compute_wait_seconds(3, None), 4.0)
        self.assertEqual(strategy.compute_wait_seconds(4, None), 8.0)

    def test_respect_retry_after_overrides_base_wait_for_rate_limit_error(self):
        strategy = RetryStrategy(
            should_retry=True, max_attempts=3, base_wait_seconds=1.0, respect_retry_after=True
        )
        exc = RateLimitError("限流", retry_after=9.5)
        self.assertEqual(strategy.compute_wait_seconds(1, exc), 9.5)

    def test_respect_retry_after_ignored_for_non_rate_limit_exception(self):
        """
        respect_retry_after=True这个配置项, 只应该对RateLimitError生效,
        如果传入的异常不是RateLimitError(比如意外传了个ModelTimeoutError),
        应该老老实实退回到base_wait_seconds, 而不是尝试读取一个可能不存在的属性。
        """
        strategy = RetryStrategy(
            should_retry=True, max_attempts=3, base_wait_seconds=1.5, respect_retry_after=True
        )
        exc = ModelTimeoutError("超时", timeout_seconds=1.0)
        self.assertEqual(strategy.compute_wait_seconds(1, exc), 1.5)


class TestPresetStrategyFactories(unittest.TestCase):
    """
    验证四个预置策略工厂函数生成的策略, 严格符合课后作业第6题里
    分析出来的结论, 这套测试实质上是把"文字分析"转换成了"可执行的断言",
    确保未来如果有人不小心改动了策略参数, 能立刻被测试发现。
    """

    def test_authentication_error_strategy_never_retries(self):
        strategy = strategy_for_authentication_error()
        self.assertFalse(strategy.should_retry)
        self.assertEqual(strategy.max_attempts, 1)

    def test_rate_limit_error_strategy_retries_and_respects_retry_after(self):
        strategy = strategy_for_rate_limit_error()
        self.assertTrue(strategy.should_retry)
        self.assertGreater(strategy.max_attempts, 1)
        self.assertTrue(strategy.respect_retry_after)

    def test_timeout_error_strategy_uses_exponential_backoff(self):
        strategy = strategy_for_timeout_error()
        self.assertTrue(strategy.should_retry)
        self.assertTrue(strategy.use_exponential_backoff)

    def test_connection_error_strategy_has_longer_base_wait_than_timeout(self):
        """
        连接失败通常意味着更严重的网络问题(而不是单纯的响应慢),
        因此其基础等待时间理应不短于超时场景的基础等待时间,
        这是一个体现"策略设计合理性"的对比性断言。
        """
        connection_strategy = strategy_for_connection_error()
        timeout_strategy = strategy_for_timeout_error()
        self.assertGreaterEqual(connection_strategy.base_wait_seconds, timeout_strategy.base_wait_seconds)


class TestResolveStrategyForException(unittest.TestCase):
    """测试根据异常实例反查对应重试策略的逻辑, 包括子类兼容与未知类型兜底。"""

    def test_exact_type_match_returns_correct_strategy(self):
        exc = AuthenticationError("鉴权失败")
        strategy = resolve_strategy_for_exception(exc)
        self.assertFalse(strategy.should_retry)

    def test_rate_limit_exact_match(self):
        exc = RateLimitError("限流")
        strategy = resolve_strategy_for_exception(exc)
        self.assertTrue(strategy.should_retry)
        self.assertTrue(strategy.respect_retry_after)

    def test_unknown_exception_type_returns_conservative_no_retry_strategy(self):
        strategy = resolve_strategy_for_exception(ValueError("未登记类型"))
        self.assertFalse(strategy.should_retry)
        self.assertEqual(strategy.max_attempts, 1)

    def test_custom_strategy_map_is_respected_over_default(self):
        """
        验证调用方可以传入一份自定义的策略映射表, 覆盖掉默认的DEFAULT_STRATEGY_MAP,
        这为不同业务场景(比如某些内部管理后台可以接受对429做更激进的重试)
        提供了灵活的定制空间。
        """
        custom_map = {RateLimitError: RetryStrategy(should_retry=True, max_attempts=10, base_wait_seconds=0.5)}
        strategy = resolve_strategy_for_exception(RateLimitError("限流"), strategy_map=custom_map)
        self.assertEqual(strategy.max_attempts, 10)


class TestExecuteWithRetryStrategyEndToEnd(unittest.TestCase):
    """
    针对execute_with_retry_strategy这个核心执行函数的端到端测试,
    覆盖"第一次就成功""重试几次后成功""最终失败"三大类场景。
    """

    def _no_op_sleep(self, seconds: float) -> None:
        """测试专用的空sleep函数, 不真正阻塞, 只是满足接口签名要求。"""
        return None

    def test_immediate_success_records_single_attempt(self):
        outcome = execute_with_retry_strategy(
            lambda: "立即成功", sleep_func=self._no_op_sleep, time_func=FakeClock()
        )
        self.assertTrue(outcome.succeeded)
        self.assertEqual(outcome.total_attempts, 1)
        self.assertEqual(outcome.final_result, "立即成功")

    def test_authentication_error_stops_after_first_attempt(self):
        call_count = 0

        def always_401():
            nonlocal call_count
            call_count += 1
            raise AuthenticationError("鉴权失败")

        outcome = execute_with_retry_strategy(always_401, sleep_func=self._no_op_sleep, time_func=FakeClock())
        self.assertFalse(outcome.succeeded)
        self.assertEqual(call_count, 1)
        self.assertIsInstance(outcome.final_error, AuthenticationError)

    def test_rate_limit_error_retries_until_success_within_limit(self):
        call_count = 0
        recorded_waits: List[float] = []

        def sleep_recorder(seconds: float) -> None:
            recorded_waits.append(seconds)

        def fail_twice_then_succeed():
            nonlocal call_count
            call_count += 1
            if call_count < 3:
                raise RateLimitError("限流", retry_after=1.5)
            return "成功结果"

        outcome = execute_with_retry_strategy(
            fail_twice_then_succeed, sleep_func=sleep_recorder, time_func=FakeClock()
        )
        self.assertTrue(outcome.succeeded)
        self.assertEqual(outcome.total_attempts, 3)
        self.assertEqual(recorded_waits, [1.5, 1.5])

    def test_rate_limit_error_gives_up_after_max_attempts(self):
        """
        429场景的max_attempts是有限的(默认4次), 如果连续失败次数超过这个上限,
        应该老老实实放弃, 而不是无限重试下去, 这里验证max_attempts真的被遵守了。
        """
        call_count = 0

        def always_rate_limited():
            nonlocal call_count
            call_count += 1
            raise RateLimitError("持续限流", retry_after=0.1)

        outcome = execute_with_retry_strategy(
            always_rate_limited, sleep_func=self._no_op_sleep, time_func=FakeClock()
        )
        self.assertFalse(outcome.succeeded)
        self.assertEqual(call_count, strategy_for_rate_limit_error().max_attempts)

    def test_timeout_error_exponential_backoff_wait_sequence(self):
        recorded_waits: List[float] = []

        def sleep_recorder(seconds: float) -> None:
            recorded_waits.append(seconds)

        def always_timeout():
            raise ModelTimeoutError("持续超时", timeout_seconds=0.01)

        outcome = execute_with_retry_strategy(
            always_timeout, sleep_func=sleep_recorder, time_func=FakeClock()
        )
        self.assertFalse(outcome.succeeded)
        expected_waits = [1.0, 2.0]  # 对应max_attempts=3, 失败2次之后触发2次等待
        self.assertEqual(recorded_waits, expected_waits)

    def test_connection_error_eventually_succeeds_after_transient_failures(self):
        call_count = 0

        def fail_once_then_succeed():
            nonlocal call_count
            call_count += 1
            if call_count == 1:
                raise ModelConnectionError("临时网络抖动")
            return "网络恢复后成功"

        outcome = execute_with_retry_strategy(
            fail_once_then_succeed, sleep_func=self._no_op_sleep, time_func=FakeClock()
        )
        self.assertTrue(outcome.succeeded)
        self.assertEqual(outcome.final_result, "网络恢复后成功")
        self.assertEqual(outcome.total_attempts, 2)

    def test_outcome_summary_text_contains_key_information(self):
        """验证summary_text()生成的文本至少包含尝试次数与最终状态这些关键信息, 便于日志排查。"""
        outcome = execute_with_retry_strategy(
            lambda: "结果", sleep_func=self._no_op_sleep, time_func=FakeClock()
        )
        text = outcome.summary_text()
        self.assertIn("成功", text)
        self.assertIn("1", text)

    def test_attempt_records_are_ordered_and_numbered_sequentially(self):
        """验证每次尝试的AttemptRecord.attempt_number严格按1,2,3...递增, 没有跳号或乱序。"""
        call_count = 0

        def fail_twice_then_succeed():
            nonlocal call_count
            call_count += 1
            if call_count < 3:
                raise ModelTimeoutError("超时", timeout_seconds=0.01)
            return "ok"

        outcome = execute_with_retry_strategy(
            fail_twice_then_succeed, sleep_func=self._no_op_sleep, time_func=FakeClock()
        )
        numbers = [record.attempt_number for record in outcome.attempts]
        self.assertEqual(numbers, [1, 2, 3])


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

### 文件十三:`conversation_history_analyzer.py`(对话历史统计分析工具)

> 说明:ask_ai.py的F8功能会把每一次会话保存成一份带时间戳的json文件,但ask_ai.py本身没有提供任何"回头去看这些历史文件、做统计分析"的能力。陈铭在整理自己这几天攒下来的对话记录时,意识到这是一个值得单独做的小工具——统计"自己最常问哪一类问题""哪个模型服务商回复平均更长""哪次对话轮次最多"这些信息,对复盘自己的学习过程、以及后续苍穹项目里"用户行为分析"模块的设计,都是一次有意义的练习。

```python
"""
文件名: conversation_history_analyzer.py
说明:
    ask_ai.py的F8功能(对话历史落盘)会把每一次会话保存成一份
    conversation_history/conversation_<provider>_<时间戳>.json文件,
    但ask_ai.py本身只负责"保存", 没有提供任何"回头去看这些历史文件,
    做一些统计分析"的能力。陈铭在周末整理自己这几天攒下来的对话记录时,
    意识到这其实是一个值得单独做的小工具——统计"自己最常问哪一类问题"
    "哪个模型服务商回复平均更长""哪次对话轮次最多"这些信息, 对复盘
    自己的学习过程、以及后续苍穹项目里"用户行为分析"模块的设计,
    都是一次有意义的练习。

    本模块提供的能力:
        1. load_conversation_file: 加载单份对话历史json文件, 做基本的
           结构校验, 校验失败时给出清晰的错误说明而不是让程序直接崩溃。
        2. load_all_conversation_files: 批量加载一个目录下的所有对话历史文件,
           自动跳过无法解析的文件, 并记录哪些文件被跳过、原因是什么。
        3. ConversationStats: 汇总统计结果的数据结构。
        4. compute_stats_for_single_conversation /
           compute_aggregate_stats: 分别计算单份对话与多份对话汇总的统计信息。
        5. render_stats_report: 把统计结果渲染成一段人类可读的中文报告文本。
"""

from __future__ import annotations

import json
from dataclasses import dataclass, field
from pathlib import Path
from typing import List, Optional, Tuple


@dataclass
class LoadedConversation:
    """成功加载的单份对话历史, 附带来源文件路径, 便于后续报告中溯源。"""

    file_path: Path
    provider: str
    saved_at: str
    messages: list


@dataclass
class SkippedFile:
    """加载失败被跳过的文件记录, 附带跳过原因, 便于排查数据质量问题。"""

    file_path: Path
    reason: str


def load_conversation_file(file_path: Path) -> LoadedConversation:
    """
    加载单份对话历史json文件, 校验其结构是否符合ask_ai.py保存时的约定格式
    (必须包含provider/saved_at/messages三个顶层字段, messages必须是列表,
    且列表中每个元素都必须是包含role/content两个字段的字典)。

    :param file_path: 对话历史json文件路径
    :raises ValueError: 文件内容不是合法JSON, 或者结构不符合约定格式
    :return: 校验通过的LoadedConversation对象
    """
    try:
        raw_text = file_path.read_text(encoding="utf-8")
    except OSError as exc:
        raise ValueError(f"无法读取文件: {exc}") from exc

    try:
        data = json.loads(raw_text)
    except json.JSONDecodeError as exc:
        raise ValueError(f"文件内容不是合法的JSON: {exc}") from exc

    if not isinstance(data, dict):
        raise ValueError("顶层结构必须是一个JSON对象(字典), 而不是列表或其他类型")

    for required_field in ("provider", "saved_at", "messages"):
        if required_field not in data:
            raise ValueError(f"缺少必需的顶层字段: {required_field}")

    messages = data["messages"]
    if not isinstance(messages, list):
        raise ValueError("messages字段必须是一个列表")

    for index, message in enumerate(messages):
        if not isinstance(message, dict):
            raise ValueError(f"messages[{index}]必须是一个字典, 实际类型: {type(message).__name__}")
        if "role" not in message or "content" not in message:
            raise ValueError(f"messages[{index}]缺少role或content字段")

    return LoadedConversation(
        file_path=file_path,
        provider=str(data["provider"]),
        saved_at=str(data["saved_at"]),
        messages=messages,
    )


def load_all_conversation_files(
    directory: Path,
) -> Tuple[List[LoadedConversation], List[SkippedFile]]:
    """
    批量加载目录下所有以.json为扩展名的对话历史文件。

    :param directory: 存放对话历史文件的目录(通常是ask_ai.py里的
        conversation_history/目录)
    :return: 二元组(成功加载的对话列表, 被跳过的文件及原因列表)
    """
    loaded: List[LoadedConversation] = []
    skipped: List[SkippedFile] = []

    if not directory.exists():
        return loaded, skipped

    for file_path in sorted(directory.glob("*.json")):
        try:
            loaded.append(load_conversation_file(file_path))
        except ValueError as exc:
            skipped.append(SkippedFile(file_path=file_path, reason=str(exc)))

    return loaded, skipped


@dataclass
class ConversationStats:
    """一份(或多份汇总)对话历史的统计结果。"""

    provider: Optional[str] = None
    total_conversations: int = 0
    total_user_messages: int = 0
    total_assistant_messages: int = 0
    total_user_characters: int = 0
    total_assistant_characters: int = 0
    longest_user_message: str = ""
    longest_assistant_message: str = ""
    max_rounds_in_single_conversation: int = 0
    per_provider_conversation_count: dict = field(default_factory=dict)

    @property
    def average_assistant_reply_length(self) -> float:
        """AI回复的平均字符长度, 用于粗略衡量"这个厂商的回复是不是普遍更啰嗦"。"""
        if self.total_assistant_messages == 0:
            return 0.0
        return self.total_assistant_characters / self.total_assistant_messages

    @property
    def average_rounds_per_conversation(self) -> float:
        """平均每次对话包含多少轮问答(一轮 = 一条user消息 + 一条assistant消息)。"""
        if self.total_conversations == 0:
            return 0.0
        return self.total_user_messages / self.total_conversations


def compute_stats_for_single_conversation(conversation: LoadedConversation) -> ConversationStats:
    """计算单份对话历史的统计信息, 常用于"我这一次对话到底聊了些什么"的场景。"""
    stats = ConversationStats(provider=conversation.provider, total_conversations=1)

    user_messages = [m for m in conversation.messages if m.get("role") == "user"]
    assistant_messages = [m for m in conversation.messages if m.get("role") == "assistant"]

    stats.total_user_messages = len(user_messages)
    stats.total_assistant_messages = len(assistant_messages)
    stats.total_user_characters = sum(len(str(m.get("content", ""))) for m in user_messages)
    stats.total_assistant_characters = sum(len(str(m.get("content", ""))) for m in assistant_messages)
    stats.max_rounds_in_single_conversation = len(user_messages)

    if user_messages:
        stats.longest_user_message = max(
            (str(m.get("content", "")) for m in user_messages), key=len
        )
    if assistant_messages:
        stats.longest_assistant_message = max(
            (str(m.get("content", "")) for m in assistant_messages), key=len
        )

    stats.per_provider_conversation_count[conversation.provider] = 1
    return stats


def compute_aggregate_stats(conversations: List[LoadedConversation]) -> ConversationStats:
    """
    汇总计算多份对话历史的整体统计信息, 用于"我这一周总共问了多少问题,
    分别用了哪些模型服务商"这类回顾性分析。

    :param conversations: 已成功加载的多份对话历史
    :return: 汇总后的ConversationStats(provider字段在多provider混合场景下
        保留为None, 表示"这是一份跨服务商的汇总统计", 具体分布见
        per_provider_conversation_count字段)
    """
    aggregate = ConversationStats(total_conversations=len(conversations))

    longest_user_candidate = ""
    longest_assistant_candidate = ""

    for conversation in conversations:
        single_stats = compute_stats_for_single_conversation(conversation)

        aggregate.total_user_messages += single_stats.total_user_messages
        aggregate.total_assistant_messages += single_stats.total_assistant_messages
        aggregate.total_user_characters += single_stats.total_user_characters
        aggregate.total_assistant_characters += single_stats.total_assistant_characters
        aggregate.max_rounds_in_single_conversation = max(
            aggregate.max_rounds_in_single_conversation,
            single_stats.max_rounds_in_single_conversation,
        )

        if len(single_stats.longest_user_message) > len(longest_user_candidate):
            longest_user_candidate = single_stats.longest_user_message
        if len(single_stats.longest_assistant_message) > len(longest_assistant_candidate):
            longest_assistant_candidate = single_stats.longest_assistant_message

        provider = conversation.provider
        aggregate.per_provider_conversation_count[provider] = (
            aggregate.per_provider_conversation_count.get(provider, 0) + 1
        )

    aggregate.longest_user_message = longest_user_candidate
    aggregate.longest_assistant_message = longest_assistant_candidate

    if len(aggregate.per_provider_conversation_count) == 1:
        aggregate.provider = next(iter(aggregate.per_provider_conversation_count))

    return aggregate


def truncate_for_display(text: str, max_length: int = 60) -> str:
    """把过长的文本截断到指定长度用于展示, 避免报告里出现一整段几百字的原文, 影响阅读。"""
    if len(text) <= max_length:
        return text
    return text[:max_length] + "……(已截断)"


def render_stats_report(stats: ConversationStats, title: str = "对话历史统计报告") -> str:
    """把ConversationStats渲染成一段结构清晰的中文文本报告。"""
    lines = [
        "=" * 50,
        title,
        "=" * 50,
        f"对话文件总数: {stats.total_conversations}",
        f"累计提问次数: {stats.total_user_messages}",
        f"累计AI回复次数: {stats.total_assistant_messages}",
        f"平均每次对话的问答轮数: {stats.average_rounds_per_conversation:.1f}",
        f"AI回复平均长度(字符数): {stats.average_assistant_reply_length:.1f}",
        f"单次对话中最多的问答轮数: {stats.max_rounds_in_single_conversation}",
    ]

    if stats.per_provider_conversation_count:
        lines.append("各模型服务商使用次数分布:")
        for provider, count in sorted(
            stats.per_provider_conversation_count.items(), key=lambda item: item[1], reverse=True
        ):
            lines.append(f"  {provider}: {count}次")

    if stats.longest_user_message:
        lines.append(f"最长的一次提问: {truncate_for_display(stats.longest_user_message)}")
    if stats.longest_assistant_message:
        lines.append(f"最长的一次AI回复: {truncate_for_display(stats.longest_assistant_message)}")

    lines.append("=" * 50)
    return "\n".join(lines)


def main() -> None:
    """
    命令行入口: 加载conversation_history/目录下的全部历史文件, 打印汇总统计报告,
    并对每个被跳过的文件给出跳过原因(这在教学阶段有意保留了两份可能残缺的
    测试文件, 用于演示"数据质量问题不应该导致整个分析工具直接崩溃"这一原则)。
    """
    history_dir = Path(__file__).resolve().parent / "conversation_history"
    loaded, skipped = load_all_conversation_files(history_dir)

    if not loaded:
        print(f"目录{history_dir}下没有找到任何有效的对话历史文件。")
        print("(提示: 先运行ask_ai.py, 在退出时选择保存对话历史, 再重新运行本工具)")
    else:
        aggregate_stats = compute_aggregate_stats(loaded)
        print(render_stats_report(aggregate_stats, title="全部对话历史汇总统计"))

    if skipped:
        print()
        print(f"以下{len(skipped)}个文件因格式问题被跳过:")
        for item in skipped:
            print(f"  {item.file_path.name}: {item.reason}")


if __name__ == "__main__":
    main()
```

### 文件十四:`test_conversation_history_analyzer.py`(对话历史分析工具单元测试)

> 说明:针对conversation_history_analyzer.py的单元测试,使用Python标准库`tempfile`在系统临时目录下创建真实的、结构或明或暗有问题的json文件,验证加载与统计逻辑在各种正常与异常输入下都表现符合预期。选择"真的写文件再读回来"这种测试方式,是因为`load_conversation_file`的核心职责之一就是"处理来自磁盘文件的、可能存在各种格式问题的原始文本",这部分逻辑必须通过真实的文件读写才能被充分覆盖测试到。

```python
"""
文件名: test_conversation_history_analyzer.py
说明:
    针对conversation_history_analyzer.py的单元测试, 使用Python标准库
    tempfile在系统临时目录下创建真实的、结构或明或暗有问题的json文件,
    验证加载与统计逻辑在各种正常与异常输入下都表现符合预期。

    之所以选择"真的写文件再读回来"这种略显"重"的测试方式, 而不是
    只在内存里构造Python字典直接调用compute_stats_for_single_conversation,
    是因为load_conversation_file这个函数本身的核心职责之一,
    就是"处理来自磁盘文件的、可能存在各种格式问题的原始文本",
    这部分逻辑必须通过真实的文件读写才能被充分覆盖测试到。

    运行方式:
        python -m unittest test_conversation_history_analyzer.py -v
"""

from __future__ import annotations

import json
import shutil
import tempfile
import unittest
from pathlib import Path

from conversation_history_analyzer import (
    compute_aggregate_stats,
    compute_stats_for_single_conversation,
    load_all_conversation_files,
    load_conversation_file,
    render_stats_report,
    truncate_for_display,
)


def write_json_file(directory: Path, filename: str, content) -> Path:
    """辅助函数: 把content序列化为JSON并写入指定目录下的filename文件, 返回完整路径。"""
    file_path = directory / filename
    if isinstance(content, str):
        file_path.write_text(content, encoding="utf-8")
    else:
        file_path.write_text(json.dumps(content, ensure_ascii=False), encoding="utf-8")
    return file_path


class ConversationHistoryTestCase(unittest.TestCase):
    """
    提供一个每个测试方法都会自动创建/清理的临时目录, 避免测试之间
    互相污染文件系统状态, 也避免测试结束后在磁盘上留下垃圾文件。
    """

    def setUp(self) -> None:
        self.temp_dir = Path(tempfile.mkdtemp(prefix="day12_conv_history_test_"))

    def tearDown(self) -> None:
        shutil.rmtree(self.temp_dir, ignore_errors=True)


class TestLoadConversationFile(ConversationHistoryTestCase):
    """测试单份文件加载与结构校验逻辑。"""

    def test_valid_file_loads_successfully(self):
        content = {
            "provider": "deepseek",
            "saved_at": "2026-07-14T10:00:00",
            "messages": [
                {"role": "system", "content": "你是助手"},
                {"role": "user", "content": "你好"},
                {"role": "assistant", "content": "你好, 有什么可以帮你"},
            ],
        }
        file_path = write_json_file(self.temp_dir, "valid.json", content)

        loaded = load_conversation_file(file_path)
        self.assertEqual(loaded.provider, "deepseek")
        self.assertEqual(len(loaded.messages), 3)

    def test_invalid_json_text_raises_value_error(self):
        file_path = write_json_file(self.temp_dir, "broken.json", "{this is not valid json")
        with self.assertRaises(ValueError) as ctx:
            load_conversation_file(file_path)
        self.assertIn("JSON", str(ctx.exception))

    def test_top_level_not_a_dict_raises_value_error(self):
        file_path = write_json_file(self.temp_dir, "list_top_level.json", ["not", "a", "dict"])
        with self.assertRaises(ValueError) as ctx:
            load_conversation_file(file_path)
        self.assertIn("JSON对象", str(ctx.exception))

    def test_missing_required_field_raises_value_error(self):
        content = {"provider": "qwen", "messages": []}  # 缺少saved_at字段
        file_path = write_json_file(self.temp_dir, "missing_field.json", content)
        with self.assertRaises(ValueError) as ctx:
            load_conversation_file(file_path)
        self.assertIn("saved_at", str(ctx.exception))

    def test_messages_not_a_list_raises_value_error(self):
        content = {"provider": "qwen", "saved_at": "x", "messages": "不应该是字符串"}
        file_path = write_json_file(self.temp_dir, "bad_messages_type.json", content)
        with self.assertRaises(ValueError) as ctx:
            load_conversation_file(file_path)
        self.assertIn("messages字段必须是一个列表", str(ctx.exception))

    def test_message_missing_role_field_raises_value_error(self):
        content = {
            "provider": "qwen",
            "saved_at": "x",
            "messages": [{"content": "缺少role字段的消息"}],
        }
        file_path = write_json_file(self.temp_dir, "bad_message_item.json", content)
        with self.assertRaises(ValueError) as ctx:
            load_conversation_file(file_path)
        self.assertIn("messages[0]", str(ctx.exception))

    def test_message_item_not_a_dict_raises_value_error(self):
        content = {"provider": "qwen", "saved_at": "x", "messages": ["这应该是字典而不是字符串"]}
        file_path = write_json_file(self.temp_dir, "message_not_dict.json", content)
        with self.assertRaises(ValueError):
            load_conversation_file(file_path)

    def test_nonexistent_file_raises_value_error_not_crash(self):
        nonexistent = self.temp_dir / "does_not_exist.json"
        with self.assertRaises(ValueError):
            load_conversation_file(nonexistent)


class TestLoadAllConversationFiles(ConversationHistoryTestCase):
    """测试批量加载目录下所有文件的逻辑, 包括正常文件与残缺文件混合的场景。"""

    def test_empty_directory_returns_empty_results(self):
        loaded, skipped = load_all_conversation_files(self.temp_dir)
        self.assertEqual(loaded, [])
        self.assertEqual(skipped, [])

    def test_nonexistent_directory_returns_empty_results_without_crashing(self):
        loaded, skipped = load_all_conversation_files(self.temp_dir / "does_not_exist_dir")
        self.assertEqual(loaded, [])
        self.assertEqual(skipped, [])

    def test_mixed_valid_and_invalid_files_are_separated_correctly(self):
        write_json_file(
            self.temp_dir,
            "conversation_deepseek_20260714_100000.json",
            {
                "provider": "deepseek",
                "saved_at": "2026-07-14T10:00:00",
                "messages": [{"role": "user", "content": "问题一"}],
            },
        )
        write_json_file(
            self.temp_dir,
            "conversation_qwen_20260714_110000.json",
            {
                "provider": "qwen",
                "saved_at": "2026-07-14T11:00:00",
                "messages": [{"role": "user", "content": "问题二"}],
            },
        )
        write_json_file(self.temp_dir, "conversation_broken.json", "not valid json at all {{{")

        loaded, skipped = load_all_conversation_files(self.temp_dir)
        self.assertEqual(len(loaded), 2)
        self.assertEqual(len(skipped), 1)
        self.assertEqual(skipped[0].file_path.name, "conversation_broken.json")

    def test_non_json_files_are_ignored_entirely(self):
        """非.json扩展名的文件(比如README.txt)不应该被当作候选文件尝试加载, 也不应该出现在skipped列表里。"""
        (self.temp_dir / "README.txt").write_text("这不是对话历史文件", encoding="utf-8")
        write_json_file(
            self.temp_dir,
            "conversation_deepseek_1.json",
            {"provider": "deepseek", "saved_at": "x", "messages": []},
        )
        loaded, skipped = load_all_conversation_files(self.temp_dir)
        self.assertEqual(len(loaded), 1)
        self.assertEqual(len(skipped), 0)

    def test_files_are_loaded_in_sorted_filename_order(self):
        """
        验证加载顺序是按文件名排序的(而不是操作系统返回的任意顺序),
        这对报告的可重复性、以及"最早一份对话是哪个"这类分析很重要。
        """
        for suffix in ("c", "a", "b"):
            write_json_file(
                self.temp_dir,
                f"conversation_{suffix}.json",
                {"provider": "deepseek", "saved_at": suffix, "messages": []},
            )
        loaded, _ = load_all_conversation_files(self.temp_dir)
        saved_at_order = [item.saved_at for item in loaded]
        self.assertEqual(saved_at_order, ["a", "b", "c"])


class TestComputeStatsForSingleConversation(unittest.TestCase):
    """测试单份对话的统计计算逻辑, 使用内存构造的LoadedConversation, 不涉及文件系统。"""

    def _build_loaded(self, provider: str, messages: list):
        from conversation_history_analyzer import LoadedConversation

        return LoadedConversation(
            file_path=Path("/tmp/fake.json"), provider=provider, saved_at="x", messages=messages
        )

    def test_counts_user_and_assistant_messages_separately(self):
        conversation = self._build_loaded(
            "deepseek",
            [
                {"role": "system", "content": "系统消息不计入统计"},
                {"role": "user", "content": "问题一"},
                {"role": "assistant", "content": "回答一"},
                {"role": "user", "content": "问题二"},
                {"role": "assistant", "content": "回答二"},
            ],
        )
        stats = compute_stats_for_single_conversation(conversation)
        self.assertEqual(stats.total_user_messages, 2)
        self.assertEqual(stats.total_assistant_messages, 2)
        self.assertEqual(stats.max_rounds_in_single_conversation, 2)

    def test_longest_message_detection(self):
        conversation = self._build_loaded(
            "qwen",
            [
                {"role": "user", "content": "短问题"},
                {"role": "user", "content": "这是一个明显更长一些的问题内容"},
                {"role": "assistant", "content": "简短回答"},
            ],
        )
        stats = compute_stats_for_single_conversation(conversation)
        self.assertEqual(stats.longest_user_message, "这是一个明显更长一些的问题内容")

    def test_empty_messages_list_produces_zeroed_stats_without_crashing(self):
        conversation = self._build_loaded("deepseek", [])
        stats = compute_stats_for_single_conversation(conversation)
        self.assertEqual(stats.total_user_messages, 0)
        self.assertEqual(stats.average_assistant_reply_length, 0.0)
        self.assertEqual(stats.longest_user_message, "")

    def test_average_assistant_reply_length_calculation(self):
        conversation = self._build_loaded(
            "deepseek",
            [
                {"role": "assistant", "content": "12345"},  # 5字符
                {"role": "assistant", "content": "1234567890"},  # 10字符
            ],
        )
        stats = compute_stats_for_single_conversation(conversation)
        self.assertAlmostEqual(stats.average_assistant_reply_length, 7.5)


class TestComputeAggregateStats(unittest.TestCase):
    """测试多份对话汇总统计的逻辑, 重点验证跨provider的分布统计是否正确。"""

    def _build_loaded(self, provider: str, messages: list):
        from conversation_history_analyzer import LoadedConversation

        return LoadedConversation(
            file_path=Path("/tmp/fake.json"), provider=provider, saved_at="x", messages=messages
        )

    def test_aggregate_across_multiple_providers(self):
        conversations = [
            self._build_loaded("deepseek", [{"role": "user", "content": "q1"}, {"role": "assistant", "content": "a1"}]),
            self._build_loaded("deepseek", [{"role": "user", "content": "q2"}, {"role": "assistant", "content": "a2"}]),
            self._build_loaded("qwen", [{"role": "user", "content": "q3"}, {"role": "assistant", "content": "a3"}]),
        ]
        aggregate = compute_aggregate_stats(conversations)

        self.assertEqual(aggregate.total_conversations, 3)
        self.assertEqual(aggregate.per_provider_conversation_count["deepseek"], 2)
        self.assertEqual(aggregate.per_provider_conversation_count["qwen"], 1)
        # 混合了多个provider, provider字段应保持为None(表示这是跨厂商汇总)
        self.assertIsNone(aggregate.provider)

    def test_aggregate_with_single_provider_sets_provider_field(self):
        conversations = [
            self._build_loaded("deepseek", [{"role": "user", "content": "q1"}]),
            self._build_loaded("deepseek", [{"role": "user", "content": "q2"}]),
        ]
        aggregate = compute_aggregate_stats(conversations)
        self.assertEqual(aggregate.provider, "deepseek")

    def test_aggregate_of_empty_list_returns_zeroed_stats(self):
        aggregate = compute_aggregate_stats([])
        self.assertEqual(aggregate.total_conversations, 0)
        self.assertEqual(aggregate.average_rounds_per_conversation, 0.0)


class TestRenderStatsReportAndTruncate(unittest.TestCase):
    """测试报告渲染文本与截断辅助函数的边界情况。"""

    def test_truncate_short_text_unchanged(self):
        self.assertEqual(truncate_for_display("短文本", max_length=60), "短文本")

    def test_truncate_long_text_appends_marker(self):
        long_text = "字" * 100
        truncated = truncate_for_display(long_text, max_length=20)
        self.assertTrue(truncated.startswith("字" * 20))
        self.assertIn("已截断", truncated)

    def test_render_report_contains_title_and_counts(self):
        from conversation_history_analyzer import ConversationStats

        stats = ConversationStats(total_conversations=5, total_user_messages=10)
        report = render_stats_report(stats, title="测试标题")
        self.assertIn("测试标题", report)
        self.assertIn("5", report)
        self.assertIn("10", report)


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

## ask_ai.py 交互指令说明

- 直接输入问题并回车: 向当前模型服务商提问
- `/help`: 查看帮助
- `/clear`: 清空当前对话历史, 开始新一轮对话
- `/provider`: 查看当前使用的模型服务商
- `/switch deepseek` 或 `/switch qwen`: 切换模型服务商
- `/save`: 将当前对话历史保存为json文件
- `/exit` 或 `/quit`: 退出程序(会询问是否保存对话历史)

## 安全注意事项

1. API Key永远不要写入任何代码文件, 只通过环境变量加载。
2. API Key永远不要提交进Git仓库, 建议在项目根目录的`.gitignore`中
   提前排除任何可能意外包含密钥的本地配置文件。
3. 打印/记录日志时, 如果确实需要展示Key用于排查问题, 务必先脱敏
   (只显示前几位与后几位, 中间用星号替代), 参考`call_deepseek_api.py`
   中的`mask_key`函数实现。
4. 本项目`conversation_history/`目录下保存的对话记录, 如果对话内容涉及
   敏感信息, 请自行注意保管, 不建议提交进公开的代码仓库。

## 已知局限(将在Day13解决)

当前版本的所有API调用都是"裸调用"——遇到超时或限流, 只会打印一条
错误提示然后放弃这次提问, 不会自动重试。这是有意为之的教学设计,
Day13会引入装饰器与重试机制, 把"自动重试"这件事优雅地补上。
````

---

## 今日复盘

晚上9点15分,老王在群里发了一段比平时更长的话,算是今天的收尾。

"今天不点评代码细节了,"他先说了这么一句开场,"因为今天真正值得说的,不是谁的代码写得更漂亮,是你们四个人今天都做成了一件此前从没做成过的事——独立地、完整地、从'什么都不知道'到'跑通一整条链路',对接了一个真实的、生产级的大模型API。这件事的价值,不会随着你们以后写的代码越来越复杂而贬值,反而会随着时间推移,越来越显出它的分量——因为往后很长一段时间里,你们判断'这件事我到底会不会'的信心来源,很大程度上就是今天这个下午。"

他接着挑了几个细节点评,依旧没有回避问题:"苏梦,你下午调DeepSeek的时候,把`messages`字段误写成了字符串,卡了快一个小时才发现,这个我知道你现在回想起来可能会觉得'当时怎么这么笨',但我要说一句公道话——这种错误,几乎是每一个第一次对接结构化API请求体的人都会踩的坑,报错信息本身'`400 Bad Request`'不会直接告诉你哪里错了,排查思路比结果本身更重要。你后来想到打印完整的`payload`去对照文档核对结构,这个思路是对的,以后遇到类似的'请求体格式不对但报错信息很笼统'的场景,这就是你的第一反应,不需要再从头摸索。"

"张凡,你问的HTTP状态码分类问题问得挺细,这说明你在认真消化协议层面的东西,这是好事,但我也要提醒一句——今天这个阶段,你不需要把所有状态码背下来,更重要的是记住'看到4字头先怀疑自己、看到5字头先怀疑对方'这个判断思路,细节遇到具体问题的时候再查文档,不用现在死记硬背。"

"韩露,"老王的语气在这里停顿了一下,"你的API Key,我看到你直接写在了`ask_ai.py`的一个字符串常量里,没有走环境变量。这个我们明天说,今天先记一句——这不是小事。"这句话在群里被韩露看到之后,她心里咯噔了一下,但当时并没有意识到接下来会发生什么,只是回复了一句"好的老王,我今晚改一下"。

"对陈铭,"老王最后说,"你今天代码写得挺规范,`PROVIDER_CONFIGS`字典的设计我提到过,值得肯定。有一个小地方可以再想想——你在`ask_ai.py`里,请求失败之后把刚追加的用户消息从`conversation`里`pop()`出来,这个处理是对的,避免了'一次失败的提问'污染对话历史,但你有没有想过,如果这次失败是限流(429),理论上你的问题本身是没问题的,只是暂时被限流了,是不是应该有一种'保留这个问题,等一等再自动重新问一次'的处理方式,而不是直接放弃?"陈铭想了想回复:"是不是就是明天要学的重试装饰器?""对,提前想到这一步,说明你今天这条链路走得很扎实。"老王回复。

陈铭在自己的成长笔记里写下了今天的感受,写得比平时长:"今天是这十一天以来最'不一样'的一天。不是因为知识点特别难——HTTP协议的概念其实没有想象中复杂,`requests`库用起来也很直观——是因为这是第一次,我写的代码真的'走出去'了,真的有一个我完全不认识、在几千公里之外某个机房里跑着的服务器,认认真真地处理了我发过去的请求,又认认真真地把一段文字发了回来。老王说这是'离最初报名转行学AI开发这件事最近的一次',我现在完全理解这句话的意思了。之前学的东西,不管是列表、字典、类、异常处理,都是在给'今天'做准备,而'今天'又在给后面几十天做准备——原来学习这件事,不是一段一段孤立的知识点堆起来的,是一条能看见方向、能感觉到自己在往前走的路。"

他又补了一句关于韩露那件事的记录,带着一点担忧:"老王今晚点名说了韩露的API Key直接写在代码里的事,说'这不是小事'。我不知道接下来会怎样,但我今晚回去第一件事,是把自己的`ask_ai.py`又检查了一遍,确认Key确实是从环境变量读的,没有任何一个地方手滑写死了字符串。这大概就是老王想要的效果——不是等出了事才后怕,是提前多检查一遍。"

窗外这时候比昨天更安静一些,楼下便利店的灯还亮着,张凡在工位上又把`ask_ai.py`跑了一轮,这次问的是一个纯属好奇的问题——"你知道你自己是被谁调用的吗",AI的回复颇为得体又带点分寸,张凡看完笑出了声,把这段对话截图发到了自己的朋友圈,配文是"今天开始正式跟AI说上话了"。

---

## 课后作业

**1.(概念题)** 请解释HTTP协议中GET和POST两种请求方法的核心语义区别,并说明为什么绝大多数大模型对话API(如DeepSeek、通义千问)都采用POST而不是GET。

**2.(概念题)** 请解释为什么"大模型没有真正的记忆能力,多轮对话的连续性是靠客户端自己维护并重新发送完整的messages列表实现的"这个说法。如果不这样做(比如每次只发送用户的最新一句话,不带历史),会出现什么现象?

**3.(代码题)** 请编写一个函数`safe_call_with_fallback(primary_provider, fallback_provider, conversation)`,优先尝试用`primary_provider`调用模型API,如果捕获到`AuthenticationError`或`ModelConnectionError`(视为该服务商当前不可用),自动切换到`fallback_provider`重试一次;如果两个服务商都失败,向上抛出最后一次捕获到的异常。可以直接复用`ask_ai.py`中的`call_model_api`函数。

**4.(代码题)** 请基于`error_handling_demo.py`中的思路,编写一个函数`classify_http_error(status_code)`,接收一个HTTP状态码,返回一段中文描述,至少覆盖400、401、403、404、429、500、502、503、504这几种常见状态码,对于表中未覆盖的状态码,返回一个通用的兜底描述(比如"未特别分类的错误状态码,请查阅具体接口文档")。

**5.(代码题)** 请为`ask_ai.py`新增一个`/history`指令,输入该指令时,打印出当前对话历史中所有的问答内容(不包括最开头的system消息),每一条前面标注序号,格式类似"1. 你: xxx" "2. AI: xxx"。

**6.(思考题)** 老王在晚自习提到,今天`ask_ai.py`的所有API调用都是"裸调用",没有任何重试机制。请结合今天学到的401、429、超时这三类错误的特点,分析一下:如果要设计一套"自动重试"策略,这三类错误分别应该采用什么样的重试策略(是否应该重试、重试前要不要等待、等待多久、最多重试几次),并说明理由。

**7.(思考题)** 韩露今天把API Key直接写在了代码文件里,被老王当场指出"这不是小事"。请结合你对软件工程实践的理解,分析一下:如果这份代码后续被提交进了一个公开的GitHub仓库,可能会带来哪些具体的风险?除了"不要写死在代码里, 用环境变量加载"之外,你还能想到哪些进一步的安全防护手段(可以从代码扫描工具、密钥轮换机制、访问权限最小化等角度展开思考,不要求你现在就会实现,只需要说明思路)?

---

## 作业参考答案

**1. 参考答案**

GET和POST的核心语义区别在于"意图"——GET表达的是"我要获取一份既有的资源",本质上是一次只读的查询操作,理论上不应该对服务器端的数据产生任何修改性的副作用(这也是为什么GET请求通常被认为是"安全"且"可重复执行多次而效果一致"的,浏览器甚至会缓存GET请求的结果);POST表达的是"我要提交一份数据, 让服务器基于这份数据做一些处理或产生某种变化",这个"变化"可能是创建一条新记录、触发一次计算、或者(在大模型场景下)触发一次推理生成,这类操作通常不应该被无脑缓存或重复执行(重复提交同一个问题两次,理论上会消耗两次调用配额,产生两次独立的推理计算,而不是复用第一次的缓存结果)。

大模型对话API普遍采用POST而不是GET,原因有三层:第一,语义上更贴切——用户提一个问题,本质上是"提交一份数据(问题内容)给服务器处理", 更贴近POST的语义,而不是"获取一份既有资源"; 第二,数据量与格式的限制——GET请求的参数只能放在URL里,长度有限制,而且不适合传输结构化的、可能很长的JSON数据(比如完整的多轮对话历史),POST的请求体没有这类限制,天然适合传输复杂的JSON结构;第三,安全性考量——GET请求的参数会明明白白出现在URL中,容易被浏览器历史记录、代理服务器日志、甚至网络监控设备记录下来,而调用大模型API时,请求头里通常要携带`Authorization`这类敏感鉴权信息,body里也可能包含用户的隐私问题内容,这些内容更适合放在不会被随意记录到URL日志里的请求体和请求头中,POST在这一点上比GET更安全。

**2. 参考答案**

大模型的推理过程,本质上是"输入一段文本(严格来说是token序列),经过模型的神经网络计算,输出下一段文本",这个计算过程本身是无状态的——同一个模型服务, 处理来自不同用户、甚至同一用户不同时刻的请求, 彼此之间默认互不知晓、互不影响, 模型服务器不会在自己内部为"陈铭这个用户"维护一份专属的、跨请求持续存在的"记忆区"。你之所以感觉AI"记得"你们之前聊过什么,是因为客户端(比如今天写的`ask_ai.py`)在每一次发起新请求时, 都把"从对话开始到现在为止的完整历史"重新打包, 通过`messages`这个列表一次性发送给模型, 模型是在"重新阅读了一遍完整的对话记录"之后, 才生成了这次的回复——它并不是"记得", 而是"每次都被重新告知了一遍"。

如果不这样做,每次只发送用户最新的一句话,不带任何历史上下文,会出现的现象是:AI会表现得像"完全失忆"一样, 无法回应任何依赖上下文的追问。比如用户第一句说"我叫陈铭", 第二句问"我刚才说我叫什么", 如果第二次请求的`messages`里只包含"我刚才说我叫什么"这一句, 模型在"读"这句话的时候, 根本无法知道"我"是谁、"刚才"说了什么, 大概率会回答类似"抱歉, 我无法获取您之前提到的信息"这样的话。这正是今天课堂上老王专门设计的"体感练习"——先用最简单的单轮版本让大家亲眼看到这个"失忆"现象,再引入`messages`历史累积的写法,让大家直观感受到"连续对话"这个体验,是完全靠客户端主动维护出来的,而不是模型自带的能力。

**3. 参考答案**

```python
from api_exceptions import AuthenticationError, ModelConnectionError, CangqiongError


def safe_call_with_fallback(primary_provider, fallback_provider, conversation):
    """
    优先尝试primary_provider, 如果遇到鉴权失败或连接失败, 自动切换到
    fallback_provider重试一次, 两者都失败则抛出最后一次捕获到的异常。

    :param primary_provider: 优先使用的模型服务商, 如"deepseek"
    :param fallback_provider: 备用的模型服务商, 如"qwen"
    :param conversation: 完整的对话历史列表
    :raises CangqiongError: 两个服务商都调用失败时, 抛出fallback阶段的异常
    :return: AI回复的文本内容
    """
    from ask_ai import call_model_api  # 复用ask_ai.py中已经实现好的统一调用入口

    try:
        return call_model_api(primary_provider, conversation)
    except (AuthenticationError, ModelConnectionError) as primary_exc:
        print(
            f"[提示] {primary_provider}调用失败({primary_exc}), "
            f"正在自动切换到{fallback_provider}重试……"
        )
        try:
            return call_model_api(fallback_provider, conversation)
        except CangqiongError as fallback_exc:
            print(f"[提示] {fallback_provider}同样调用失败: {fallback_exc}")
            raise


if __name__ == "__main__":
    demo_conversation = [
        {"role": "system", "content": "你是一个简洁的助手。"},
        {"role": "user", "content": "用一句话介绍Python语言。"},
    ]
    try:
        reply = safe_call_with_fallback("deepseek", "qwen", demo_conversation)
        print(f"最终得到的回复: {reply}")
    except CangqiongError as exc:
        print(f"两个服务商均不可用: {exc}")
```

解析:本题考查的核心思路是"服务降级"(fallback)——工程实践中,当某个外部依赖(这里是某个模型服务商)出现问题时,一种常见的高可用策略就是自动切换到备用方案,而不是让整个功能直接不可用。这里特意只对`AuthenticationError`和`ModelConnectionError`这两类"当前这个服务商大概率短期内都用不了"的错误做fallback,而不是对所有异常都无条件fallback——比如`RateLimitError`(限流)其实更适合"等一等再用原服务商重试",而不是立刻换一家,因为换一家不一定能解决"这次问题本身出在我的请求内容或者调用频率"这类情况;`ModelResponseParseError`也不适合无脑fallback,因为如果是请求内容本身有问题,换个服务商大概率还是会遇到类似的解析问题。这种"针对不同错误类型设计不同应对策略"的思路,是Day13重试装饰器设计时会进一步深化的核心思想。

**4. 参考答案**

```python
def classify_http_error(status_code: int) -> str:
    """
    根据HTTP状态码返回一段中文描述, 覆盖常见的客户端错误与服务端错误,
    未覆盖的状态码返回通用兜底描述。

    :param status_code: HTTP状态码
    :return: 中文描述字符串
    """
    known_errors = {
        400: "请求格式错误(Bad Request), 通常是请求体JSON结构不符合接口要求, "
             "建议打印完整的payload对照官方文档核对字段类型",
        401: "鉴权失败(Unauthorized), 通常是API Key未设置、错误、包含多余空白字符, "
             "或者Key已过期/被禁用",
        403: "权限不足(Forbidden), 鉴权本身通过了, 但当前身份没有权限访问这个具体资源或接口",
        404: "资源不存在(Not Found), 通常是请求的URL路径写错了, 检查接口地址是否正确",
        429: "触发限流(Too Many Requests), 请求过于频繁, 建议等待一段时间后重试, "
             "不建议立即重试",
        500: "服务器内部错误(Internal Server Error), 问题通常出在服务端, "
             "不一定是你的请求有问题, 可以稍后重试或联系服务商",
        502: "网关错误(Bad Gateway), 通常是服务端内部的某个上游服务暂时出了问题",
        503: "服务暂时不可用(Service Unavailable), 常见于服务器过载或正在维护, "
             "建议稍后重试",
        504: "网关超时(Gateway Timeout), 服务端等待上游服务响应超时, "
             "和客户端设置的请求timeout是两个不同层面的超时概念",
    }
    return known_errors.get(
        status_code, f"未特别分类的错误状态码({status_code}), 请查阅具体接口文档确认含义"
    )


if __name__ == "__main__":
    for code in [200, 400, 401, 403, 404, 429, 500, 502, 503, 504, 418]:
        print(f"{code}: {classify_http_error(code)}")
```

解析:本题是对上午课堂笔记里"状态码分类表"的一次代码化整理——用一个字典把状态码和对应的中文描述关联起来,`dict.get()`的第二个参数天然提供了兜底默认值的能力,不需要写额外的`if...else`分支去处理"未知状态码"的情况,这是Python里处理"查表+兜底默认值"场景的一个常用惯用法(idiom)。这个函数在真实项目里可以进一步扩展成一个独立的小工具模块,被所有需要展示错误信息的地方复用,避免每个模块都各自维护一份不一致的错误描述文案。

**5. 参考答案**

```python
"""
在 ask_ai.py 的 run_interactive_loop 函数中, 新增以下分支
(建议放在处理"/save"指令的分支之后, 处理普通问题的逻辑之前):

        if user_input == "/history":
            print_conversation_history(conversation)
            continue

并在文件中新增以下函数:
"""


def print_conversation_history(conversation: list) -> None:
    """
    打印当前对话历史中的问答内容(跳过最开头的system消息),
    每条消息前标注序号, 格式为"序号. 角色: 内容"。

    :param conversation: 完整的messages历史列表
    """
    real_messages = [msg for msg in conversation if msg["role"] != "system"]
    if not real_messages:
        print("[提示] 当前还没有任何对话记录。")
        return

    print("=" * 40)
    print("当前对话历史:")
    for index, msg in enumerate(real_messages, start=1):
        speaker = "你" if msg["role"] == "user" else "AI"
        print(f"{index}. {speaker}: {msg['content']}")
    print("=" * 40)
```

解析:本题考查的是在已有代码结构基础上做"合理扩展"的能力——新增指令的实现方式,完全遵循了`ask_ai.py`里已有的"以`/`开头的指令走特殊分支, 其余输入当作普通问题"的既有设计模式, 没有引入新的代码组织范式, 保持了和已有代码风格的一致性。用列表推导式`[msg for msg in conversation if msg["role"] != "system"]`过滤掉system消息, 是因为system消息是"设定AI人设"用的, 不属于真正的"问答对话", 展示给用户看历史记录时应该排除掉, 这个细节体现了"从用户视角出发设计交互内容"的意识。

**6. 参考答案**(思考题, 以下给出一种有理有据的分析框架)

**401鉴权失败**:不应该自动重试。401的根本原因是"这次请求携带的凭证本身有问题"(Key错误、缺失、过期), 如果不改变Key本身,不管重试多少次,结果都会是同一个401,自动重试在这种场景下是完全无意义的重复劳动,甚至可能因为反复无效请求而触发额外的风控。正确的处理方式是:立即终止本次调用,给出清晰的"请检查API Key"提示, 交给人工介入排查, 而不是自动重试。

**429限流**:应该重试,但必须"等待后重试",而不是立即重试。限流的本质是"你请求得太快了, 慢一点就好", 立即重试大概率会再次落在同一个限流时间窗口内, 继续失败, 甚至可能进一步恶化限流状态。合理的策略是: 读取响应头里的`Retry-After`字段(如果服务商提供了这个信息), 按建议的秒数等待后重试; 如果没有提供, 采用一个相对保守的固定等待时间(比如3-5秒), 或者更优雅地采用"指数退避"策略(第一次等1秒, 第二次等2秒, 第三次等4秒, 逐次增加等待时间), 并设置一个最大重试次数上限(比如3-5次), 避免无限重试下去。

**超时**:应该重试, 且大多数情况下可以立即重试或者短暂等待后重试。超时的常见原因往往是网络的"临时抖动"或者服务端某一瞬间的短暂繁忙, 具有较大的偶然性和临时性, 立即重试(或者等待1-2秒后重试)往往就能成功。但同样需要设置一个最大重试次数上限, 因为如果连续多次都超时, 大概率说明存在更持续性的问题(比如本机网络完全不通, 或者服务商正在经历较长时间的服务异常), 这时应该放弃重试, 给出明确提示, 而不是无休止地卡在原地。

总结起来,这三类错误的应对策略, 背后遵循同一个更普适的判断原则:先判断"如果什么都不改变, 单纯重试一次, 结果有没有可能变得不一样"——如果答案是"不会变"(如401), 就不应该重试;如果答案是"有可能变, 但需要给对方一点时间"(如429和超时), 就应该"等待后重试", 且必须设置重试次数上限, 避免异常情况下的死循环或者资源浪费。这正是Day13重试装饰器要解决的核心设计问题——一个好的重试装饰器, 应该支持"针对不同异常类型采用不同的重试策略", 而不是对所有异常一视同仁地"无脑重试N次"。

**7. 参考答案**(思考题, 以下给出一种可参考的分析框架)

如果这份包含真实API Key的代码被提交进了一个公开的GitHub仓库, 具体的风险包括: 第一, 任何人都可以直接在仓库里看到这个Key, 拿去调用对应的大模型API, 消耗掉本该属于你自己(或者公司)的调用额度, 如果是按量计费的账户, 可能会产生真实的经济损失; 第二, 即使你事后发现问题并从最新代码中删除了这个Key, 只要它曾经出现在某一次Git提交记录里, 它依然会永久保留在Git的历史记录中(可以通过`git log`、`git show`等命令追溯到任何一个历史提交), 除非做非常复杂的历史重写操作(比如`git filter-branch`或者`BFG Repo-Cleaner`这类工具, 而且即使重写了本地历史, 如果早已被别人克隆或者被搜索引擎、GitHub自身的缓存索引到, 依然可能残留在别处), 想要"彻底抹去"一个曾经泄露过的密钥, 远比想象中困难, 这也是为什么行业里的标准做法永远是"一旦确认泄露, 第一时间去服务商后台吊销这个Key、重新生成一个新的", 而不是寄希望于"删掉代码就没事了"; 第三, 一些自动化的"密钥扫描机器人"(包括GitHub自己的Secret Scanning功能, 以及一些第三方安全扫描工具)会持续扫描公开仓库, 一旦检测到符合已知密钥格式特征的字符串, 会自动发出警报甚至自动通知对应的服务商, 也就是说, 密钥泄露这件事,很可能在你自己都没意识到的情况下, 已经被别人(或者机器人)率先发现。

除了"不要写死在代码里、用环境变量加载"这个最基础的措施之外,更完善的安全防护体系还可以从以下几个角度展开:第一,**代码提交前的自动扫描**——在Git提交流程中接入`pre-commit`钩子, 结合类似`detect-secrets`、`gitleaks`这类开源工具, 在代码真正被提交之前, 自动扫描本次改动是否包含符合已知密钥格式的字符串, 一旦发现直接阻止提交, 把"人的疏忽"这个环节, 用工具做一层前置拦截; 第二,**密钥的定期轮换机制**——不依赖一个密钥"一直用到出问题为止", 而是建立制度性的定期更换密钥的流程, 即使某个密钥在某个时间点意外泄露, 由于它会在不久之后的正常轮换周期内被替换掉, 泄露造成的风险窗口期也会被压缩; 第三,**访问权限最小化**——如果服务商支持创建"权限受限"的子密钥(比如只允许调用某几个特定的接口、设置每日调用额度上限、绑定特定的IP地址范围), 应该优先使用这种权限受限的密钥, 而不是使用拥有账户全部权限的"主密钥", 一旦泄露, 受限密钥造成的潜在损失也相对可控; 第四, **秘钥管理系统的引入**——在更成熟的团队和生产环境里, 密钥通常不会直接以环境变量的方式散落在每个人的本地机器和每台服务器上, 而是统一托管在专门的密钥管理服务(比如云厂商提供的KMS/Secrets Manager类产品)里, 应用程序在运行时动态地、按需地去获取密钥, 而不是长期以明文形式静态存储在任何一个具体的地方, 这样即使某台服务器或者某个人的账号权限被攻破, 影响范围也能被严格限定和快速响应。这几层防护手段叠加起来, 才能构成一套真正经得起考验的密钥安全体系, 而不是仅仅依赖"大家自觉不写死密钥"这一条相对脆弱的人为约束。

---

## 明日预告

晚上9点半,老王在群里发了今天最后一段话,语气比平时更认真一点:"今天你们跑通的`ask_ai.py`,现在能问能答、能连续追问、切换厂商也没问题,看起来已经是个'能用'的小工具了。但我要提醒你们一件事——今天这个版本,是最脆弱的版本, 遇到网络稍微不稳定, 或者你问得稍微频繁一点, 它随时可能崩掉, 而且崩掉的姿态还不算优雅, 是直接把一堆报错信息甩在终端上, 整个程序退出。"

他停顿了一下:"明天(Day13),你们会亲手体验一次这种'脆弱'——我会拿着你们今天写出来的代码,故意在网络不太稳定的情况下反复调用,大概率会让你们看到程序真实地卡住、真实地报出一个叫`ReadTimeout`的错误然后崩溃退出。到那时候,你们会理解一件事——一个真正能长期稳定运行的程序,不能假设外部世界永远配合,必须提前设计好'如果对方不配合,我该怎么应对'的方案。"

"这就引出明天的核心内容——装饰器。"老王写道,"你们会学到怎么用一个装饰器,给任何一个可能失败的函数,自动加上'失败后等一等、再试一次'的能力,而不需要在每个调用的地方,重复写一遍`try/except`加`time.sleep`的代码。除此之外,明天还会讲生成器、`typing`类型注解,以及`asyncio`异步编程的入门——你们会第一次感受到'同时发起多个请求、不用排队等待'是什么体验,这对以后要处理'一个用户请求需要同时问好几个模型'这类场景至关重要。"

他最后补了一句,没有说明具体是什么事,只是留了一个悬念:"另外明天早上,我会先讲一件昨晚发生的、和API Key有关的事情——这件事我今天已经提过一句,但具体细节,留到明天当面说。"

陈铭看到这句话,心里嘀咕了一下——是不是韩露那个写死在代码里的Key出了什么问题?他把这句话记在笔记本上,在旁边画了一个问号,准备明天早上第一个到会议室,看看到底发生了什么。窗外,便利店的灯还亮着,楼道里飘来一点晚风,陈铭合上电脑之前,又把自己的`ask_ai.py`跑了最后一轮——这次他问的是:"如果一个程序每天要被调用几百万次, 网络出问题的概率是不是也会跟着放大?"AI给出的回答,提到了"大数定律下, 小概率事件在足够大的调用量面前, 会变成必然会发生的事情"这样一句话。陈铭盯着这句话,忽然对明天要学的内容,多了一层更具体的期待。

