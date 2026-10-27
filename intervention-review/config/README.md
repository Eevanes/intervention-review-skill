# 配置说明

默认配置为 [`default.json`](default.json)。项目可在当前工作目录放置 `.intervention-review.json` 覆盖其中的字段；[`example.json`](example.json) 是虚构示例，可复制后根据实际情况修改。配置为 UTF-8 JSON，不支持注释。

Agent 先读取默认配置，再递归合并项目配置：对象按键覆盖，数组整体替换，未指定字段保留默认值。只允许下列字段；拒绝未知字段、类型错误和不支持的值，并指出位置，不默默忽略或当作授权。`version` 必须是整数 `1`。项目配置缺失不视为错误；若文件存在但无效，停止使用无效配置进行自动决策，提示修复。当前请求的明确要求优先，但不能绕过授权及 Skill 的安全边界。

| 字段 | 类型与有效值 | 含义 |
| --- | --- | --- |
| `version` | 整数，`1` | 配置格式版本。 |
| `project.name` | 字符串，可为空 | 项目名称。 |
| `project.goal` | 字符串，可为空 | 判断偏差时使用的项目目标；空值表示待提供。 |
| `project.timezone` | 有效 IANA 时区字符串 | 解释明确时间和复查时间；默认 `Asia/Shanghai`。 |
| `sources` | 对象数组 | 资料来源清单；空数组表示配置中没有预设来源。 |
| `sources[].name` | 非空字符串 | 来源名称。 |
| `sources[].location` | 非空字符串 | 文件路径或来源标识；相对路径以项目工作目录为基准。 |
| `sources[].enabled` | 布尔值 | `false` 不主动读取；`true` 也不等于授权。 |
| `review.default_recheck_hours` | 1–720 的整数 | 需要复查但未给出具体时间时的建议间隔，以可知的当前时间为基准；不创建定时任务。 |
| `review.max_next_steps` | 1–5 的整数 | 每个事项建议的下一步数量上限。 |
| `notifications.decisions` | 以下值组成的无重复数组：`intervene_now`、`offer_support`、`gather_evidence` | 主动提醒候选；不影响用户主动询问的回答。 |
| `notifications.gather_evidence_requires_time_cost` | 布尔值 | 为 `true` 时，补充证据仅在明确等待代价且需负责人行动时进入主动提醒候选。 |
| `notifications.repeat_only_on_actionable_change` | 布尔值 | 为 `true` 时，仅在变化影响下一步时重复提醒；宿主须提供可用的历史记录才能执行去重。 |
| `delivery.mode` | 字符串，固定 `draft_only` | 生成审核稿；对外发送和状态修改须明确授权并由宿主执行。 |

`sources` 每个对象必须包含且只包含 `name`、`location`、`enabled`。其他对象可以只提供要覆盖的字段，但合并后的配置必须完整有效。不要在配置中放密钥、token、个人隐私数据或真实聊天内容。

开启来源只表示它可以被考虑；它不会建立连接或授予读取权限。`notifications` 是筛选规则，不会注册定时任务或实际发送。无法读取来源、判断时区或取得当前时间时，保留未知状态，不据此认定项目异常。
