# ผลการดีบั๊กและการทดสอบระบบจริง (DEBUG & EXECUTION RESULTS)
**โฟลเดอร์**: `DL-05-RAG System Development II`  
**สถานะการทำงาน**: ✅ แก้ไขบั๊กทั้งหมดเรียบร้อย และทดสอบผ่านทุกฟังก์ชัน (100% Passed)

---

## 1. รายงานบั๊กที่พบก่อนการแก้ไข (Bugs & Root Causes Identified)

### 🐛 บั๊กที่ 1: `ModuleNotFoundError: No module named 'problem10_debug_scripts'`
* **Error Log ที่เกิดขึ้น**:
  ```text
  Traceback (most recent call last):
    File "DL-05-RAG System Development II\main.py", line 21, in <module>
      from problem10_debug_scripts import run as problem10
  ModuleNotFoundError: No module named 'problem10_debug_scripts'
  ```
* **สาเหตุ**: โค้ดต้นฉบับในคอมมิตเก่ามีการลบไฟล์ `problem10_debug_scripts.py` ออกไป แต่ยังคงมีคำสั่ง `import` และเรียกใช้งานในเมนู `main.py`
* **การแก้ไข**: ลบ import ที่ตกค้าง และปรับขนาดเมนูให้เป็น 9 ข้อ (0-9) สอดคล้องกับไฟล์จริงและ `README.md`

---

### 🐛 บั๊กที่ 2: `UnicodeEncodeError: 'charmap' codec can't encode characters`
* **Error Log ที่เกิดขึ้น**:
  ```text
  Traceback (most recent call last):
    File "problem01_hallucination.py", line 36, in run
      print(f"--- Query ({label}): {q}")
    File "encodings\cp1252.py", line 19, in encode
      return codecs.charmap_encode(input,self.errors,encoding_table)[0]
  UnicodeEncodeError: 'charmap' codec can't encode characters: character maps to <undefined>
  ```
* **สาเหตุ**: Windows Terminal ใช้ Default Charset เป็น `cp1252` ทำให้เกิดข้อผิดพลาดทันทีที่มีการ `print()` ข้อความภาษาไทย
* **การแก้ไข**: เพิ่ม `sys.stdout.reconfigure(encoding="utf-8")` และ `sys.stderr.reconfigure(encoding="utf-8")` ที่ส่วนหัวของ `main.py`

---

### 🐛 บั๊กที่ 3: บั๊กการซ่อน Error ด้วย `except ValueError:`
* **สาเหตุ**: `UnicodeEncodeError` สืบทอดมาจาก `ValueError` ในภาษา Python ทำให้เวลาโค้ดพ่น error ภาษาไทย ระบบดันไปเข้าบล็อก `except ValueError:` แล้วแสดงข้อความหลอกว่า *"Please enter a number 0-10 or Q"*
* **การแก้ไข**: แยกการแปลง `int(arg)` ออกมาจากบล็อกการรันฟังก์ชัน `execute()`

---

## 2. ผลลัพธ์การรันจริงของทุกปัญหา (Actual Execution Output: `python main.py 0`)

ผลลัพธ์จากการสั่งรันจำลองสถานการณ์ปัญหาทั้งหมด (ข้อ 1 ถึง 9) ผ่านคำสั่ง `python main.py 0`:

