# AGENTS.md — Blueprint Visual Page Builder

ไฟล์นี้เขียนแบบ tool-agnostic ตามมาตรฐาน AGENTS.md ซึ่ง OpenCode CLI อ่านได้เองโดยตรง (ไม่ต้องมีไฟล์ config เฉพาะ tool เพิ่ม) และ AI coding agent ตัวอื่น (Cursor, Aider, Codex ฯลฯ) ก็อ่านแล้วทำงานในโปรเจคนี้แบบเดียวกันได้

โฟกัสของไฟล์นี้คือ 3 กลไกที่ทำให้ agent ทำงานข้าม session ได้โดยไม่ต้องไล่อ่านทุกอย่างใหม่ทุกครั้ง: **memory, current state, และ summary** — ทั้งหมดเป็น **flat file ในโปรเจค ไม่พึ่ง MCP server ใดๆ** (โปรเจคนี้เลิกใช้ MCP แล้ว)

อ้างอิงเอกสารหลักของโปรเจคที่ `docs/01-PRD.md` ถึง `docs/04-CODING-GUIDELINES.md` — ไฟล์นี้ไม่ duplicate เนื้อหาจากที่นั่น แต่บอกว่า "เมื่อไหร่ต้องอ่านอะไร" และ "จะจำอะไรข้าม session อย่างไร"

## 0. โครงไฟล์ state ที่ไฟล์นี้ดูแล

```
personal-portfolio/
├── AGENTS.md                 # ไฟล์นี้
├── instruction.md                 # instruction
├── docs/                     # PRD, architecture, backlog, guidelines (static, ไม่ค่อยเปลี่ยน)
│   └── tickets/
│       └── phase-<N>/
│           └── <ticket-id>.md   # ticket ละเอียดต่อใบ, generate จาก backlog (ดูหัวข้อ 7) — ไม่ overwrite มือ ถ้า backlog เปลี่ยนให้รัน /generate-tickets ใหม่เฉพาะ ticket ที่กระทบ
├── .opencode/
│   └── command/
│       ├── save-memory.md       # คำสั่ง /save-memory — สั่งบันทึก memory+summary+CURRENT.md ครบในทีเดียว (ดูหัวข้อ 5)
│       ├── generate-tickets.md  # คำสั่ง /generate-tickets — สร้าง docs/tickets/ จาก backlog ทีละ epic (ดูหัวข้อ 7)
│       └── start-ticket.md      # คำสั่ง /start-ticket — เช็ค CURRENT.md+memory+phase gate+ticket spec ก่อนเริ่มเขียนโค้ด (ดูหัวข้อ 5)
└── .agent/
    ├── CURRENT.md             # สถานะล่าสุดของโปรเจค (มีไฟล์เดียว, overwrite ตลอด)
    ├── memory/
    │   └── notes.md           # ที่เก็บ memory ถาวรข้าม session (ดูหัวข้อ 2) — ไฟล์เดียว ไม่มี MCP
    └── summaries/
        └── YYYY-MM-DD-<ticket-id>.md   # สรุปราย session/ticket (ไฟล์ใหม่ทุกครั้ง)
```

## 1. Tooling — ไม่ใช้ MCP server แล้ว

โปรเจคนี้เลิกพึ่ง MCP server (filesystem, git, supabase, memory ฯลฯ) ทั้งหมด agent ทำงานผ่าน **bash/CLI ตรงๆ** เท่านั้น:

| งาน | ใช้ | กติกา |
|---|---|---|
| อ่าน/เขียนไฟล์ | เครื่องมือแก้ไฟล์ปกติของ agent | จำกัดขอบเขตไว้ที่ root ของ repo เท่านั้น |
| git | คำสั่ง `git` ผ่าน bash | ห้าม force-push หรือ rewrite history โดยไม่ถามผู้ใช้ก่อนเสมอ |
| Supabase / DB | Supabase CLI (`supabase ...`) ผ่าน bash แทน MCP | **ห้ามรัน migration หรือคำสั่งที่แก้ข้อมูลจริงบน environment ที่ไม่ใช่ local/dev โดยไม่ยืนยันกับผู้ใช้ก่อนทุกครั้ง** |
| e2e test / debug | `pnpm playwright test` ผ่าน bash | ใช้ headed/`--debug` mode เฉพาะตอน test fail และต้องการ debug interactive |
| **Memory ข้าม session** | ไฟล์ `.agent/memory/notes.md` เท่านั้น | ดูหัวข้อ 2 — ห้ามพึ่ง memory MCP อีกต่อไป |

**กติกาทั่วไปเมื่อคำสั่ง bash ล้มเหลว:** ถ้า error เป็น auth/credential ให้แจ้งผู้ใช้ให้ตั้งค่าใหม่แทนการเดา credential เอง

