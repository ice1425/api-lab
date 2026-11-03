# API lab results

## Part 0

- Step 0.1, the number of pizza names: 18
- Step 0.3, status code of the request without a token: 401
- Step 0.4, the name of your pizza: The Bold Quattro Stagioni
- Step 0.12, results of the three runs without a session (optional): Run 1: passed, Run 2: failed (401), Run 3: failed (401)
- Step 0.16, blocks of "how many of the three tools are excluded": 0 tools ([]), 1 tool (["Knife"], ["Scissors"]), 2 tools (["Knife", "Scissors"]), 3 tools (["Knife", "Pizza cutter", "Scissors"]). The original test covered only the 1-tool block. The 3-tool block revealed the defect.

## Defect report

- Title: Excluding all three tools returns 500 with an empty body
- Request: POST https://quickpizza.grafana.com/api/pizza
  Header: Authorization: token <token of a logged-in user>
  Body: {"excludedTools": ["Knife", "Pizza cutter", "Scissors"]}
- Expected result: Status 400 Bad Request with a JSON error message, because no pizza can satisfy the request
- Actual result: Status 500 Internal Server Error with an empty body
- Severity: Medium. A normal customer input crashes the request handler, and the app that calls the API gets no message to show or handle. The rest of the API still works, so it is not High.

## Part A

- Step A4, status codes when the other user reads, changes and deletes: read 404 {"error":"not found"}, change 400 {"error":"rating ID <id> not found"}, delete 400 {"error":"rating ID <id> not found"}. Every attempt was refused, and the owner read the rating back unchanged.

## Part B

1. ความล้มเหลวสองแบบที่ test กับเซิร์ฟเวอร์จริงหาเจอ แต่ unit test ที่ใช้ stub หาไม่เจอ
   (ก) ปัญหาของโครงสร้างระบบที่อยู่ระหว่างส่วนต่างๆ เช่น load balancer ที่ส่ง request ไปคนละเซิร์ฟเวอร์ ใน Step 0.12 การ login ล้มเหลวด้วย 401 เมื่อไม่ได้เก็บ cookie AWSALB
   (ข) ปัญหาในการที่เซิร์ฟเวอร์จริงแปลงและเก็บข้อมูล เช่น error ของ Go JSON ใน Step A2 และ error 500 ใน Step 0.16 เพราะ stub จะตอบแค่สิ่งที่เรากำหนดไว้ จึงไม่มีทางเจอปัญหาเหล่านี้
   เหตุผลที่ยังควรเขียน unit test ที่ใช้ stub: รันได้เร็ว ไม่ขึ้นกับอินเทอร์เน็ตหรือเซิร์ฟเวอร์กลางว่าล่มหรือไม่ ชี้ได้แม่นว่าฟังก์ชันไหนผิด และทดสอบกรณีที่สร้างผ่าน API ได้ยาก

2. rating ที่สร้างด้วย token สาธารณะเป็นของทุกคน นักศึกษาคนอื่นอาจแก้หรือลบระหว่างที่ test ของเรากำลังรัน ทำให้ test fail ด้วยเหตุผลที่ไม่เกี่ยวกับโค้ดของเรา การสร้าง user ใหม่ด้วยชื่อใหม่ทำให้แต่ละ test มีข้อมูลของตัวเอง test จึงเป็นอิสระต่อกันและรันซ้ำได้ผลเหมือนเดิม
   ใน Project 2 จะให้แต่ละ test สร้างลูกค้า ตะกร้า หรือการจองของตัวเองด้วยชื่อที่ไม่ซ้ำ แล้วลบทิ้งหลัง test จบด้วย fixture แบบ yield และจะไม่แก้ข้อมูลของกลุ่มอื่น

3. QuickPizza มีเซิร์ฟเวอร์หลายตัวอยู่หลัง load balancer และแต่ละตัวเก็บข้อมูล user แยกกัน session จะเก็บ cookie AWSALB ไว้ ทำให้ทุก request ไปถึงเซิร์ฟเวอร์ตัวเดียวกับที่สร้าง user ถ้าไม่มี session การ login อาจไปถึงเซิร์ฟเวอร์ที่ไม่รู้จัก user นั้น
   ผลการรัน Step 0.12 สามรอบ: รอบที่ 1 ผ่าน รอบที่ 2 และ 3 fail ด้วย 401 ทั้งที่ไม่ได้แก้โค้ดเลย test แบบนี้เรียกว่า flaky test

