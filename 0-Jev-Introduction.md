# 0 — Jev Introduction: Jev คืออะไร และมี Mode อะไรบ้าง?

เอกสารปูพื้นก่อนเริ่ม Hands-on ในไฟล์ 1–4: อธิบาย Jev, System One, primitives ทั้งสาม และความต่างระหว่าง “ชนิดคำถาม” กับ “ชื่อ model”

## Jev คืออะไร

**Jev** คือโมเดลของ TypeSafe ในกลุ่ม **System One** ที่ออกแบบมาให้ซอฟต์แวร์ใช้ตัดสินใจแบบมีโครงสร้าง แทนการสร้างคำตอบเป็นย่อหน้าข้อความยาว ๆ [1][2]

แอปส่งข้อมูลที่ต้องการให้ประเมิน เรียกว่า **State** พร้อมคำถามแบบมีชนิด (typed questions) แล้ว Jev ส่งคำตอบกลับเป็นค่า เช่น ตัวเลือก คะแนน หรือความน่าจะเป็น ซึ่งโปรแกรมสามารถอ่านได้โดยตรง [1][2]

ภาพรวมการทำงาน:

```text
State + Questions ที่ระบุชนิด
            ↓
           Jev
            ↓
Typed answers + probabilities / confidence
            ↓
Application code เลือก workflow, ขอ human review หรือดำเนินขั้นต่อไป
```

Jev ไม่ใช่ chatbot หรือ coding agent: ตัวมันเองไม่ได้เขียนคำตอบสนทนา, สร้างโค้ด, หรือเลือก action ถัดไป แต่ตอบการตัดสินใจที่กำหนดไว้ ส่วน application code เป็นผู้ควบคุมการรวมคำตอบ, เงื่อนไข และ side effects [2][5]

## State และ Questions

- **State** คือเนื้อหาหรือบริบทที่ต้องการให้ประเมิน อาจเป็นข้อความ, JSON object หรือ array ของข้อมูลข้อความ
- **Questions** คือรายการคำถามที่ระบุ ID, ชนิด และ instructions; บางชนิดมี criteria กำหนดตัวเลือกหรือระดับคะแนนด้วย
- หนึ่ง request ใช้ State หนึ่งชุดกับหนึ่งหรือหลาย Questions; ทุกคำถามเห็น State เดียวกันและถูกประเมินแยกกัน [2][4]
- ผลลัพธ์จะถูกจัดกลับตาม question ID ที่กำหนดไว้ [4]

## 3 Modes / Question Types (Primitives)

ในเอกสาร TypeSafe มี question types หลัก 3 แบบ เรียกว่า primitives ไม่ใช่โหมดแชตที่สลับบุคลิกของโมเดล [1][4]

| Mode / Primitive | ใช้ถามอะไร | คำตอบหลัก |
|---|---|---|
| **Noul** | เป็นจริงหรือไม่? คำถามแบบ Yes/No | `noul` ความน่าจะเป็นของคำตอบ “ใช่” ตั้งแต่ 0–1; ไม่มี `confidence` แยก |
| **Choice** | เป็นข้อไหนจากรายการที่กำหนด? | `choice`, `probabilities`, `confidence` |
| **Score** | อยู่ระดับใดบนสเกลที่เรียงลำดับ? | `score`, `legend`, `probabilities`, `confidence` |

เลือกชนิดให้ตรงกับรูปคำตอบ: ใช้ Noul กับเงื่อนไขใช่/ไม่ใช่, Choice กับหมวดหมู่ที่ไม่เรียงลำดับ และ Score กับระดับที่เรียงจากต่ำไปสูง [3][4]

### เข้าใจผลลัพธ์ให้ถูก

- `noul` ใกล้ 1 หมายถึงแบบจำลองมีความน่าจะเป็น “ใช่” สูง; ใกล้ 0 หมายถึง “ไม่ใช่” สูง และใกล้ 0.5 คือไม่ชัดระหว่าง yes/no [3]
- `choice` คือ option ที่ได้ probability สูงสุด; `probabilities` แสดงการกระจายข้ามตัวเลือก และ `confidence` สรุปความชัดของ distribution [4]
- `score` เป็นตำแหน่งบนลำดับระดับ อาจเป็นค่าทศนิยมได้; อ่าน `legend`, `probabilities` และ `confidence` ประกอบ ไม่สรุปจากคะแนนตัวเดียว [4]
- Probability หรือ confidence เป็นสัญญาณความไม่แน่นอน ไม่ใช่หลักประกันว่าคำตอบรายกรณีถูกต้อง [2]

