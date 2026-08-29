# Blueprint — Visual Page Builder

Visual Page Builder แบบ self-hosted friendly และไม่ผูก vendor (no vendor lock-in) พัฒนาด้วยวิธี Vibe Coding ทั้งหมด

> เอกสารชุดนี้ (ในโฟลเดอร์ `docs/`) ทำหน้าที่เป็น **single source of truth** ให้ AI coding agent อ่านแล้วเขียนโค้ดต่อได้อย่างสอดคล้องกัน

## ภาพรวม

โปรเจคนี้มุ่งแก้ปัญหาที่ Page builder ส่วนใหญ่ในตลาด (Webflow, Framer, Builder.io) เป็น SaaS ปิด ผูกกับ cloud ของเจ้าของบริการ ผู้ใช้ที่ต้องการโฮสต์เอง ควบคุมข้อมูลเอง ปรับแต่ง component/plugin เอง และ export โค้ดจริงไปใช้ต่อ จึงไม่มีตัวเลือกที่ครบวงจรและทันสมัยพอ

เป้าหมายหลัก:
1. Visual Editor ที่ลาก-วาง-ปรับขนาด component บน canvas ได้แบบ real-time
2. หน้าเพจเก็บเป็น JSON schema ที่ portable — save/load/export ได้อิสระจาก vendor
3. Render ทั้งในรูปแบบ preview (ใน editor) และ export เป็น HTML/CSS หรือ React/Next.js component จริง
4. รองรับการขยายผ่าน plugin system (ระยะยาว)
5. Deploy ได้ง่ายบน edge platform (Vercel/Netlify/Cloudflare)

## โครงสร้างโปรเจค

โปรเจคใช้ **pnpm workspaces + Turborepo** (วางแผนไว้ใน `docs/02-ARCHITECTURE.md`) แบ่งเป็น:

- `apps/editor/` — Next.js app ตัว visual editor
- `packages/schema/` — JSON schema + Zod types ของ page tree (ใช้ร่วมทุก package)
- `packages/renderer/` — แปลง page-tree JSON → React elements
- `packages/component-library/` — Component ที่ลากวางได้
- `packages/editor-core/` — Editor logic (Zustand store, history, dnd-kit)
- `packages/export/` — Export engine (JSON → HTML/CSS, JSON → React/Next)
- `packages/db/` — Drizzle schema + Supabase client wrapper
- `docs/` — เอกสารกำหนดโปรเจค (PRD, Architecture, Roadmap, Guidelines, Requirements)
- `.agent/` — ไฟล์ state ข้าม session สำหรับ AI agent
- `.opencode/` — ตั้งค่า opencode + skills + commands

## เอกสารประกอบ

| ไฟล์ | เนื้อหา |
|---|---|
| [`docs/01-PRD.md`](docs/01-PRD.md) | Product Requirements — ภาพรวม, ปัญหา, เป้าหมาย, scope, หลักการออกแบบ |
| [`docs/02-ARCHITECTURE.md`](docs/02-ARCHITECTURE.md) | โครงสร้าง monorepo, tech stack, data flow, schema หลัก |
| [`docs/03-ROADMAP-BACKLOG.md`](docs/03-ROADMAP-BACKLOG.md) | แผน 4 เฟส + backlog ของ ticket |
| [`docs/04-CODING-GUIDELINES.md`](docs/04-CODING-GUIDELINES.md) | กติกาการเขียนโค้ด, naming, test, PR checklist |
| [`docs/05-REQUIREMENTS.md`](docs/05-REQUIREMENTS.md) | ความต้องการเชิงฟังก์ชันและไม่เชิงฟังก์ชันที่สรุปจาก PRD |
| [`docs/STYLE.md`](docs/STYLE.md) | แนวทางสไตล์การเขียน |
| [`AGENTS.md`](AGENTS.md) | บลูปรินท์การทำงานของ AI agent (memory / current state / summary) |
| [`instruction.md`](instruction.md) | คำสั่งประจำโปรเจคสำหรับ AI agent |

## สถานะปัจจุบัน

อยู่ระหว่างเตรียมโครงสร้างโปรเจค (ยังไม่เริ่มเขียนโค้ดแอป) — ดูความคืบหน้าแผนใน `docs/03-ROADMAP-BACKLOG.md`

## คำเตือนสำคัญเรื่องการพัฒนา

- **ห้ามข้าม phase gate** — ไม่เริ่ม ticket ของเฟสถัดไปจนกว่า acceptance criteria ของเฟสก่อนหน้าจะผ่าน
- **Page Tree JSON คือ source of truth** — ห้ามเก็บ state สำคัญไว้ที่อื่นโดยไม่ sync กลับ schema
- **เคารพ dependency direction** — `schema` ← `renderer`/`component-library` ← `editor-core` ← `apps/editor` ห้าม import ย้อนทาง

---

MIT License

Copyright (c) 2026 indyapi

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
