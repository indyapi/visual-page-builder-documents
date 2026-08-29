# ARCHITECTURE — Blueprint Visual Page Builder

## 1. Monorepo Structure

ใช้ monorepo ตั้งแต่ day 1 (แนะนำ **pnpm workspaces + Turborepo**) เพื่อแชร์ type และ schema ระหว่าง package โดยไม่ duplicate code

```
blueprint/
├── apps/
│   ├── editor/              # Next.js app — ตัว visual editor (dashboard, canvas UI)
│   └── docs/                # (เฟสหลัง) เว็บ docs ของโปรเจคเอง
├── packages/
│   ├── schema/              # JSON schema + Zod types ของ "page tree" — ใช้ร่วมกันทุก package
│   ├── renderer/             # แปลง page-tree JSON -> React elements (ใช้ทั้งใน editor canvas และ production render)
│   ├── component-library/   # Component ที่ลากวางได้ (Section, Container, Text, Image, Button, Form ฯลฯ)
│   ├── editor-core/          # Editor logic: Zustand store, history (undo/redo), selection, dnd-kit setup
│   ├── export/                # (เฟส 3-4) Export engine: JSON -> HTML/CSS, JSON -> React/Next component
│   ├── db/                    # Drizzle schema + Supabase client wrapper
│   └── cli/                   # (เฟส 4) CLI สำหรับ scaffold/deploy project
├── e2e/                        # Playwright tests
├── turbo.json
├── pnpm-workspace.yaml
└── package.json
```

**หลักการแบ่ง package:**
- `schema` เป็น package ที่ **ไม่มี dependency ต่อ package อื่นในระบบ** (leaf node) — ทุกอย่างพึ่งมัน
- `renderer` พึ่ง `schema` + `component-library` เท่านั้น ห้าม import จาก `editor-core`
- `editor-core` พึ่ง `schema` + `renderer` + `component-library`
- `apps/editor` ประกอบร่างทุกอย่างเข้าด้วยกัน

การแยกแบบนี้บังคับให้ renderer เป็น "pure function of JSON" จริง ๆ — คือหัวใจของหลักการ JSON-first ใน PRD

## 2. Tech Stack และเหตุผล

| เลเยอร์ | เทคโนโลยี | เหตุผล |
|---|---|---|
| Language | TypeScript (strict mode) | Type safety ข้าม package ใน monorepo, จำเป็นมากเมื่อ vibe coding เพื่อกัน AI เขียนโค้ดหลุด schema |
| Framework | Next.js (App Router) | SSR/SSG สำหรับทั้ง editor app และหน้าที่ publish ออกไป, ecosystem ใหญ่ |
| UI | Tailwind CSS + shadcn/ui | Utility-first, component ที่ copy เข้าโปรเจคได้ตรง ไม่ผูก dependency runtime หนัก |
| State | Zustand | เบา ไม่ boilerplate เหมือน Redux เหมาะกับ editor state ที่เปลี่ยนบ่อย (selection, drag state) |
| Drag & Drop | dnd-kit | Accessible, modular, ควบคุม sensor ได้ละเอียด เหมาะกับ nested container |
| Resize | Moveable | รองรับ resize/rotate/scale หลายรูปแบบ, แยกความรับผิดชอบจาก dnd-kit ชัดเจน |
| Rich Text | Lexical | Extensible, มี node schema ของตัวเองที่ serialize เป็น JSON ได้ตรงกับหลัก JSON-first |
| Database | Supabase (Postgres) | Auth + Storage + DB ในตัว, self-hostable ถ้าไม่อยากผูก cloud ของ Supabase เอง |
| ORM | Drizzle | Type-safe, SQL-like, เบา — เป็นชั้นกันไม่ให้ business logic ผูกกับ Supabase client โดยตรง |
| Test (unit) | Vitest | เร็ว, compatible กับ Vite ecosystem, ใช้กับ schema/renderer/store logic |
| Test (e2e) | Playwright | ทดสอบ drag-drop/resize จริงในเบราว์เซอร์ ซึ่ง unit test ทำไม่ได้ |
| Deploy | ดูหัวข้อ 3 | — |

## 3. คำแนะนำ Deployment: Vercel

