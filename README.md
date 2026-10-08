# 120ask Q&A Dataset

Q&A data from 120ask (寻医问药网, xywy.com).

## File Inventory

| File | Size | Contents |
|---|---|---|
| `120ask_ai_replies.jsonl` | 46.83 MB | Model-generated doctor replies, 32,601 records |
| `120ask_eligible_replies.csv` | 26.88 MB | Real doctor replies from the 120ask site, 27,003 records |

The `question_id` sets in the two files are **exactly identical**, covering 10,867 questions in both cases.

---

## 1. `120ask_ai_replies.jsonl`

JSON Lines; one generated record per line, **32,601** lines in total, covering **10,867** questions.

Each question was generated once at each of 3 sampling temperatures, so `temperature` takes the value `0.0`, `1.0`, or `2.0` for 10,867 records each; the temperature set for every `question_id` is always `{0.0, 1.0, 2.0}`.

This file takes its `question_id` values from `120ask_eligible_replies.csv`, and its question set matches that file.

### Fields

| Field | Type | Values |
|---|---|---|
| `status` | string | Always `"ok"` |
| `question_id` | string | Question ID, corresponding to `question_id` in the CSV |
| `temperature` | float | `0.0` / `1.0` / `2.0` |
| `repeat_index` | int | Always `1` |
| `model` | string | Always `"deepseek-v4-flash"` |
| `base_url` | string | Always `"https://api.deepseek.com"` |
| `target_chars` | int | Always `180` |
| `generated_reply` | string | Model-generated doctor reply |
| `prompt_version` | string | Always `"120ask_zhang_style_v1"` |
| `system_prompt` | string | Always `"You are a helpful assistant"` |
| `user_prompt` | string | Full prompt, see the template below |
| `generated_at` | string | ISO 8601 UTC timestamp |

### Prompt Template

All 32,601 `user_prompt` values **match the same template exactly** (verified record by record, 100.0000% hit rate), with no empty variable slots:

```
请根据以下患者在在线健康问答平台提出的问题，生成一条医生回复，长度约180个中文字符。

患者问题标题：{标题}
科室：{科室}
患者性别：{性别}
患者年龄：{年龄}
患者问题内容：{内容}

医生回复：
```

(The template is stored verbatim in Chinese: "Based on the question the following patient asked on an online health Q&A platform, generate a doctor's reply of about 180 Chinese characters." — followed by slots for question title, department, patient gender, patient age, and question body, then the reply cue.)

The phrase "约180个中文字符" (about 180 Chinese characters) in the template matches the `target_chars: 180` field.

### Notes

- **All 3 records for each `question_id` have `repeat_index` equal to `1`**, so the three samples can only be distinguished by `temperature`; deduplicating on `(question_id, repeat_index)` would collapse 32,601 records down to 10,867.
- Dirty values in the data (these come from the source data, not from the prompt): `患者性别` (patient gender) includes the value `保密` ("confidential"); `患者年龄` (patient age) mixes units, such as `50个月` ("50 months"); and `科室` (department) actually holds disease/topic labels (e.g. `感冒` "cold", `腹泻` "diarrhea", `肾结石` "kidney stone") rather than clinical departments.
- Reply length: mean 166.0 characters, median 166 characters (the template asks for roughly 180), minimum 65 characters, maximum 357 characters.

---

## 2. `120ask_eligible_replies.csv`

UTF-8 (with BOM), with a header row, and **27,003** data rows covering **10,867** questions. Each question has multiple doctor replies, and 10,867 rows have `is_accepted` equal to `1` (exactly one accepted reply per question).

### Fields

`question_id`, `reply_id`, `question_url`, `reply_order`, `is_accepted`, `doctor_key`, `doctor_profile_url`, `doctor_name`, `doctor_title`, `hospital`, `specialty`, `help_count`, `praise_count`, `reply_time`, `reply_time_text`, `response_time_seconds`, `reply_text`, `answer_length`, `reply_has_image`, `reply_image_count`, `reply_doctor_info_text`, `profile_doctor_info_text`

### Relationship to the JSONL File

- This CSV is the authoritative source of `question_id` values; the question set in `120ask_ai_replies.jsonl` matches it exactly (10,867 questions), with none missing and none extra.
