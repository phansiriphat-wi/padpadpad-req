# แผนงานเทคนิค: จัดกลุ่มและฝ่ายงาน (Group Management)

## 1. สรุปแนวทาง
- ฟีเจอร์นี้ให้คณะกรรมการจัดสังกัดผู้เข้าร่วมที่ผ่านการคัดเลือกแล้วให้กลุ่มหรือฝ่ายที่ต้องการตาม FR-GRP-01 ถึง FR-GRP-06
- ผู้ใช้หลักคือคณะกรรมการและผู้เข้าร่วมที่ได้รับการจัดสังกัด
- ระบบจะแสดงรายชื่อผู้ผ่านคัดเลือกพร้อมสถานะและอนุญาตให้เลือกสังกัดใหม่ก่อนบันทึก
- ก่อนบันทึกจะมีการยืนยันข้อมูลสังกัดเดิมและสังกัดใหม่ พร้อมปุ่มยืนยันและยกเลิก
- เมื่อบันทึกสำเร็จ ระบบจะแสดงข้อมูลปัจจุบันและแจ้งเตือนผู้เข้าร่วมหากมีการย้ายสังกัด

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
| --- | --- | --- |
| React (Vite) สำหรับส่วนหน้า | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้สำหรับแสดงรายชื่อผู้เข้าร่วมและหน้าต่างยืนยันการย้ายสังกัด |
| Python FastAPI สำหรับส่วนหลังบ้าน | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้สำหรับจัดการข้อมูลสังกัดและการยืนยันการบันทึก |
| PostgreSQL | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้เก็บข้อมูลผู้เข้าร่วม กลุ่ม/ฝ่าย ประวัติการย้ายสังกัด |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับ FR |
| --- | --- | --- |
| Participant | participant_id, full_name, selection_status, current_assignment_id, assignment_status, created_at, updated_at | FR-GRP-01, FR-GRP-02, FR-GRP-05 |
| Assignment | assignment_id, assignment_name, assignment_type, is_active, created_at | FR-GRP-03, FR-GRP-05 |
| AssignmentHistory | history_id, participant_id, previous_assignment_id, new_assignment_id, changed_by, changed_at, is_notified, work_started_flag | FR-GRP-04, FR-GRP-06 |
| CommitteeActionLog | log_id, committee_id, participant_id, action_type, action_time, result | FR-GRP-04, NFR-SEC-01 |

หมายเหตุ: ระบบจะไม่เก็บข้อมูลเลขบัตรประชาชน หรือรหัสประจำตัวส่วนบุคคลสำหรับกระบวนการจัดสังกัดนี้ เนื่องจากยังไม่ปรากฏใน spec และไม่จำเป็นต่อฟีเจอร์นี้

## 4. API / หน้าจอ

- หน้า: ParticipantsListPage — GET /participants/eligible — Input: filter status, role — Output: รายชื่อผู้เข้าร่วมที่ผ่านคัดเลือกพร้อมสถานะ “รอจัดกลุ่ม” / “จัดกลุ่มแล้ว” — รองรับ FR-GRP-01, FR-GRP-05
- หน้า: AssignParticipantPage — GET /participants/{id}/assignment-preview — Input: participantId — Output: ผู้เข้าร่วม, สังกัดปัจจุบัน, รายการกลุ่ม/ฝ่ายที่เลือกได้ — รองรับ FR-GRP-02, FR-GRP-03
- API: POST /participants/{id}/assignments — Input: assignmentId, confirmedByCommittee — Output: assignment record, success status — รองรับ FR-GRP-04
- API: POST /participants/{id}/assignments/confirm-change — Input: previousAssignmentId, newAssignmentId, confirm — Output: updated current assignment + notification trigger — รองรับ FR-GRP-06
- หน้า: AssignmentHistoryPage — GET /participants/{id}/assignment-history — Input: participantId — Output: สังกัดเดิมและสังกัดใหม่ (เฉพาะเมื่อเริ่มงานหรือประกาศงานแล้ว) — รองรับ FR-GRP-06

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
| --- | --- | --- |
| CON-GRP-01 | Phase 1 ของลำดับงาน: ดึงรายการผู้เข้าร่วมที่ผ่านการคัดเลือกแล้วก่อนแสดงสังกัด | ใช้แล้ว |
| CON-GRP-02 | การเปิดหน้าจอกำหนดสังกัดและการยืนยันบันทึกถูกจำกัดให้คณะกรรมการเท่านั้น | ใช้แล้ว |
| CON-GRP-03 | ขั้นตอนยืนยันก่อนบันทึกใน Phase 3 และ modal ยกเลิก/ยืนยัน | ใช้แล้ว |
| IF-GRP-01 | ParticipantsListPage และ GET /participants/eligible ดึงข้อมูลจากผู้เข้าร่วมที่ผ่านคัดเลือกแล้วเท่านั้น | ใช้แล้ว |
| IF-GRP-02 | AssignParticipantPage และ API เลือกสังกัดใช้รายการกลุ่ม/ฝ่ายที่มีอยู่ในระบบเท่านั้น | ใช้แล้ว |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
| --- | --- | --- |
| AC-GRP-01 | test_AC_GRP_01_assign_new_participant_to_department | ตรวจว่าเมื่อคณะกรรมการเลือกผู้เข้าร่วมที่ยังไม่ได้จัดสังกัดและระบุกอง/ฝ่าย จะเห็นข้อมูล preview และสถานะเป็น “รอจัดกลุ่ม” ก่อนยืนยัน |
| AC-GRP-02 | test_AC_GRP_02_confirm_assignment_updates_participant_and_notifies | ตรวจว่าหลังกดยืนยัน ระบบบันทึกสังกัดปัจจุบันและแจ้งเตือนผู้เข้าร่วมเมื่อการย้ายสังกัดสำเร็จ |
| AC-GRP-03 | test_AC_GRP_03_show_current_assignment_after_assignment | ตรวจว่าหลังจัดสังกัดแล้ว ระบบแสดงกลุ่ม/ฝ่ายปัจจุบันและสถานะเป็น “จัดกลุ่มแล้ว” |
| AC-GRP-04 | test_AC_GRP_04_confirm_reassignment_modal_and_history | ตรวจว่าหน้าต่างยืนยันมีผู้เข้าร่วม + สังกัดเดิม + สังกัดใหม่ + ปุ่มยืนยัน/ยกเลิก และบันทึกประวัติเมื่อเริ่มงานหรือประกาศงานแล้ว |

