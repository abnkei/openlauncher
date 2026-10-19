# OpenLauncher: ประเมินก่อนติดตั้งบนจอรถ

วันที่ตรวจ: 19 ตุลาคม 2026

## ข้อสรุป

เหมาะสำหรับทดลองเป็น launcher เพิ่มเติมบนจอ Android aftermarket หลังตรวจเครื่องจริง ยังไม่แนะนำตั้งเป็น default ทันที ความเข้ากันได้กับจอรถของผู้ใช้ยังไม่ทราบ เพราะ ADB เชื่อมต่อไม่ได้

## หลักฐานที่ตรวจ

- Repository: https://github.com/dw2lam/openlauncher
- Source main ที่อ่าน: `8ee7d3a0f351609ec42674b03fd292d7ccf671d5` เป็น static review เฉพาะจุด ไม่ใช่ full security audit
- README, Gradle, manifest, weather/network, notification listener, location, media และ radio integration
- ดาวน์โหลด APK มาตรฐาน v0.0.4 และ v0.0.5 จาก GitHub releases
- SHA-256 ตรงกับ asset digest ใน GitHub API ทั้งสองไฟล์
- keytool อ่าน certificate ทั้งสองไฟล์ได้ signer fingerprint SHA-256 เดียวกัน: `6CDE47725FDF92BDE52B0912196499140C59BA2261FFB884245D8C277BDBCC97` ชื่อ subject `CN=Android Debug, O=Android, C=US` ไม่ใช่การยืนยันตัวตนผู้เผยแพร่ และยังไม่ได้ตรวจ APK signature ทุก scheme ด้วย apksigner
- ADB 37.0.1 ดาวน์โหลดจาก Google; connect `<device-ip>:5555` ถูกปฏิเสธ (10061) ก่อนและหลังผู้ใช้แจ้งเชื่อม hotspot อีกครั้ง; devices ว่าง และ mDNS ไม่พบบริการ

## ความเหมาะสมและข้อจำกัด

| ประเด็น | ผลประเมิน |
|---|---|
| ระบบเป้าหมาย | Android aftermarket head unit; ไม่ใช่การเปลี่ยน firmware หรือเพิ่ม Android Auto/CarPlay receiver ให้เครื่อง |
| SDK | Source ระบุ minSdk 21, targetSdk 36; ยังไม่ยืนยัน minimum ที่ติดตั้งได้จริงจาก merged APK manifest/dependencies |
| หน้าจอ | Activity ล็อก landscape; ต้องตรวจ resolution/density และการกดจริง |
| เวอร์ชัน | v0.0.4 เป็น Latest release; v0.0.5 เป็น pre-release ผู้พัฒนาระบุ Untested บน physical head units |
| วิทยุ | Source main มี MCU integration สำหรับ szchoiceway; เครื่องอื่นต้องอาศัย MediaSession ของแอปวิทยุผู้ผลิต จึงไม่รับประกัน seek/preset/metadata |
| รถและ MCU | กล้องถอย, CANBUS, ปุ่มพวงมาลัย, HVAC, Bluetooth และการปลุกจาก sleep ต้องทดสอบกับจอจริง |
| PiP | ไม่แนะนำ test-pip/platform test-key build; ใช้ APK มาตรฐานในการทดลอง |

## Security / Privacy

- Manifest ขอ location, network, การดูรายการแอป และอ่านรูป/ไฟล์ตาม Android version; มี notification listener ที่ผู้ใช้ต้องอนุญาต
- Notification access ให้สิทธิ์กว้าง แม้ code ที่อ่านใช้เพื่อ MediaSession/Now Playing ควรเปิดเมื่อจำเป็นเท่านั้น
- Weather ส่ง latitude/longitude ไป `https://api.open-meteo.com/v1/forecast`; source เรียกเมื่อมี location และวนประมาณทุก 30 นาที โดยไม่ตรวจว่า weather widget ยังแสดงอยู่หรือไม่ การเอา widget ออกจึงไม่ใช่ privacy switch
- ไม่พบ root/shell subprocess, dynamic code loading หรือ credential hardcode จากการค้น code paths ที่ตรวจ; ไม่ใช่หลักฐานรับรองว่าปลอดช่องโหว่หรือว่า APK ตรงกับ source ทุกส่วน
- Dependencies หลักเป็น AndroidX/Compose, DataStore, Retrofit/OkHttp, Coil, Gson และ Coroutines; ยังไม่ได้ทำ dependency vulnerability audit หรือ resolve dependency graph
- Onboarding/Settings มีทางไปอนุญาต overlay แต่ manifest main ที่อ่านไม่มี SYSTEM_ALERT_WINDOW จึงไม่ควรคาดหวัง floating-window capability จาก APK มาตรฐาน
- หลายจุดจับ exception แล้วไม่รายงาน อาจทำให้ฟังก์ชันไม่ทำงานโดยไม่มีข้อความช่วยวินิจฉัย

## ไฟล์ APK และ hash

| ไฟล์ | SHA-256 |
|---|---|
| artifacts/openlauncher-0.0.4.apk | `dbc68a02be729496ef14351ba70c2ddededb9cd15754c496b3c3be15ccde100c` |
| artifacts/openlauncher-0.0.5.apk | `a25d0640a04b7ee3a4b78ffe43c56441b21398d0458045229ef90986ddfd4937` |

Metadata บันทึกใน `artifacts/releases.json`; binary hashes ใช้ยืนยันความตรงกับ GitHub asset ไม่ใช่การรับรองความปลอดภัย

## ขั้นตอนต่อไปเมื่อ ADB ใช้งานได้

1. อ่าน manufacturer/model, SDK/release, ABI, RAM, resolution/density และ Home component เดิม โดยไม่ดึงข้อมูลส่วนตัว
2. ตรวจ package เดิมและ signing ก่อน install/update; บันทึกทางกลับ launcher เดิม
3. เลือกรุ่นจาก SDK จริงและข้อจำกัดที่พบ แล้วติดตั้ง APK มาตรฐานเป็นแอปเพิ่มเติม เปิดแบบ explicit component ก่อนตั้ง default
4. ทดสอบเปิด/ปิด, permission denied, offline, เปิดแอปนำทาง/CarPlay receiver, music/radio, Home, sleep/wake และ cold boot ขณะรถจอด การเช็กกล้องถอย/ปุ่มรถให้เจ้าของรถทำในสภาพแวดล้อมที่เหมาะสม
5. ตรวจ logcat เฉพาะ package และ memory; หากมีปัญหา กลับ launcher เดิมและถอนแพ็กเกจนี้โดยไม่แตะแอประบบ

ยังไม่ได้ติดตั้งแอป เปลี่ยน default launcher เปลี่ยน firmware หรือทดสอบ runtime/build เครื่องจริง ต้องได้ ADB endpoint ที่เปิดใช้งานก่อน: IP อย่างเดียวไม่ระบุ port; wireless debugging รุ่นใหม่อาจต้อง pairing port/code แยกจาก connection port