## 2. Memory Protocol — จำอะไรข้าม session

**Memory ในที่นี้ ≠ เอกสารใน `docs/`** เอกสารใน docs/ คือ spec ที่ตั้งใจเขียนไว้ล่วงหน้า ส่วน memory คือสิ่งที่ **ค้นพบระหว่างทาง** ที่ยังไม่ถูกจดไว้ที่ไหน และเก็บทั้งหมดไว้ที่ **`.agent/memory/notes.md` ไฟล์เดียว** (ไม่มี MCP server แล้ว)

### ควรบันทึกอะไร
- การตัดสินใจทางเทคนิคที่เบี่ยงจาก spec เดิมเล็กน้อย พร้อมเหตุผล (เช่น "เปลี่ยนจาก Moveable resize handle แบบ 8 จุดเป็น 4 จุด เพราะชนกับ dnd-kit sensor")
- Gotcha/บั๊กที่เจอซ้ำๆ และวิธีแก้ที่ยืนยันแล้วว่าใช้ได้ (เช่น "Lexical serialize ต้อง call `.toJSON()` ไม่ใช่ `JSON.stringify()` ตรงๆ ไม่งั้น node ชนิด custom หาย")
- ข้อจำกัดของ environment ที่ค้นพบเอง (เช่น "Supabase local dev ต้องรัน `supabase start` ก่อน `pnpm dev` ไม่งั้น connection refused")

### ไม่ควรบันทึกอะไร
- อะไรก็ตามที่มีอยู่แล้วใน `docs/` — ถ้าพบว่า docs ผิด/ไม่ตรงกับของจริง ให้ไปแก้ docs โดยตรง ไม่ใช่จด exception ไว้ใน memory
- รายละเอียดของแต่ละ ticket ที่ทำ (นั่นคือหน้าที่ของ **Summary**, หัวข้อ 4)
- ความลับ/credential ใดๆ

### วิธีบันทึก (คำสั่งมาตรฐานสำหรับ agent)

Append เข้า `.agent/memory/notes.md` ด้วยรูปแบบนี้เสมอ (ห้ามเขียนทับไฟล์เดิม, ห้ามลบของเก่า):

```
## [YYYY-MM-DD] <หัวข้อสั้น>
Package/Component: <ที่เกี่ยวข้อง>
บันทึก: <รายละเอียด 1-3 บรรทัด>
```

กติกาการใช้ไฟล์นี้:
1. **ก่อนเริ่ม ticket ใหม่ที่แตะ package/component เดิม** — grep หา keyword ที่เกี่ยวข้องใน `.agent/memory/notes.md` ก่อนเสมอ (เร็วกว่าและกัน bug ซ้ำ) เช่น `grep -i "lexical" .agent/memory/notes.md`
2. **ทันทีที่ค้นพบ decision/gotcha/environment constraint ระหว่างทำงาน** — append เข้าไฟล์ทันที ไม่ต้องรอจบ session (กันลืม)
3. **เมื่อไฟล์เริ่มยาวเกิน ~300 บรรทัด** — จัดกลุ่มรายการที่เกี่ยวกับ package/component เดียวกันให้อยู่ติดกัน และรวมรายการซ้ำซ้อน/ล้าสมัยเป็นบรรทัดเดียว แทนการปล่อยให้ไฟล์ยาวไปเรื่อยๆ (ห้ามลบเนื้อหาที่ยังใช้ได้จริง)

## 3. Current State Protocol — `.agent/CURRENT.md`

ไฟล์เดียวที่สะท้อนสถานะโปรเจค ณ ปัจจุบัน overwrite ทับของเดิมทุกครั้ง (ไม่ append) เพื่อไม่ให้ agent อ่านข้อมูลเก่าที่ไม่จริงแล้ว

### Template

```markdown
# Current State
Last updated: <YYYY-MM-DD HH:MM> by <agent/session ref>

## Phase ปัจจุบัน
Phase <N> — <ชื่อเฟส>
Acceptance criteria ของเฟสนี้: <ผ่านแล้ว / ยังไม่ผ่าน, ขาดอะไร>

## Ticket ที่กำลังทำ
<ticket-id> — <ชื่อ> — สถานะ: <in-progress / blocked / รอ review>

## Test status ล่าสุด
lint: ✅/❌  typecheck: ✅/❌  unit: ✅/❌  e2e: ✅/❌/ยังไม่ได้รัน

## Blocker (ถ้ามี)
<คำอธิบายสั้น + สิ่งที่ต้องการจากผู้ใช้เพื่อไปต่อ>

## ticket ถัดไปตาม backlog
<ticket-id ถัดไป>
```

