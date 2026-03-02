# 3D Hero Section — WazapSuite Landing Page

**Tanggal:** 2026-02-28  
**Status:** Desain Divalidasi ✅

---

## Ringkasan Konsep

**"WazapSuite Universe"** — Dashboard aplikasi WazapSuite dalam tampilan 3D tilted/perspektif mengambang di tengah layar, dikelilingi 4 floating node yang masing-masing mewakili fitur utama. Scene berputar 360° secara auto-rotate, ditampilkan sebagai **scroll-driven image sequence** yang dipicu guliran pengguna.

---

## Keputusan Desain

| Aspek             | Keputusan                                                  |
| ----------------- | ---------------------------------------------------------- |
| **Objek Utama**   | Dashboard App WazapSuite (bukan smartphone)                |
| **Gaya Visual**   | Solid Futuristik — dark background, neon hijau/biru accent |
| **Jumlah Node**   | 4 node fitur utama mengorbit di sekeliling dashboard       |
| **Animasi**       | Auto-Rotate 360°                                           |
| **Metode Tampil** | Scroll-driven Image Sequence                               |

---

## Fitur yang Menjadi Node (4 Utama)

| Node               | Ikon/Label          | Warna Accent  |
| ------------------ | ------------------- | ------------- |
| 🚀 Bulk Messaging  | "10K Messages Sent" | Biru #3B82F6  |
| 🛡️ Anti-Ban Tech   | "Account Protected" | Hijau #22C55E |
| 🗺️ Maps Scraper    | "500 Leads Found"   | Amber #F59E0B |
| 🤖 AI Flow Builder | "Bot Active"        | Ungu #A855F7  |

---

## Pipeline Produksi Aset

```
1. GENERATE IMAGE (Google Whisk)
   → Render ~60–120 frame posisi rotasi berbeda (0°, 3°, 6°, ... 360°)

2. ANIMATE TO VIDEO (Google Flow)
   → Smooth 360° rotation video, 3–4 detik, 30fps

3. EXTRACT FRAMES (Video → Image Sequence)
   → Export ~90–120 frame PNG/WebP
   → Simpan di: /public/hero-3d/frame_{001..120}.webp

4. SCROLL-DRIVEN PLAYBACK
   → Canvas/img swap saat user scroll
   → Total scroll distance: ~200vh untuk satu putaran penuh
```

---

## Prompts Google Whisk & Google Flow

### 🖥️ PROMPT 1A — Dashboard 3D: Frame Pertama / Keyframe Depan (Google Whisk)

> **Keyframe 1 dari 2.** Tampak depan-kiri — ini adalah posisi awal sebelum rotasi dimulai.

```text
A premium 3D SaaS application dashboard rendered floating in space.
Camera angle: front-left perspective, tilted 20 degrees above horizon,
the dashboard faces slightly toward the viewer (front-facing primary view).
The left sidebar is clearly visible. Maximum content area visible.

Dashboard layout (recreate accurately):
- Left sidebar: narrow dark teal/navy panel (#0F172A) with navigation icons
  and menu items: Accounts, Inbox, Dashboard, AI Studio, Blast, Contacts,
  Maps Scraper, Settings. Top-left: teal circular logo + bold text "Wazap".
- Main content: light gray background (#F8FAFC), 4 white stat cards in a row:
  "Active Accounts 3/3", "Sent Today 10,247", "Active Jobs 2", "Success Rate 98%"
  each with a small colored icon (teal, blue, amber, green).
- Below stat cards: large white panel "Recent Activity" with two tabs
  "Blast Jobs" | "Warmer Jobs", showing a live progress list.
- Top-right header: date "28 February, 2026", bell icon, user avatar circle "AD".

Floating effect: the entire dashboard UI panel floats slightly above center,
casting a soft shadow downward. Neon teal glow (#14B8A6) radiates from edges.
Background: pure deep dark navy (#0A0F1E), completely clean with no other objects.
Lighting: cinematic studio light from top-left, crisp shadows.
Style: ultra-detailed 3D product render, 4K quality, centered composition.
No surrounding objects. Dashboard UI panel only.
```

---

### 🖥️ PROMPT 1B — Dashboard 3D: Frame Kedua / Keyframe Belakang (Google Whisk)

> **Keyframe 2 dari 2.** Tampak belakang-kanan — posisi akhir setelah memutar ~150°. Google Flow akan menganimasikan transisi antara Frame 1 → Frame 2 → Frame 1 (loop).

