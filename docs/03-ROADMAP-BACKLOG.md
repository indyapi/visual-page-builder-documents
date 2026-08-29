# ROADMAP & BACKLOG — Blueprint Visual Page Builder

โครงสร้าง: **Phase → Epic → User Story → Ticket**
ใช้ prefix ticket ตามเฟส: `P1-xxx`, `P2-xxx`, `P3-xxx`, `P4-xxx` เพื่ออ้างอิงใน commit/PR ได้ตรง (เช่น `feat(P1-003): implement canvas drop zone`)

**Phase Gate Rule:** ห้ามเริ่มทำ ticket ของเฟสถัดไปจนกว่า Acceptance Criteria ระดับ Epic ของเฟสปัจจุบันจะผ่านครบ

---

## PHASE 1 — Core Editor

### Epic P1-E1: Monorepo & Project Scaffold
- **Story:** ในฐานะทีมพัฒนา ต้องการโครง monorepo ที่รันได้ทันที เพื่อเริ่มเขียนโค้ดได้โดยไม่เสียเวลา setup
  - `P1-001` Setup pnpm workspace + Turborepo config
  - `P1-002` Setup `packages/schema` พร้อม Zod + TypeScript strict config
  - `P1-003` Setup `apps/editor` เป็น Next.js App Router เปล่า พร้อม Tailwind + shadcn/ui
  - `P1-004` Setup Vitest config ระดับ monorepo (shared config)
  - `P1-005` Setup ESLint + Prettier + Husky pre-commit hook

### Epic P1-E2: Page Tree Schema & Renderer
- **Story:** ในฐานะระบบ ต้องมี schema กลางที่ทุก package อ้างอิงได้ เพื่อไม่ให้ editor/renderer หลุด sync กัน
  - `P1-006` ออกแบบ `PageNode` / `PageTree` type + Zod schema
  - `P1-007` เขียน `renderer` package: แปลง PageTree → React tree (recursive render)
  - `P1-008` Unit test: schema validation (valid/invalid case)
  - `P1-009` Unit test: renderer output snapshot สำหรับ node type พื้นฐาน (section, container, text)

### Epic P1-E3: Canvas & Basic Components
- **Story:** ในฐานะผู้ใช้ ต้องการลาก section/container ลงบน canvas เพื่อเริ่มประกอบหน้าเพจ
  - `P1-010` Component: `Section` (full-width block)
  - `P1-011` Component: `Container` (constrained-width, รับ children)
  - `P1-012` Component: `Text` (static, ยังไม่ rich text — รอเฟส 3)
  - `P1-013` Canvas component: render PageTree จาก Zustand store แบบ live
  - `P1-014` dnd-kit: drag component จาก sidebar palette → drop ลง canvas
  - `P1-015` dnd-kit: reorder/nest container ภายใน canvas

### Epic P1-E4: Property Panel
- **Story:** ในฐานะผู้ใช้ ต้องการเลือก element แล้วแก้ property (สี, padding, ข้อความ) ผ่าน panel ด้านข้าง
  - `P1-016` Selection state ใน Zustand (selectedNodeId)
  - `P1-017` Property panel UI (shadcn/ui form controls) — bind กับ props ของ node ที่เลือก
  - `P1-018` Property change → update store → re-render canvas ทันที (two-way binding)

### Epic P1-E5: Save/Load JSON
- **Story:** ในฐานะผู้ใช้ ต้องการบันทึกหน้าเพจแล้วโหลดกลับมาแก้ไขต่อได้
  - `P1-019` Setup `packages/db`: Drizzle schema สำหรับตาราง `pages` (id, name, tree_json, updated_at)
  - `P1-020` Setup Supabase client wrapper ใน `packages/db`
  - `P1-021` API route: `POST /api/pages` (save), `GET /api/pages/:id` (load)
  - `P1-022` Integration test: save → load round-trip ต้องได้ JSON เดิม

**Phase 1 Acceptance Criteria:** ผู้ใช้เปิด editor, ลาก section+container+text ลง canvas, แก้ property, กด save, refresh หน้า, กด load แล้วได้ผลลัพธ์ตรงเดิม 100%

---

## PHASE 2 — Responsive & Templates

### Epic P2-E1: Layers Panel
  - `P2-001` UI panel แสดง tree structure ของ PageTree (แบบ file explorer)
  - `P2-002` คลิก layer → select node บน canvas (sync กับ selection state เดิม)
  - `P2-003` Drag reorder ใน layers panel → update PageTree children order

### Epic P2-E2: History (Undo/Redo)
  - `P2-004` ออกแบบ history middleware สำหรับ Zustand store (command pattern หรือ snapshot diff)
  - `P2-005` Keyboard shortcut: Ctrl/Cmd+Z, Ctrl/Cmd+Shift+Z
  - `P2-006` Unit test: undo/redo กับ action ทุกประเภท (add/remove/move/edit prop)