### กติกา
- **อ่านไฟล์นี้เป็นอย่างแรกสุดของทุก session ก่อนอ่านอะไรอื่น** — ถ้า `CURRENT.md` บอกว่ามี blocker ให้ถามผู้ใช้เรื่องนั้นก่อนเริ่มงานใหม่
- **อัปเดตไฟล์นี้ทุกครั้งที่จบ ticket หรือจบ session** แม้ ticket จะยังไม่เสร็จก็ต้องอัปเดตสถานะปัจจุบันไว้
- ห้ามปล่อยให้ไฟล์นี้ค้างข้อมูลเก่าข้ามหลาย session — ถ้าพบว่าไม่ตรงกับความเป็นจริง (เช่น commit ล่าสุดไปไกลกว่าที่ไฟล์บอก) ให้ sync ให้ตรงก่อนทำงานต่อ

## 4. Summary Protocol — `.agent/summaries/`

ต่างจาก `CURRENT.md` ตรงที่ summary คือ **log ประวัติ** ไม่ overwrite — ไฟล์ใหม่ 1 ไฟล์ต่อ session หรือต่อ ticket ที่ปิดงาน

### Template ต่อไฟล์ (`.agent/summaries/YYYY-MM-DD-<ticket-id>.md`)

```markdown
# Session Summary — <ticket-id>: <ชื่อ>
วันที่: <YYYY-MM-DD>

## สิ่งที่ทำ
- <รายการสั้นๆ>

## การตัดสินใจสำคัญ
- <ถ้ามี — ถ้าเป็นการตัดสินใจถาวรที่กระทบ spec ให้ลิงก์ไปที่ memory entry ด้วย>

## ผลตรวจก่อนปิดงาน
lint/typecheck/test: <ผ่าน/ไม่ผ่าน>, e2e: <ผ่าน/ไม่ผ่าน/ข้าม>

## สิ่งที่ยังไม่เสร็จ / ต่อยอดได้
- <ถ้ามี>
```

### กติกาการอ่าน summary (สำคัญเรื่องประหยัด token)
- **ไม่ต้องอ่านทุกไฟล์ใน `.agent/summaries/` ทุกครั้ง** — `CURRENT.md` ควรมีข้อมูลพอสำหรับ 90% ของการเริ่มงานต่อ
- อ่าน summary ย้อนหลังเฉพาะตอน: ผู้ใช้ถามถึง ticket เก่าที่ปิดไปแล้ว, หรือกำลัง debug บางอย่างที่สงสัยว่าเกี่ยวกับการตัดสินใจในอดีต
- เมื่อไฟล์ใน `.agent/summaries/` เริ่มเยอะ (>30 ไฟล์) ให้รวบ summary ที่เก่ากว่า 1 เฟสก่อนหน้าเป็นไฟล์เดียว (`.agent/summaries/phase-1-archive.md`) แทนการเก็บทุกไฟล์แยกไว้ตลอดไป — เก็บของ 1-2 เฟสล่าสุดแบบละเอียดพอ

## 5. ลำดับการทำงานเมื่อเริ่ม session ใหม่ (รวมทุกอย่างเข้าด้วยกัน)

> **Skills คือเครื่องยนต์ workflow ของโปรเจคนี้** (ดูรายละเอียดใน `.opencode/skills/`): `blueprint-ticket-runner` (entry + execution + DoD), `blueprint-context-budget` (โหลด context เท่าที่จำเป็น), `blueprint-schema-guard` (แตะ schema กลาง), `blueprint-component-scaffold` (สร้าง component ใหม่), `blueprint-pr-gatekeeper` (ตรวจก่อนปิดงาน) — **ใช้ตามบริบทตลอดการทำ ticket ทุกครั้ง** ไม่ว่าจะผ่าน `/start-ticket` หรือผู้ใช้ขอทำ ticket โดยตรง

1. รัน `/start-ticket <ticket-id>` (นิยามไว้ที่ `.opencode/command/start-ticket.md`) — คำสั่งนี้ทำ pre-flight ให้ครบ: อ่าน `.agent/CURRENT.md`, เช็ค phase gate เทียบกับ `docs/03-ROADMAP-BACKLOG.md`, อ่าน ticket spec จาก `docs/tickets/`, grep `.agent/memory/notes.md` หา context เก่าที่เกี่ยวข้อง, แล้วสรุป scope ให้ผู้ใช้ยืนยันก่อนเริ่ม — **จากนี้ให้ทำงานต่อโดยเรียกใช้ skill `blueprint-ticket-runner` (เครื่องยนต์ workflow) ซึ่งจะรับช่วงประกาศ scope, โหลด context ผ่าน `blueprint-context-budget`, ทำงานทีละข้อ, และปิดงานด้วย DoD**
2. ระหว่างทำ ถ้าแตะ schema กลางให้ใช้ skill `blueprint-schema-guard`, ถ้าสร้าง component ใหม่ให้ใช้ `blueprint-component-scaffold` → ระหว่างทางถ้าเจอ decision/gotcha/environment constraint ใหม่ ให้ append เข้า `.agent/memory/notes.md` ทันที (ห้ามรอจบงาน)
3. ก่อน merge/commit สุดท้าย ตรวจตาม PR Checklist ใน `docs/04-CODING-GUIDELINES.md` หัวข้อ 6 ด้วยตัวเอง (lint/typecheck/test, dependency direction, AC ของ ticket)
4. จบงาน: รันคำสั่ง `/save-memory` (นิยามไว้ที่ `.opencode/command/save-memory.md`) เพื่อ append memory, เขียน summary ใหม่ที่ `.agent/summaries/`, และ overwrite `.agent/CURRENT.md` ให้ครบในขั้นตอนเดียว — ถ้าผู้ใช้ตัดจบ session โดยไม่ได้สั่งคำสั่งนี้ ให้ agent เสนอรันเองก่อนจบงาน

