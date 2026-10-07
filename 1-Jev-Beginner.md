# Jev Hands-on สำหรับมือใหม่: 3 Modes × 2 ตัวอย่าง

ลองใช้ Jev บน TypeSafe Playground ด้วย JSON สั้น ๆ แยก **State** (ข้อมูล) กับ **Questions** (สิ่งที่ถาม) ให้ชัด

- มี 3 Modes: **Noul, Choice, Score**
- Mode ละ 2 ตัวอย่าง: ข้อ 1 ใช้ State เดียว + Question เดียว; ข้อ 2 ใช้ State เดียว + หลาย Questions
- เรียงจากง่ายไปยากขึ้น
- ใช้ Playground ได้โดยไม่ต้องเขียนโค้ด [1]

> ชื่อที่ถูกต้องคือ Jev (โมเดล) และ Noul (primitive) ตัวอย่างเป็นข้อมูลฝึกทั่วไป ไม่ใช่ข้อมูล production

## วิธีใช้ Playground

1. เปิด [TypeSafe Playground](https://console.typesafe.ai/)
2. เลือกตัวอย่างด้านล่าง
3. คัดลอก JSON ในกรอบ **STATE** ไปวางในช่อง State
4. คัดลอก JSON ในกรอบ **QUESTIONS** ไปวางในช่อง Questions / Prompts
5. กด Run / Evaluate แล้วดูผลลัพธ์

State คือข้อมูลที่โมเดลประเมิน ส่วน Questions คือคำถามที่ต้องการให้ตอบ สามารถถามหลายข้อกับ State เดียวกันในคำขอเดียวได้ [1][2][3]

> หาก Playground แสดงช่องแยก ให้คัดลอกแต่ละบล็อกลงช่องตามชื่อ หากเป็น request editor ให้ใส่ object ของ State ใน `state` และ object ของ Questions ใน `questions` อย่าวาง Questions เป็นส่วนหนึ่งของ State

## สรุป 3 Modes

| Mode | ใช้เมื่อ | ฟิลด์คำตอบหลัก |
|---|---|---|
| **Noul** | ถามใช่/ไม่ใช่ | `noul`: ความน่าจะเป็นของ “ใช่” ตั้งแต่ 0–1 |
| **Choice** | เลือกหนึ่งข้อจากรายการ | `choice`, `probabilities`, `confidence` |
| **Score** | ให้คะแนนตามระดับที่เรียงลำดับ | `score`, `legend`, `probabilities`, `confidence` |

เลือก mode ตามคำตอบที่โค้ดหรือผู้ใช้ต้องการ: Noul สำหรับ yes/no, Choice สำหรับตัวเลือกที่ไม่มีลำดับ และ Score สำหรับ rubric แบบไล่ระดับ [2][3][4][5]

---

# Mode 1 — Noul

Noul คืนความน่าจะเป็นว่า “ใช่” สำหรับคำถาม yes/no และไม่มี `confidence` แยก [3]

## Noul 1 — Single: ข้อความพูดถึงแมวไหม?

หนึ่ง State กับหนึ่ง Question

### STATE — วางในช่อง State

```json
{
  "text": "A cat is sleeping on the chair."
}
```

### QUESTIONS — วางในช่อง Questions

```json
{
  "mentions_cat": {
    "type": "noul",
    "instructions": "Does the text mention a cat?"
  }
}
```

**ดูผล:** `answers.mentions_cat.noul` ค่าใกล้ 1 หมายถึงมีแนวโน้มว่าใช่ ค่าใกล้ 0 หมายถึงมีแนวโน้มว่าไม่ใช่

## Noul 2 — Multi: ตรวจข้อความและนโยบายใน State เดียว

หนึ่ง State กับหลาย Questions โดยแต่ละ Question ตรวจคนละข้อมูล

### STATE

```json
{
  "ticket": {
    "message": "My order arrived late. What are my options?"
  },
  "policy": "Late deliveries may be eligible for a refund."
}
```

### QUESTIONS

```json
{
  "customer_requested_refund": {
    "type": "noul",
    "instructions": "Does `ticket.message` explicitly request a refund?"
  },
  "policy_mentions_refund": {
    "type": "noul",
    "instructions": "Does `policy` say that a refund may be available?"
  },
  "customer_reports_late_delivery": {
    "type": "noul",
    "instructions": "Does `ticket.message` say that the order arrived late?"
  }
}
```

**ดูผล:** มีคำตอบแยกตาม ID ทั้งสามข้อ ใช้ State เดียวกัน แต่ทุก Question เป็น judgment แยกอิสระ [2] สังเกตว่าถามถึง “ลูกค้าขอ refund” ต่างจาก “policy พูดถึง refund”

---

# Mode 2 — Choice

Choice เลือกหนึ่งตัวเลือกจากรายการที่กำหนด และคืน probabilities กับ confidence ประกอบ [4]

## Choice 1 — Single: ท้องฟ้าเป็นสีอะไรตามข้อความ?

หนึ่ง State กับหนึ่ง Question

### STATE

```json
{
  "observation": "The clear daytime sky appears blue."
}
```

### QUESTIONS

```json
{
  "sky_color": {
    "type": "choice",
    "instructions": "What color does the observation say the sky appears?",
    "criteria": {
      "blue": "The sky is described as blue",
      "green": "The sky is described as green",
      "red": "The sky is described as red"
    }
  }
}
```

**ดูผล:** `answers.sky_color.choice` คือ option ที่เลือก; ดู `probabilities` และ `confidence` เพื่อเห็นความกระจาย/ความชัดของคำตอบ [4]

## Choice 2 — Multi: จำแนกปัญหาและสิ่งที่ลูกค้าต้องการ

หนึ่ง State กับหลาย Questions แบบ Choice

### STATE

```json
{
  "message": "My headphones arrived broken. I would like a replacement."
}
```

### QUESTIONS

```json
{
  "issue_type": {
    "type": "choice",
    "instructions": "What is the main issue described in the message?",
    "criteria": {
      "damaged_item": "The item arrived broken or damaged",
      "shipping_delay": "The delivery is late or has not arrived",
      "billing_problem": "There is an issue with a charge or payment",
      "other": "The issue does not fit the listed options"
    }
  },
  "requested_resolution": {
    "type": "choice",
    "instructions": "What resolution does the customer ask for?",
    "criteria": {
      "replacement": "Send the same item again",
      "refund": "Return the money",
      "repair": "Repair the existing item",
      "information": "The customer asks a question but requests no action"
    }
  },
  "tone": {
    "type": "choice",
    "instructions": "What tone does the customer use?",
    "criteria": {
      "calm": "Neutral or polite wording",
      "frustrated": "Clearly dissatisfied but not strongly angry",
      "angry": "Strongly angry or hostile wording"
    }
  }
}
```

**ดูผล:** เปรียบเทียบ `issue_type.choice`, `requested_resolution.choice` และ `tone.choice` ว่าแต่ละ Question คืนคนละการจัดประเภท แม้ใช้ State เดียวกัน ตัวเลือก `other` ช่วยรองรับกรณีที่ไม่เข้ากลุ่มที่ระบุ [2][4]

---

# Mode 3 — Score

Score ใช้ระดับคะแนนที่เรียงลำดับกัน ต้องอธิบายความหมายของแต่ละระดับให้ชัดเจน [5]

## Score 1 — Single: กาแฟร้อนแค่ไหน?

หนึ่ง State กับหนึ่ง Question

### STATE

```json
{
  "coffee": "The coffee is steaming hot and too hot to drink right now."
}
```

### QUESTIONS

```json
{
  "temperature": {
    "type": "score",
    "instructions": "How hot is the coffee according to the text?",
    "criteria": [
      "0 - Cold: described as cold or chilled",
      "1 - Warm: warm but comfortable to drink",
      "2 - Hot: steaming or too hot to drink immediately"
    ]
  }
}
```

**ดูผล:** ตรวจ `answers.temperature.score`, `legend`, `probabilities` และ `confidence` ไม่ต้องคาดหวังว่า score จะเป็นจำนวนเต็มเสมอ [5]

## Score 2 — Multi: ให้คะแนนรายงานบั๊กสองด้าน

หนึ่ง State กับหลาย Questions แบบ Score แต่ละข้อใช้ rubric ของตัวเอง

### STATE

```json
{
  "bug_report": "The mobile app crashes when I tap Save. It happened three times on Android 14. I do not know how to reproduce it reliably."
}
```

### QUESTIONS

```json
{
  "problem_detail": {
    "type": "score",
    "instructions": "How clearly does the report describe the problem?",
    "criteria": [
      "0 - Vague: only says something is wrong",
      "1 - Partial: names the problem but gives little detail",
      "2 - Clear: describes the failure and when it happens"
    ]
  },
  "reproduction_readiness": {
    "type": "score",
    "instructions": "How ready is the report for an engineer to reproduce the problem?",
    "criteria": [
      "0 - Not ready: no environment or reproduction information",
      "1 - Some information: environment or occurrence details are present, but steps are unclear",
      "2 - Ready: clear reproduction steps and relevant environment are provided"
    ]
  },
  "impact_level": {
    "type": "score",
    "instructions": "How serious is the reported impact on the user?",
    "criteria": [
      "0 - Low: cosmetic issue with no meaningful function impact",
      "1 - Medium: a feature is degraded but a workaround may exist",
      "2 - High: a key function is blocked and no workaround is stated"
    ]
  }
}
```

**ดูผล:** เปรียบเทียบคะแนนด้านรายละเอียด, ความพร้อมในการทำซ้ำ และ impact แยกกัน `score` สรุปตำแหน่งบน rubric; `probabilities` แสดงการกระจายตามระดับ และ `confidence` สรุป distribution [5] อย่าตีความคะแนนนี้ว่าเป็นข้อเท็จจริงที่ยืนยันแล้ว

---

## แบบฝึกสั้น ๆ

สำหรับแต่ละตัวอย่าง Multi ให้แก้เฉพาะข้อความใน State แล้วรัน Questions ชุดเดิมอีกครั้ง:

- **Noul:** เปลี่ยนข้อความลูกค้าเป็น “Can you explain the return policy?” แล้วดูว่า `customer_requested_refund` เปลี่ยนอย่างไร
- **Choice:** เปลี่ยนข้อความเป็น “My package has not arrived. Please refund the shipping fee.” แล้วดูว่าคำตอบสองข้อเปลี่ยนอย่างไร
- **Score:** เพิ่มขั้นตอนทำซ้ำที่ชัดเจนใน bug report แล้วเปรียบเทียบ `reproduction_readiness`

## เช็กลิสต์

- [ ] บท Single แต่ละบทมี State หนึ่งชุดและ Question หนึ่งข้อ
- [ ] บท Multi แต่ละบทมี State หนึ่งชุดและ Questions หลายข้อ
- [ ] Noul คืนค่า yes probability; Choice เลือกจาก options; Score ประเมินตามระดับ
- [ ] อ่าน probabilities/confidence ประกอบตามชนิดคำตอบ
- [ ] จดผลที่ Playground แสดงจริง เพราะผลอาจแตกต่างตามอินพุตและการประเมิน [1]

## เอกสารทางการ

- [1] [TypeSafe Quick start](https://docs.typesafe.ai/introduction/quickstart)
- [2] [Primitives (Questions)](https://docs.typesafe.ai/primitives.md)
- [3] [Noul](https://docs.typesafe.ai/primitives/noul.md)
- [4] [Choice](https://docs.typesafe.ai/primitives/choice.md)
- [5] [Score](https://docs.typesafe.ai/primitives/score.md)

## Sources

[1] https://docs.typesafe.ai/introduction/quickstart — Quick start
[2] https://docs.typesafe.ai/primitives.md — Primitives
[3] https://docs.typesafe.ai/primitives/noul.md — Noul
[4] https://docs.typesafe.ai/primitives/choice.md — Choice
[5] https://docs.typesafe.ai/primitives/score.md — Score
