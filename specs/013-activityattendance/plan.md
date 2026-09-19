# แผนงานเทคนิค: เช็คการเข้าร่วมค่ายด้วย GPS (GPS Attendance)

## 1. สรุปแนวทาง
- ฟีเจอร์นี้ให้ผู้เข้าร่วมค่ายยืนยันการเข้าร่วมด้วย GPS ตามวันที่และรอบเวลาที่เลือก โดยแต่ละวันมี 3 รอบคือ เช้า / กลางวัน / เย็น และแต่ละรอบให้เช็คได้หนึ่งครั้งเท่านั้น
- ผู้ใช้หลักคือ ผู้เข้าร่วมค่าย และคณะกรรมการที่ต้องตรวจสอบสถานะและรายละเอียดการเช็คเข้าร่วม
- ระบบจะขอสิทธิ์เข้าถึง GPS ก่อนการเช็คเข้าร่วม ตรวจสอบระยะห่างจากจุดอ้างอิงค่าย และ บันทึกผลเป็น “เข้าร่วม” หรือ “ยังไม่เช็ค” ตามเงื่อนไขที่กำหนด
- หาก GPS ไม่พร้อม ผู้ใช้ไม่อนุญาต หรือระบบไม่สามารถยืนยันตำแหน่งได้ ระบบจะปฏิเสธการเช็คและไม่บันทึกข้อมูล
- สำหรับการป้องกันการเช็คซ้ำ ระบบจะใช้ constraint หรือ unique key ตามผู้เข้าร่วม + วันที่ + รอบเวลา และยึด record แรกเมื่อเกิดการส่งซ้ำหรือค้าง

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
| --- | --- | --- |
| React (Vite) สำหรับหน้าบ้าน | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้แสดงหน้าเช็คเข้าร่วม ประวัติผู้เข้าร่วม และแดชบอร์ดคณะกรรมการ |
| Python FastAPI สำหรับหลังบ้าน | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้รับ GPS ตรวจสอบระยะทาง บันทึกข้อมูลการเช็คเข้าร่วม และตรวจสอบสิทธิ์ |
| PostgreSQL | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้เก็บข้อมูลผู้เข้าร่วม ข้อมูล GPS และผลการเช็คเข้าร่วม |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับ FR |
| --- | --- | --- |
| Participant | participant_id, user_id, status, is_eligible, created_at | CON-ATT-01, FR-ATT-01, FR-ATT-12, FR-ATT-13, FR-ATT-16 |
| CampLocation | camp_location_id, name, latitude, longitude, radius_meters, updated_at | FR-ATT-02, FR-ATT-04, FR-ATT-05, ASM-ATT-03, ASM-ATT-04 |
| AttendanceSlot | slot_id, attendance_date, time_slot, description | FR-ATT-01, CON-ATT-03 |
| AttendanceRecord | attendance_id, participant_id, attendance_date, time_slot, status, checked_at, latitude, longitude, distance_meters, gps_validation_status | FR-ATT-06, FR-ATT-07, FR-ATT-08, FR-ATT-09, FR-ATT-10, FR-ATT-11, FR-ATT-15, NFR-DATA-01, NFR-DATA-03 |
| AttendanceAudit | audit_id, attendance_id, action_type, reason, created_at | IF-ATT-01, IF-ATT-02, NFR-REL-01, NFR-REL-02 |
| UserRole | user_id, role, is_committee | NFR-SEC-01, NFR-SEC-02, CON-ATT-07, CON-ATT-08 |

หมายเหตุ: ไม่มีฟิลด์ข้อมูลที่ขัดกับ constraint เช่น ไม่มีการเก็บข้อมูลบัตรประชาชน หรือการติดตาม GPS แบบต่อเนื่อง เนื่องจาก spec ระบุชัดว่ารับ GPS เฉพาะตอนเช็คเข้าร่วมเท่านั้น

## 4. API / หน้าจอ

