# PRD — Visual Page Builder (Codename: Blueprint)

## 1. ภาพรวมโปรเจค

**ชื่อโปรเจค:** Blueprint — Visual Page Builder
**ประเภท:** Self-hosted friendly, No vendor lock-in
**วิธีพัฒนา:** Vibe Coding ทั้งหมด (AI-assisted development) — เอกสารในชุดนี้ทำหน้าที่เป็น "single source of truth" ให้ AI agent อ่านแล้วเขียนโค้ดต่อได้อย่างสอดคล้องกัน

## 2. ปัญหาที่ต้องการแก้ (Problem Statement)

Page builder ส่วนใหญ่ในตลาด (Webflow, Framer, Builder.io) เป็น SaaS ปิด ผูกกับ cloud ของเจ้าของบริการ ผู้ใช้ที่ต้องการ:
- โฮสต์เอง (self-host)
- ควบคุมข้อมูลเอง (own the data)
- ปรับแต่ง component/plugin เอง
- Export โค้ดจริงไปใช้ต่อในโปรเจค React/Next.js ของตัวเอง

ไม่มีตัวเลือกที่ครบวงจรและทันสมัยพอ

## 3. เป้าหมายผลิตภัณฑ์ (Product Goals)

1. สร้าง Visual Editor ที่ลาก-วาง-ปรับขนาด component บน canvas ได้แบบ real-time
2. โครงสร้างข้อมูลหน้าเพจเป็น JSON schema ที่ portable — save/load/export ได้อิสระจาก vendor ใดๆ
3. Render ผลลัพธ์ได้ทั้งในรูปแบบ preview (ภายใน editor) และ export เป็น HTML/CSS หรือ React/Next.js component จริง
4. รองรับการขยายผ่าน plugin system ในระยะยาว
5. Deploy ได้ง่ายบน edge platform ยอดนิยม (Vercel/Netlify/Cloudflare)

## 4. กลุ่มผู้ใช้เป้าหมาย (Target Users)

- นักพัฒนาที่ต้องการ page builder แบบ self-hosted สำหรับลูกค้าหรือโปรเจคตัวเอง
- ทีมเล็กที่ต้องการสร้าง landing page/marketing site โดยไม่พึ่ง SaaS รายเดือน
- นักพัฒนา plugin/theme ที่ต้องการต่อยอด ecosystem

## 5. ขอบเขตโปรเจค (Scope)

โปรเจคแบ่งเป็น 4 เฟส (ดูรายละเอียดใน `03-ROADMAP-BACKLOG.md`):

| เฟส | ชื่อ | แก่นสำคัญ |
|---|---|---|
| 1 | Core Editor | โครง canvas, section, container, property panel, save/load JSON |
| 2 | Responsive & Templates | layers panel, undo/redo, duplicate, snap guides, zoom/pan, template engine |
| 3 | Production Ready | asset manager, rich text, forms, image upload, theme system, publish, export HTML/CSS |
| 4 | Developer Features | React/Next export, plugin system, CLI |

**นอกขอบเขต (Out of Scope) ในระยะแรก:**
- Multi-user real-time collaboration (คล้าย Figma) — เป็น candidate สำหรับเฟส 5 ในอนาคต
- Built-in CMS/headless content management เต็มรูปแบบ (ใช้ Supabase table ตรงๆ ไปก่อน)
- Mobile app editor

## 6. หลักการออกแบบ (Design Principles)

1. **JSON-first** — ทุกหน้าเพจคือ JSON tree ที่ serializable 100% ไม่มี state ที่ผูกกับ runtime เฉพาะ
2. **Renderer แยกจาก Editor** — renderer package ต้อง render JSON → UI ได้โดยไม่ต้องพึ่ง editor logic (ใช้ทั้งใน editor canvas และใน production site จริง)
3. **No hidden vendor coupling** — Supabase ใช้เป็น default database แต่ data layer ต้องผ่าน Drizzle ORM เพื่อสลับ provider ได้ในอนาคต
4. **Composable component library** — component ทุกตัวเป็น pure, portable, ไม่ผูกกับ editor state โดยตรง (สื่อสารผ่าน props/schema)
5. **Progressive disclosure** — UI เรียบง่ายในเฟสแรก ค่อยๆ เพิ่มความซับซ้อน (layers, snap guide ฯลฯ) ในเฟสถัดไป

## 7. ตัวชี้วัดความสำเร็จ (Success Criteria)

- เฟส 1 เสร็จ: ผู้ใช้สร้างหน้าเพจง่ายๆ ด้วย section/container, ปรับ property, save แล้วโหลดกลับมาได้ตรงเดิม 100%
- เฟส 3 เสร็จ: export เพจเป็น static HTML/CSS ที่เปิดในเบราว์เซอร์ได้ตรงกับที่เห็นใน editor (pixel-parity)
- เฟส 4 เสร็จ: export เป็น Next.js component แล้วนำไปวางในโปรเจค Next.js อื่นได้โดยไม่ error

## 8. ความเสี่ยงหลัก (Key Risks)

| ความเสี่ยง | ผลกระทบ | แนวทางบรรเทา |
|---|---|---|
| Renderer กับ Editor drift กันเมื่อ schema เปลี่ยน | preview ไม่ตรงกับ export จริง | ใช้ shared schema package เดียว + snapshot test |
| dnd-kit + Moveable ทำงานร่วมกันซับซ้อน (drag vs resize conflict) | UX สะดุด | จำกัด interaction mode ชัดเจนต่อ element (drag mode / resize mode) ตั้งแต่ต้น |
| Scope creep จากฟีเจอร์เฟส 3-4 | ทำให้เฟส 1-2 ไม่เสร็จ | ล็อค phase gate เข้มงวด ไม่เริ่มเฟสถัดไปจนกว่า acceptance criteria เฟสก่อนหน้าผ่าน |
