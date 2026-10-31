# app.json：应用清单

每个应用的根目录有一份 `app.json`，告诉商城、Yay 和平台：它叫什么、怎么打开、能用哪些云能力。版本号跟着固件一起改。我们审核时用 Tikpal 的正式契约逐字段核对；不认识的字段一律不收。`./kit check` 核其中主要的几条，`records` 一段照契约逐条核。

## 最小的一份（模板）

```json
{
  "slug": "my-timer",
  "kind": "extension",
  "version": "0.1.0",
  "name": {"zh": "计时器", "en": "Timer"},
  "boards": ["tikpal-r1"],
  "layout": "yaykit-v1",
  "minYay": "0.1.10",
  "apiLevel": 3,
  "capabilities": {},
  "launch": {
    "summary": {"zh": "计时器：说多久就计多久，到点提醒。", "en": "Timer: counts down as long as you say."},
    "examples": {"zh": ["打开计时器", "帮我计时十分钟"], "en": ["Open the timer", "Set a ten minute timer"]}
  },
  "aliases": {"zh": ["定时器"], "en": ["Countdown"]},
  "scenes": ["tools"],
  "audience": "all",
  "pricing": {"kind": "free"}
}
```

## 字段

| 字段 | 必填 | 说明 |
|---|---|---|
| `slug` | 是 | 小写字母、数字、连字符，字母开头，2–24 个字符（项目名 `yaykit_<slug>` 最长 31 个字符，圆屏从它读回 slug）。和项目名对得上。定了就不改。`yay`、`launcher`、`recovery`、`template`、`system` 是保留的 |
| `kind` | 是 | 扩展应用都写 `extension` |
| `version` | 是 | `主.次.修`，和 `CMakeLists.txt` 的 `PROJECT_VER` 一样。每次提交都比上一次大 |
| `name` | 是 | 名字，`zh` 和 `en` 必填，`ja`、`es-419` 可选；每种不超过 24 个字符 |
| `boards` | 是 | 写 `["tikpal-r1"]` |
| `layout` | 是 | 写 `"yaykit-v1"` |
| `minYay` | 建议 | 最低要哪一版 Yay。至少写开发包对应的那一版（`./kit version` 的 `yay`） |
| `apiLevel` | 建议 | 开发包的接口级别（`./kit version` 的 `apiLevel`） |
| `capabilities` | 是 | 要用的云能力和限额，最小权限：设备令牌只许这些。不要云能力就写 `{}`。要什么，立项时告诉我们，我们帮你写这一段 |
| `launch` | 要让 Yay 用说话打开就要 | `summary`：一句话说它能做什么（≤ 300 字）。`examples`：用户会说的话，每种语言最多 12 句、每句 ≤ 80 字，两类都要有：点了名的（「打开计时器」）和没点名、只说想做什么的（「帮我计时十分钟」），Yay 找应用主要靠后一类。`params`：Yay 能带给你的启动参数（见下） |
| `aliases` | 是 | 别的叫法，每种语言最多 8 个；没有就写空数组 `{"zh": [], "en": []}` |
| `scenes` | 是 | 在商城哪一类里，1–3 个：`party` 聚会、`learning` 学习、`relax` 放松、`language` 语言、`focus` 专注、`tools` 工具 |
| `audience` | 是 | `kids`、`adults` 或 `all` |
| `pricing` | 是 | 现在只有 `{"kind": "free"}` |
| `settings` 等 | 否 | 可以在手机 App 里调的设置（`settings`、`settingOptions`、`settingLabels`、`settingRules`）。要的话立项时说，我们一起定 |
| `records` | 用了 `yk_app_record()` 就要 | 记录进「动态」的格式，和 `capabilities.records` 一起写，见下面的[记录 `records`](#记录-records)。立项时一起定 |
| `ephemeral` | 否 | `true`：什么都不留（不写记录、不存数据和文件），比如翻译 |

### 启动参数 `launch.params`

Yay 打开你时能带的参数，键名是参数名，值说明它：

```json
"params": {
  "minutes": {"type": "number", "description": "要计时多少分钟"},
  "sound": {"type": "string", "description": "到点用什么声音", "enum": ["bell", "chime"]}
}
```

- `type`：`string`、`number` 或 `boolean`；`description`：给 Yay 的模型看，它按这个决定传什么；`enum`：只能是这几个值。
- 只写固件真读的参数。读法见[应用开发指南](app-guide.md)第 5 节。

## 记录 `records`

固件用 `yk_app_record(kind, data)` 写记录（[应用开发指南](app-guide.md)第 5 节），平台按这一段收进「动态」。要写两处，种类要对上：

- `capabilities.records.kinds`：圆屏能写哪几种（设备令牌只许这几种）；
- `records.kinds`：每一种的 `kind` 和 `schema`（`data` 长什么样）。

两处是同样几种，多一种少一种都不过。用户在 Yay 问过（`ask`）、答应了以后，平台才收这个应用的记录。

这两段立项时跟我们一起定：记哪几种、每种记什么、多久整理一次、「动态」用哪个模板。写全了是这样（计时器每计完一次记一条 `{"minutes": 10, "completed": true}`）：

```json
"capabilities": {
  "records": {"kinds": ["mytimer.session"]}
},
"records": {
  "ask": {
    "zh": "计时器计过的时间，要我记下来吗？只记哪天、计了几分钟、有没有计完，不录音；每周一在「动态」里整理一次。要的话回我一声。",
    "en": "Shall I keep the time you count with Timer? Only the day, the minutes and whether it finished, no recordings; it is summed up in Feed every Monday. Just reply if you'd like that."
  },
  "kinds": [
    {
      "kind": "mytimer.session",
      "schema": {
        "type": "object",
        "properties": {
          "minutes": {"type": "number", "minimum": 0, "maximum": 600},
          "completed": {"type": "boolean"},
          "sound": {"type": "string", "enum": ["bell", "chime"]}
        },
        "required": ["minutes", "completed"],
        "additionalProperties": false
      }
    }
  ],
  "milestones": [
    {"type": "streak", "days": 7},
    {"type": "total", "kind": "mytimer.session", "field": "minutes", "reach": 1000}
  ],
  "feed": {"cadence": "weekly", "feedDays": 60},
  "pages": {"template": "mytimer@1"}
}
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `ask` | 是 | Yay 问用户要不要记的那句话：记什么、不记什么、多久整理一次。`zh`、`en` 必填，`ja`、`es-419` 可选，每种不超过 400 个字符 |
| `kinds` | 是 | 1–16 种记录。每种有 `kind`（和 `yk_app_record()` 的第一个参数一字不差：小写字母、数字和点，字母开头，3–41 个字符）和 `schema`（见下）；可选 `noMemory: true`（Yay 的记忆不读这种记录），`collect`（收进集合，见下） |
| `collections` | 否 | 最多 8 个集合，比如「查过的字」：`name`（写法同字段名）、`title` 和 `unit`（`zh`、`en` 必填，每种不超过 20 个字符）、`every`（每满多少个算一个里程碑） |
| `milestones` | 否 | 最多 16 个：`{"type": "streak", "days": 7}` 连着几天有记录（2–3650）；`{"type": "total", "kind": …, "field": …, "reach": …}` 这种记录的这个字段加起来到多少；`{"type": "first", "kind": …, "by": …}` 这个字段每个值第一次出现；`{"type": "first", "kind": …, "name": …, "when": {…}}` 第一次出现 `when` 里这些值。`by` 和 `name` 加 `when` 只写一种 |
| `feed` | 是 | 「动态」多久整理一次：`cadence` 是 `daily`（每天早上）、`weekly`（每周一）或 `session`（用完一次）；`feedDays` 读最近几天的记录，1–366 |
| `pages` | 是 | `{"template": "名字@版本"}`：「动态」和应用页用哪个模板画。模板在平台上，不在你的仓库里，名字立项时我们给 |

### 记录的 `schema`

`data` 长什么样，用 JSON Schema（2020-12）的一小部分写，只能用这些词：`type`、`description`（不超过 200 个字符）、`properties`、`required`、`additionalProperties`、`items`、`minItems`、`maxItems`、`minLength`、`maxLength`、`minimum`、`maximum`、`enum`、`anyOf`、`default`。`type` 是 `object`、`array`、`string`、`number`、`integer`、`boolean`、`null` 之一。

- 最外层是 `"type": "object"`：一条记录是一个对象。
- 每个 `"type": "object"` 都列出 `properties`，并写 `"additionalProperties": false`（只能写 `false`）：没列的字段不收。
- 每个 `"type": "array"` 都写 `items`。
- `required` 里的字段要在 `properties` 里。
- 字段名小写字母开头，只用字母、数字和下划线，1–41 个字符：`bestCombo`、`new_words` 可以，`best-combo` 不行。
- 可以为空的写 `{"anyOf": [{"type": "integer"}, {"type": "null"}]}`。
- `data` 本身不超过 512 字节（应用开发指南第 5 节）。

### 收进集合 `collect`

```json
"collect": [{"collection": "words", "each": "newWords", "key": "word", "keyAs": "lower", "fields": ["word", "meaning"], "with": {"lang": "language"}}]
```

把记录（写了 `each` 就是记录里这个列表的每一项）按字段 `key` 收进集合 `collection`（要在 `collections` 里）。`keyAs`：`lower` 转小写；`phrase` 转小写、只留字母和数字，什么都不剩的不收。`fields`：每项留哪些字段，不写就全留。`with`：再从记录本身带上几个字段，`{"lang": "language"}` 是每项多一个 `lang`，取记录的 `language`。

### `./kit check` 怎么核

- `records` 和 `capabilities.records` 照 Tikpal 的契约逐条核，不过就不能提交。
- 固件里 `yk_app_record("…")` 写的种类，`records` 里没有的，会打一行「提示」（不算不过）：服务端不收这种记录。模板（`./kit new`）在 `on_leave` 里写了一条 `<简称>.count`，没有 `records`：要留进「动态」就照上面补上，不要就删掉那一句。

## 常见的错

- `version` 忘了跟 `PROJECT_VER` 一起改。
- `examples` 只写了「打开 xxx」：Yay 只有在用户点名时才找得到你。
- `capabilities` 里写了没用到的能力：审核会请你删掉。
- 写了 `launch.params`，固件却不读：用户说了参数也没用。
- `records` 写成 `{"<种类>": {"fields": {…}}}`：不是这个写法，照上面「记录 `records`」的例子写。
- `capabilities.records.kinds` 和 `records.kinds` 没对上，或者 `kind` 和固件里 `yk_app_record()` 写的差一个字。