- GET /attendance/slots?date={date} — แสดงวันที่และรอบเวลาที่มีให้เลือก, input: date, output: รายการรอบเวลา, รองรับ FR-ATT-01, AC-ATT-01
- POST /attendance/checkin — รับ participant_id, attendance_date, time_slot, latitude, longitude, output: status, message, รองรับ FR-ATT-03, FR-ATT-04, FR-ATT-05, FR-ATT-06, FR-ATT-07, FR-ATT-08, FR-ATT-09, FR-ATT-10, FR-ATT-11, IF-ATT-01, IF-ATT-02, IF-ATT-03, AC-ATT-02, AC-ATT-03, AC-ATT-04, AC-ATT-05, AC-ATT-06
- GET /attendance/history/{participant_id} — แสดงประวัติการเช็คที่สำเร็จของผู้ใช้ที่เข้าสู่ระบบ, รองรับ FR-ATT-12, AC-ATT-07
- GET /committee/attendance-summary — แสดงรายชื่อผู้เข้าร่วมพร้อมสถานะ “เข้าร่วม” หรือ “ยังไม่เช็ค”, รองรับ FR-ATT-13, FR-ATT-16, AC-ATT-08, AC-ATT-10
- GET /committee/attendance/{participant_id} — แสดงรายละเอียดการเช็คที่สำเร็จของผู้เข้าร่วมแต่ละคน, รองรับ FR-ATT-14, FR-ATT-15, AC-ATT-09

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
| --- | --- | --- |
| CON-ATT-01 | ตรวจสอบสถานะผู้เข้าร่วมก่อนเปิดหน้าเช็คชื่อและเรียก API เช็คชื่อ | ใช้แล้ว |
| CON-ATT-02 | บังคับให้ผู้ใช้ยินยอมให้เข้าถึง GPS ก่อนเรียก API checkin | ใช้แล้ว |
| CON-ATT-03 | กำหนด date + time_slot จาก 3 รอบต่อวัน และ 1 ครั้งต่อรอบ | ใช้แล้ว |
| CON-ATT-04 | ตรวจสอบระยะห่างจาก CampLocation ก่อนยอมให้บันทึกสถานะ “เข้าร่วม” | ใช้แล้ว |
| CON-ATT-05 | ใช้ unique key ตาม participant_id + attendance_date + time_slot | ใช้แล้ว |
| CON-ATT-06 | บันทึก latitude, longitude, distance_meters, checked_at และค่า GPS ที่ใช้ตรวจสอบ | ใช้แล้ว |
| CON-ATT-07 | ตรวจสอบเจ้าของบัญชีก่อนอนุญาตเช็คเข้าร่วมในบัญชีของตนเอง | ใช้แล้ว |
| CON-ATT-08 | ให้คณะกรรมการเข้าดูข้อมูลการเข้าร่วมของผู้เข้าร่วมทั้งหมด | ใช้แล้ว |
| IF-ATT-01 | ปฏิเสธการเช็คเมื่อ GPS ไม่พร้อม/ไม่อนุญาต/ยืนยันไม่ได้ | ใช้แล้ว |
| IF-ATT-02 | ไม่บันทึกการพยายามเช็คเมื่ออยู่นอกรัศมี และคงสถานะเป็น “ยังไม่เช็ค” | ใช้แล้ว |
| IF-ATT-03 | ป้องกันการเช็คซ้ำด้วยการยึด record แรกสำหรับรอบนั้น | ใช้แล้ว |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
| --- | --- | --- |
| AC-ATT-01 | test_AC_ATT_01_show_selected_date_and_slot | ตรวจว่าผู้เข้าร่วมเลือกวันที่และรอบเวลาแล้ว ระบบแสดงข้อมูลที่เลือกอย่างถูกต้อง |
| AC-ATT-02 | test_AC_ATT_02_request_gps_before_checkin | ตรวจว่าหลังกดเช็คเข้าร่วม ระบบขอสิทธิ์ GPS ก่อนดำเนินการต่อ |
| AC-ATT-03 | test_AC_ATT_03_successful_checkin_records_attendance | ตรวจว่าถ้าพบว่าภายในรัศมี ระบบบันทึกสถานะ “เข้าร่วม” พร้อมข้อมูลวัน เวลา latitude longitude และระยะห่าง |
| AC-ATT-04 | test_AC_ATT_04_out_of_range_checkin_is_rejected | ตรวจว่าถ้าอยู่นอกรัศมี ระบบปฏิเสธการเช็คและคงสถานะเป็น “ยังไม่เช็ค” |
| AC-ATT-05 | test_AC_ATT_05_duplicate_checkin_keeps_first_record | ตรวจว่าการเช็คซ้ำในรอบเดียวกันไม่สร้าง record ใหม่ และยึด record แรก |
| AC-ATT-06 | test_AC_ATT_06_unavailable_gps_is_rejected | ตรวจว่ากรณี GPS ไม่พร้อม/ไม่อนุญาต/ยืนยันไม่ได้ ระบบไม่บันทึกและแจ้งให้ลองใหม่ |
| AC-ATT-07 | test_AC_ATT_07_participant_history_shows_only_successful_records | ตรวจว่าประวัติของผู้เข้าร่วมแสดงเฉพาะการเช็คที่สำเร็จเท่านั้น |
| AC-ATT-08 | test_AC_ATT_08_committee_sees_all_attendance_status | ตรวจว่าคณะกรรมการเห็นรายชื่อพร้อมสถานะ “เข้าร่วม” หรือ “ยังไม่เช็ค” สำหรับทุกคน |
| AC-ATT-09 | test_AC_ATT_09_committee_detail_view_contains_required_fields | ตรวจว่าคณะกรรมการเห็นวันที่ รอบเวลา สถานะ เวลา latitude longitude และระยะห่าง |
| AC-ATT-10 | test_AC_ATT_10_unchecked_attendance_status_is_displayed | ตรวจว่าผู้เข้าร่วมที่ยังไม่ได้เช็คจะแสดงสถานะ “ยังไม่เช็ค” |

