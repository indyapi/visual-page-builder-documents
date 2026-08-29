# STYLE.md — Default Theme (ก่อนที่แอดมินจะปรับแต่งเอง)

เอกสารนี้อธิบาย visual identity เริ่มต้นของเว็บพอร์ตโฟลิโอ — ทุกค่าที่ระบุด้านล่าง
ถูก seed ไว้ใน `site_settings.theme` (ตาราง Supabase) และเป็นค่าที่ปุ่ม
"รีเซ็ตเป็นค่าเริ่มต้น" ใน Style popover จะอ้างอิงถึง เปลี่ยนได้ทั้งหมดภายหลังผ่าน
ThemeDrawer (แอดมิน) โดยไม่ต้องแก้โค้ด

แนวคิดหลัก: **"ภาพถ่ายที่ถูกครอปขอบจอ"** — จำลองความรู้สึกของช่างภาพที่ frame
ภาพหลุดขอบ, ตัวอักษรพาดผ่านภาพเหมือนสติกเกอร์บนอัลบั้มภาพ, พื้นดำแบบสตูดิโอถ่ายภาพ

---

## 1. Color tokens

| Token | ตัวแปร CSS | ค่าเริ่มต้น | ใช้ที่ไหน |
|---|---|---|---|
| Background | `--c-bg` | `#0a0a0a` | พื้นหลังทั้งเว็บ (เกือบดำสนิท ไม่ใช่ดำเพียว เพื่อลด eye strain) |
| Text | `--c-text` | `#f3f2ec` | ข้อความหลัก (ขาวอมครีม ไม่ใช่ขาวเพียว ให้ contrast นุ่มกว่า) |
| Accent | `--c-accent` | `#4de8d9` | สี teal — ปุ่มหลัก, ลิงก์ที่ active, ตัวเลขในแกลเลอรี/list |
| Accent 2 | `--c-accent2` | `#ff6a3d` | สีส้ม/ember — ใช้กับ section แบบ "feature" (แบนเนอร์เต็มจอ) เพื่อสร้าง contrast จาก teal |
| Muted | `--c-muted` | `#8a8a85` | ข้อความรอง, eyebrow label, timestamp |

Gradient ที่ผสมจาก accent สองตัว:
- `teal-gradient` = `linear-gradient(135deg, var(--c-accent), #1fa89b)`
- `ember-gradient` = `linear-gradient(135deg, var(--c-accent2), #ffc53d)`

**Contrast**: `--c-text` บน `--c-bg` เริ่มต้น = อัตราส่วนประมาณ 16.8:1 (เกิน AAA)
ถ้าแอดมินเปลี่ยนสี ควรเช็ค contrast เองคร่าวๆ (มีคำเตือนใน ThemeDrawer)

---

## 2. Typography

| Role | Font | น้ำหนัก | ใช้ที่ไหน |
|---|---|---|---|
| Display / crop-heading | `Archivo Black` (fallback `Noto Sans Thai`) | 900 (ตัวเดียว) | หัวข้อใหญ่ที่ครอปขอบจอ เช่น "GRIT", "MY PERFORMANCE" |
| Body | `Inter` (fallback `Noto Sans Thai`) | 400/500/600/700 | เนื้อหาทั่วไป, bio, ปุ่ม |
| Eyebrow / mono | `Space Mono` | 400 | label เล็กๆ ตัวพิมพ์ใหญ่ เว้นระยะ เช่น "Explore : list of contents", เลขกำกับแกลเลอรี |
| ภาษาไทย | `Noto Sans Thai` | 400–900 | fallback ทุก role ด้านบน เพื่อให้ข้อความไทยแสดงผลถูกต้องแม้ font หลักเป็นละติน |

Class หลักที่กำหนด typography ไว้ใน `index.css`:
- `.crop-heading` — line-height 0.85, letter-spacing -0.02em, uppercase, ใช้ font display
- `.eyebrow` — font mono, letter-spacing 0.15em, uppercase, ขนาดเล็ก (0.7rem)

---

## 3. Layout & spacing

- Container padding แนวนอน: `px-6` (mobile) → `md:px-16` (desktop)
- Section แต่ละอันคั่นด้วย `border-t border-white/5` — เส้นบางๆ แทนการ์ดแยก
- Section padding แนวตั้งมาตรฐาน: `py-24`
- Grid หลัก: 12 คอลัมน์ (`md:grid-cols-12`) สำหรับ hero/image_text, 2–4 คอลัมน์สำหรับ gallery

---

## 4. Texture & effect เริ่มต้น

