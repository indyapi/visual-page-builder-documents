# CODING GUIDELINES — สำหรับ Vibe Coding

เอกสารนี้มีไว้ให้ AI agent (Claude Code หรือเครื่องมืออื่น) และตัวคุณเองยึดเป็นมาตรฐานเดียวกันทุกครั้งที่เขียนโค้ด เพื่อกันไม่ให้โค้ดแต่ละ session ไม่สอดคล้องกัน

## 1. หลักการทั่วไป

1. **อ่านเอกสาร 01-03 ก่อนเริ่ม ticket ใดๆ เสมอ** — ห้ามเดา schema/architecture เอง ถ้าเอกสารไม่ครอบคลุม ให้เพิ่มลงเอกสารก่อน ไม่ใช่ตัดสินใจเองเงียบๆ ในโค้ด
2. **ทำทีละ ticket** — 1 ticket = 1 PR/commit ที่ scope ชัดเจน ห้ามรวมหลาย ticket เข้าด้วยกัน เพราะ review ยากและ AI มักเบี่ยง scope เมื่อทำหลายอย่างพร้อมกัน
3. **schema เปลี่ยนแปลงต้องมาก่อนเสมอ** — ถ้า ticket ต้องแก้ `PageNode`/`PageTree` type ให้แก้ที่ `packages/schema` ก่อน แล้วค่อยตามด้วย renderer/editor-core/component-library
4. **ห้าม hardcode ค่าที่ควรมาจาก schema/theme** — สี, spacing, font ต้องอ้างอิง theme token (ตั้งแต่เฟส 3) หรืออย่างน้อยเป็น constant ที่มาจากที่เดียว

## 2. Naming Convention

| สิ่งที่ตั้งชื่อ | รูปแบบ | ตัวอย่าง |
|---|---|---|
| React component | PascalCase | `SectionBlock.tsx` |
| Hook | camelCase, prefix `use` | `useCanvasSelection.ts` |
| Zustand store slice | camelCase, suffix `Store` | `editorStore.ts` |
| Zod schema | PascalCase, suffix `Schema` | `PageNodeSchema` |
| TypeScript type/interface | PascalCase | `PageNode`, `ResponsiveStyle` |
| ไฟล์ test | ชื่อเดียวกับไฟล์ต้นฉบับ + `.test.ts` | `renderer.test.ts` |
| Ticket/branch | `<prefix>-<number>-slug` | `p1-014-canvas-drop-zone` |

## 3. โครงสร้างไฟล์ต่อ Component

ทุก component ใน `component-library` ต้องมีไฟล์ครบชุดนี้:

```
components/Section/
├── Section.tsx           # React component
├── Section.schema.ts     # Zod schema สำหรับ props ของ component นี้
├── Section.test.tsx      # Unit/snapshot test
└── index.ts              # re-export
```

## 4. Commit Message Convention

ใช้ Conventional Commits ผูกกับ ticket id:

```
<type>(<ticket-id>): <คำอธิบายสั้น>

ตัวอย่าง:
feat(p1-014): เพิ่ม drag component จาก palette ลง canvas
fix(p2-011): แก้ conflict ระหว่าง dnd-kit กับ Moveable
test(p1-009): เพิ่ม snapshot test สำหรับ renderer node พื้นฐาน
docs(p1): อัปเดต schema เอกสารหลังเพิ่ม field responsive style
```

Type ที่ใช้: `feat`, `fix`, `test`, `docs`, `refactor`, `chore`

## 5. กติกาเขียน Test ควบคู่โค้ดเสมอ

- Component/component-library ใหม่ → ต้องมี snapshot test ก่อน merge
- แก้ schema → ต้องมี validation test ทั้ง valid/invalid case
- Editor action ใหม่ (drag, resize, undo) → ต้องมี Playwright test อย่างน้อย 1 happy-path

**กติกาเฉพาะ vibe coding:** ถ้า AI agent เขียนโค้ด feature ใหม่โดยไม่มี test แนบมาด้วยในรอบเดียวกัน ให้ถือว่า ticket นั้นยังไม่เสร็จ

## 6. Pull Request Checklist

ก่อน merge ทุก PR ต้องผ่าน:

- [ ] อ้างอิง ticket id ใน PR title
- [ ] `pnpm turbo run lint typecheck test` ผ่านทั้งหมด
- [ ] ไม่มี package ใด import ข้าม boundary ที่ผิดกฎ (ดู dependency direction ใน `02-ARCHITECTURE.md` หัวข้อ 1)
- [ ] ถ้าแก้ schema — เอกสาร `02-ARCHITECTURE.md` หัวข้อ 5 ได้อัปเดตตามแล้ว
- [ ] Acceptance criteria ของ ticket นั้น (จาก `03-ROADMAP-BACKLOG.md`) ผ่านครบ

## 7. เมื่อ AI Agent ไม่แน่ใจ

ให้ AI หยุดถามแทนการเดา ในกรณีต่อไปนี้:
- ต้องเพิ่ม field ใหม่ใน schema กลางที่กระทบหลาย package
- ต้องตัดสินใจ UX ที่ไม่ได้ระบุใน PRD (เช่น keyboard shortcut ที่ชนกัน)
- Ticket ดู scope ใหญ่กว่าที่ backlog ระบุ (อาจต้องแตกเป็นหลาย ticket ย่อย)

ห้าม "เดาไปก่อนแล้วค่อยแก้ทีหลัง" กับจุดที่กระทบ schema กลาง เพราะ schema คือจุดที่ทุก package พึ่งพา การแก้ย้อนหลังจะกระทบวงกว้าง
