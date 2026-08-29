# instruction.md — คำสั่งประจำโปรเจคสำหรับ AI Coding Agent

> ไฟล์นี้วางไว้ที่ root ของ repo เพื่อให้  AI agent โหลดเป็น context อัตโนมัติทุกครั้งที่เริ่มงานในโปรเจคนี้

## โปรเจคนี้คืออะไร

Blueprint — Visual Page Builder, ไม่ผูก vendor, พัฒนาแบบ vibe coding ทั้งหมด
อ่านรายละเอียดเต็มที่ `/docs/01-PRD.md`, `/docs/02-ARCHITECTURE.md`, `/docs/03-ROADMAP-BACKLOG.md`, `/docs/04-CODING-GUIDELINES.md` ก่อนเริ่มงานทุกครั้ง

## กติกาบังคับ (ต้องทำตามเสมอ)

1. **ก่อนเขียนโค้ด ticket ใดๆ ให้เปิดอ่าน `03-ROADMAP-BACKLOG.md` หา ticket id ที่ตรงกันก่อน** ถ้าไม่มี ticket ที่ตรง ให้ถามผู้ใช้ว่าจะเพิ่ม ticket ใหม่หรือ scope ผิด
2. **ห้ามแก้ package มากกว่าที่ ticket ระบุ** ยกเว้น dependency chain ที่จำเป็นจริงๆ (เช่นแก้ schema แล้วต้องตาม renderer)
3. **ทุก feature ใหม่ต้องมาพร้อม test ในรอบเดียวกัน** (ดู `04-CODING-GUIDELINES.md` หัวข้อ 5)
4. **เคารพ dependency direction ของ monorepo** (`02-ARCHITECTURE.md` หัวข้อ 1): `schema` ← `renderer`/`component-library` ← `editor-core` ← `apps/editor`. ห้าม import ย้อนทาง
5. **PageTree JSON คือ source of truth** ห้ามเก็บ state สำคัญไว้ที่อื่นโดยไม่ sync กลับ schema

## คำสั่งที่ใช้บ่อย

```bash
pnpm install                    # ติดตั้ง dependency ทั้ง monorepo
pnpm turbo run dev              # รัน apps/editor แบบ dev
pnpm turbo run lint typecheck   # ตรวจก่อน commit
pnpm turbo run test             # unit test (Vitest) ทุก package
pnpm playwright test            # e2e test
```

## เมื่อเริ่ม session ใหม่

1. เช็คว่ากำลังอยู่ phase ไหน (ดู acceptance criteria ล่าสุดที่ผ่านใน `03-ROADMAP-BACKLOG.md`)
2. ถามผู้ใช้ว่าจะทำ ticket ไหนต่อ ถ้าไม่ได้ระบุมา ให้เสนอ ticket ถัดไปตามลำดับใน backlog ของเฟสปัจจุบัน
3. ห้ามข้าม phase gate — ถ้า acceptance criteria ของเฟสปัจจุบันยังไม่ผ่าน ห้ามเริ่ม ticket ของเฟสถัดไป แม้ผู้ใช้จะขอ ให้เตือนก่อน

## สิ่งที่ต้องระวังเป็นพิเศษ

- **dnd-kit vs Moveable conflict**: element หนึ่งควรมี interaction mode เดียวชัดเจนในแต่ละขณะ (drag mode หรือ resize mode) ห้ามเปิดทั้งคู่พร้อมกันโดยไม่ตั้งใจ
- **Renderer ต้องเป็น pure function ของ PageTree** ห้ามให้ renderer เข้าถึง Zustand store ของ editor โดยตรง (เพราะ renderer ต้องใช้ซ้ำตอน export/publish ที่ไม่มี editor context)
- **Schema versioning**: ถ้าแก้ schema แบบ breaking change ต้องเพิ่ม migration function และ bump `version` field ใน `PageTree`
