# Hands-on 4: Advanced NeoWork Triage — ผสม 3 Modes

รันเคสตัวอย่าง NeoWork ด้วย **State 1 ชุด** และ **Questions 1 ชุด** ซึ่งผสม Noul, Choice และ Score ใน request เดียว [1][2]

- ใช้ข้อมูลจำลองเพื่อการเรียนรู้เท่านั้น ไม่ใช่ข้อมูลจาก NeoWork, Oracle, Hive หรือโรงงานจริง
- Questions ทุกข้อเห็น State ชุดเดียวกันและประเมินแยกจากกัน [1][2] ให้ระบบ application เป็นผู้รวมผลและกำหนด workflow ต่อ
- ห้ามใช้ผลจากแบบฝึกหัดนี้ปล่อย hold, อนุมัติ release, ปิดเคส หรือสั่งการ production โดยอัตโนมัติ

## วิธีรันบน Playground

1. เปิด [TypeSafe Playground](https://console.typesafe.ai/)
2. สร้าง request ใหม่ โดยใช้ JSON ในส่วน **STATE** เป็น State
3. เพิ่ม Questions จาก JSON ในส่วน **QUESTIONS**; ถ้า Playground ให้เพิ่มทีละข้อ ให้สร้างหนึ่ง question ต่อหนึ่ง key/ID ตามตัวอย่าง
4. เลือก Jev หาก Playground มีตัวเลือกโมเดล แล้วกด Run / Evaluate
5. อ่านผลตามหัวข้อ “วิธีอ่านคำตอบ” ด้านล่าง

ใน API รูปแบบคำขอประกอบด้วย `state`, `model` และ map ของ `questions`; question IDs จะเป็น key ของคำตอบที่ส่งกลับ [2] ในแบบฝึกหัดนี้มี State block เดียวและ Questions block เดียวเท่านั้น

## STATE — คัดลอกทั้งก้อนลงช่อง State

```json
{
  "case": {
    "case_id": "NW-DEMO-ADV-001",
    "site": "NRM",
    "title": "Multiple lots on SPC hold after APC error",
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "lot_count_reported": 5,
    "hold_reason": "ENGINEERING",
    "hold_code": "SPC_Shutdown APC_TERMFAIL_AVG_LOW",
    "current_status": "HOLD",
    "repeat_within_24h_reported": true
  },
  "evidence": {
    "hold_snapshot": {
      "status": "listed_as_available",
      "contents": "No real Oracle extract is included; this is only a sample source label."
    },
    "wip_snapshot": {
      "status": "listed_as_available",
      "contents": "No real WIP data is included; this is only a sample source label."
    },
    "engineer_assessment": {
      "status": "missing",
      "contents": null
    },
    "root_cause_verification": {
      "status": "not_provided",
      "contents": null
    },
    "production_or_customer_impact": {
      "status": "unknown",
      "contents": null
    },
    "containment_verification": {
      "status": "not_provided",
      "contents": null
    },
    "release_approval": {
      "status": "not_provided",
      "contents": null
    }
  },
  "known_facts": [
    "The case description reports five lots on hold.",
    "The stated current status is HOLD.",
    "The case description reports that the event was observed again within 24 hours.",
    "The case description says the root cause has not been confirmed."
  ],
  "unknowns": [
    "The actual production, customer, or shipping impact is unknown.",
    "No verified engineer assessment is provided.",
    "No root-cause verification or containment verification is provided.",
    "No authorized release approval is provided."
  ],
  "safety_constraints": [
    "This is synthetic training data, not a real production case.",
    "Do not release, disposition, close, or change production status from this model assessment.",
    "A qualified human must verify evidence and authorize any operational action."
  ]
}
```

## QUESTIONS — คัดลอกทั้งก้อนลงช่อง Questions

คำถามแต่ละข้อมี ID ของตัวเอง เพื่อให้ผลลัพธ์อ่านแยกกันได้ [2]

Noul ใช้กับคำถาม yes/no เพียงประเด็นเดียว [3] Choice ใช้ตัวเลือกพร้อมคำอธิบาย [4] Score ใช้ criteria ที่เรียงจากระดับต่ำไปสูง [5]

```json
{
  "case_type": {
    "type": "choice",
    "instructions": "Which category best describes `case.hold_code` and `case.description`, using only the supplied state? Do not infer a root cause.",
    "criteria": {
      "SPC_HOLD": "A statistical process control or process-monitoring hold",
      "WIP_CAPACITY": "A work-in-progress group capacity or fullness constraint",
      "EQUIPMENT_CONSTRAINT": "An equipment availability, failure, or qualification constraint",
      "DATA_QUALITY": "A data integrity, missing-data, or measurement-quality issue",
      "OTHER_OR_UNCLEAR": "The available facts do not support the other categories"
    }
  },
  "triage_severity": {
    "type": "choice",
    "instructions": "Classify the triage severity using only impact explicitly documented in the state. Do not treat unknown production, customer, or shipping impact as confirmed impact.",
    "criteria": {
      "LOW": "No material operational impact is documented; routine review may be sufficient",
      "MEDIUM": "A limited issue or repeated event is reported, but broad impact is not established",
      "HIGH": "Material operational impact is explicitly supported by supplied evidence",
      "CRITICAL": "Severe immediate production, safety, or customer impact is explicitly supported by supplied evidence",
      "UNKNOWN": "The supplied state does not support a reliable severity classification"
    }
  },
  "next_review_stage": {
    "type": "choice",
    "instructions": "Choose the most appropriate review stage, not an operational action. Do not select a choice that releases or changes the hold.",
    "criteria": {
      "INITIAL_TRIAGE": "Confirm case details and identify the responsible owner",
      "ENGINEERING_INVESTIGATION": "A responsible engineer should investigate the event and supporting evidence",
      "EVIDENCE_COMPLETION": "Collect or verify missing evidence before further review",
      "PROMPT_HUMAN_ESCALATION": "A qualified owner should promptly assess explicitly documented high impact",
      "AUTHORIZED_CLOSURE_REVIEW": "Evidence is ready for an authorized person to review closure; this is not automatic closure"
    }
  },
  "primary_evidence_gap": {
    "type": "choice",
    "instructions": "Select the most important evidence gap that blocks a well-supported next review, based only on the supplied state.",
    "criteria": {
      "ROOT_CAUSE_VERIFICATION": "Evidence that tests or verifies the suspected root cause",
      "IMPACT_SCOPE": "Verified scope of affected lots and production, customer, or shipping impact",
      "CONTAINMENT_VERIFICATION": "Evidence that containment was applied and verified",
      "ENGINEER_ASSESSMENT": "A qualified engineer's assessment or disposition rationale",
      "RELEASE_APPROVAL": "Authorized approval required for release or closure",
      "OTHER_OR_UNCLEAR": "Another gap is present or the supplied state does not identify one clearly"
    }
  },
  "event_repeated_within_24h": {
    "type": "noul",
    "instructions": "Does `case.description` explicitly say that the event was observed again within 24 hours?",
    "criteria": {
      "true": "The description explicitly reports a repeat within 24 hours",
      "false": "The description does not report a repeat within 24 hours"
    }
  },
  "root_cause_confirmed": {
    "type": "noul",
    "instructions": "Does the supplied state provide verified evidence that the root cause is confirmed?",
    "criteria": {
      "true": "A confirmed root cause is explicitly supported by evidence in the state",
      "false": "The state says the cause is unconfirmed or supplies no verification"
    }
  },
  "release_approval_provided": {
    "type": "noul",
    "instructions": "Does the supplied state include an authorized release approval?",
    "criteria": {
      "true": "An explicit authorized release approval is included",
      "false": "No authorized release approval is included"
    }
  },
  "rca_evidence_readiness": {
    "type": "score",
    "instructions": "How ready is the supplied case information for an authorized root-cause-analysis review? Rate evidence completeness, not the likelihood that a guessed cause is correct.",
    "criteria": [
      "Not ready: key evidence, impact details, or verification is missing",
      "Partly ready: some useful facts are present, but important evidence or validation is still missing",
      "Review-ready: relevant evidence, impact, and verification are documented for an authorized review"
    ]
  },
  "impact_documentation": {
    "type": "score",
    "instructions": "How complete is the documentation of operational impact in the supplied state?",
    "criteria": [
      "Insufficient: no verified affected scope or operational impact is documented",
      "Partial: some affected scope is stated, but verified production, customer, or shipping impact is missing",
      "Complete: affected scope and verified operational impact are documented with supporting evidence"
    ]
  }
}
```

## วิธีอ่านคำตอบ

คำตอบจะกลับมาใต้ ID ที่กำหนดไว้ใน Questions [2]:

| Question IDs | Mode | ฟิลด์หลักที่อ่าน | ใช้ตอบเรื่อง |
|---|---|---|---|
| `case_type`, `triage_severity`, `next_review_stage`, `primary_evidence_gap` | Choice | `choice`, `probabilities`, `confidence` | ประเภทเคส, ความรุนแรง, ขั้น review, ช่องว่างหลักฐาน |
| `event_repeated_within_24h`, `root_cause_confirmed`, `release_approval_provided` | Noul | `noul` (0–1) | ความน่าจะเป็นที่คำตอบ yes เป็นจริง; Noul ไม่มี confidence แยก |
| `rca_evidence_readiness`, `impact_documentation` | Score | `score`, `legend`, `probabilities`, `confidence` | ตำแหน่งบน rubric; คะแนนอาจอยู่ระหว่างระดับ |

Choice แสดงตัวเลือกที่มี probability สูงสุดพร้อม distribution ของทุกตัวเลือก [4] ส่วน Score เป็นตำแหน่งบนลำดับระดับและอาจเป็นทศนิยม จึงควรอ่าน probabilities ควบคู่กัน ไม่พิจารณาจาก score ตัวเดียว [5] ค่าเหล่านี้เป็นการประเมินจากข้อมูลที่ให้ ไม่ใช่หลักฐานยืนยันหรือการอนุมัติ

## วิเคราะห์ผลแบบ Advanced

1. เปรียบเทียบ `case_type` กับ `primary_evidence_gap`: ประเภทเคสตอบว่าเป็นงานกลุ่มใด ส่วน evidence gap บอกข้อมูลสำคัญที่ยังขาด [4]
2. เปรียบเทียบ `triage_severity` กับ `impact_documentation`: การมีหลายล็อตบน hold ไม่เท่ากับมีหลักฐานยืนยันผลกระทบด้าน production/customer/shipping
3. เทียบ `event_repeated_within_24h` กับ `root_cause_confirmed`: การเกิดซ้ำเป็นข้อมูลเกี่ยวกับ pattern แต่ไม่ได้ยืนยันสาเหตุ
4. อ่าน `rca_evidence_readiness.score` พร้อม probabilities และ confidence; คะแนน readiness ไม่ใช่ probability ว่า RCA ถูกต้อง [5]
5. ตรวจว่าคำตอบแต่ละข้ออ้างอิง State ได้หรือไม่ [1] หาก State ไม่มีข้อมูลนั้นให้ถือเป็นช่องว่างเพื่อ human review ไม่ใช่เติมข้อเท็จจริงเอง

คำถามหลายข้อใน request เดียวกันถูกประเมินแยกกัน แม้จะใช้ State เดียวกัน; อย่าคาดหวังให้คำตอบหนึ่ง question คำนวณหรือบังคับคำตอบอีก question โดยอัตโนมัติ [1][2] การรวมผลควรทำใน application code [6] การตั้ง threshold และควบคุม side effects ต้องกำหนดตามความเสี่ยงของ workflow [6]

## ทดลองเปลี่ยน State

เก็บ Questions เดิมไว้ แล้วเปลี่ยนข้อมูล State ทีละจุด:

- **Scenario A — ไม่มีข้อความว่าเกิดซ้ำ:** แก้ `case.description` ให้ไม่ระบุ event ซ้ำภายใน 24 ชั่วโมง แล้วสังเกต `event_repeated_within_24h`
- **Scenario B — เพิ่ม impact assessment จำลอง:** เพิ่มหลักฐานที่ระบุ scope และผลกระทบอย่างชัดเจนใน `evidence.production_or_customer_impact` แล้วสังเกต `triage_severity` และ `impact_documentation`
- **Scenario C — เพิ่มหลักฐาน RCA จำลอง:** เพิ่ม engineer assessment, root-cause verification และ containment verification ที่ระบุว่าเป็นข้อมูลฝึก แล้วสังเกต `root_cause_confirmed`, `primary_evidence_gap` และ `rca_evidence_readiness`

ใช้ข้อมูลสังเคราะห์เท่านั้น และอย่าเขียน Scenario ที่ทำให้ดูเหมือนมี approval จริง ผลลัพธ์อาจเปลี่ยนตามข้อความที่ป้อนและไม่ใช่ค่าที่รับประกัน; ควรทดสอบเกณฑ์กับข้อมูล use case ก่อนกำหนด workflow จริง [6]

## Checklist

- [ ] มี JSON State เพียงหนึ่ง block และ Questions เพียงหนึ่ง block
- [ ] Questions ผสม Noul, Choice และ Score ใน request เดียว
- [ ] ทุกคำตอบตรวจได้จาก question ID โดยตรง
- [ ] แต่ละ Noul ถามเงื่อนไข yes/no เดียว
- [ ] Choice มีตัวเลือกที่อธิบายความหมาย รวมตัวเลือก fallback
- [ ] Score มี levels เรียงลำดับและเป็น rubric มิติเดียว
- [ ] ไม่มีการนำผลไปเปลี่ยนสถานะ production โดยอัตโนมัติ

## Sources

[1] https://docs.typesafe.ai/concepts/state — State
[2] https://docs.typesafe.ai/primitives — Primitives
[3] https://docs.typesafe.ai/primitives/noul — Noul
[4] https://docs.typesafe.ai/primitives/choice — Choice
[5] https://docs.typesafe.ai/primitives/score — Score
[6] https://docs.typesafe.ai/concepts/how-to-build-with-system-one — How to build with TypeSafe
