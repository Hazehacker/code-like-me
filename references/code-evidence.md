# 代码证据与取舍

分析日期：2026-10-15。用途：核对 skill 中的个人偏好、解决与旧代码的冲突，以及后续根据真实 Review 更新。无需每次编码都加载。

## 取样边界

用户明确提供的理念来自知识库 `_个人 Agent 系统/编码习惯&架构 SKILL——设计.md`：减小对已有行为的影响，先辨认能力、状态和生命周期的归属，用日志和注释降低验证成本，从可复现、可迁移的问题中沉淀判断顺序。当前 AGENTS.md 又明确“长期总成本”优先，因而不能把“最小改动”误解为永远局部补丁。

本次阅读了三个仓库的选定 Controller、Service、Mapper、DTO、Client、Listener 及历史差异，属于有针对性的抽样，不是全仓审计。未运行三个项目，也未认定全部代码均为用户手写。

| 仓库 | 固定取样版本 | 使用方式 |
| --- | --- | --- |
| [blog.hazenix.top](https://github.com/Hazehacker/blog.hazenix.top) | `97d978451e4e13562cd927734d44270aa666f24e`，2025-11-12；另读 `6959b38b` 规范化差异 | 以早期业务实现与主动风格调整为主，排除本地未提交改动 |
| [HazeEats](https://github.com/Hazehacker/HazeEats) | `6e58d4e36a5f5bc793f74fd528df3bc084286ba2` | 读取远端快照的订单、菜品、购物车、Client 与 Listener；未完成全历史取样 |
| [Auris](https://github.com/Hazehacker/Auris) | `5eec9f423086c2151259dfbcdcdf16e756d5be0d`，2026-01-08 | 查看提交历史与歌单、歌曲业务；结合独立功能提交辨认变化边界 |

网络读取遇到 GitHub API `HTTP Error 403: rate limit exceeded` 和完整克隆 `TimeoutExpired`。改用博客本地已提交历史、Auris 克隆、HazeEats 浅克隆与固定 SHA 的原始文件完成取样。版本选择不是“最新代码都已验证”的声明。

**证据权重：** 用户明确理念 > 主动表达取舍的修改记录 > 多个业务样本的共同写法 > 单个现状或模板。提交者身份与 `@author` 只能辅助定位，不能证明逐行手写或原创。下文“较强”也不等于排除了 AI 辅助。

## 进入 skill 的偏好

### 1. 构造器注入：有主动选择，证据较强

博客 [6959b38b](https://github.com/Hazehacker/blog.hazenix.top/commit/6959b38bcf70dece6a22a7c631e22a8756c5c3dd) 的提交说明明确写了阅读《阿里巴巴开发规范》后调整开发风格。差异包括：多个 Controller、Service、配置类从 `@Autowired` 字段注入改为 `@RequiredArgsConstructor` 加 `private final`；将多处业务提示提取到 `MessageConstant`；部分标准输出改用日志。

后续样本：[博客 CategoryServiceImpl L24–31](https://github.com/Hazehacker/blog.hazenix.top/blob/97d978451e4e13562cd927734d44270aa666f24e/blog-hazenix-top/blog-server/src/main/java/top/hazenix/service/impl/CategoryServiceImpl.java#L24-L31)、[Auris PlaylistServiceImpl L25–31](https://github.com/Hazehacker/Auris/blob/5eec9f423086c2151259dfbcdcdf16e756d5be0d/backend/auris-server/src/main/java/top/hazenix/auris/service/impl/PlaylistServiceImpl.java#L25-L31)、[HazeEats PayStatusListener L16–20](https://github.com/Hazehacker/HazeEats/blob/6e58d4e36a5f5bc793f74fd528df3bc084286ba2/backend/order-service/src/main/java/top/hazenix/order/listener/PayStatusListener.java#L16-L20)。

采用为新增 Spring 类的默认值。HazeEats 仍存在字段注入，说明历史代码并不统一；不据此要求清扫整个旧项目。也不机械继承旧提交中异常日志级别或拼写错误。

### 2. 业务用例直接组织，接口按业务命名：中等证据

- [博客新增和修改文章 L135–217](https://github.com/Hazehacker/blog.hazenix.top/blob/97d978451e4e13562cd927734d44270aa666f24e/blog-hazenix-top/blog-server/src/main/java/top/hazenix/service/impl/ArticleServiceImpl.java#L135-L217)：Service 组织文章及标签关联写入。
- [HazeEats saveWithFlavor L41–74](https://github.com/Hazehacker/HazeEats/blob/6e58d4e36a5f5bc793f74fd528df3bc084286ba2/backend/dish-service/src/main/java/top/hazenix/dish/service/impl/DishServiceImpl.java#L41-L74)：菜品写入后回填主键，再批量写入口味。
- [Auris addTrackToPlaylists L124–189](https://github.com/Hazehacker/Auris/blob/5eec9f423086c2151259dfbcdcdf16e756d5be0d/backend/auris-server/src/main/java/top/hazenix/auris/service/impl/PlaylistServiceImpl.java#L124-L189)：校验、查询、逐步组装、批量写入；对应 [d4769d4 独立功能提交](https://github.com/Hazehacker/Auris/commit/d4769d422c6e00f811b04e95c5e120fed7568992) 涉及 DTO、Controller、Mapper、Service 及 SQL。

提炼的是“能顺着方法读懂业务”的形态。未证明用户反对所有泛型或设计模式，也不把逐条查询、重复查询当成偏好。

### 3. 按用途组装输出，显式维护关系：中等证据

- [博客 getPopularArticles L526–543](https://github.com/Hazehacker/blog.hazenix.top/blob/97d978451e4e13562cd927734d44270aa666f24e/blog-hazenix-top/blog-server/src/main/java/top/hazenix/service/impl/ArticleServiceImpl.java#L526-L543)：选择 `ArticleShortVO`，注释说明排行榜不需要 category。
- [HazeEats conditionalSearch L359–385](https://github.com/Hazehacker/HazeEats/blob/6e58d4e36a5f5bc793f74fd528df3bc084286ba2/backend/order-service/src/main/java/top/hazenix/order/service/impl/OrderServiceImpl.java#L359-L385)：因接口需要 `orderDishes`，组装 `OrderVO`。
- [Auris AddTrackToPlaylistsDTO L13–23](https://github.com/Hazehacker/Auris/blob/5eec9f423086c2151259dfbcdcdf16e756d5be0d/backend/auris-pojo/src/main/java/top/hazenix/auris/dto/AddTrackToPlaylistsDTO.java#L13-L23)：一次业务动作有明确的输入形态。

因此保留 DTO / Entity / VO 的责任区分与现有响应契约，不要求每个接口都创建全套对象。Mapper 可以在现有查询模式下直接返回投影 VO；不为符合教条增加一层无价值转换。

### 4. 注释解释业务限制，日志辅助定位：理念明确、代码支持

- [博客 deleteCategory L75–83](https://github.com/Hazehacker/blog.hazenix.top/blob/97d978451e4e13562cd927734d44270aa666f24e/blog-hazenix-top/blog-server/src/main/java/top/hazenix/service/impl/CategoryServiceImpl.java#L75-L83)：解释有关联文章的分类不能删除。
- [HazeEats 取消订单 L300–315](https://github.com/Hazehacker/HazeEats/blob/6e58d4e36a5f5bc793f74fd528df3bc084286ba2/backend/order-service/src/main/java/top/hazenix/order/service/impl/OrderServiceImpl.java#L300-L315)：解释即使前端没有按钮，后端仍需检查状态。
- [HazeEats LogServiceAspect L35–55](https://github.com/Hazehacker/HazeEats/blob/6e58d4e36a5f5bc793f74fd528df3bc084286ba2/backend/eats-server/src/main/java/top/hazenix/aspect/LogServiceAspect.java#L35-L55)：存在可开关的定位日志。

“多加日志与注释”的目的，是降低验证和排障成本。采用中文业务说明、必要标识和失败原因；不将 AOP 全量记录、请求个人信息或重复注释变成默认要求。

### 5. 接口边界比固定架构标签更重要：理念明确、样本有限

[博客 Controller L53–65](https://github.com/Hazehacker/blog.hazenix.top/blob/97d978451e4e13562cd927734d44270aa666f24e/blog-hazenix-top/blog-server/src/main/java/top/hazenix/controller/admin/ArticleController.java#L53-L65) 展示薄入口；[HazeEats CartClient L13–38](https://github.com/Hazehacker/HazeEats/blob/6e58d4e36a5f5bc793f74fd528df3bc084286ba2/backend/eats-api/src/main/java/top/hazenix/api/client/CartClient.java#L13-L38) 展示跨服务协议；[PayStatusListener L30–34](https://github.com/Hazehacker/HazeEats/blob/6e58d4e36a5f5bc793f74fd528df3bc084286ba2/backend/order-service/src/main/java/top/hazenix/order/listener/PayStatusListener.java#L30-L34) 将业务动作交给 Service。

但这些常见结构本身不足以证明个人原创。采用用户笔记明确提出的“先归属、后拆分”，不宣称微服务、DDD、事件驱动或某个框架是所有项目的个人默认。

## 没有固化的内容

| 观察 | 处理 |
| --- | --- |
| 博客 [ee052744](https://github.com/Hazehacker/blog.hazenix.top/commit/ee052744b871607985a4ce7a0f63759f691e1d5b) 明确写了让 AI 修改前端；Auris [7e246cd](https://github.com/Hazehacker/Auris/commit/7e246cd96d5924e85a23d198afefd0e20d1458f1) 的前端框架提交署名与主要后端提交署名不同，未完成逐行归属确认 | 不把前端组件组织和审美归为用户手写偏好；保留未来确认空间 |
| HazeEats 同时有单体 `eats-server` 和拆分服务，支付代码还保留“苍穹外卖订单”文本 | 存在模板或课程来源迹象，不能将重复模块视为多份独立证据；本次未比对原课程代码，不能判定每段来源 |
| 三项目都出现 Result、DTO、AutoFill 等常见模式 | 只当作兼容现有项目的约定，不宣称独创或永久架构选择 |
| Auris [addTrack L134–150](https://github.com/Hazehacker/Auris/blob/5eec9f423086c2151259dfbcdcdf16e756d5be0d/backend/auris-server/src/main/java/top/hazenix/auris/service/impl/TrackServiceImpl.java#L134-L150) 在 finally 中写数据并 return | 该结构可能掩盖上传异常；明确不模仿。本次是静态判断，未执行该路径 |
| Auris [removeTrack L166–176](https://github.com/Hazehacker/Auris/blob/5eec9f423086c2151259dfbcdcdf16e756d5be0d/backend/auris-server/src/main/java/top/hazenix/auris/service/impl/TrackServiceImpl.java#L166-L176) 删除关联后还删除歌曲实体，而项目也支持多歌单关联 | 提醒核对资源归属和删除语义；未在本次替用户裁定或修改产品行为 |
| HazeEats [submitOrder L79–165](https://github.com/Hazehacker/HazeEats/blob/6e58d4e36a5f5bc793f74fd528df3bc084286ba2/backend/order-service/src/main/java/top/hazenix/order/service/impl/OrderServiceImpl.java#L79-L165) 在本地事务中调用远端购物车及 MQ | 不能由注解推导跨系统原子性；作为正确性边界，而非用户已经实现可靠分布式事务的证据 |
| 历史代码中的超长 Service、逐条补数据、废弃注释、拼写差异、格式差异 | 不复制为风格；沿当前项目约定，按任务所需改善 |

## 更新依据

新证据应写清：适用场景、可验证的原始文件或提交、实际选择、替代方案为何不合适、适用边界。用户现在的明确取舍可以覆盖历史推断；不要让旧项目成为限制其演进的模板。