```text
A premium 3D SaaS application dashboard rendered floating in space.
Camera angle: back-right perspective, tilted 20 degrees above horizon.
The dashboard has rotated approximately 150 degrees horizontally —
now showing mostly the BACK and RIGHT SIDE of the floating UI panel.

What is visible from this angle:
- The back face of the dashboard panel: a clean dark charcoal surface
  (#1E293B) with subtle grid texture and soft reflective finish.
- The right edge: a thin sliver of the sidebar's right border visible,
  glowing faintly with teal neon (#14B8A6).
- The top edge: a thin strip showing the header bar from behind.
- The bottom edge: a subtle soft shadow cast downward on invisible ground.

The panel itself is the same shape and size as Frame 1, just rotated 150°.
The teal neon glow on the edges is still visible, now wrapping around
the far edges from this view.
Background: pure deep dark navy (#0A0F1E), completely clean.
Lighting: cinematic studio light from top-left, strong side rim light
on the right edge highlighting the teal glow.
Style: ultra-detailed 3D product render, 4K quality, centered composition.
No surrounding objects. Dashboard panel only.
```

### 🟦 PROMPT 2 — Node: Bulk Messaging (Google Whisk)

> Generate **1 gambar** floating stat card untuk fitur Blast/Bulk Messaging.

```text
A small futuristic floating UI stat card on a transparent background.
Design inspired by modern SaaS dashboard metric cards.
The card has a dark (#1E293B) background with a vivid blue (#3B82F6)
neon glow border and subtle glassmorphism effect.

Card layout:
- Top row: paper-plane send icon in blue + label "Messages Sent"
- Center: large bold white number "10,247"
- Bottom badge: green pill badge with text "↑ Active Now"
- Card size: wide rectangle, rounded corners (16px radius)

Style: 3D floating card, slight right-tilt perspective, soft drop shadow
with blue glow underneath, transparent/PNG background.
Ultra-detailed, 4K, premium SaaS UI style.
```

---

### 🟩 PROMPT 3 — Node: Anti-Ban Protection (Google Whisk)

> Generate **1 gambar** floating stat card untuk fitur Anti-Ban & Warmer.

```text
A small futuristic floating UI stat card on a transparent background.
The card has a dark (#1E293B) background with a vivid green (#22C55E)
neon glow border and subtle glassmorphism effect.

Card layout:
- Top row: shield-check icon in green + label "Account Protection"
- Center: large bold white text "All Safe"
- Bottom: small green text "0 Bans • Warmer Active"
- Card size: wide rectangle, rounded corners

Style: 3D floating card, slight left-tilt perspective, soft drop shadow
with green glow, transparent/PNG background.
Ultra-detailed, 4K, premium SaaS UI style.
```

---

### 🟨 PROMPT 4 — Node: Maps Scraper Results (Google Whisk)

> Generate **1 gambar** floating stat card untuk fitur Google Maps Scraper.

```text
A small futuristic floating UI stat card on a transparent background.
The card has a dark (#1E293B) background with a vivid amber (#F59E0B)
neon glow border and subtle glassmorphism effect.

Card layout:
- Top row: map-pin location icon in amber + label "Leads Scraped"
- Center: large bold white number "13,565"
- Bottom: small gray text "from Google Maps • Completed"
- Progress bar at bottom: full amber bar at 100%
- Card size: wide rectangle, rounded corners

Style: 3D floating card, slight top-right tilt perspective, soft drop shadow
with amber glow, transparent/PNG background.
Ultra-detailed, 4K, premium SaaS UI style.
```

---

### 🟪 PROMPT 5 — Node: AI Flow Builder (Google Whisk)

> Generate **1 gambar** floating mini-preview card untuk fitur AI Flow Builder.

```text
A small futuristic floating UI card on a transparent background showing
a miniature AI flow builder diagram.
The card has a dark (#1E293B) background with a vivid purple (#A855F7)
neon glow border.

Card layout:
- Top row: brain/neural-network icon in purple + label "AI Flow Builder"
- Center: tiny node-graph diagram showing 3 connected circular nodes
  with arrows between them (like a chatbot flow), glowing lines
- Bottom: small text "Bot Active • Auto-Reply ON"
- Card size: slightly taller rectangle, rounded corners

Style: 3D floating card, bottom-left tilt perspective, soft drop shadow
with purple glow, transparent/PNG background.
Ultra-detailed, 4K, premium SaaS UI style.
```

---

### 🎬 PROMPT VIDEO — Google Flow (Generate Baru, v2)