### Epic P2-E3: Duplicate & Multi-select
  - `P2-007` Duplicate node (deep clone พร้อม generate id ใหม่ทั้ง subtree)
  - `P2-008` Multi-select (shift-click) — เบื้องต้นรองรับ duplicate/delete แบบกลุ่ม

### Epic P2-E4: Resize & Snap Guides (Moveable)
  - `P2-009` ผสาน Moveable เข้ากับ canvas element ที่เลือก
  - `P2-010` Snap guide: ชิดขอบ container/element อื่นเมื่อ drag/resize
  - `P2-011` แก้ conflict ระหว่าง dnd-kit (drag reorder) กับ Moveable (resize) — กำหนด interaction mode ต่อ element ชัดเจน

### Epic P2-E5: Zoom & Pan
  - `P2-012` Canvas zoom controls (25%-200%)
  - `P2-013` Canvas pan (space+drag หรือ scroll)

### Epic P2-E6: Responsive Style System
  - `P2-014` ขยาย `ResponsiveStyle` schema รองรับ breakpoint (mobile/tablet/desktop)
  - `P2-015` UI breakpoint switcher ใน editor toolbar
  - `P2-016` Property panel แสดง/แก้ style ตาม breakpoint ที่เลือก

### Epic P2-E7: Template Engine
  - `P2-017` Schema สำหรับ "template" (PageTree ที่ mark เป็น reusable)
  - `P2-018` UI: บันทึกหน้าปัจจุบันเป็น template, เลือก template ตอนสร้างหน้าใหม่
  - `P2-019` ตาราง Supabase `templates` + API route

**Phase 2 Acceptance Criteria:** ผู้ใช้สร้างหน้าเพจ, ปรับ layout ต่างกันในแต่ละ breakpoint, undo/redo ได้ถูกต้องทุก action, บันทึกเป็น template แล้วนำมาสร้างหน้าใหม่ได้

---

## PHASE 3 — Production Ready

### Epic P3-E1: Rich Text (Lexical)
  - `P3-001` ผสาน Lexical แทน `Text` component พื้นฐาน
  - `P3-002` Toolbar: bold/italic/link/heading/list
  - `P3-003` Serialize Lexical state ↔ PageNode props (ต้อง JSON-safe)

### Epic P3-E2: Asset Manager & Image Upload
  - `P3-004` Supabase Storage bucket setup + upload API
  - `P3-005` Asset manager UI (grid, search, delete)
  - `P3-006` Component: `Image` พร้อม lazy-load, alt text field

### Epic P3-E3: Forms
  - `P3-007` Component: `Form`, `Input`, `Textarea`, `Submit button`
  - `P3-008` Form submission handler → เก็บลง Supabase table `form_submissions`

### Epic P3-E4: Theme System
  - `P3-009` Schema: theme tokens (สี, font, spacing scale)
  - `P3-010` Theme editor UI
  - `P3-011` Component library อ่านค่าจาก theme token แทน hardcode

### Epic P3-E5: Publish & Export (HTML/CSS)
  - `P3-012` `packages/export`: PageTree → static HTML/CSS
  - `P3-013` Publish flow: build → เก็บผลลัพธ์ (Supabase Storage หรือ deploy target)
  - `P3-014` E2E test: pixel-parity ระหว่าง editor preview กับ exported HTML

**Phase 3 Acceptance Criteria:** ผู้ใช้สร้างหน้าที่มีข้อความจัดรูปแบบ, รูปภาพ, ฟอร์ม, ธีมสีของตัวเอง แล้ว publish ออกมาเป็น HTML/CSS ที่ตรงกับ preview

---

## PHASE 4 — Developer Features

### Epic P4-E1: React/Next.js Export
  - `P4-001` `packages/export`: PageTree → React component source code (.tsx)
  - `P4-002` Export ทั้งโปรเจคเป็น Next.js app structure พร้อม package.json

### Epic P4-E2: Plugin System
  - `P4-003` ออกแบบ Plugin API (register custom component type, custom property control)
  - `P4-004` Plugin loader ใน `editor-core`
  - `P4-005` เอกสาร + ตัวอย่าง plugin

### Epic P4-E3: CLI
  - `P4-006` `packages/cli`: คำสั่ง `blueprint init`, `blueprint dev`, `blueprint export`
  - `P4-007` CLI สำหรับ deploy shortcut (Vercel/Netlify/Cloudflare)

**Phase 4 Acceptance Criteria:** นักพัฒนาภายนอกเขียน plugin เพิ่ม component type ใหม่ได้โดยไม่แก้ core code, export หน้าเพจเป็น Next.js project แล้วรันได้จริง
