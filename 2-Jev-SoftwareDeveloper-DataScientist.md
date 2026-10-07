# Hands-on 2: Jev สำหรับงาน IT, Developer และ Data Science

แบบฝึกหัดคัดลอก JSON ลง TypeSafe Playground แล้วทดลองรันกับโจทย์งานเทคนิค

- รวม 6 ตัวอย่าง ครอบคลุม 3 Modes: **Noul, Choice, Score**
- Mode ละ 2 ตัวอย่าง: ข้อแรก Single Question; ข้อสอง Multi Questions บน State เดียว
- ใช้ข้อมูลสมมติสำหรับฝึก ไม่ใช่คำแนะนำให้ deploy, ปิดระบบ, เปลี่ยนสิทธิ์ หรือแก้ข้อมูลจริงโดยอัตโนมัติ

## วิธีลอง

1. เปิด [TypeSafe Playground](https://console.typesafe.ai/)
2. วาง JSON ในส่วน STATE ลงช่อง State และ JSON ในส่วน QUESTIONS ลงช่อง Questions / Prompts
3. กด Run / Evaluate แล้วพิจารณาคำตอบ, probabilities และ confidence ตามชนิดของคำถาม
4. ทดลองแก้ข้อความใน State แล้วรัน Questions เดิมซ้ำ

หากใช้ request editor ให้แยก object ของข้อมูลเป็น `state` และคำถามเป็น `questions` ตามหน้าจอ/เอกสารของ Playground อย่ารวม Questions เข้า State

---

# Mode 1 — Noul

## 1.1 Single — มีการระบุ test failure หรือไม่?

### STATE

```json
{
  "ci_log": "Build passed. Unit tests passed. Integration test test_payment_timeout failed: expected 200, got 504."
}
```

### QUESTIONS

```json
{
  "has_test_failure": {
    "type": "noul",
    "instructions": "Does the CI log report at least one test failure?"
  }
}
```

**ดูผล:** `answers.has_test_failure.noul` ค่าใกล้ 1 คือข้อความมี test failure; ค่าใกล้ 0 คือไม่พบตามคำถามนี้ ตัวอย่างนี้จัดประเภทจาก log เท่านั้น ไม่ได้วิเคราะห์สาเหตุของ 504

## 1.2 Multi — ตรวจคุณภาพข้อมูลสำหรับงานวิเคราะห์

### STATE

```json
{
  "dataset_note": "The table has 12,400 rows. The customer_id column is missing for 3% of rows. Event timestamps use UTC. The note does not describe how duplicate events are handled."
}
```

### QUESTIONS

```json
{
  "mentions_missing_values": {
    "type": "noul",
    "instructions": "Does the note explicitly mention missing values in a column?"
  },
  "states_timezone": {
    "type": "noul",
    "instructions": "Does the note specify the timezone used for event timestamps?"
  },
  "describes_duplicate_handling": {
    "type": "noul",
    "instructions": "Does the note explain how duplicate events are handled?"
  }
}
```

**ดูผล:** คำตอบ 3 ข้อเป็นการตรวจข้อความแยกกัน ไม่ได้พิสูจน์ว่า dataset ไม่มีปัญหาอื่น ลองเพิ่มประโยคเรื่อง duplicate handling แล้วรันซ้ำ [1]

---

# Mode 2 — Choice

## 2.1 Single — จัดประเภทสาเหตุหลักจาก incident note

### STATE

```json
{
  "incident": "The API started returning 503 after the database connection pool reached its configured limit. Restarting the API temporarily restored service."
}
```

### QUESTIONS

```json
{
  "primary_area": {
    "type": "choice",
    "instructions": "Which area is most directly implicated by the incident note?",
    "criteria": {
      "database_capacity": "Database connections or database capacity are the primary issue",
      "application_bug": "Application logic or an application defect is the primary issue",
      "network": "Network connectivity or routing is the primary issue",
      "unknown": "The note does not support choosing one of the listed areas"
    }
  }
}
```

**ดูผล:** `answers.primary_area.choice` เป็นตัวเลือกหลัก; ดู `probabilities` และ `confidence` ด้วย โดยเฉพาะเมื่อหลายสาเหตุยังเป็นไปได้ ผลนี้เป็นการจัดประเภทโน้ต ไม่ใช่ root-cause analysis

## 2.2 Multi — เลือกประเภทงานจาก pull request summary

### STATE

```json
{
  "pull_request": "Add an index on events.created_at and update the dashboard query to filter by time range. Includes a migration and query-plan check."
}
```

### QUESTIONS

```json
{
  "change_area": {
    "type": "choice",
    "instructions": "What is the main engineering area of this pull request?",
    "criteria": {
      "database": "Schema, index, migration, or database-query change",
      "frontend": "User interface or browser-side change",
      "infrastructure": "Deployment, networking, or runtime-platform change",
      "documentation": "Documentation-only change",
      "other": "The summary does not fit the listed areas"
    }
  },
  "risk_kind": {
    "type": "choice",
    "instructions": "What kind of review risk is most relevant based on the summary?",
    "criteria": {
      "performance": "Query latency, resource use, or performance regression",
      "data_migration": "Migration correctness, locking, or compatibility of stored data",
      "user_interface": "Visual or interaction regression in the user interface",
      "no_clear_risk": "The summary does not indicate a specific risk area"
    }
  },
  "change_scope": {
    "type": "choice",
    "instructions": "How broad is the described change?",
    "criteria": {
      "narrow": "A localized change affecting one small component or query",
      "moderate": "Several related parts of one service or workflow",
      "broad": "Multiple services or unrelated areas are changed"
    }
  }
}
```

**ดูผล:** แยกดู `change_area`, `risk_kind` และ `change_scope` เพราะเป็นคนละการตัดสินใจ แม้ใช้ summary เดียวกัน ตัวเลือกถูกกำหนดไว้ใน `criteria`; หากตัวเลือกสับสนกัน ให้ปรับคำอธิบายให้แยกขอบเขตชัดขึ้น [2]

---

# Mode 3 — Score

## 3.1 Single — ให้คะแนนความรุนแรงของ bug report

### STATE

```json
{
  "bug_report": "Export fails in Safari for large reports. Export still works in Chrome. No data loss has been observed."
}
```

### QUESTIONS

```json
{
  "bug_severity": {
    "type": "score",
    "instructions": "How severe is the user impact described in this bug report?",
    "criteria": [
      "Cosmetic: no meaningful functionality is affected",
      "Limited: a feature is degraded, but a practical workaround is available",
      "Blocking: a key function is unavailable and no workaround is stated"
    ]
  }
}
```

**ดูผล:** อ่าน `answers.bug_severity.score`, `probabilities`, `legend` และ `confidence` ร่วมกัน ผลคือการให้คะแนนตามข้อความที่ให้มา ไม่ได้ยืนยันผลกระทบกับผู้ใช้ทุกคน [3]

## 3.2 Multi — ให้คะแนนความพร้อมของ experiment summary

### STATE

```json
{
  "experiment": "Model v2 was evaluated against the existing baseline on a held-out set of 8,000 records. The report says the metric improved, but does not name the metric, provide a value, or describe the split strategy."
}
```

### QUESTIONS

```json
{
  "method_detail": {
    "type": "score",
    "instructions": "How clearly does the summary describe the evaluation method?",
    "criteria": [
      "Low: no evaluation method or dataset is described",
      "Partial: an evaluation set or comparison is mentioned, but key method details are missing",
      "Clear: the evaluation set, comparison, and key method details are stated"
    ]
  },
  "result_reproducibility": {
    "type": "score",
    "instructions": "How reproducible are the reported results from the information in the summary?",
    "criteria": [
      "Low: no metric or usable result is reported",
      "Partial: a result or metric is mentioned, but important values or setup details are missing",
      "High: metrics, values, data split, and enough setup detail are provided to repeat the comparison"
    ]
  },
  "claim_specificity": {
    "type": "score",
    "instructions": "How specific is the claim about model performance?",
    "criteria": [
      "Low: only a vague claim such as 'better' is made",
      "Partial: a direction or metric is named, but quantitative evidence is incomplete",
      "High: the metric, measured values, comparison, and evaluation context are stated"
    ]
  }
}
```

**ดูผล:** คะแนนทั้งสามวัดคนละมิติ—วิธีประเมิน, การทำซ้ำผล และความเฉพาะเจาะจงของ claim—อย่ารวมเป็นคะแนนเดียวโดยอัตโนมัติ ลองแก้ State เพิ่มชื่อ metric, ค่า baseline/model และ split strategy แล้วเปรียบเทียบ distribution; rubric ของ Score ควรมีลำดับและคำอธิบายที่แยกกันได้ [3]

---

## แบบฝึกต่อยอด

- แก้ CI log ให้มีแต่ build failure แล้วรัน Noul เดิม สังเกตว่าคำถามเกี่ยวกับ test failure ตอบอย่างไร
- เพิ่มหลักฐานสองทางใน incident note แล้วดูว่า Choice probabilities เปลี่ยนหรือไม่
- เพิ่ม metric และวิธีแบ่งข้อมูลใน experiment summary แล้วเทียบ Score ก่อนและหลัง

ใช้ผลลัพธ์เป็นตัวช่วยทดลองและตรวจสอบโดยคน ไม่ใช้แทน code review, incident investigation, data validation หรือการตัดสินใจ deploy จริง

## Sources

[1] https://docs.typesafe.ai/primitives/noul — Noul: yes/no questions, criteria และการแยกคำถามหลายข้อ
[2] https://docs.typesafe.ai/primitives/choice — Choice: ตัวเลือกที่กำหนด, probabilities และ confidence
[3] https://docs.typesafe.ai/primitives/score — Score: ระดับเรียงลำดับ, score, probabilities และ confidence