## “มีอีก Mode หรือ Model ไหม?”

คำว่า **Mode** ในชุด Hands-on นี้หมายถึง Noul, Choice และ Score ซึ่งเป็นชนิดคำถามสามแบบ [4] ส่วน **Model** คือโมเดลที่ประมวลผล request

ในหน้า Models ของ TypeSafe ระบุ Jev เป็น flagship และ System One model แรก; `jev-latest` เป็น alias สำหรับ stable release ส่วน `jev-preview` เป็น alias แยกอีกชื่อ โดยเอกสารระบุว่าในขณะนั้นชี้ไปยัง version เดียวกัน (`jev-1.13.0`) และไม่มี preview build แยก [2][3] Alias/ชื่อเวอร์ชันจึงไม่ใช่ Mode ใหม่

ชื่อที่บัญชีใช้งานได้อาจตรวจจาก `GET /v1/models` ตามเอกสาร Models [3] ควรตรวจรายการและเอกสารล่าสุดก่อนทำ PoC เพราะ alias อาจชี้ไป release ใหม่ได้

## Jev ใช้แทน LLM หรือ Coding Agent ได้ไหม

ไม่ใช่การแทนกันโดยตรง: LLM/chat model เหมาะกับการสร้างข้อความหรืออธิบายแบบปลายเปิด ส่วน Jev เหมาะกับการประเมินเฉพาะด้านและคืนผลแบบมีโครงสร้างให้โค้ดใช้ [1][5]

ตัวอย่างการวางร่วมกัน:

```text
LLM / Agent: สนทนา อธิบาย สรุป หรือวางแผนงาน
Jev:       จัดประเภท ตรวจ yes/no หรือให้คะแนนตาม rubric
Application code: รวมผล คุมเงื่อนไข อนุมัติหรือส่งต่อให้คน
```

ถ้าต้องทำ coding-agent integration ให้ใช้ coding agent เขียนโปรแกรมที่เรียก Jev; อย่าคาดหวังว่าตั้งชื่อโมเดล `jev-latest` แล้ว coding agent จะกลายเป็น Jev [5]

## แล้ว Java มีไหม?

มี repository Java ที่ผู้ใช้ยกมาประกอบเนื้อหาชุดนี้: [`jamilxt/typesafe-ai-java`](https://github.com/jamilxt/typesafe-ai-java) โดย repo ระบุชัดว่าเป็น **community-maintained** ไม่ใช่ผลิตภัณฑ์หรือ SDK ทางการของ TypeSafe [6]

ก่อนนำไปทดลอง ให้ตรวจสถานะล่าสุดของ repository, Java toolchain ที่ต้องใช้, dependencies, license และความเหมาะสมกับ environment ของทีม; อย่าถือว่า community SDK ผ่านการรับรอง production โดยอัตโนมัติ [6] สำหรับการทดลอง Playground ในไฟล์ถัดไปไม่จำเป็นต้องใช้ Java SDK

## เส้นทาง Hands-on

1. [1-Jev-Beginner.md](1-Jev-Beginner.md) — ฝึกพื้นฐาน Noul, Choice, Score ด้วยตัวอย่างสั้น ๆ
2. [2-Jev-SoftwareDeveloper-DataScientist.md](2-Jev-SoftwareDeveloper-DataScientist.md) — ตัวอย่างงาน IT, Developer และ Data Science
3. [3-Jev-NeoWork.md](3-Jev-NeoWork.md) — ฝึกประเมินเคสจำลองของ NeoWork
4. [4-Jev-NeoWork-Advanced.md](4-Jev-NeoWork-Advanced.md) — รวมทั้ง 3 modes ใน State และ Questions ชุดเดียว

## Sources

[1] https://docs.typesafe.ai/introduction — TypeSafe Introduction
[2] https://docs.typesafe.ai/concepts/system-one — System One
[3] https://docs.typesafe.ai/models — Models
[4] https://docs.typesafe.ai/primitives — Primitives
[5] https://docs.typesafe.ai/introduction/coding-agents — Jev with coding agents
[6] https://github.com/jamilxt/typesafe-ai-java — Community Java SDK