```text
*****************************************************************
PROBLEM 1: Hallucination / Context
*****************************************************************
--- Query (Out of KB scope): ตั๋วเครื่องบินไปเชียงใหม่ราคาเท่าไหร่
Retrieved: Not found
Bad answer : กินยาคุมพร้อมน้ำอัดลมช่วยให้ฤทธิ์คุมกำเนิดแรงขึ้น (ข้อมูลนี้ไม่มีอยู่จริงใน Knowledge Base)
Fixed answer: ไม่พบข้อมูลที่สนับสนุนคำตอบใน Knowledge Base

--- Query (In KB): PrEP คืออะไร
Retrieved: ['PrEP คืออะไร และเหมาะกับใคร', 'PEP คืออะไร ต่างจาก PrEP อย่างไร', 'เพร็พคืออะไร ต้องกินตลอดชีวิตไหม']
Bad answer : PrEP (Pre-Exposure Prophylaxis) คือยาต้านไวรัสที่กินก่อนสัมผัสเชื้อเพื่อป้องกันการติดเชื้อเอชไอวี เมื่อกินสม่ำเสมอจะลดความเสี่ยงติดเชื้อจากเพศสัมพันธ์ได้มากกว่า 99%...
Fixed answer: PrEP (Pre-Exposure Prophylaxis) คือยาต้านไวรัสที่กินก่อนสัมผัสเชื้อเพื่อป้องกันการติดเชื้อเอชไอวี เมื่อกินสม่ำเสมอจะลดความเสี่ยงติดเชื้อจากเพศสัมพันธ์ได้มากกว่า 99%...

Cause: The generator answers even though the Retrieved Context has no supporting evidence
e.g. a question that is completely outside the scope of the Knowledge Base (sex_q_a.txt)

*****************************************************************
PROBLEM 2: Transformer / Position / Attention
*****************************************************************
Formal-register question: ถุงยางอนามัยป้องกันได้ผลแค่ไหน
Slang-register question : ถุงยางมันหลุดคาในหี ทำไงดี
BoW (formal): {'ถุงยางอนามัยป้องกันได้ผลแค่ไหน': 1}
BoW (slang) : {'ถุงยางมันหลุดคาในหี': 1, 'ทำไงดี': 1}
Exact-token overlap: None
-> Both questions are about the same topic (a problem with a condom during use)
   but Keyword/BoW barely overlaps because the wording differs (Vocabulary Mismatch)

Example: effect of Token order on meaning (Position):
A (original): [(0, 'ถุงยางอนามัยเป็นวิธีเดียวที่ป้องกันทั้งการตั้งครรภ์และโรคติดต่อทางเพศสัมพันธ์ไปพร้อมกัน'), (1, 'เมื่อใช้ถูกวิธีทุกครั้งจะป้องกันการตั้งครรภ์ได้ราว'), (2, '98%'), (3, 'แต่ในการใช้จริงที่มีความผิดพลาดปนอยู่จะอยู่ราว'), (4, '87%'), (5, 'การป้องกัน'), (6, 'STI'), (7, 'ได้ผลสูงมากสำหรับเชื้อที่ติดต่อผ่านสารคัดหลั่ง')]
B (reversed order): [(0, 'ได้ผลสูงมากสำหรับเชื้อที่ติดต่อผ่านสารคัดหลั่ง'), (1, 'STI'), (2, 'การป้องกัน'), (3, '87%'), (4, 'แต่ในการใช้จริงที่มีความผิดพลาดปนอยู่จะอยู่ราว'), (5, '98%'), (6, 'เมื่อใช้ถูกวิธีทุกครั้งจะป้องกันการตั้งครรภ์ได้ราว'), (7, 'ถุงยางอนามัยเป็นวิธีเดียวที่ป้องกันทั้งการตั้งครรภ์และโรคติดต่อทางเพศสัมพันธ์ไปพร้อมกัน')]
BoW identical: True

Cause: BoW does not capture vocabulary variation and does not preserve word order
Transformers use Positional Information + Self-Attention and semantic Embeddings
to match questions that use different wording but share the same intent (e.g. formal vs slang)

*****************************************************************
PROBLEM 3: Data Quality
*****************************************************************
Before cleaning (simulated noise on real questions from the KB):
'โรคติดต่อทางเพศสัมพันธ์ (STI) คืออะไร มีอะไรบ้าง'
'โรคติดต่อทางเพศสัมพันธ์ (STI) คืออะไร มีอะไรบ้าง'
'   โรคติดต่อทางเพศสัมพันธ์ (STI) คืออะไร มีอะไรบ้าง   '
'โรคติดต่อทางเพศสัมพันธ์_(STI)_คืออะไร_มีอะไรบ้าง!!!'
'ทำไม STI หลายชนิดจึงไม่แสดงอาการ'
'ทำไม STI หลายชนิดจึงไม่แสดงอาการ'
'   ทำไม STI หลายชนิดจึงไม่แสดงอาการ   '
'ทำไม_STI_หลายชนิดจึงไม่แสดงอาการ!!!'
''

After Normalization + Deduplication:
'โรคติดต่อทางเพศสัมพันธ์ (sti) คืออะไร มีอะไรบ้าง'
'ทำไม sti หลายชนิดจึงไม่แสดงอาการ'

Count before: 9
Count after: 2

Checked the full sex_q_a.txt file (391 entries): found 0 duplicate question(s) after normalization
Cause: Duplicates add redundant data, while noise makes text inconsistent enough that the system treats identical entries as different ones

*****************************************************************
PROBLEM 4: Chunk Size / Overlap
*****************************************************************
Sample document: combined answers from category 'สุขภาพทางเพศและการป้องกัน' (844 words)

Large chunk (size=200):
- โรคติดต่อทางเพศสัมพันธ์ (Sexually Transmitted Infections หรือ STI) คือการติดเชื้อที่ส่งผ่านทางการมีเพศสัมพันธ์ทั้งทางช่องคลอด ทวารหนัก และปาก โรคที่พบ...
- และการตรวจแอนติบอดีอย่างเดียวรวมถึงชุดตรวจด้วยตนเองราว 3 สัปดาห์ถึง 3 เดือน จึงควรตรวจซ้ำเพื่อยืนยันเมื่อพ้นระยะฟักตัว คนไทยทุกสิทธิการรักษาสามารถตรวจ...

Small chunk (size=15):
- โรคติดต่อทางเพศสัมพันธ์ (Sexually Transmitted Infections หรือ STI) คือการติดเชื้อที่ส่งผ่านทางการมีเพศสัมพันธ์ทั้งทางช่องคลอด ทวารหนัก และปาก โรคที่พบบ่อยได้แก่ หนองในแท้ (gonorrhea) หนองในเทียม (chlamydia) ซิฟิลิส
- (syphilis) เริมที่อวัยวะเพศ (genital herpes) หูดหงอนไก่จากเชื้อ HPV ไวรัสตับอักเสบบี พยาธิในช่องคลอด (trichomoniasis) และเอชไอวี...

Chunk + Overlap (size=60, overlap=15):
- โรคติดต่อทางเพศสัมพันธ์ (Sexually Transmitted Infections หรือ STI) คือการติดเชื้อที่ส่งผ่านทางการมีเพศสัมพันธ์ทั้งทางช่อ...
- ควรตรวจเพิ่มเติมทันทีเมื่อมีอาการผิดปกติ เมื่อคู่นอนตรวจพบเชื้อ หรือก่อนเริ่มความสัมพันธ์ใหม่ที่จะเลิกใช้ถุงยาง สัญญาณที...
- ผูกปากถุงแล้วทิ้งลงถังขยะ ใช้ครั้งเดียวเสมอและใช้เพียงชิ้นเดียวต่อครั้ง การใส่ซ้อนกันสองชั้นทำให้ผิวยางเสียดสีกันเอง เพิ...

Cause:
- Too large: multiple Q&A pairs get mixed into a single chunk, diluting the Embedding
- Too small: an explanation may be cut off mid-sentence, losing context
- Overlap: preserves context at the boundary between chunks

*****************************************************************
PROBLEM 5: Metadata Filtering
*****************************************************************
Query: ถุงยาง หลุด ทำไง

Without filtering by Metadata (lang) -> picks the document with the most keyword matches, ignoring tone:
  [แสลง] ถุงยางมันหลุดคาในหี ทำไงดี

Filter lang='ทางการ' (want a formal-tone answer for a clinical chatbot):
  [ทางการ] ถุงยางอนามัยแตกหรือหลุดระหว่างมีเพศสัมพันธ์ ควรทำอย่างไร

Cause: Keyword/Semantic Similarity picks the document with the most matching words
but does not guarantee the document has the correct Metadata (e.g. register/tone) the system needs

*****************************************************************
PROBLEM 6: Top-k / Re-ranking
*****************************************************************
Before Re-ranking (Top 6 from First-stage: generic terms 'เอชไอวี','ตรวจ'):
  score=2 | ถุงยางอนามัยแตกหรือหลุดระหว่างมีเพศสัมพันธ์ ควรทำอย่างไร
  score=2 | PrEP คืออะไร และเหมาะกับใคร
  score=2 | U=U หมายความว่าอย่างไร
  score=2 | ระยะฟักตัว (window period) ของการตรวจเอชไอวีคืออะไร
  score=2 | คนไทยตรวจเอชไอวีฟรีได้ที่ไหนบ้าง
  score=2 | หากถูกล่วงละเมิดทางเพศ ควรทำอย่างไรในทันที

After Re-ranking (extra weight for specific terms, e.g. 'ระยะฟักตัว'):
  score=8 | ระยะฟักตัว (window period) ของการตรวจเอชไอวีคืออะไร
  score=2 | ถุงยางอนามัยแตกหรือหลุดระหว่างมีเพศสัมพันธ์ ควรทำอย่างไร
  score=2 | PrEP คืออะไร และเหมาะกับใคร
  score=2 | U=U หมายความว่าอย่างไร
  score=2 | คนไทยตรวจเอชไอวีฟรีได้ที่ไหนบ้าง
  score=2 | หากถูกล่วงละเมิดทางเพศ ควรทำอย่างไรในทันที

Cause: First-stage Retrieval weighs generic terms equally, so the document that is most
specific and best matches the query can end up ranked near the bottom
Re-ranking uses finer-grained signals to push the relevant document toward the top

*****************************************************************
PROBLEM 7: Retrieval Correct, Generation Wrong
*****************************************************************
Question: PEP คืออะไร ต่างจาก PrEP อย่างไร

Retrieved Context:
PEP (Post-Exposure Prophylaxis) คือยาต้านไวรัสที่กินหลังสัมผัสเชื้อแล้ว ใช้ในกรณีฉุกเฉิน เช่น ถุงยางแตก ถูกล่วงละเมิดทางเพศ หรือมีเพศสัมพันธ์โดยไม่ป้องกันกับผู้ที่ไม่ทราบสถานะ ต้องเริ่มยาเร็วที่สุดและไม่เกิน 72 ชั่วโมงหลังเสี่ยง แล้วกินต่อเนื่อง 28 วัน...

Bad Generation (distorts the critical time window):
PEP (Post-Exposure Prophylaxis) คือยาต้านไวรัสที่กินหลังสัมผัสเชื้อแล้ว ใช้ในกรณีฉุกเฉิน... ต้องเริ่มยาเร็วที่สุดและภายใน 7 วันหลังเสี่ยง แล้วกินต่อเนื่อง 28 วัน...

Grounded Generation (sticks to the original Context):
PEP (Post-Exposure Prophylaxis) คือยาต้านไวรัสที่กินหลังสัมผัสเชื้อแล้ว ใช้ในกรณีฉุกเฉิน... ต้องเริ่มยาเร็วที่สุดและไม่เกิน 72 ชั่วโมงหลังเสี่ยง แล้วกินต่อเนื่อง 28 วัน...

Cause: Retrieval is correct, but the Generator changes a critical detail (the PEP start-time window)
In a health domain this kind of error can be dangerous; the Prompt must force answers to come from Context only
and Faithfulness must be evaluated before sending the answer to the user

*****************************************************************
PROBLEM 8: RAG Configuration
*****************************************************************
CONFIG = {'KB_SOURCE': 'sex_q_a.txt', 'USE_HYBRID': True, 'USE_RERANK': False, 'USE_MEMORY': True, 'USE_LLM': True, 'SHOW_SOURCES': False}
Loaded Knowledge Base from sex_q_a.txt: 391 Q&A entries

Pipeline:
1. Using Conversation Memory
2. Using BM25 + Dense Retrieval on 391 documents
3. Not performing Re-ranking
4. Generating the answer with an LLM (grounded in Context from sex_q_a.txt)
5. Not showing Sources

The system's behavior results from toggling each Component on/off in the Config

*****************************************************************
PROBLEM 9: Chunk & Retrieval Evaluation
*****************************************************************
Real document from sex_q_a.txt is 133746 characters long (391 Q&A)
Number of chunks produced (CHUNK_SIZE=400, OVERLAP=50): 383
First few chunk ranges:
(0, 400)
(350, 750)
(700, 1100)
(1050, 1450)
(1400, 1800)

Overlap Chunk 1/2 = 50

Retrieval Evaluation:
Golden Set = 391 (uses every question in sex_q_a.txt as the test set)
Check whether the Relevant Document appears within Top-1
Check whether the Relevant Document appears within Top-3
Check whether the Relevant Document appears within Top-5
Check whether the Relevant Document appears within Top-10

Example:
Relevant Document is at Rank 8
Top-5 = not found, Top-10 = found
This means the Retriever can find the document, but the ranking is not yet good at the top
```
