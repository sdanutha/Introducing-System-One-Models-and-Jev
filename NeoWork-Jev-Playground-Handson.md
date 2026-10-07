# Hands-on: ทดลอง Jev กับ NeoWork บน TypeSafe Playground

คู่มือเริ่มเร็วแบบ copy–paste เพื่อเห็นภาพว่า Jev รับ **State** (ข้อมูลเคส) และ **Questions** (คำถามที่ต้องการให้ประเมิน) แยกจากกันอย่างไร

> เวลาโดยประมาณ: 10–20 นาที  
> ตัวอย่างในเอกสารนี้เป็น **ข้อมูลจำลองเพื่อการเรียนรู้เท่านั้น** ไม่ใช่ข้อมูลจาก NeoWork, Oracle, Hive หรือโรงงานจริง และห้ามใช้ผลลัพธ์นี้ปล่อย hold, อนุมัติ release, ปิดเคส หรือสั่งการ production โดยอัตโนมัติ

## 1. เปิด Playground

เปิด [TypeSafe Playground](https://console.typesafe.ai/) แล้วเข้าสู่ระบบ จากนั้นสร้าง request ใหม่ เอกสาร Quick start แนะนำให้ใส่ข้อความหรือข้อมูลเป็น state แล้วเพิ่มคำถาม และสามารถเพิ่มหลายคำถามต่างชนิดใน request เดียวได้ [1][2]

ใน Playground ให้แยกวางดังนี้:

- ช่อง **State**: วาง JSON ในหัวข้อ 2 เท่านั้น
- ส่วน **Questions**: เพิ่มคำถามตาม JSON ในหัวข้อ 3 (ถ้า UI ให้เพิ่มทีละข้อ ให้คัดลอก object แต่ละตัวภายใน `questions` ไปเพิ่มตาม ID)
- เลือกโมเดล `jev-latest` หาก Playground ให้เลือกโมเดล
- กด Run / Evaluate แล้วอ่านคำตอบในหัวข้อ 4

`State` คือข้อมูลที่โมเดลจะพิจารณา ส่วน `Questions` ระบุ judgments ที่ต้องการ คำถามใน request เดียวกันใช้ state เดียวกัน และคำตอบผูกกับ ID ที่เรากำหนด [2][3]

## 2. คัดลอกไปวางใน State

วาง JSON ทั้งก้อนต่อไปนี้ลงในช่อง **State**:

```json
{
  "case": {
    "case_id": "NW-DEMO-001",
    "site": "NRM",
    "title": "Multiple lots on SPC hold after APC error",
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "lot_count": 5,
    "hold_reason": "ENGINEERING",
    "hold_code": "SPC_Shutdown APC_TERMFAIL_AVG_LOW",
    "current_status": "HOLD",
    "repeat_within_24h": true
  },
  "evidence_available": [
    {
      "source": "Oracle hold/release extract",
      "status": "available",
      "note": "Example source label only; no real extract is included."
    },
    {
      "source": "Current WIP hold snapshot",
      "status": "available",
      "note": "Example source label only; no real snapshot is included."
    },
    {
      "source": "Engineer comment",
      "status": "missing",
      "note": "No verified engineer comment is provided in this demo."
    }
  ],
  "known_facts": [
    "The case description reports five lots on hold.",
    "The current status is HOLD.",
    "The description says the event was observed again within 24 hours."
  ],
  "unknowns": [
    "Confirmed root cause is not provided.",
    "Production or shipping impact is not provided.",
    "No release approval or containment verification is provided."
  ],
  "safety_policy": "Do not release, disposition, or change production status based only on this model assessment. A qualified human must verify evidence and approve operational actions."
}
```

### อ่าน State ก่อนรัน

- `case`: รายละเอียดตัวอย่างของเคส
- `evidence_available`: ระบุว่ามี/ไม่มีหลักฐานชนิดใด โดยตัวอย่างนี้ไม่ได้แนบข้อมูลระบบจริง
- `known_facts`: ข้อเท็จจริงที่มีอยู่ในตัวอย่าง
- `unknowns`: ข้อมูลที่ยังไม่มี ห้ามให้โมเดลอนุมานเป็นข้อเท็จจริง
- `safety_policy`: ข้อจำกัดว่าโมเดลไม่มีอำนาจอนุมัติ action

State จะเป็นข้อความธรรมดาหรือ JSON object ก็ได้; การตั้งชื่อ field และจัดข้อมูลที่เกี่ยวข้องไว้เป็นโครงสร้างช่วยให้ตั้งคำถามอ้างถึงข้อมูลได้ชัดเจน [3]

## 3. คัดลอกไปวางใน Questions

หาก Playground มีช่อง JSON สำหรับคำถาม ให้วาง object นี้ในส่วน **Questions** โดยไม่ต้องใส่ State ซ้ำ:

```json
{
  "case_type": {
    "type": "choice",
    "instructions": "Classify the operational case using only the supplied case facts. Do not infer a root cause that is not provided.",
    "criteria": {
      "SPC_HOLD": "A statistical process control or process-monitoring hold",
      "WIPGROUP_FULL": "A WIP group capacity or fullness constraint",
      "EQUIPMENT_CONSTRAINT": "An equipment availability, failure, or qualification constraint",
      "DATA_QUALITY": "A data integrity, missing data, or measurement-quality issue",
      "ACCESS_REQUEST": "A user access or permission request",
      "OTHER": "The case does not fit the listed categories"
    }
  },
  "severity": {
    "type": "choice",
    "instructions": "Classify the triage severity based only on the impact facts explicitly present in the state. If production, customer, or shipping impact is unknown, do not assume it is critical; use the available evidence and the human-review question.",
    "criteria": {
      "LOW": "No material operational impact is stated; routine handling appears sufficient",
      "MEDIUM": "A limited issue is stated, but broad impact or immediate escalation is not established",
      "HIGH": "Material operational impact or repeated disruption is explicitly supported by the state",
      "CRITICAL": "Severe, immediate production, safety, or customer impact is explicitly supported by the state",
      "UNKNOWN": "Available facts do not support a reliable severity classification"
    }
  },
  "route_to": {
    "type": "choice",
    "instructions": "Choose the next review stage, not an operational action. Base the route on the supplied facts and unknowns. Never select a route that implies releasing or changing the hold.",
    "criteria": {
      "TRIAGE": "Initial review and confirmation of case details",
      "INVESTIGATION": "A responsible engineer or team should investigate the evidence and cause",
      "ESCALATION": "A qualified owner should promptly assess a clearly supported high-impact case",
      "CLOSURE_REVIEW": "Evidence is complete enough for an authorized person to review closure",
      "HUMAN_REVIEW": "The case is ambiguous or missing important facts and needs human judgment"
    }
  },
  "needs_human_review": {
    "type": "noul",
    "instructions": "Does this case require a qualified human to review the evidence before any operational decision or status change?"
  },
  "rca_readiness_score": {
    "type": "score",
    "instructions": "Score how ready the case is for a root-cause-analysis review based only on evidence explicitly present in the state. This is evidence completeness, not probability that a guessed cause is correct.",
    "criteria": [
      "0 - Not ready: key evidence or impact details are missing",
      "1 - Partly ready: some useful facts are present, but important evidence or validation is missing",
      "2 - Review-ready: relevant evidence, impact, and verification are documented sufficiently for an authorized RCA review"
    ]
  },
  "missing_evidence_category": {
    "type": "choice",
    "instructions": "Select the most important missing evidence category that blocks confident investigation or closure. Choose NONE only if the state indicates required evidence is available.",
    "criteria": {
      "ROOT_CAUSE_EVIDENCE": "Verified evidence identifying or testing the suspected root cause",
      "IMPACT_SCOPE": "Evidence of affected lots, production, customer, or shipping impact",
      "CONTAINMENT_VERIFICATION": "Evidence that containment was applied and verified",
      "ENGINEER_ASSESSMENT": "A qualified engineer's assessment or disposition rationale",
      "RELEASE_APPROVAL": "Required authorized release or closure approval",
      "NONE": "No important evidence gap is indicated by the state",
      "OTHER": "A different evidence gap is present"
    }
  }
}
```

คำถาม `Choice` ใช้รายการตัวเลือกที่กำหนดไว้, `Score` ใช้ระดับเรียงลำดับพร้อมคำอธิบาย, และ `Noul` ใช้คำถามใช่/ไม่ใช่ซึ่งคืนค่าความน่าจะเป็น 0–1 [2] กำหนดเกณฑ์ให้ชัดและถามหนึ่ง judgment ต่อหนึ่ง question เพื่อให้ตรวจคำตอบและนำไปประกอบในโค้ดได้ง่าย [2][5]

### ถ้า Playground ให้เพิ่มคำถามทีละข้อ

เพิ่มตาม ID เหล่านี้ แล้วคัดลอกเนื้อหาจาก JSON ด้านบนของแต่ละ ID ไปใส่:

1. `case_type` — Choice
2. `severity` — Choice
3. `route_to` — Choice
4. `needs_human_review` — Noul
5. `rca_readiness_score` — Score
6. `missing_evidence_category` — Choice

> ระวัง: ถ้าหน้าจอมีช่องแยกสำหรับ `type`, `instructions`, `criteria` ให้คัดลอกเฉพาะค่าของ field นั้น ๆ ไม่วาง JSON ชั้นนอกทั้งก้อนลงในช่อง text ของ instructions

## 4. อ่านผลลัพธ์ที่ได้

ชื่อคำตอบจะตรงกับ question IDs ที่ตั้งไว้:

- `case_type.choice`: ประเภทเคสที่เลือก
- `case_type.probabilities` และ `case_type.confidence`: การกระจายความน่าจะเป็นและ confidence ของ Choice
- `severity.choice`: ระดับ triage ที่เลือก; ตรวจ probability/confidence ประกอบ
- `route_to.choice`: ขั้นตอนถัดไปที่เสนอ ไม่ใช่คำสั่งให้ระบบทำ action
- `needs_human_review.noul`: ความน่าจะเป็นที่คำตอบใช่สำหรับการต้อง review โดยมนุษย์
- `rca_readiness_score.score`: คะแนน readiness ตาม rubric; ดู `legend`, probabilities และ confidence ประกอบ
- `missing_evidence_category.choice`: ประเภทหลักฐานที่ควรตามเพิ่ม

Jev คืน typed values และ distributions ตามชนิดคำถาม; `Noul` ไม่มี `confidence` แยก ส่วน Choice/Score มี confidence ตามรูปแบบคำตอบของเอกสาร [2][4]

ผลจริงอาจแตกต่างจากที่คาดไว้ อย่ากำหนดคำตอบตายตัวให้ Playground: เป้าหมายคือเห็นโครงสร้างผลลัพธ์ แล้วตรวจว่าโมเดลยึดเฉพาะข้อมูลใน State หรือไม่

## 5. คำถามฝึกคิดหลังรัน

ตอบจากผลที่ Playground แสดง:

1. ทำไม `case_type` เป็น Choice แต่ `rca_readiness_score` เป็น Score?
2. State ระบุว่ามี root cause ที่ยืนยันแล้วหรือไม่? ถ้าไม่ คำตอบใดใน instructions ช่วยกันไม่ให้เดาสาเหตุ?
3. มีข้อความใดใน State ที่ยืนยัน production/shipping impact หรือ release approval หรือไม่?
4. ถ้า `needs_human_review.noul` สูง หรือ Choice confidence ต่ำ ควรทำอะไรต่อ?
5. จะเพิ่มหลักฐานอะไรใน State ก่อนยกระดับ `rca_readiness_score`?

หลักการออกแบบคือให้ model ประเมินข้อมูลที่ส่งมา แต่ให้ code และผู้มีอำนาจคุม workflow, threshold และ side effects; ความมั่นใจเป็นข้อมูลประกอบ ไม่ใช่การรับประกันความถูกต้อง [4][5][7]

## 6. ลองแก้ State เพื่อเปรียบเทียบ

ทำสำเนา State เดิม แล้วแก้เพียงข้อเดียวในแต่ละครั้ง จากนั้นรัน Questions ชุดเดิม:

### ทดลอง A — ลดจำนวนล็อต

เปลี่ยน `lot_count` จาก `5` เป็น `1` และแก้ข้อความใน `description` ให้สอดคล้องกัน สังเกต `severity` และ `route_to`

### ทดลอง B — เพิ่มหลักฐาน impact

เพิ่มใน `known_facts` ว่ามีข้อมูลยืนยันจำนวนล็อตและผลกระทบ แต่ต้องเป็นข้อความสมมติสำหรับแบบฝึกหัดเท่านั้น สังเกตว่าคำตอบเปลี่ยนเพราะข้อมูลใด

### ทดลอง C — เติมหลักฐาน RCA

เพิ่มหลักฐานจำลอง เช่น engineer assessment และ containment verification ที่ระบุชัดว่าเป็นตัวอย่าง จากนั้นดู `rca_readiness_score` และ `missing_evidence_category`

### ตารางบันทึกผล

| Run | เปลี่ยนอะไรใน State | case_type | severity | route_to | human review Noul | RCA score | จุดที่อยากตรวจต่อ |
|---|---|---|---|---|---:|---:|---|
| Base | State ต้นฉบับ |  |  |  |  |  |  |
| A | ลดจำนวนล็อต |  |  |  |  |  |  |
| B | เพิ่ม impact evidence (จำลอง) |  |  |  |  |  |  |
| C | เพิ่ม RCA evidence (จำลอง) |  |  |  |  |  |  |

เปลี่ยนทีละตัวแปรเพื่อแยกให้ออกว่า input ส่วนใดมีผลต่อ judgment อย่าสรุปคุณภาพของระบบจากเคสเดียว; PoC จริงควรใช้ชุดเคสที่มีคำตอบจาก SME เป็น ground truth และวัดผลก่อนตั้ง threshold [4][5]

## 7. ตัวอย่าง request แบบรวม (สำหรับดูภาพรวม ไม่ใช่ช่อง State หรือ Questions แยก)

ถ้าในอนาคตใช้ API โดยตรง Request จะรวม `state`, `model`, และ `questions` ไว้ใน object เดียว โดยส่งไปยัง endpoint ตาม Quick start [1]:

```json
{
  "model": "jev-latest",
  "state": {
    "case": {
      "case_id": "NW-DEMO-001",
      "site": "NRM",
      "title": "Multiple lots on SPC hold after APC error",
      "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
      "lot_count": 5,
      "hold_reason": "ENGINEERING",
      "hold_code": "SPC_Shutdown APC_TERMFAIL_AVG_LOW",
      "current_status": "HOLD",
      "repeat_within_24h": true
    },
    "known_facts": [
      "Five lots are reported on hold.",
      "Current status is HOLD.",
      "Root cause is not confirmed."
    ],
    "unknowns": [
      "Production or shipping impact is unknown.",
      "No release approval is included."
    ],
    "safety_policy": "Human approval is required before operational status changes."
  },
  "questions": {
    "needs_human_review": {
      "type": "noul",
      "instructions": "Does this case require a qualified human to review the evidence before any operational decision or status change?"
    }
  }
}
```

Request รวมด้านบนเป็นตัวอย่างย่อสำหรับ API เท่านั้น หากทำ Playground ให้ใช้ State เต็มในหัวข้อ 2 และ Questions เต็มในหัวข้อ 3 แยกกัน อย่าวาง request รวมนี้ลงช่อง State อย่างเดียว [1]

## 8. ขั้นถัดไปสำหรับ PoC จริง

ก่อนต่อเข้ากับ NeoWork:

1. ใช้ historical cases ที่ลบ/ปกปิดข้อมูลอ่อนไหวแล้ว และมี SME labels
2. ให้ SMEs กำหนดนิยามของ case type, severity และ readiness score
3. ประเมินผลแยกตาม site, hold type และระดับความมั่นใจ
4. กำหนด fallback ให้คน review สำหรับ uncertainty, unknown category และ high-impact action
5. ให้ output เป็นข้อเสนอการ route เท่านั้น จนกว่าจะผ่าน validation, audit และ approval ภายใน
6. อย่าส่งข้อมูล production ที่เป็นความลับไปยังบริการภายนอกจนกว่าจะผ่าน security/privacy review

เอกสาร TypeSafe แนะนำให้เริ่มจากคำถามที่แคบและแยก judgment ที่ซับซ้อนเป็นหลายคำถาม [5]. ให้ code ควบคุมการตัดสินใจปลายทาง [5][6]. Threshold ต้องปรับจากข้อมูลและความเสี่ยงของ use case [4][7].

## แหล่งอ้างอิง

[1] Quick start: Playground, API request และ Python SDK  
[2] Primitives: Choice, Score, Noul และคำถามหลายข้อ  
[3] State: วิธีจัดข้อมูลและแยก state ออกจาก questions  
[4] Confidence: วิธีอ่าน probability/confidence  
[5] How to build with TypeSafe: ให้ code คุม workflow  
[6] Intent routing: route ไป deterministic handler, specialist หรือ human  
[7] Confidence-gated routing: threshold ตามความเสี่ยง

## Sources

[1] https://docs.typesafe.ai/introduction/quickstart — Quick start
[2] https://docs.typesafe.ai/primitives.md — Primitives
[3] https://docs.typesafe.ai/concepts/state.md — State
[4] https://docs.typesafe.ai/confidence.md — Confidence
[5] https://docs.typesafe.ai/concepts/how-to-build-with-system-one.md — How to build with TypeSafe
[6] https://docs.typesafe.ai/patterns/intent-routing.md — Intent routing
[7] https://docs.typesafe.ai/patterns/confidence-routing.md — Confidence-gated routing
