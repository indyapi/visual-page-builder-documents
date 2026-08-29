# REQUIREMENTS — Blueprint Visual Page Builder

เอกสารสรุปความต้องการของโปรเจค (Functional & Non-Functional Requirements) ที่ดึงมาจาก `docs/01-PRD.md` และ `docs/02-ARCHITECTURE.md` เพื่อใช้เป็น checklist ตรวจสอบการทำงานของแต่ละเฟส

> สำหรับรายละเอียดเชิงแนวคิด ให้อ่าน PRD เดิม สำหรับโครงสร้างทางเทคนิคให้อ่าน Architecture

## 1. Functional Requirements (FR)

### FR1 — Visual Editor (เฟส 1)
- FR1.1 ผู้ใช้ลาก component จาก palette ลงบน canvas ได้
- FR1.2 ผู้ใช้วาง/ย้าย component ระหว่าง container ได้แบบ nested
- FR1.3 ผู้ใช้ปรับขนาด (resize) และย้าย (drag) element บน canvas แบบ real-time
- FR1.4 ผู้ใช้แก้ไข property ของ element ผ่าน Property Panel
- FR1.5 ระบบบันทึกหน้าเพจเป็น Page Tree JSON และโหลดกลับมาได้ตรงเดิม 100%

### FR2 — Data Model (เฟส 1)
- FR2.1 ทุกหน้าเพจเก็บเป็น JSON tree ที่ serializable 100% (ไม่มี state ผูกกับ runtime เฉพาะ)
- FR2.2 Node เก็บแบบ flat map (`Record<NodeId, PageNode>`) เพื่อย้าย node แบบ O(1)
- FR2.3 Schema มี `version` field รองรับ migration ในอนาคต

### FR3 — Renderer (เฟส 1+)
- FR3.1 Renderer แปลง Page Tree JSON → React elements ได้โดยไม่พึ่ง editor logic
- FR3.2 Renderer ใช้ซ้ำได้ทั้งใน editor canvas และใน production site จริง

### FR4 — Responsive & Editing UX (เฟส 2)
- FR4.1 Layer panel แสดงโครงสร้างต้นไม้ของหน้าเพจ
- FR4.2 Undo/Redo อย่างน้อย 1 ระดับเต็ม
- FR4.3 Duplicate element
- FR4.4 Snap guides / alignment helper
- FR4.5 Zoom / Pan ของ canvas
- FR4.6 Style แยกตาม breakpoint (responsive)

### FR5 — Templates (เฟส 2)
- FR5.1 ระบบ template engine — save/load template ได้

### FR6 — Production Ready (เฟส 3)
- FR6.1 Asset manager (จัดการไฟล์อัปโหลด)
- FR6.2 Rich text editing (Lexical)
- FR6.3 Form component + form submission handler เก็บลง database
- FR6.4 Image upload ผ่าน Storage
- FR6.5 Theme system
- FR6.6 Publish flow: build → เก็บผลลัพธ์ (Storage หรือ deploy target)
- FR6.7 Export เพจเป็น static HTML/CSS (pixel-parity กับ editor)

### FR7 — Developer Features (เฟส 4)
- FR7.1 Export เป็น React/Next.js component
- FR7.2 Plugin system
- FR7.3 CLI สำหรับ scaffold/deploy project

### FR8 — Data & Auth (ระยะยาว)
- FR8.1 ใช้ Supabase เป็น default database (Auth + Storage + DB)
- FR8.2 Data layer ผ่าน Drizzle ORM เพื่อสลับ provider ได้ในอนาคต
- FR8.3 Self-hostable ได้ถ้าไม่ต้องการผูก cloud ของ Supabase

## 2. Non-Functional Requirements (NFR)

- **NFR1 — Type Safety:** ใช้ TypeScript strict mode ตลอด monorepo
- **NFR2 — No Vendor Coupling:** business logic ไม่ผูกกับ Supabase client โดยตรง (ผ่าน Drizzle)
- **NFR3 — Composable:** component ทุกตัวเป็น pure/portable สื่อสารผ่าน props/schema ไม่ผูก editor state
- **NFR4 — Performance:** re-render canvas จาก Page Tree JSON ต้องรวดเร็ว (Zustand)
- **NFR5 — Deployability:** deploy ได้บน Vercel/Netlify/Cloudflare ได้ง่าย สถาปัตยกรรมไม่ผูก vendor
- **NFR6 — Testability:** unit (Vitest) + e2e (Playwright) รันบน CI ทุก PR
- **NFR7 — Progressive Disclosure:** UI เรียบง่ายในเฟสแรก ค่อยๆ เพิ่มความซับซ้อน

## 3. Success Criteria

- เฟส 1: สร้างหน้าเพจด้วย section/container, ปรับ property, save แล้วโหลดกลับมาได้ตรงเดิม 100%
- เฟส 3: export เพจเป็น static HTML/CSS ที่เปิดในเบราว์เซอร์ได้ตรงกับที่เห็นใน editor (pixel-parity)
- เฟส 4: export เป็น Next.js component แล้วนำไปวางในโปรเจค Next.js อื่นได้โดยไม่ error

## 4. Out of Scope (ระยะแรก)

- Multi-user real-time collaboration (คล้าย Figma) — candidate สำหรับเฟส 5
- Built-in CMS/headless CMS เต็มรูปแบบ (ใช้ Supabase table ตรงๆ ไปก่อน)
- Mobile app editor

## 5. Key Risks

| ความเสี่ยง | ผลกระทบ | แนวทางบรรเทา |
|---|---|---|
| Renderer กับ Editor drift กันเมื่อ schema เปลี่ยน | preview ไม่ตรงกับ export จริง | shared schema package เดียว + snapshot test |
| dnd-kit + Moveable ทำงานร่วมกันซับซ้อน | UX สะดุด | จำกัด interaction mode ชัดเจน (drag / resize) ต่อ element |
| Scope creep จากฟีเจอร์เฟส 3-4 | เฟส 1-2 ไม่เสร็จ | ล็อค phase gate เข้มงวด |
