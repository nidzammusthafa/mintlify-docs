# Homepage Performance Analysis

Analyzed `new-client/src/app/[locale]/(marketing)/page.tsx` against `web-performance-seo` and `remotion-best-practices`.

## 1. CSS & Rendering Performance (Web Performance SEO)

The `web-performance-seo` skill flagged several areas that might impact CLS (Cumulative Layout Shift) and LCP (Largest Contentful Paint), particularly regarding complex CSS filters.

### Findings

- **Excessive Backdrop Blurs:**
  - **Hero Background:** `src/app/[locale]/(marketing)/page.tsx:32`: `blur-3xl` on a large div. This forces the browser to calculate blur for a large area.
  - **Dashboard Preview Glow:** `src/app/[locale]/(marketing)/page.tsx:65`: `blur` on the glow effect.
  - **Contact Section Blobs:** `src/app/[locale]/(marketing)/page.tsx:276-277`: multiple `blur-3xl` elements.
  - **Navbar:** `src/components/marketing/layout/navbar.tsx` uses `backdrop-blur-md` and `backdrop-blur-lg`.

- **Gradient Text:**
  - `src/app/[locale]/(marketing)/page.tsx:33`: `bg-clip-text text-transparent` used for the H1 title.
  - **Risk:** Can cause accessibility issues in high-contrast modes or if the gradient fails to load.

- **Low Opacity Backgrounds:**
  - Multiple instances of `/10`, `/20`, `/30` opacity (e.g., `bg-primary/10`, `bg-muted/30`).
  - **Risk:** Potential contrast issues if text is placed on top without verification.

### Recommendations

1.  **Replace CSS Blurs:** For static background blobs, use pre-blurred PNG/WebP images instead of CSS `filter: blur()`. This significantly reduces main-thread rendering time.
2.  **Gradient Fallback:** Add a `@media (prefers-contrast: no-preference)` block to ensure gradient text degrades gracefully to a solid color.
3.  **Contrast Check:** Verify all low-opacity backgrounds against text colors to ensure a contrast ratio of at least 4.5:1.

## 2. Video & Animation Performance (Remotion Best Practices)

The `remotion-best-practices` skill emphasizes efficient media handling to avoid layout shifts and heavy downloads.

### Findings

- **Heavy Initial Load (Hero):**
  - `HeroShowcase` is loaded immediately (client-side via `ssr: false`).
  - The `HeroShowcase` component itself contains multiple complex scenes (`FeatureExplosion`, `DashboardZoom`, etc.) and calculates interpolations every frame.
  - **Impact:** High JS execution time on page load, potentially delaying TTI (Time to Interactive).

- **Multiple Autoplaying Players:**
  - The page includes 3 `FeatureDeepDive` sections, each with a potentially autoplying `FeatureVideo`.
  - `src/components/marketing/FeatureVideo.tsx`: Sets `autoPlay={true}` and `loop={true}` explicitly.
  - **Impact:** If all 4 players (Hero + 3 Features) load and play simultaneously (even if off-screen), it will consume significant CPU/GPU and bandwidth.

### Recommendations

1.  **Lazy Load Videos:** Use `IntersectionObserver` (or `framer-motion`'s `useInView`) to only play `FeatureVideo` components when they are in the viewport. Pause them when they leave to save resources.
2.  **Optimize Hero:**
    - Ensure `HeroShowcase` is simplified for the initial frame.
    - Consider a "Click to Play" strategy for mobile.
3.  **Poster Images:** Add a `poster` prop or a placeholder image to `FeatureVideo` so the layout is stable before the heavy player loads.

## Action Plan

1.  **Phase 1: Quick Wins (CSS)**
    - Add gradient text fallbacks.
    - Audit and replace non-essential `backdrop-blur` with static images where possible.

2.  **Phase 2: Video Optimization**
    - Implement `useInView` for `FeatureVideo` to toggle `playing` prop.
    - Refactor `HeroShowcase` to be lighter on initial load.
