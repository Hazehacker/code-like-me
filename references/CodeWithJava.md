# Java / Spring 的默认写法

下面是代码偏好，不代表新项目必须采用 Spring

- **依赖显式。** 新 Spring 类默认构造器注入；项目已有 Lombok 时用 `@RequiredArgsConstructor` 与 `private final`。这是博客主动规范化提交体现的选择，优先于旧模板中的 `@Autowired` 字段注入。仅给旧类增加业务方法时，不顺带批量改造其注入方式；项目明确要求字段注入时遵守项目约定。
- **业务动作就是方法名。** 使用 `addTrackToPlaylists`、`saveWithFlavor` 这样的动词与业务对象组合。变量保留 `playlistIds`、`orderDetailList` 等语义；无需照抄历史拼写、单字母或 `temp` 命名。
- **沿现有层次承接职责。** Controller 做协议适配并调用 Service；Service 组织用例和一致性边界；Mapper 做查询与持久化；跨服务访问经已有 Client。现有 Service 接口继续维护，不为每个小函数新增接口、基类或通用执行框架。
- **业务主线直写。** 简单映射可用 Stream；含分支、状态变化、批量组装时选容易跟踪的循环。先识别真正独立的职责，再决定是否抽 Service、组件或框架；不默认引入 DDD、微服务、Redis、MQ 或泛型 CRUD。