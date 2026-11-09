# RiverTable · 河牌德州

基于 Cocos Creator 3.8.8 的经典德州扑克项目。采用客户端与服务器分离架构，计划支持本地服务器及局域网多人对局，使用虚拟筹码。

## 当前状态

- Client 已由 Cocos Creator 3.8.8 创建，已补齐业务资源和脚本目录。
- Server、Shared、Tools、Docs 已建立目录骨架。
- 尚未实现游戏逻辑、服务器入口、依赖配置、协议同步工具和数据库。
- 本文中的技术方案和功能为开发规划，不代表目前已可运行。

## 技术规划

| 模块 | 技术与职责 |
| --- | --- |
| Client | Cocos Creator 3.8.8、TypeScript；界面、输入、动画、音效 |
| Server | 计划使用 Node.js、TypeScript、WebSocket；房间、规则、发牌、结算 |
| Shared | TypeScript 公共协议、公开数据类型、错误码 |
| 本地存储 | 计划使用 SQLite，存放玩家资料及对局记录 |

服务器负责洗牌、发牌、行动合法性、底池计算和最终结算。客户端只提交操作并展示服务器下发的状态；服务器应按玩家生成可见状态，避免泄露其他玩家的底牌和牌堆。

## 目录结构

业务目录使用大驼峰命名（PascalCase）。Cocos 管理的 assets、settings、profiles 等目录以及用于动态加载的 resources 保留约定名称。不要直接重命名 Cocos 工程目录或带有 .meta 的资源；后续资源调整应在编辑器中进行。

```text
RiverTable/
├─ README.md
├─ Client/                         # Cocos Creator 3.8.8 工程
│  ├─ assets/
│  │  ├─ Scenes/                   # Boot、Lobby、Table 场景（待创建）
│  │  ├─ Scripts/
│  │  │  ├─ Core/                  # 初始化、事件、场景切换
│  │  │  ├─ Network/               # 连接、心跳、断线重连
│  │  │  ├─ Models/                # 客户端公开状态
│  │  │  ├─ Features/
│  │  │  │  ├─ Lobby/              # 大厅、创建及加入房间
│  │  │  │  ├─ Table/              # 牌桌、座位、公共牌、底池
│  │  │  │  ├─ Actions/            # 弃牌、过牌、跟注、加注
│  │  │  │  └─ Settlement/         # 摊牌与结算展示
│  │  │  ├─ UI/                    # 通用界面组件
│  │  │  ├─ Utils/                 # 工具函数
│  │  │  └─ Generated/Shared/      # 公共定义同步目标，勿手动修改
│  │  ├─ Prefabs/
│  │  │  ├─ Table/
│  │  │  └─ UI/
│  │  ├─ Art/                     # 图片、图集、字体
│  │  ├─ Audio/                   # 音乐与音效
│  │  └─ resources/Config/        # 动态加载配置
│  ├─ settings/                   # Cocos 工程设置
│  ├─ profiles/                   # Cocos 编辑器配置
│  ├─ package.json
│  └─ tsconfig.json
├─ Server/
│  ├─ Src/
│  │  ├─ Config/                  # 服务配置
│  │  ├─ Transport/               # WebSocket 与消息路由
│  │  ├─ Sessions/                # 会话、心跳、重连绑定
│  │  ├─ Lobby/                   # 房间列表及加入流程
│  │  ├─ Rooms/                   # 房间生命周期、座位管理
│  │  ├─ Game/
│  │  │  ├─ Engine/               # 对局状态机、行动顺序
│  │  │  ├─ Deck/                 # 洗牌、发牌
│  │  │  ├─ Rules/                # 盲注与下注规则
│  │  │  ├─ Evaluator/            # 最佳五张牌与牌型比较
│  │  │  ├─ Pots/                 # 主池、边池、筹码分配
│  │  │  └─ Views/                # 按玩家生成可见状态
│  │  ├─ Persistence/             # 数据访问与迁移
│  │  └─ Logging/                 # 日志与对局事件
│  ├─ Tests/
│  │  ├─ Rules/
│  │  ├─ Evaluator/
│  │  ├─ Pots/
│  │  └─ Integration/
│  └─ Data/                       # 本地数据库运行目录
├─ Shared/Src/
│  ├─ Protocol/                   # 请求、响应、广播
│  ├─ Types/                      # 卡牌、行动、公开状态
│  └─ Constants/                  # 阶段枚举与错误码
├─ Tools/                         # 公共定义同步等工具（待实现）
└─ Docs/                          # 架构、规则、协议文档（待编写）
```

Cocos 自动生成的 library、temp、.creator 等目录未在上方展开。

## 本地开发

1. 在 Cocos Dashboard 中添加并使用 Cocos Creator 3.8.8 打开 Client 目录。
2. 等待编辑器导入新资源目录并生成对应 .meta 文件。
3. 在编辑器中创建 Boot、Lobby、Table 场景，保存到 assets/Scenes。
4. 后续实现 Server 的依赖、配置和入口后，再补充具体安装与启动命令。目前没有可执行的服务器启动脚本。

服务器规划地址为 `ws://127.0.0.1:3000`。局域网设备调试时，使用服务器电脑的局域网 IP；服务端需配置监听地址并允许对应端口访问。此地址尚未启用。

## 公共协议约定

Shared/Src 是公共定义的唯一来源。后续由 Tools 中的同步工具生成 Client/assets/Scripts/Generated/Shared 内的代码，服务器直接引用公共定义。协议同步脚本尚未实现。

完整牌堆、其他玩家的隐藏手牌等服务器内部状态不得放入发给客户端的公共状态中。

## 开发顺序

1. 明确 2–9 人无限注德州扑克规则、消息协议和对局状态。
2. 实现规则引擎、牌型比较、主池与边池结算，并验证关键规则。
3. 实现本地服务器、会话、房间及 WebSocket 通信。
4. 实现 Cocos 大厅、牌桌和操作界面。
5. 补充断线重连、对局记录、动画及音效。

## 版本管理

空目录默认不会被 Git 跟踪；需要保留在仓库中的空目录使用 .gitkeep 占位。运行产生的数据库、日志及 Cocos 缓存不应提交。

客户端、服务器和共享代码统一由根目录 RiverTable 的 Git 仓库管理，Client 不再包含独立仓库。Client/.gitignore 用于排除客户端缓存、依赖和构建产物；资源及其 .meta 文件、工程设置应一同提交。