สรุปสั้น: **แนะนำ Vercel เป็นค่าเริ่มต้น** สำหรับ `apps/editor`

เหตุผล:
1. Next.js App Router (โดยเฉพาะ feature ใหม่อย่าง Server Actions, Partial Prerendering) ถูก maintain โดยทีมเดียวกับ Vercel — compatibility และ preview deployment ลื่นไหลที่สุด
2. Preview deployment ต่อ PR ช่วยการทำงานแบบ vibe coding มาก — ให้ AI/คุณเห็นผลลัพธ์จริงทุกครั้งที่ commit โดยไม่ต้อง config เพิ่ม
3. Image optimization, Edge Functions ผูกกับ Next.js native ไม่ต้องเขียน adapter เพิ่ม

**เมื่อไหร่ควรเลือกอย่างอื่นแทน:**
- **Cloudflare (Pages/Workers)** — ถ้าลำดับความสำคัญคือต้นทุนต่ำสุดระยะยาวและ edge runtime ทั่วโลก แต่ต้องระวัง Next.js feature บางตัว (เช่น ISR แบบเต็มรูปแบบ) รองรับผ่าน `@cloudflare/next-on-pages` ซึ่งตามหลัง Vercel เสมอ
- **Netlify** — ใกล้เคียง Vercel ในแง่ DX แต่ community/adapter สำหรับ Next.js App Router ใหม่ๆ มักตามหลังเล็กน้อย

**สถาปัตยกรรมที่ไม่ผูก vendor:** เพราะ renderer/schema/component-library แยก package ชัดเจน การจะย้าย deploy target ภายหลัง (เช่นจาก Vercel ไป self-hosted Node server) ทำได้โดยแทบไม่แตะ business logic — กระทบแค่ config การ build/deploy เท่านั้น

## 4. Data Flow หลัก

```
User action (drag/resize/edit) 
   → dnd-kit / Moveable event 
   → editor-core: update Zustand store (page-tree JSON)
   → renderer: re-render canvas จาก page-tree JSON (React re-render ปกติ)
   → (on save) editor-core → API route → Drizzle → Supabase Postgres
   → (on publish, เฟส 3+) export package: page-tree JSON → HTML/CSS bundle → เก็บใน Supabase Storage / deploy target
```

## 5. Page Tree Schema (แนวคิดหลัก)

Schema เป็นหัวใจของทั้งระบบ ออกแบบคร่าวๆ (รายละเอียดเต็มจะอยู่ใน `packages/schema` เมื่อเริ่มเฟส 1):

```ts
type NodeId = string;

interface PageNode {
  id: NodeId;
  type: string;            // "section" | "container" | "text" | "image" | ... (component-library กำหนด type ที่รองรับ)
  props: Record<string, unknown>;   // validate ด้วย Zod schema เฉพาะ type นั้น
  style: ResponsiveStyle;   // style แยกตาม breakpoint (เฟส 2)
  children: NodeId[];
}

interface PageTree {
  version: string;         // schema version สำหรับ migration ในอนาคต
  rootId: NodeId;
  nodes: Record<NodeId, PageNode>;
}
```

หลักการสำคัญ: เก็บ node แบบ flat map (ไม่ nested object ลึก) เพื่อให้ drag-drop ย้าย node เป็นการแก้ `children` array แบบ O(1) ไม่ต้อง deep clone tree ทั้งต้น

## 6. Testing Strategy

- **Unit (Vitest):** schema validation, store reducers/actions, renderer pure-function output (snapshot)
- **Integration:** save → load round-trip ต้องได้ JSON เดิม 100% (byte-for-byte หลัง normalize)
- **E2E (Playwright):** drag component ลง canvas, resize, undo/redo, publish flow — รันบน CI ทุก PR

## 7. CI/CD Pipeline (แนวทาง)

1. Lint + typecheck (turbo run lint typecheck)
2. Unit test (Vitest) ทุก package
3. Build ทุก app/package (ตรวจ monorepo dependency graph ถูกต้อง)
4. E2E test (Playwright) รันบน preview deployment
5. Deploy preview อัตโนมัติทุก PR (Vercel), merge เข้า main = deploy production
