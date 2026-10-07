# Jev Hands-on: ทดลองกับตัวอย่างเคส NeoWork

คู่มือสำหรับทดลอง Jev บน TypeSafe Playground ด้วย JSON ที่แยก **State** (ข้อมูลเคส) กับ **Questions** (สิ่งที่ต้องการให้ประเมิน) ชัดเจน

- ครบ 3 Modes: **Noul, Choice, Score**
- Mode ละ 2 ตัวอย่าง: ข้อแรกเป็น Single State + Single Question; ข้อสองเป็น Multi Questions บน State เดียว
- ใช้ข้อมูลจำลองเพื่อฝึกเท่านั้น ไม่ใช่ข้อมูลจาก NeoWork, Oracle, Hive หรือโรงงานจริง
- ห้ามใช้ผลลัพธ์นี้ปล่อย hold, อนุมัติ release, ปิดเคส หรือสั่งการ production โดยอัตโนมัติ

## วิธีใช้ Playground

1. เปิด [TypeSafe Playground](https://console.typesafe.ai/)
2. เลือกตัวอย่างที่ต้องการทดลอง
3. วาง JSON ในส่วน **STATE** ลงในช่อง State
4. วาง JSON ในส่วน **QUESTIONS** ลงในช่อง Questions / Prompts
5. กด Run / Evaluate แล้วดูคำตอบตาม ID ของแต่ละ question

State คือข้อมูลที่ให้โมเดลประเมิน; คำถามทั้งหมดใน request ใช้ State เดียวกันและถูกประเมินแยกจากกัน, ดังนั้นควรเก็บข้อมูลใน State และ judgment ที่ต้องการใน Questions คนละส่วน [1]

> หาก Playground แสดงช่องให้เพิ่ม Questions ทีละข้อ ให้เพิ่มแต่ละ object โดยใช้ชื่อ ID เป็น question ID หากหน้าจอมีช่องแยก `type`, `instructions`, `criteria` ให้ใส่ค่าแต่ละ field ให้ตรงช่อง

## สรุป 3 Modes

| Mode | ใช้เมื่อ | ผลลัพธ์หลัก |
|---|---|---|
| **Noul** | ต้องการคำตอบใช่/ไม่ใช่ | `noul`: ความน่าจะเป็นที่คำตอบคือ “ใช่” ตั้งแต่ 0–1 |
| **Choice** | ต้องเลือกหนึ่งตัวเลือกจากรายการ | `choice`, `probabilities`, `confidence` |
| **Score** | ต้องประเมินบนระดับที่เรียงลำดับ | `score`, `legend`, `probabilities`, `confidence` |

ใช้ Noul กับแต่ละเงื่อนไข yes/no, Choice กับประเภทที่แยกเป็นตัวเลือก และ Score กับมิติที่ให้ระดับจากต่ำไปสูงได้ [2][3][4]

---

# Mode 1 — Noul

Noul ตอบคำถาม yes/no โดยคืนค่าความน่าจะเป็นของ “ใช่” การแยกหลายเงื่อนไขเป็นหลายคำถามช่วยให้แต่ละผลลัพธ์มีความหมายชัดเจน [2]

## Noul 1 — Single: ระบุว่ามีการรายงานเหตุซ้ำหรือไม่?

หนึ่ง State กับหนึ่ง Question

### STATE

```json
{
  "case": {
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "repeat_within_24h": true
  }
}
```

### QUESTIONS

```json
{
  "event_repeated_within_24h": {
    "type": "noul",
    "instructions": "Does the case description say that the event was observed again within 24 hours?"
  }
}
```

**อ่านผล:** ดู `answers.event_repeated_within_24h.noul` ซึ่งประเมินว่าข้อความใน description ระบุเหตุซ้ำภายใน 24 ชั่วโมงหรือไม่ [2] นี่เป็นการอ่านข้อมูลในตัวอย่าง ไม่ใช่การยืนยันจากระบบโรงงาน

## Noul 2 — Multi: ตรวจข้อมูลที่มีและข้อมูลที่ยังขาด

หนึ่ง State กับหลาย Questions แบบ yes/no แยกอิสระ

### STATE

```json
{
  "case": {
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "current_status": "HOLD"
  },
  "evidence": {
    "engineer_assessment": "missing",
    "production_impact": "unknown",
    "release_approval": "not_provided"
  }
}
```

### QUESTIONS

```json
{
  "root_cause_confirmed": {
    "type": "noul",
    "instructions": "Does the state provide a confirmed root cause for the case?"
  },
  "production_impact_documented": {
    "type": "noul",
    "instructions": "Does the state document a verified production impact?"
  },
  "release_approval_provided": {
    "type": "noul",
    "instructions": "Does the state provide an authorized release approval?"
  }
}
```

**อ่านผล:** ตรวจคำตอบสาม ID แยกกันว่า State ระบุ root cause, production impact และ release approval หรือไม่ [1][2] การไม่มีข้อมูลใน State ไม่ได้พิสูจน์ว่าไม่มีข้อมูลในระบบต้นทาง; มันหมายถึงข้อมูลดังกล่าวไม่ได้ให้มาในตัวอย่างนี้

---

# Mode 2 — Choice

Choice ใช้เมื่อต้องเลือกหนึ่งคำตอบจากตัวเลือกที่กำหนดไว้ และส่งคืนตัวเลือกพร้อม probability distribution และ confidence [3]

## Choice 1 — Single: จัดประเภทเคสจากคำอธิบาย

หนึ่ง State กับหนึ่ง Question

### STATE

```json
{
  "case": {
    "title": "Multiple lots on SPC hold after APC error",
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "hold_code": "SPC_Shutdown APC_TERMFAIL_AVG_LOW"
  }
}
```

### QUESTIONS

```json
{
  "case_type": {
    "type": "choice",
    "instructions": "Which category best describes the operational case, using only the supplied facts?",
    "criteria": {
      "SPC_HOLD": "A statistical process control or process-monitoring hold",
      "WIP_CAPACITY": "A work-in-progress group capacity or fullness constraint",
      "EQUIPMENT": "An equipment availability, failure, or qualification constraint",
      "DATA_QUALITY": "A data integrity, missing-data, or measurement-quality issue",
      "OTHER_OR_UNCLEAR": "The supplied facts do not support the other categories"
    }
  }
}
```

**อ่านผล:** `answers.case_type.choice` คือประเภทที่เลือก; อ่าน `probabilities` และ `confidence` ประกอบ [3] หากข้อมูลไม่พอหรือคำตอบกระจายหลายตัวเลือก ให้ตรวจหลักฐานเพิ่มเติม อย่าถือว่า Choice ยืนยันสาเหตุราก

## Choice 2 — Multi: ประเมินขั้นตอน review และหลักฐานที่ควรตามเพิ่ม

หนึ่ง State กับหลาย Questions แบบ Choice

### STATE

```json
{
  "case": {
    "title": "Multiple lots on SPC hold after APC error",
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "current_status": "HOLD",
    "repeat_within_24h": true
  },
  "evidence": {
    "verified_engineer_assessment": "missing",
    "production_or_shipping_impact": "unknown",
    "containment_verification": "not_provided",
    "release_approval": "not_provided"
  },
  "safety_note": "This is a training example. Any operational decision requires qualified human review and authorized approval."
}
```

### QUESTIONS

```json
{
  "triage_severity": {
    "type": "choice",
    "instructions": "Classify triage severity using only impact facts explicitly present. Do not assume unknown production, customer, or shipping impact.",
    "criteria": {
      "LOW": "No material operational impact is stated; routine handling may be sufficient",
      "MEDIUM": "A limited issue or repeated event is stated, but broad impact is not established",
      "HIGH": "Material operational impact is explicitly supported by the supplied facts",
      "CRITICAL": "Severe and immediate production, safety, or customer impact is explicitly supported",
      "UNKNOWN": "Available facts do not support a reliable severity classification"
    }
  },
  "next_review_stage": {
    "type": "choice",
    "instructions": "Choose a review stage, not an operational action. Do not choose a stage that releases or changes the hold.",
    "criteria": {
      "INITIAL_TRIAGE": "Confirm case details and identify the responsible owner",
      "ENGINEERING_INVESTIGATION": "A responsible engineer should investigate evidence and possible cause",
      "PROMPT_HUMAN_ESCALATION": "A qualified owner should promptly assess explicitly supported high impact",
      "EVIDENCE_COMPLETION": "Collect or verify missing evidence before further review",
      "AUTHORIZED_CLOSURE_REVIEW": "Evidence is ready for an authorized person to review closure; this is not automatic closure"
    }
  },
  "most_important_evidence_gap": {
    "type": "choice",
    "instructions": "Choose the most important missing evidence category for the next review, based only on the state.",
    "criteria": {
      "ROOT_CAUSE": "Verified evidence testing or identifying the suspected root cause",
      "IMPACT_SCOPE": "Verified evidence of affected lots or production, customer, or shipping impact",
      "CONTAINMENT": "Evidence that containment was applied and verified",
      "ENGINEER_ASSESSMENT": "A qualified engineer's assessment or disposition rationale",
      "RELEASE_APPROVAL": "Required authorized approval for release or closure",
      "OTHER_OR_UNCLEAR": "A different gap is present or the supplied facts do not identify one clearly"
    }
  }
}
```

**อ่านผล:** ดู `triage_severity.choice`, `next_review_stage.choice` และ `most_important_evidence_gap.choice` แยกกัน ตรวจ probabilities/confidence โดยเฉพาะเมื่อคำตอบไม่ชัด [3] Choice เป็นผลประเมินเพื่อช่วย review เท่านั้น ไม่ใช่คำสั่งปล่อย hold หรือเปลี่ยนสถานะ [5]

---

# Mode 3 — Score

Score ใช้ระดับที่เรียงลำดับจากต่ำไปสูง โดย `criteria` เป็นรายการระดับพร้อมคำอธิบาย คะแนนอาจอยู่ระหว่างระดับ จึงควรอ่าน `probabilities`, `legend` และ `confidence` ประกอบ [4]

## Score 1 — Single: ประเมินความพร้อมของข้อมูลสำหรับ RCA review

หนึ่ง State กับหนึ่ง Question

### STATE

```json
{
  "case": {
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "current_status": "HOLD"
  },
  "evidence": {
    "engineer_assessment": "missing",
    "production_impact": "unknown",
    "containment_verification": "not_provided"
  }
}
```

### QUESTIONS

```json
{
  "rca_evidence_readiness": {
    "type": "score",
    "instructions": "How ready is the supplied case information for an authorized root-cause-analysis review? Rate evidence completeness, not the probability that any suspected cause is correct.",
    "criteria": [
      "Not ready: key evidence, impact details, or verification is missing",
      "Partly ready: some useful facts are present, but important evidence or validation is still missing",
      "Review-ready: relevant evidence, impact, and verification are documented for an authorized review"
    ]
  }
}
```

**อ่านผล:** ตรวจ `answers.rca_evidence_readiness.score`, `legend`, `probabilities` และ `confidence` [4] คะแนนนี้ประเมินความพร้อมของข้อมูลที่ให้มา ไม่ได้ยืนยันว่า RCA ถูกต้องหรืออนุมัติการปิดเคส

## Score 2 — Multi: ให้คะแนนความครบถ้วนของหลักฐานคนละด้าน

หนึ่ง State กับหลาย Questions แบบ Score แต่ละข้อวัดคนละมิติ

### STATE

```json
{
  "case": {
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "current_status": "HOLD"
  },
  "evidence": {
    "affected_lots": "Five lots are reported on hold; no source extract is attached.",
    "production_impact": "Unknown; no impact assessment is included.",
    "containment": "No containment verification is included.",
    "engineer_assessment": "No verified engineer assessment is included."
  }
}
```

### QUESTIONS

```json
{
  "impact_documentation": {
    "type": "score",
    "instructions": "How complete is the documentation of operational impact in the supplied state?",
    "criteria": [
      "Insufficient: affected scope or impact is not described",
      "Partial: some affected scope is stated, but verified operational impact is missing",
      "Complete: affected scope and verified production, customer, or shipping impact are documented"
    ]
  },
  "containment_verification": {
    "type": "score",
    "instructions": "How complete is the evidence that containment was applied and verified?",
    "criteria": [
      "Insufficient: no containment action or verification is documented",
      "Partial: an action is mentioned, but completion or verification is unclear",
      "Complete: containment action, scope, and verification evidence are documented"
    ]
  },
  "technical_assessment": {
    "type": "score",
    "instructions": "How complete is the qualified technical assessment for an RCA review?",
    "criteria": [
      "Insufficient: no qualified assessment or supporting evidence is provided",
      "Partial: an assessment is mentioned, but supporting evidence or validation is incomplete",
      "Complete: a qualified assessment is supported by documented evidence and validation"
    ]
  }
}
```

**อ่านผล:** เปรียบเทียบคะแนน impact, containment และ technical assessment แต่ละ `score` เป็นตำแหน่งบนสเกล ส่วน probabilities แสดงการกระจายระหว่างระดับ [4] หากคะแนนหรือ confidence ทำให้ไม่แน่ใจ ให้ตรวจหลักฐานจริงกับผู้รับผิดชอบ ไม่ใช้คะแนนแทนการอนุมัติ [5]

---

## แบบฝึกต่อยอด

ลองเปลี่ยน State ทีละส่วน แล้วรัน Questions ชุดเดิมอีกครั้ง:

- เพิ่มหลักฐานจำลองที่ยืนยันจำนวนล็อตที่ได้รับผลกระทบ แล้วดู `impact_documentation`
- เพิ่ม engineer assessment ที่มีหลักฐานรองรับ โดยระบุชัดว่าเป็นข้อมูลสมมติ แล้วเปรียบเทียบ `rca_evidence_readiness` และ `technical_assessment`
- เพิ่มข้อความยืนยันว่า containment ถูกตรวจสอบแล้ว แล้วดู `containment_verification`
- เปลี่ยนคำอธิบายเคสให้ไม่มีการระบุเหตุซ้ำ แล้วตรวจว่า Noul single ประเมินข้อความอย่างไร

เปลี่ยนทีละตัวแปรเพื่อดูว่า input ใดสัมพันธ์กับ judgment ใด ผลจากตัวอย่างเดียวไม่เพียงพอสำหรับสรุปความแม่นยำหรือกำหนด threshold สำหรับใช้งานจริง; การกำหนด workflow และการกระทำปลายทางควรอยู่ในการควบคุมของโค้ดและผ่านการทดสอบกับข้อมูล use case [5]

## เช็กลิสต์

- [ ] ตัวอย่าง Single แต่ละบทมี State หนึ่งชุดและ Question หนึ่งข้อ
- [ ] ตัวอย่าง Multi แต่ละบทมี State หนึ่งชุดและ Questions หลายข้อ
- [ ] Noul ถาม yes/no ทีละประเด็น; Choice เลือกจาก criteria; Score ให้ระดับเรียงลำดับ
- [ ] อ่าน probabilities/confidence ประกอบกับคำตอบหลัก
- [ ] ผลลัพธ์เป็นข้อมูลช่วยประเมิน ไม่ใช่การอนุมัติหรือคำสั่ง production

## Sources

[1] https://docs.typesafe.ai/concepts/state — State: เนื้อหาที่ประเมิน, หนึ่ง State ต่อหนึ่งหรือหลาย Questions และคำถามถูกประเมินแยกกัน
[2] https://docs.typesafe.ai/primitives/noul — Noul: yes/no, criteria และการแยกคำถามหลายข้อ
[3] https://docs.typesafe.ai/primitives/choice — Choice: ตัวเลือกที่กำหนด, probabilities และ confidence
[4] https://docs.typesafe.ai/primitives/score — Score: ระดับเรียงลำดับ, score, legend, probabilities และ confidence
[5] https://docs.typesafe.ai/concepts/how-to-build-with-system-one — แยกคำถามให้แคบและให้โค้ดคุม workflow/side effects
