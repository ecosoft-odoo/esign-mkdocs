## As-Is

เมื่อสถานะของเอกสาร Purchase Order เปลี่ยนจาก Bid Selection เป็น **Waiting to Verify**
หัวหน้าพัสดุจะเป็นคนกดปุ่ม "Verify" เพื่อเปลี่ยนสถานะเป้น **Waiting to Release** โดยไม่ต้องมีการอนุมัติใดๆเพิ่มเติม

และกดปุ่ม "Release" เพื่อเปลี่ยนสถานะเป็น **PO Released**

## To-Be



เมื่อสถานะของเอกสารเปลี่ยนเป้น **Waiting to Verify** เราต้องการให้มีการลงนาม
จาก 1) บุคคลภายในผ่าน Camunda Workflow และตามด้วย 2) บุคคลภายนอกผ่านระบบ eSign (Supplier's Contacts)

โดยระบบจะเพิ่มให้มี **Sub States** ระหว่างสถานะ **Waiting to Verify** ดังรูป เพื่อติดตามสถานะของการลงนามก่อนที่จะเปลี่ยนสถานะเป็น **Waiting to Release** เพื่อกลับเข้า Flow ปกติ

![](pics/po1.png)

1. Ready: สถานะเริ่มต้นเมื่อ State = Waiting to Verify
2. PO Verified: หลังจากหัวหน้าเจ้าหน้าที่พัสดุ Verify with eSign Process
3. Waiting Approval: หลังจากเจ้าหน้าที่พัสดุ Send to Approval เพื่อลงนามโดยบุคคลภายในผ่าน **Camunda Workflow**
4. PO Approved: เมื่อจบ Camunda Workflow
5. Waiting Supplier Confirm: เมื่อพนักงานพัสดุ Send to Supplier Confirm เพื่อการลงนามโดยบุคคลภายนอกโดยระบบ **eSign**

!!! info

    หากไม่ต้องการขั้นตอนการลงนาม หัวหน้าพัสดุสามารถเลี่ยงได้โดยกดปุ่ม Verify แบบปกติ

---

### Substate: Ready

เมื่อ State = Bid Approved, Substate จะเปลี่ยนเป็น Ready โดยอัตโนมัติ

![](pics/po2.png)

โดยจะมี 2 ทางเลือกสำหรับหัวหน้าพัสดุ

1. Verify with eSign Process เพื่อเข้ากระบวนการลงนาม
2. Verify เพื่อข้ามขั้นตอนนี้ และเปลี่ยนสถานะเป็น Waiting to Release ทันที

---

### Substate: PO Verified

![](pics/po3.png)

เจ้าหน้าที่พัสดุจะเข้ามาทำงานต่อ

1. เลือก Approve PO Form เพื่อเตรียมแนบเป็น attachment
2. กดปุ่ม Send to Approval

---

### Substate: Waiting Approval

![](pics/po4.png)

สถานะนี้เป็นการส่งต่อไปที่ Camunda Workflow โดยระบบจะดำเนินการดังต่อไปนี้

1. พิมพ์ฟอร์ม Approve PO Form เป็น attachment แนบไว้กับเอกสาร (on background)
2. ส่ง Intray Message เพื่อแจ้งเตือนผู้มีส่วนได้เสียให้เข้าไปอนุมัติต่อที่ Camunda
3. ยิง API (data + attachment links) เพื่อ start Camunda Workflow
4. ดึงข้อมูล Workflow จาก Camunda มาไว้ที่ Approval History

#### พิมพ์ฟอร์ม Approve PO Form

    NOT DONE YET, this part should be done by coding

#### ส่ง Intray Message

    NOT DONE - Waiting for message sample message from O+

#### ยิง API start Camunda Wofkflow

    NOT DONE - Need to discuss with Bon Bordin for required data/links
    Public attachment is doe with route /get_purchase_attachment?attachment_id=1234

#### ดึงข้อมูล Workflow จาก Camunda

    NOT DONE - this is similar to Approval History in Call for Bid, need to talk to Bon Bordin

![](pics/po_approval_history.png)

1. ข้อมูลเริ่มต้นได้มาตอนยิง API
2. สามารถกดปุ่มเพื่อดึง updates
3. ตารางแสดง Approval History

---

### Substate: PO Approved

![](pics/po5.png)

เมื่อ Camunda Workflow จบ จะส่ง API กลับมาเพื่ออัพเดทสถานะเป็น PO Approved

    NOT DONE, need to create incoming API for Camunda

เจ้าหน้าที่พัดสุจะกดปุ่ม Send to Supplier Confirm โดย

* ระบบจะแสดง Wizard ให้เลือก Supplier Contacts
* จะมีบาง Contact ที่ยังไม่ได้ลงทะเบียนในระบบ เจ้าหน้าที่พัสดุจะแจ้งไปที่ Contact นั้นๆเพื่อเข้ามาลงทะเบียนกับสวทช
* เมื่อลงทะเบียนแล้วระบบ PABI2 จะต้องมีความสามารถไป Sync ข้อมูลกลับมาที่ Supplier 
    
        NOT DONE, need to talk to Ball Siam on how to sync

---

### Substate: Waiting Supplier Confirm

![](pics/po6.png)

สถานะนี้เป็นการส่งต่อไปที่ระบบ eSign โดยระบบจะดำเนินการดังต่อไปนี้

1. ยิง API ไปที่ eSign โดยส่งไฟล์ PDF ของ Approve PO Form ไปด้วย
2. ส่ง Email ไปที่ Contacts พร้อมแนบไฟล์ไปด้วย

        NOT DONE, need to talk to Ball Siam on how to API

---

### Email เพิ่มเติมตอน State = PO Released

* เพิ่มการส่ง email to [PO committee + PR’s requested by, prepared by + PO’s supplier’s Email]
* แนบไฟล์ ???

        NOT DONE, waiting for O+ to design eamil/form

---

### หมายเหตุเพิ่มเติม


!!! note

    * ปุ่ม Verify แบบปกติจะใช้ได้เมื่อ Substate ยังเป็น Ready เท่านั้น เมื่อ Substate ทำงานแล้ว ระบบจะไม่ยอมให้กดปุ่ม (show warning)