- **Grain overlay** (`.grain`) — noise SVG opacity 5% ทับพื้นหลังของ hero section
  จำลองเกรนฟิล์ม/กล้อง ให้ความรู้สึกภาพถ่ายมากกว่าดิจิทัลล้วน
- **Grayscale → color on hover** — รูป hero ใช้ `grayscale` เป็นค่าเริ่มต้น แล้ว
  `hover:grayscale-0` เผยสีตอน hover (จำลองฟีลรูปในห้องมืด/ห้องล้างฟิล์ม)
- **Text-stroke outline** — คำใหญ่ที่ครอปขอบจอใช้ `text-transparent` +
  `-webkit-text-stroke: 2px var(--c-accent)` แทนตัวอักษรทึบ ให้ดูเบาและทันสมัย

---

## 5. Motion / Animation เริ่มต้น

| Interaction | Easing/Spring | Duration | หมายเหตุ |
|---|---|---|---|
| Scroll reveal (`Reveal`) | `cubic-bezier(0.16,1,0.3,1)` | 0.7s | fade + translateY(40px→0), `viewport once: true` |
| Scroll reveal scale (`RevealScale`) | เดียวกัน | 0.6s | fade + scale(0.92→1), ใช้กับการ์ด/รูป |
| Parallax hero word/portrait | scroll-linked (`useScroll`+`useTransform`) | — | เลื่อนช้ากว่าคอนเทนต์หลัก สร้างมิติความลึก |
| Gallery card hover | CSS transition | 0.5s | รูปขยาย `scale-110` ภายใน overflow-hidden |
| Feature banner background | `whileInView` scale 1.15→1 | 1.2s | เข้าเฟรมแบบซูมออกช้าๆ ตอน scroll เข้ามาเห็น |
| Drawer (slide-over) | spring `stiffness: 340, damping: 34` | — | ใช้กับ ThemeDrawer, PagesDrawer, SectionEditDrawer |
| Modal (ImagePicker, ItemEditor) | cubic-bezier(0.16,1,0.3,1) | 0.2–0.25s | scale(0.94→1) + fade |
| `prefers-reduced-motion` | — | 0.001ms | ทุก animation ปิดอัตโนมัติถ้าผู้ใช้ตั้งค่าระบบไว้ |

หลักการ: **reveal ครั้งเดียวตอนแรกเห็น (`once: true`)** ไม่ animate ซ้ำตอน scroll
กลับไปกลับมา เพื่อไม่ให้ลายตา, และ **springy แต่ไม่หวือหวา** — เน้นความรู้สึก
"เนียน" มากกว่า "ตื่นเต้น"

---

## 6. ค่าเริ่มต้นแบบ raw (สำหรับ seed / reset)

```json
{
  "site_title": "My Portfolio",
  "favicon_url": null,
  "logo_image_url": null,
  "logo_text": "",
  "footer_text": "",
  "footer_links": [],
  "theme": {
    "bg": "#0a0a0a",
    "text": "#f3f2ec",
    "accent": "#4de8d9",
    "accent2": "#ff6a3d",
    "muted": "#8a8a85"
  }
}
```

ตรงกับ default ที่ column `site_settings.theme` ตั้งไว้ใน `schema.sql` — ถ้าต้องการ
รีเซ็ตทั้งเว็บกลับค่าเริ่มต้น ให้รัน SQL:

```sql
update site_settings set theme = '{
  "bg": "#0a0a0a",
  "text": "#f3f2ec",
  "accent": "#4de8d9",
  "accent2": "#ff6a3d",
  "muted": "#8a8a85"
}'::jsonb where id = 1;
```

---

## 7. กติกาเวลาปรับ theme ใหม่ (คำแนะนำ ไม่ใช่ข้อบังคับทางเทคนิค)

1. เปลี่ยนทีละคู่ (bg+text) ก่อน แล้วเช็ค contrast ให้อ่านง่ายสุดก่อนค่อยไปปรับ accent
2. accent กับ accent2 ควรต่างโทนกันชัดเจน (เช่น เย็น vs อุ่น) เพราะใช้แยก section
   ปกติ (teal) กับ section แบบ feature/highlight (ember) ออกจากกัน
3. per-section override (ไอคอน 🎨 ตอนอยู่ในโหมดแก้ไข) จะสำคัญกว่าค่า global เสมอ —
   ถ้าอยาก reset section ใดให้กลับไปใช้สี global ให้กด "รีเซ็ตเป็นค่าเริ่มต้น" ใน
   popover ของ section นั้น
