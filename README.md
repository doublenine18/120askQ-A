# 120ask Q&A 数据集

120ask（寻医问药网）问答数据，用于「张医生风格」医生回复的生成与评估。

## 文件清单

| 文件 | 大小 | 内容 |
|---|---|---|
| `120ask_ai_replies.jsonl` | 46.83 MB | 模型生成的医生回复，32,601 条 |
| `120ask_eligible_replies.csv` | 26.88 MB | 120ask 站内真实医生回复，27,003 条 |

两个文件的 `question_id` 集合**完全一致**，均为 10,867 个问题。

---

## 1. `120ask_ai_replies.jsonl`

JSON Lines，每行一条生成记录，共 **32,601** 行，覆盖 **10,867** 个问题。

每个问题在 3 个采样温度下各生成 1 条，因此 `temperature` 为 `0.0` / `1.0` / `2.0` 各 10,867 条；每个 `question_id` 的温度集合恒为 `{0.0, 1.0, 2.0}`。

本文件以 `120ask_eligible_replies.csv` 的 `question_id` 为准，问题集合与之一致。

### 字段

| 字段 | 类型 | 取值 |
|---|---|---|
| `status` | string | 恒为 `"ok"` |
| `question_id` | string | 问题 ID，与 CSV 的 `question_id` 对应 |
| `temperature` | float | `0.0` / `1.0` / `2.0` |
| `repeat_index` | int | 恒为 `1` |
| `model` | string | 恒为 `"deepseek-v4-flash"` |
| `base_url` | string | 恒为 `"https://api.deepseek.com"` |
| `target_chars` | int | 恒为 `180` |
| `generated_reply` | string | 模型生成的医生回复 |
| `prompt_version` | string | 恒为 `"120ask_zhang_style_v1"` |
| `system_prompt` | string | 恒为 `"You are a helpful assistant"` |
| `user_prompt` | string | 完整提示词，见下方模板 |
| `generated_at` | string | ISO 8601 UTC 时间戳 |

### 提示词模板

32,601 条的 `user_prompt` **全部精确匹配**同一模板（逐条校验，命中率 100.0000%），变量槽位无空值：

```
请根据以下患者在在线健康问答平台提出的问题，生成一条医生回复，长度约180个中文字符。

患者问题标题：{标题}
科室：{科室}
患者性别：{性别}
患者年龄：{年龄}
患者问题内容：{内容}

医生回复：
```

模板中的「约180个中文字符」与字段 `target_chars: 180` 一致。

### 注意事项

- **每个 `question_id` 的 3 条记录 `repeat_index` 都为 `1`**，只能靠 `temperature` 区分三次采样；按 `(question_id, repeat_index)` 去重会把 32,601 条压成 10,867 条。
- 数据中的脏值（源数据本身，非提示词问题）：`患者性别` 存在 `保密` 取值；`患者年龄` 存在 `50个月` 这类单位混用；`科室` 实际填的是疾病/主题标签（如 `感冒`、`腹泻`、`肾结石`），并非临床科室。
- 回复长度：平均 166.0 字、中位数 166 字（模板要求约 180 字），最短 65 字、最长 357 字。

---

## 2. `120ask_eligible_replies.csv`

UTF-8（含 BOM），首行为表头，共 **27,003** 行数据，覆盖 **10,867** 个问题。每个问题有多条医生回复，`is_accepted` 为 `1` 的有 10,867 条（每个问题恰好 1 条被采纳）。

### 字段

`question_id`, `reply_id`, `question_url`, `reply_order`, `is_accepted`, `doctor_key`, `doctor_profile_url`, `doctor_name`, `doctor_title`, `hospital`, `specialty`, `help_count`, `praise_count`, `reply_time`, `reply_time_text`, `response_time_seconds`, `reply_text`, `answer_length`, `reply_has_image`, `reply_image_count`, `reply_doctor_info_text`, `profile_doctor_info_text`

### 与 jsonl 的关系

- 本 CSV 是 `question_id` 的权威来源；`120ask_ai_replies.jsonl` 的问题集合与之一致（10,867 个），无多无少。
