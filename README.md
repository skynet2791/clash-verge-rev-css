# Clash Verge Rev CSS Customization (v1.0.0)

This directory manages the custom CSS overrides for optimizing the Clash Verge Rev UI, styled after a clean, high-contrast, premium **Shadcn Light** design system.

---

## 🎨 Theme & Design Specs (Shadcn Light)

- **Page Background**: Light Slate-50 (`#f8fafc`).
- **Sidebar**: Clean solid white (`#ffffff`) with a subtle `1px` right border (`rgba(0, 0, 0, 0.06)`).
- **Cards**: Solid white (`#ffffff`) with default padding/radii, matching native layout heights.
- **Accent Color**: Premium modern blue (`#3b82f6` for primary outlines, `#2563eb` for active text highlights).
- **Active Navigation**: Soft transparent blue capsule background (`rgba(59, 130, 246, 0.08)`) with blue text and icons.

---

## 🛠️ Critical Layout Bugs & DOM Fixes (Gotchas to Remember)

When continuing optimization, **always adhere to the following scoping selectors** to avoid breaking default React/Mui layout calculations:

### 1. Card Scoping (Home Page)
- **Problem**: Global `.base-content .MuiBox-root` styling broke nested layouts inside card bodies (especially the canvas chart controls in "Traffic Statistics").
- **Fix**: Use exact first-level selectors:
  ```css
  .base-content > .MuiGrid-container > [class*="MuiGrid-grid-"] > .MuiBox-root
  ```
  This targets only the 5 main cards (Subscription, Node, Network Settings, Proxy Mode, Traffic Stats) and leaves nested canvas controllers untouched.

### 2. Button Groups Collision (Settings & Proxies Tab)
- **Problem**: Overriding `.MuiButton-outlined` directly with `border-radius` and `border` broke Mui's `ButtonGroup` component, causing adjacent borders to double up and rounded corners to overlap.
- **Fix**: Exclude button group children from standard outline styles:
  ```css
  .MuiButton-outlined:not(.MuiButtonGroup-grouped) { ... }
  ```
  And style grouped items specifically to preserve native border collapse:
  ```css
  .MuiButtonGroup-grouped.MuiButton-outlined { ... }
  ```

### 3. Alert/Description Box Overflow (Network Settings)
- **Problem**: Forcing `padding` on outer wrappers (`.css-f9z05k`) combined with default `width: 100%` caused alert containers to overflow the right edge of the card due to `content-box` calculations.
- **Fix**: Remove padding from the outer wrapper and apply borders/backgrounds to the inner text element (`> .MuiTypography-root`) with `box-sizing: border-box`:
  ```css
  .base-content [class*="css-f9z05k"] {
    background-color: transparent !important;
    border: none !important;
  }
  .base-content [class*="css-f9z05k"] > .MuiTypography-root {
    box-sizing: border-box !important;
    background-color: rgba(59, 130, 246, 0.05) !important;
    border: 1px solid rgba(59, 130, 246, 0.22) !important;
  }
  ```

### 4. Active Profile & Proxy Node Highlight (Subscription & Proxy Tab)
- **Problem**: Active elements did not respond to standard `.active` selectors, and a thick vertical left-border line (`border-left: 4px solid ...`) disrupted card symmetry.
- **Fix**: Combine MUI state classes with HTML attribute selectors, and explicitly force a thin uniform border on all four sides:
  ```css
  .layout-content__right [class*="Mui-selected"],
  .layout-content__right [aria-selected="true"] {
    border: 1px solid rgba(59, 130, 246, 0.35) !important;
    border-left: 1px solid rgba(59, 130, 246, 0.35) !important; /* Forces thick line down to 1px */
    background-color: rgba(59, 130, 246, 0.14) !important; /* 14% opacity for clear visibility */
  }
  ```
  Additionally, hide any left vertical lines implemented via pseudo-elements:
  ```css
  .layout-content__right [aria-selected="true"]::before { display: none !important; }
  ```

### 5. High-Contrast Test Result Chips (Test Tab)
- **Problem**: Overriding `.MuiChip-root` globally turned success ("支持") and error ("Failed") chips grey, masking symbols.
- **Fix**: Exclude color variations from default chip overrides, and style them separately to color both background, text, and SVG icons:
  ```css
  .base-content .MuiChip-root:not(.MuiChip-colorSuccess):not(.MuiChip-colorError) { ... }
  .base-content .MuiChip-colorSuccess {
    background-color: #f0fdf4 !important;
    border: 1px solid rgba(22, 163, 74, 0.25) !important;
  }
  .base-content .MuiChip-colorSuccess svg { color: #16a34a !important; }
  ```

---

## 🔮 Future Suggestions & Optimization Roadmap

If starting a new session or doing further updates, consider the following roadmap:

1. **Dark Theme Harmonization**:
   - Create a sister file `theme-dark.css` mapped to **Shadcn Dark** specs (off-black backgrounds `#09090b`, dark zinc borders `#27272a`, and clean dark-mode typography).
2. **Hover State Transition Tuning**:
   - Add transition timings to card outlines and sidebar buttons (`transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1)`) to make elements feel premium and alive when hovered.
3. **Connections (连接) Page Optimization**:
   - The connections page lists real-time active connections. Apply unified headers, table borders, and customize speed metrics with distinct telemetry colors.
4. **Rules (规则) Page Color Customization**:
   - Style the rules list (Direct, Reject, Proxy rule cards) with custom border indicators to help quickly audit active routing definitions.
