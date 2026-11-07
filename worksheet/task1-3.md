# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** Putthitorn Chalitchaiya

**Student ID / รหัสนิสิต:** 6870197421

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 490 | 594 | 93.5% | 100% |
| t0-small | 96 | 221 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> large แม่นกว่า small มาก (93.5% กับ 72.5%) แต่ช้ากว่าเกือบ 5 เท่า (p50 490 กับ 96 ms) และทุกรีวิวต้องเรียก large (large calls 100%) เลยแพงกว่า large ผ่าน QR3 ที่ต้องได้อย่างน้อย 90% ส่วน small ไม่ผ่าน

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | ~31.8 | 252 | 411 | 21 | 0 | 92.7% |
| t1-after | 9.8 | 421 | 1016 | 0 | 22 | 93.3% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> p50 คิดจาก request ที่ได้คำตอบเท่านั้น 21 request ที่ timeout ไม่ถูกนับรวม ตัวเลขเลยดูดีเกินจริง ต้องดู `errors` (21) กับเวลารวมที่ใช้รัน ซึ่งนานถึงประมาณ 31.8 วินาที เทียบกับ t1-after ที่ใช้ 9.8 วินาที

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> call ที่ค้างไม่ได้กิน slot ของ large model แต่ request นั้นค้างอยู่ใน gateway กับ review service นาน 20 วินาที และกิน 1 ใน 10 ช่องที่ client ส่งพร้อมกันได้ พอค้างหลายตัวพร้อมกัน request ใหม่ก็เข้าไม่ได้ ทั้งระบบเลยช้าลง ไม่ใช่แค่ 10% ที่ช้า

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> เอารีวิว pending ไปเข้าคิวไว้ แล้วให้ worker มาติด label ทีหลังตอนที่ large model ว่าง ซึ่งก็คือ T4 (asynchronous/batch) ใน t1-after มี pending 22 รายการ แต่ errors เป็น 0 เพราะไม่ต้องรอ label ก่อนตอบผู้ใช้

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> ถ้าตั้ง 0.1 วินาที จะสั้นกว่าเวลาปกติของ large model (ประมาณ 200 ms) call ที่ไป large แทบทุกตัวจะ timeout แล้วกลายเป็น pending ทั้งที่โมเดลไม่ได้เสีย ถ้าตั้ง 10 วินาที จะเกิน QR2 ที่ให้รอไม่เกิน 1 วินาที ผู้ใช้ต้องรอนาน และ request ที่ค้างจะกินทรัพยากรเหมือนใน t1-before

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 28 | 535 | 91.5% | 38% |
| t2-0.7 | 0.7 | 360 | 613 | 93.0% | 53% |
| t2-0.8 | 0.8 | 452 | 643 | 93.5% | 64% |
| t2-0.9 | 0.9 | 461 | 574 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> เลือก 0.6 เพราะ accuracy 91.5% ยังผ่าน QR3 และเรียก large แค่ 38% ซึ่งน้อยที่สุด ตรงกับ QR4 สิ่งที่ยอมเสียคือ accuracy ประมาณ 2% เมื่อเทียบกับ 0.8 (93.5%)

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> รีวิวง่ายจบที่ small เลยเร็วมาก p50 จึงลดลงเยอะ แต่ยังมีรีวิวที่ต้องไป large มากกว่า 5% (38–77%) request ที่ช้าที่สุด 5% ก็ยังเป็นพวกที่ไปรอ large อยู่ดี p95 เลยแทบไม่ลด

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> เช่น "Honestly the salad is not cold." เป็นรีวิวบวก แต่ small ตอบ negative ด้วย confidence 0.702 ถ้าใช้ threshold 0.6 หรือ 0.7 รีวิวนี้จะไม่ถูกส่งไป large เจ้าของร้านจะเห็นว่าลูกค้าไม่พอใจ ทั้งที่จริงๆ ลูกค้าชม

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 79 | 549 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> รีวิวสั้นๆ ที่คนพิมพ์ซ้ำกันบ่อย เช่น "Delicious!", "Too salty", "Cold food" ได้ประโยชน์ เพราะเจอครั้งที่สองก็ใช้ label เดิมได้เลย ส่วนรีวิวยาวที่เขียนไม่ซ้ำใครจะไม่มีวัน hit ใน t3-cache hit 35% ทำให้ large calls ลดจาก 53% เหลือ 38% และ p50 จาก 360 เหลือ 79 ms

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> ต้องล้าง cache ของโมเดลเก่าทิ้ง ไม่งั้นจะยังได้ label จากโมเดลเก่าอยู่ ถ้าออกแบบไว้ล่วงหน้า ให้ใส่ model version ไว้ใน cache key พอเปลี่ยนโมเดล key ก็เปลี่ยนตาม และตั้ง TTL ไว้ด้วยให้ข้อมูลเก่าหมดอายุเอง

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> ที่แปลกใจคือ cascade 0.7 กับ cache ยังได้ accuracy 93.0% แต่เรียก large แค่ 38% เท่ากับ 0.6 ที่แม่นน้อยกว่า แปลว่าไม่จำเป็นต้องใช้โมเดลที่แม่นที่สุดกับทุกรีวิว ต้องดู accuracy, latency และ cost ไปพร้อมกัน