## 6. สิ่งที่ agent ต้องไม่ทำเด็ดขาดเกี่ยวกับ state files เหล่านี้

- ห้ามลบไฟล์ใน `.agent/summaries/` เอง (เป็น history) — รวบ/archive ได้ แต่ไม่ลบทิ้งเงียบๆ
- ห้ามเขียนทับ (`.agent/memory/notes.md`) ทั้งไฟล์ — append เท่านั้น ยกเว้นตอนรวบ/จัดกลุ่มตามกติกาในหัวข้อ 2 ข้อ 3 ซึ่งต้องคงเนื้อหาที่ยังใช้ได้ไว้ครบ
- ห้ามเขียน credential, API key, หรือข้อมูลลูกค้าใดๆ ลงใน memory/summary/current ไฟล์เหล่านี้ (จะถูก commit เข้า git)
- ห้ามให้ `CURRENT.md` กับ `docs/03-ROADMAP-BACKLOG.md` ขัดแย้งกันโดยไม่แจ้งผู้ใช้ — ถ้า ticket ใน `CURRENT.md` ไม่มีอยู่จริงใน backlog แล้ว ให้ถือว่า backlog หรือ CURRENT.md ผิดพลาด และถามผู้ใช้ก่อนไปต่อ

## 7. Ticket Generation Protocol — `docs/tickets/`

**ต่างจาก memory/summary/CURRENT.md ตรงที่ `docs/tickets/` ไม่ใช่ state ที่เปลี่ยนทุก session** แต่เป็นเอกสารกึ่ง static — generate ครั้งแรกจาก `03-ROADMAP-BACKLOG.md` แล้วอัปเดตเฉพาะตอน backlog เปลี่ยนจริง

### ที่มา
สร้างผ่านคำสั่ง `/generate-tickets` (นิยามไว้ที่ `.opencode/command/generate-tickets.md`) — อ่าน `01-PRD.md`, `02-ARCHITECTURE.md`, `03-ROADMAP-BACKLOG.md`, `04-CODING-GUIDELINES.md` แล้วสร้าง `docs/tickets/phase-<N>/<ticket-id>.md` ทีละใบตาม ticket ที่ระบุใน backlog

### กติกา
- **ห้ามแต่ง acceptance criteria หรือรายละเอียดที่เอกสารต้นทางไม่ได้ระบุ** — ถ้าไม่มีให้ใส่ `[TBD - ต้องกำหนดเพิ่มก่อนเริ่มงานจริง]` แทนการเดา (ตามหลัก "หยุดถามแทนเดา" ใน `04-CODING-GUIDELINES.md` หัวข้อ 7)
- Generate ทีละ Epic ไม่ใช่ทั้ง backlog รวดเดียว เพื่อลด context drift และให้ตรวจทานได้ทีละก้อน
- หลัง generate เสร็จแต่ละ epic ให้ไล่เทียบ ticket id ที่สร้างกับ `03-ROADMAP-BACKLOG.md` ว่าไม่ตกหล่น/ไม่แต่งเพิ่มเอง
- `docs/tickets/<ticket-id>.md` ที่มีอยู่แล้ว **ห้าม overwrite เงียบๆ** เมื่อรัน `/generate-tickets` ซ้ำ — ถ้าพบว่ามีไฟล์อยู่แล้วและเนื้อหาต่างจาก backlog ปัจจุบัน ให้แจ้งผู้ใช้และถามก่อนว่าจะ overwrite หรือ skip
- เมื่อ ticket ถูกปิดงานจริงและมี `.agent/summaries/YYYY-MM-DD-<ticket-id>.md` แล้ว ไม่ต้องแก้ไฟล์ ticket.md เดิม (เป็นคนละหน้าที่กัน — ticket.md คือ spec ก่อนทำ, summary คือ log หลังทำ)