4. block ที่เจอ defect คือ "ตัดเครื่องมือออกทั้ง 3 ชิ้น"
   500 ที่ body ว่างแย่กว่า เพราะแอปที่เรียก API แยกไม่ออกว่า user กรอกผิดหรือเซิร์ฟเวอร์พัง ไม่มีข้อความให้แสดงกับ user และอาจลองส่ง request ซ้ำไปเรื่อยๆ ทั้งที่ไม่มีทางสำเร็จ ส่วน 400 พร้อมข้อความบอกแอปได้ชัดว่าต้องเปลี่ยน input และควรแจ้ง user ว่าอะไร
   test นี้เก็บไว้เป็น xfail แทนการลบ เพื่อให้ defect ยังแสดงอยู่ในทุกรายงานไม่ถูกลืม suite ที่เหลือยังเขียวอยู่ และ strict=True จะทำให้การรัน fail เมื่อ QuickPizza แก้ defect แล้ว เป็นสัญญาณให้ถอด mark ออก

5. Base choice coverage = test หลัก 1 ตัว + (จำนวน block - 1) ของแต่ละ characteristic
   = 1 + (3-1) + (4-1) + (2-1) + (3-1) = 1 + 2 + 3 + 1 + 2 = 9 test
   All combinations coverage = 3 x 4 x 2 x 3 = 72 test

6. ผลที่ได้: อ่าน 404 {"error":"not found"}, แก้ไข 400 {"error":"rating ID <id> not found"}, ลบ 400 {"error":"rating ID <id> not found"}
   ที่คาดไว้คือ 404 ทั้งสามกรณี เพราะสำหรับ user อื่น rating นี้ควรดูเหมือนไม่มีอยู่ (หรือ 403 Forbidden ก็ยอมรับได้) 400 Bad Request ไม่ถูกต้อง เพราะ request มีรูปแบบถูกต้อง และยังขัดกับข้อความของตัวเองที่บอกว่า "not found"
   ความต่างนี้เป็น defect แต่ความรุนแรงต่ำ เพราะทุกความพยายามถูกปฏิเสธและ rating ไม่เปลี่ยน จึงไม่ใช่ช่องโหว่ด้านความปลอดภัย แต่เป็นความไม่สม่ำเสมอที่ทำให้แอปจัดการ error ได้ยากขึ้น

7. ได้ status 200 พร้อมรายการ rating ประมาณ 50 รายการ (เช่น {"id":1,"stars":5,"pizza_id":1}) ทั้งที่ token 0000000000000000 เป็น token ที่แต่งขึ้นเองและไม่ใช่ของ user คนไหนเลย ดูเหมือน QuickPizza ตรวจแค่ว่า token ยาว 16 ตัวอักษร เพราะ token สั้นใน Step A2 ได้ 401 แต่ token ปลอมที่ยาว 16 ตัวผ่านได้
   นี่คือ defect ด้านความปลอดภัยที่ร้ายแรง เพราะ API ไม่ได้ตรวจว่า token ออกให้ user จริงหรือไม่ ถ้าเป็นระบบจริง คนแปลกหน้าสามารถแต่ง token ขึ้นมาเองแล้วอ่านข้อมูลของผู้ใช้คนอื่นได้โดยไม่ต้อง login และอาจสร้าง แก้ หรือลบข้อมูลได้ด้วย ซึ่งทำให้ระบบยืนยันตัวตนไม่มีความหมาย

8. สิ่งที่ API test ตรวจได้ดีกว่า test ผ่านหน้าเว็บ: ตรวจ status code และทุก field ของ response ได้โดยตรง และส่ง input ที่หน้าเว็บไม่ยอมให้ส่งได้ เช่น 3.5 ดาว หรือ request ที่ไม่มี token อีกทั้งยังรันเร็วกว่า
   สิ่งที่ต้องใช้ test ผ่านหน้าเว็บเท่านั้น: สิ่งที่ user เห็นและทำจริงบนหน้าจอ เช่น layout ปุ่มกดได้หรือไม่ และข้อความ error แสดงถูกตำแหน่งหรือไม่