## 7. ลำดับงาน

1. สร้างโครงสร้างข้อมูลพื้นฐานสำหรับ Participant, Assignment, AssignmentHistory และการปลอดภัยตาม NFR-SEC-01 — เกี่ยวข้องกับ FR-GRP-01, FR-GRP-02, FR-GRP-04, AC-GRP-01, AC-GRP-02
2. สร้างหน้ารายชื่อผู้เข้าร่วมที่ผ่านการคัดเลือกแล้วพร้อมสถานะ “รอจัดกลุ่ม” / “จัดกลุ่มแล้ว” — เกี่ยวข้องกับ FR-GRP-01, AC-GRP-01
3. สร้างหน้าจอเลือกผู้เข้าร่วมและเลือกสังกัดจากกลุ่ม/ฝ่ายที่มีอยู่จริง — เกี่ยวข้องกับ FR-GRP-02, FR-GRP-03, IF-GRP-02
4. เพิ่มตรวจสอบสิทธิ์คณะกรรมการและข้อจำกัดให้ผู้เข้าร่วมมีสังกัดได้เพียงหนึ่งหน่วย — เกี่ยวข้องกับ CON-GRP-02, NFR-SEC-01, ASM-03, ASM-05
5. สร้างความสามารถบันทึกสังกัดปัจจุบันและแจ้งเตือนผู้เข้าร่วมหลังย้ายสังกัดสำเร็จ — เกี่ยวข้องกับ FR-GRP-04, AC-GRP-02
6. สร้างหน้าต่างยืนยันการย้ายสังกัดพร้อมแสดงสังกัดเดิมและสังกัดใหม่ — เกี่ยวข้องกับ FR-GRP-06, AC-GRP-04
7. เพิ่มการเก็บประวัติสังกัดเดิมเมื่อเริ่มงานหรือประกาศงานแล้ว พร้อมการยกเลิก/ไม่บันทึกเมื่อยกเลิก — เกี่ยวข้องกับ FR-GRP-06, ASM-06, AC-GRP-04
8. ทดสอบครบทุก AC และตรวจความสอดคล้องของข้อมูลปัจจุบัน — เกี่ยวข้องกับ AC-GRP-01 ถึง AC-GRP-04

## 8. สิ่งที่ยังไม่ทำ
- ไม่มี Open Questions ใน spec ปัจจุบันหลังจากขั้น clarify แล้ว เนื่องจากคำถามทั้งหมดได้รับคำตอบและย้ายไปเป็น ASM-05 และ ASM-06
- ส่วนที่เกี่ยวข้องกับข้อสรุปดังกล่าวจะยังไม่สร้างจนกว่าจะได้คำตอบเพิ่มเติมจากทีม หากมีการเปลี่ยนแปลงในอนาคต

---