> Upload **Prompt 1A** (keyframe depan) dan **Prompt 1B** (keyframe belakang) ke Google Flow sebagai reference images, lalu gunakan prompt teks berikut untuk generate video baru dari awal.

```text
A cinematic 3D product showcase animation. A sleek SaaS analytics dashboard
panel floats in a dark deep navy space with teal neon edge glow.

The panel slowly rotates 360 degrees revealing both its front face
(showing a clean light-gray web application UI with sidebar navigation
and data cards) and its dark textured back face.

As it rotates, 4 small dark rectangular floating metric cards appear
around the panel like orbiting satellites. Each card is flat, small,
with rounded corners and a colored glowing border:

- Upper-left card: blue glowing border, send/arrow icon, label "Sent Today",
  number "10,247", small green badge
- Upper-right card: green glowing border, shield icon, label "Security Status",
  text "Protected", small subtitle
- Lower-left card: amber glowing border, chart icon, label "New Leads",
  number "13,565", progress bar
- Lower-right card: purple glowing border, brain icon, label "AI Flows",
  small connected-nodes diagram, subtitle text

At the end the panel faces forward fully, dashboard UI clearly visible
with all stats and charts. Tiny glowing particles drift upward.
Camera is static. Loop seamlessly.
Style: ultra-premium cinematic SaaS product reveal, dark futuristic.
```

---

## Implementasi Web (Scroll-Driven Image Sequence)

### Struktur File

```text
new-client/
└── public/
    └── hero-3d/
        ├── frame_001.webp
        ├── frame_002.webp
        └── ... (90–120 frames)
```

### Komponen React (`Hero3DCanvas.tsx`)

```tsx
// Konsep implementasi scroll-driven image sequence
// Mirip teknik Apple iPhone page
"use client";
import { useEffect, useRef, useState } from "react";

const TOTAL_FRAMES = 120;
const SCROLL_RANGE = 2000; // px

export function Hero3DCanvas() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [images, setImages] = useState<HTMLImageElement[]>([]);
  const containerRef = useRef<HTMLDivElement>(null);

  // Preload semua frame
  useEffect(() => {
    const imgs: HTMLImageElement[] = [];
    let loaded = 0;
    for (let i = 1; i <= TOTAL_FRAMES; i++) {
      const img = new Image();
      const frameNum = String(i).padStart(3, "0");
      img.src = `/hero-3d/frame_${frameNum}.webp`;
      img.onload = () => {
        loaded++;
        if (loaded === TOTAL_FRAMES) setImages(imgs);
      };
      imgs.push(img);
    }
  }, []);

  // Render frame berdasarkan scroll
  useEffect(() => {
    if (images.length === 0) return;
    const canvas = canvasRef.current;
    const ctx = canvas?.getContext("2d");
    if (!canvas || !ctx) return;

    const handleScroll = () => {
      const scrollTop = window.scrollY;
      const progress = Math.min(scrollTop / SCROLL_RANGE, 1);
      const frameIndex = Math.floor(progress * (TOTAL_FRAMES - 1));
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(images[frameIndex], 0, 0, canvas.width, canvas.height);
    };

    window.addEventListener("scroll", handleScroll, { passive: true });
    handleScroll(); // render frame pertama
    return () => window.removeEventListener("scroll", handleScroll);
  }, [images]);

  return (
    <div ref={containerRef} style={{ height: `${SCROLL_RANGE + 100}vh` }}>
      <div className="sticky top-0 h-screen flex items-center justify-center">
        <canvas
          ref={canvasRef}
          width={1280}
          height={720}
          className="w-full max-w-5xl"
        />
      </div>
    </div>
  );
}
```

---

## Optimisasi Performa

- [ ] Gunakan format **WebP** untuk file lebih kecil
- [ ] Target ukuran per frame: **< 50KB**
- [ ] Total aset: **< 6MB** untuk 120 frame
- [ ] Tambahkan **loading skeleton** saat preload berlangsung
- [ ] Gunakan `IntersectionObserver` untuk lazy-start preloading

---

## Checklist Produksi

- [ ] Generate 1 frame referensi di Google Whisk (validasi visual)
- [ ] Generate video 360° di Google Flow
- [ ] Extract frames dengan FFmpeg: `ffmpeg -i input.mp4 -vf fps=30 -s 1280x720 public/hero-3d/frame_%03d.webp`
- [ ] Implementasi komponen `Hero3DCanvas.tsx`
- [ ] Integrasi ke `page.tsx` marketing (ganti `HeroShowcase`)
- [ ] Test scroll behavior di mobile & desktop
