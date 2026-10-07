# Hands-on Lab: สร้างระบบคัดแยกข้อความลูกค้าด้วย TypeSafe / Jev

เอกสารเรียนรู้แบบลงมือทำทีละขั้น: ตั้งแต่ทดลองใน Playground ไปจนถึงเรียก Python SDK และสร้างตัวอย่าง intent routing ที่ใช้ confidence ช่วยตัดสินใจ

- ระดับ: เริ่มต้นถึงกลาง
- เวลา: ประมาณ 60–90 นาที
- สิ่งที่จะได้: สคริปต์ Python ที่ถาม Jev หลายคำถามกับข้อความเดียว แล้ว route ไปยังตัวอย่าง handler โดยยังให้โค้ดเป็นผู้ควบคุม
- ตัวอย่างใช้ข้อความภาษาอังกฤษตาม Quick start ของเอกสารทางการ เพราะเอกสารระบุว่า Jev ฝึกหลักด้วยภาษาอังกฤษ และความแม่นยำภาษาอื่นอาจต่ำกว่า ควรประเมินภาษาไทยแยกด้วยข้อมูลของตนเอง [1][3]

> References in brackets map to the numbered official docs below. แบบฝึกหัดนี้เป็นตัวอย่างเพื่อการเรียนรู้ ไม่ทำธุรกรรม ไม่คืนเงิน ไม่ส่งข้อความให้ลูกค้าจริง และไม่ควรนำ threshold ตัวอย่างไปใช้ production โดยไม่ทดสอบกับข้อมูลจริง

## สิ่งที่เราจะสร้าง

รับข้อความลูกค้า 1 รายการ แล้วถามในคำขอเดียวว่า:

1. ควรส่งให้ทีมใด (`Choice`)
2. เรื่องซับซ้อนระดับใด (`Score`)
3. ข้อความเร่งด่วนหรือไม่ (`Noul`)

จากนั้นโค้ดจะเลือกเพียง “ชื่อเส้นทางตัวอย่าง” เพื่อสาธิตการ routing: งานง่ายอาจใช้ deterministic code, คำถามสินค้าอาจส่งให้ specialist LLM, เรื่องซับซ้อนหรือความมั่นใจต่ำให้คนตรวจสอบ ทั้งหมดนี้เป็นการพิมพ์ผลลัพธ์จำลอง ไม่ได้เรียก handler จริง [5][6]

## ก่อนเริ่ม

เตรียมไว้ก่อน:

- Python 3.10 ขึ้นไป [1]
- บัญชี TypeSafe และ API key จาก dashboard หากต้องการทำส่วน API/SDK จริง [1]
- Terminal บน Windows (ตัวอย่างคำสั่งด้านล่างใช้ PowerShell)
- อย่าใส่ API key ลงในไฟล์ `.py`, Markdown, Git หรือภาพหน้าจอ

ตรวจ Python:

```powershell
python --version
```

ถ้าใช้ VS Code ให้เปิดโฟลเดอร์งาน แล้วเปิด Terminal แบบ PowerShell ในโฟลเดอร์นั้น

## บทที่ 1 — ทดลองโดยยังไม่เขียนโค้ด (10 นาที)