## 7. ลำดับงาน

1. สร้างโครงสร้างข้อมูลพื้นฐานสำหรับ Participant, CampLocation, AttendanceSlot, AttendanceRecord และ unique constraint สำหรับผู้เข้าร่วม + วันที่ + รอบเวลา — เกี่ยวข้องกับ CON-ATT-01, CON-ATT-03, CON-ATT-05, NFR-DATA-02
2. สร้างหน้าเลือกวันที่และรอบเวลา และแสดงข้อมูลสถานที่ค่ายและรัศมีที่ใช้ตรวจสอบ — เกี่ยวข้องกับ FR-ATT-01, FR-ATT-02, AC-ATT-01
3. เพิ่มการขอสิทธิ์ GPS และตรวจสอบก่อนยอมให้เช็คเข้าร่วม — เกี่ยวข้องกับ FR-ATT-03, IF-ATT-01, NFR-REL-01, AC-ATT-02, AC-ATT-06
4. เพิ่มฟังก์ชันคำนวณระยะห่างจากจุดค่าย และตัดสินใจอนุญาตหรือปฏิเสธเช็คตามรัศมี — เกี่ยวข้องกับ FR-ATT-04, FR-ATT-05, CON-ATT-04, NFR-REL-02, AC-ATT-03, AC-ATT-04
5. บันทึก AttendanceRecord แบบ atomic และป้องกันการสร้าง record ซ้ำด้วย logic ที่ยึด record แรก — เกี่ยวข้องกับ FR-ATT-06, FR-ATT-08, FR-ATT-09, FR-ATT-10, FR-ATT-11, IF-ATT-03, AC-ATT-05
6. สร้างหน้าประวัติการเข้าร่วมของผู้เข้าร่วมเฉพาะตนเอง โดยคัดเฉพาะรายการที่สำเร็จ — เกี่ยวข้องกับ FR-ATT-12, NFR-SEC-01, AC-ATT-07
7. สร้างแดชบอร์ดคณะกรรมการเพื่อดูรายชื่อและรายละเอียดการเข้าร่วม — เกี่ยวข้องกับ FR-ATT-13, FR-ATT-14, FR-ATT-15, FR-ATT-16, NFR-SEC-02, AC-ATT-08, AC-ATT-09, AC-ATT-10
8. ทดสอบความครอบคลุมจาก AC และตรวจ traceability กับ FR, NFR, CON, IF — เกี่ยวข้องกับ Acceptance Criteria ทั้งหมด

## 8. สิ่งที่ยังไม่ทำ
- ไม่มี Open Questions ใน spec ปัจจุบันหลังจากขั้น clarify แล้ว
- ส่วนที่เกี่ยวข้องกับข้อสรุปดังกล่าวจะยังไม่สร้างจนกว่าจะได้คำตอบเพิ่มเติมจากทีม หากมีการเปลี่ยนแปลงในอนาคต
- ไม่ได้เดาเรื่องค่าเริ่มต้นของรัศมีหรือรูปแบบ GPS อื่น ๆ เพราะ spec ยังไม่ได้ระบุรายละเอียดเชิงตัวเลข และต้องรอการตัดสินใจจากทีม