1. เปิด [TypeSafe Playground](https://console.typesafe.ai/) แล้วเข้าสู่ระบบ [1]
2. ใส่ข้อความตัวอย่างนี้เป็น state:

```text
Hi, I've been trying to connect my Stripe account for 3 days and the integration keeps failing. I'm losing sales. Please help ASAP.
```

3. เพิ่มคำถาม Noul:

```json
{
  "urgency": {
    "type": "noul",
    "instructions": "Does this message express urgency?"
  }
}
```

4. รันและดูผลลัพธ์ จากนั้นเพิ่มคำถาม Choice และ Score จากบทถัดไป

### ทำความเข้าใจผล

- `Noul` ตอบความน่าจะเป็นของ “ใช่” ตั้งแต่ 0 ถึง 1 ไม่มี field `confidence` แยกต่างหาก [2][4]
- `Choice` เลือกจากตัวเลือกที่เรากำหนด พร้อม probabilities และ confidence [2]
- `Score` วัดระดับตามเกณฑ์เรียงลำดับที่เราให้ พร้อม probabilities และ confidence [2]

จดสิ่งที่เห็นไว้: คำตอบใดเป็น typed value? มี uncertainty แบบไหน? อย่าคาดหวังว่าค่าจะตรงกับตัวอย่างเอกสารทุกครั้ง เพราะผลลัพธ์อาจเปลี่ยนตามโมเดลและอินพุต

## บทที่ 2 — เลือกชนิดคำถามให้ตรงงาน (10 นาที)

หลักง่าย ๆ คือเลือกชนิดคำตอบที่โค้ดจะนำไปใช้ได้ตรงที่สุด: `Choice` สำหรับรายการที่ไม่เรียงลำดับ, `Score` สำหรับระดับที่เรียงจากน้อยไปมาก, และ `Noul` สำหรับคำถามใช่/ไม่ใช่ที่ชัดเจน [2]

| สิ่งที่ต้องการรู้ | ชนิด | ตัวอย่าง |
|---|---|---|
| ทีมใดควรรับเรื่องนี้? | Choice | billing / technical / sales / other |
| ความซับซ้อนอยู่ระดับใด? | Score | simple / moderate / complex |
| ลูกค้าขอคืนเงินจริงหรือไม่? | Noul | yes probability 0–1 |

กิจกรรมสั้น ๆ: เปลี่ยน “ลูกค้ารู้สึกอย่างไร?” ให้เป็นคำถามที่วัดได้ เช่น “ข้อความนี้แสดงความไม่พอใจระดับใด?” แล้วกำหนดระดับแต่ละคะแนนให้แยกจากกันชัดเจน

## บทที่ 3 — ติดตั้ง SDK และตั้ง API key (10 นาที)

สร้าง virtual environment และติดตั้ง SDK ตาม Quick start ทางการ [1]:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install typesafe-sdk
```

ถ้า PowerShell ปฏิเสธการ activate ให้เปิด terminal ใหม่หรือใช้คำสั่งนี้เฉพาะหน้าต่างปัจจุบัน:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

ตั้ง API key ใน environment ของ terminal ชั่วคราว โดยรับค่าแบบ masked และไม่บันทึกไว้ใน source code:

```powershell
$secure = Read-Host "TypeSafe API key" -AsSecureString
$ptr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($secure)
try {
    $env:TYPESAFE_API_KEY = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($ptr)
} finally {
    [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ptr)
}
```

SDK อ่าน `TYPESAFE_API_KEY` จาก environment และใช้ `jev-latest` โดยปริยาย [1] อย่าแชร์ terminal log ที่มี secret และเมื่องานเสร็จให้ล้างค่าจาก session:

```powershell
$env:TYPESAFE_API_KEY = $null
```

> หากไม่มี API key ให้ทำบท Playground และเขียน/ตรวจโค้ดได้ แต่จะเรียก API จริงไม่ได้ ห้ามใช้ key ของผู้อื่นโดยไม่ได้รับอนุญาต

## บทที่ 4 — สร้างคำขอแรกด้วย Python (15 นาที)

สร้างไฟล์ `lab.py` แล้วใส่โค้ดต่อไปนี้ โครงสร้างยึดตาม Quick start ของ TypeSafe: สร้าง `TypeSafeClient`, ส่ง `state` และ `questions`, แล้วอ่านคำตอบผ่าน `response.answers` [1]

```python
from typesafe_sdk import Choice, Noul, Score, TypeSafeClient

client = TypeSafeClient()

state = {
    "ticket": {
        "subject": "Stripe integration failing",
        "message": (
            "I've been trying to connect my Stripe account for 3 days "
            "and the integration keeps failing. I'm losing sales. Please help ASAP."
        ),
    }
}

questions = {
    "department": Choice(
        instructions="Which team should handle the customer's main issue?",
        criteria={
            "billing": "Payment, invoice, or subscription issues",
            "technical": "Bugs, setup, or integration problems",
            "sales": "Pricing or pre-sales questions",
            "other": "None of the listed categories fits",
        },
    ),
    "complexity": Score(
        instructions="How complex is this request to resolve?",
        criteria=[
            "Simple lookup or standard procedure",
            "Requires some judgment or a few steps",
            "Unusual edge case or likely needs escalation",
        ],
    ),
    "urgent": Noul(
        instructions="Does the message express urgency or time sensitivity?",
    ),
}

response = client.system_one(state=state, questions=questions)

for question_id, answer in response.answers.items():
    print(f"\\n{question_id}: {answer}")

print("\\nModel:", response.model)
print("Usage:", response.usage)
```

รัน:

```powershell
python .\lab.py
```

คำถามหลายข้อใน request เดียวใช้ state เดียวกัน แต่แต่ละคำถามถูกประเมินอย่างอิสระ และคำตอบผูกกับ ID ที่เราตั้งไว้ [2][3]

### จุดตรวจ (Checkpoint)

- โปรแกรมเริ่มทำงานโดยไม่พิมพ์ API key ออกมา
- เห็นคำตอบ `department`, `complexity`, `urgent`
- `department` มีค่าเลือกและ probabilities/confidence
- `complexity` มี score และ legend/probabilities/confidence
- `urgent` มีค่า `noul` ระหว่าง 0–1

หากผลไม่ตรง Checkpoint ให้ตรวจว่า virtual environment ถูก activate, ติดตั้ง `typesafe-sdk`, ตั้ง `TYPESAFE_API_KEY` ใน terminal เดียวกับที่รัน และชื่อ field สะกดตรงกัน

## บทที่ 5 — แยก state ออกจาก questions (10 นาที)

`state` คือข้อมูลที่ให้โมเดลประเมิน ส่วน `questions` คือ judgments ที่ต้องการ ควรใช้ object เมื่อข้อมูลมีหลายส่วน เพื่ออ้างถึงแต่ละ field ได้ชัดเจน [3]

ลองปรับ state ให้มีข้อความและข้อมูลประกอบ:

```python
state = {
    "ticket": {
        "subject": "Duplicate charge on order A-104",
        "message": "I was charged twice for order A-104. Please refund the duplicate.",
    },
    "order": {
        "id": "A-104",
        "charges": [49, 49],
    },
    "policy": "Duplicate charges are eligible for a refund.",
}
```

เพิ่มคำถาม Noul อีกข้อ:

```python
"policy_supports_refund": Noul(
    instructions=(
        "Does `policy` support the refund requested in `ticket.message`, "
        "given the charge information in `order.charges`?"
    ),
),
```

คำถามที่อ้างข้อมูลเฉพาะควรระบุ path แบบชัดเจน เช่น `` `ticket.message` `` เพื่อลดความกำกวม [2][5]

### แบบฝึก

1. สร้าง state ที่มี `customer_message` และ `account_status`
2. ถามว่า `customer_message` ขอความช่วยเหลือเรื่อง password reset หรือไม่
3. ถามแยกอีกข้อว่า `account_status` เป็น `active` หรือไม่
4. ระบุในคำสั่งให้ชัดว่าคำถามแต่ละข้ออ้าง field ไหน

## บทที่ 6 — ทำ intent routing โดยให้โค้ดเป็นผู้ควบคุม (15 นาที)

แนวทาง TypeSafe คือใช้โค้ดกับกฎ deterministic และ side effects; แยกคำถาม AI ให้แคบและมีโครงสร้าง; แล้วให้โค้ดประกอบคำตอบและเลือก handler [5] รูปแบบ intent routing สามารถส่งงานไป deterministic handler, specialist LLM หรือ human review ตาม intent และความซับซ้อน [6]

เพิ่มฟังก์ชันนี้ท้าย `lab.py` โดยให้มันพิมพ์ชื่อเส้นทางเท่านั้น:

```python
def choose_demo_route(response):
    intent = response.answers["department"]
    complexity = response.answers["complexity"]
    urgent = response.answers["urgent"].noul

    # Threshold เหล่านี้เป็นค่าตัวอย่างสำหรับฝึกเท่านั้น
    if intent.confidence < 0.50:
        route = "human_review: intent confidence is low"
    elif complexity.confidence < 0.50:
        route = "human_review: complexity confidence is low"
    elif complexity.score > 1:
        route = "human_review: request is complex"
    elif intent.choice == "technical":
        route = "technical_support_queue (simulated)"
    elif intent.choice == "billing":
        route = "billing_queue (simulated)"
    elif intent.choice == "sales":
        route = "sales_queue (simulated)"
    else:
        route = "human_review: other or unmapped intent"

    print("Demo route:", route)
    print("Urgency probability:", urgent)
```

เรียกฟังก์ชันหลังพิมพ์คำตอบ:

```python
choose_demo_route(response)
```

ใช้ confidence เป็นข้อมูลประกอบ ไม่ใช่คำรับประกันความถูกต้อง; ค่า threshold ต้องตั้งจากความเสี่ยงและผลทดสอบของ use case จริง [4][7] ตัวอย่างนี้ตั้งใจให้ fallback ไปคนตรวจสอบ และไม่สั่งคืนเงินหรือทำ action จริง

## บทที่ 7 — รันทดสอบกับตัวอย่างหลายแบบ (10–15 นาที)

ทำซ้ำโดยแทนข้อความใน `state["ticket"]["message"]` ทีละตัว:

### ตัวอย่าง A — ปัญหาการเชื่อมต่อ

```text
Our API integration started returning 500 errors on every request about 20 minutes ago. We cannot process orders.
```

### ตัวอย่าง B — คำถามก่อนซื้อ

```text
Does your plan include team access, and can I upgrade next month?
```

### ตัวอย่าง C — มีคำขอที่ไม่อยู่ในรายการ

```text
Please update the company address on my account.
```

บันทึกผลลงตารางด้วยมือ:

| ตัวอย่าง | intent ที่เลือก | intent confidence | complexity score/confidence | urgent probability | route ที่โค้ดเลือก |
|---|---|---:|---:|---:|---|
| A |  |  |  |  |  |
| B |  |  |  |  |  |
| C |  |  |  |  |  |

อย่าตัดสินระบบจากตัวอย่างเดียว หากจะประเมินคุณภาพ ให้เตรียมข้อมูลที่มี label จริง เปรียบเทียบกับ baseline และดูผลแยกตามช่วงความมั่นใจ ก่อนเลือก threshold [4][5]

## บทที่ 8 — แบบฝึกปลายทาง (15–20 นาที)

ขยาย lab ให้รองรับข้อความตั๋วจาก list อย่างน้อย 5 ข้อ แล้วทำให้โปรแกรม:

- ใช้ Choice แยก `order_status`, `product_question`, `return_exchange`, `complaint`, `other`
- ใช้ Score ประเมิน complexity 3 ระดับ พร้อมคำอธิบายแต่ละระดับ
- ใช้ Noul ตรวจ urgency
- ถามทุกคำถามใน request เดียวเมื่อใช้ state เดียวกัน [2]
- ส่ง `order_status` ไปยังเส้นทาง deterministic (เพียงพิมพ์ข้อความจำลอง)
- ส่ง `product_question` ไปยัง specialist LLM (พิมพ์ข้อความจำลอง)
- ส่ง confidence ต่ำ, intent ที่ไม่รู้จัก หรือ complaint ที่ซับซ้อน ไป `human_review`

เกณฑ์ผ่าน:

1. ใช้ typed answer fields ไม่ parse ข้อความธรรมชาติจากคำตอบ
2. โค้ดกำหนด route และ side effects เอง
3. ทุกกรณีมี fallback ที่ปลอดภัยเมื่อ confidence ต่ำหรือ intent ไม่อยู่ใน mapping
4. โปรแกรมไม่ execute action กับลูกค้าหรือบัญชีจริง
5. เพิ่มตัวอย่างทดสอบที่คาดหวังเส้นทางสำหรับข้อความง่าย/ซับซ้อนอย่างน้อย 1 กรณีต่อแบบ

## ตรวจความเข้าใจ

ตอบคำถามเหล่านี้ด้วยคำพูดของคุณเอง:

1. เมื่อไรควรใช้ `Noul` แทน `Score`?
2. `Choice.confidence` กับ probability ของตัวเลือกที่ชนะเป็นค่าเดียวกันหรือไม่?
3. ทำไมควรถามเรื่อง intent และ complexity แยกกัน แทนคำถามกว้าง ๆ ว่า “ควรทำอย่างไรกับข้อความนี้”?
4. ถ้า confidence สูง แต่ action มีความเสี่ยงสูง ควรให้โค้ดทำอะไรเพิ่ม?
5. ถ้าระบบมีเงื่อนไขแน่นอนที่เขียนด้วย Python ได้ ควรให้โมเดลเป็นผู้ตัดสินหรือไม่?

คำตอบแนวทาง: ใช้คำถามแคบและ atomic; เลือก primitive ตามชนิดคำตอบ [2]. ใช้ code คุม workflow [5]. ตั้ง threshold ตามความเสี่ยงและทดสอบกับข้อมูลจริงก่อน production [4][7].

## Troubleshooting เบื้องต้น

- `ModuleNotFoundError: typesafe_sdk`: ตรวจว่า activate `.venv` แล้วติดตั้ง `typesafe-sdk` ใน environment เดียวกัน [1]
- authentication error: ตรวจว่าตั้ง `TYPESAFE_API_KEY` ถูกต้องใน terminal session ปัจจุบัน อย่าแปะ key ในแชตหรือไฟล์
- ได้ `KeyError`: ตรวจชื่อ question ID ใน `questions` ให้ตรงกับ `response.answers[...]`
- ผลลัพธ์ต่างจากตัวอย่าง: เป็นไปได้ ผลลัพธ์ตัวอย่างในเอกสารใช้เพื่อแสดงรูปแบบ ไม่ใช่ค่าที่รับประกัน [1]
- คะแนนหรือ confidence ต่ำ: อย่าฝืน route อัตโนมัติ ปรับข้อมูล/เกณฑ์คำถาม หรือส่งต่อให้คนตรวจ
- ใช้ภาษาไทย: ทำได้ แต่ควรสร้างชุดทดสอบภาษาไทยและวัดคุณภาพเอง เนื่องจากเอกสารระบุว่าภาษาอังกฤษเป็นภาษาฝึกหลัก [3]

## แหล่งอ้างอิง

เอกสารนี้สรุปและดัดแปลงขั้นตอนจากคู่มือ TypeSafe AI โดยคงการอ้างอิงไว้หลังเนื้อหาที่เกี่ยวข้อง ตรวจเอกสารต้นทางอีกครั้งก่อนใช้งานจริง เพราะ SDK, โมเดล และ API อาจเปลี่ยนแปลงได้

[1] Quick start และตัวอย่าง Python SDK
[2] Primitives: Choice, Score, Noul และการถามหลายข้อ
[3] State: การออกแบบ state และข้อสังเกตด้านภาษา
[4] Confidence: วิธีคิดเรื่อง confidence
[5] How to build with TypeSafe: ออกแบบ workflow โดยให้ code ควบคุม
[6] Intent routing: เส้นทาง deterministic / specialist / human
[7] Confidence-gated routing: ปรับเกณฑ์ตามความเสี่ยง

## Sources

[1] https://docs.typesafe.ai/introduction/quickstart — Quick start
[2] https://docs.typesafe.ai/primitives.md — Primitives (Questions)
[3] https://docs.typesafe.ai/concepts/state.md — State
[4] https://docs.typesafe.ai/confidence.md — Confidence
[5] https://docs.typesafe.ai/concepts/how-to-build-with-system-one.md — How to build with TypeSafe
[6] https://docs.typesafe.ai/patterns/intent-routing.md — Intent routing
[7] https://docs.typesafe.ai/patterns/confidence-routing.md — Confidence-gated routing
