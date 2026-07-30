---
name: Lumière Noir
colors:
  surface: '#121414'
  surface-dim: '#121414'
  surface-bright: '#37393a'
  surface-container-lowest: '#0c0f0f'
  surface-container-low: '#1a1c1c'
  surface-container: '#1e2020'
  surface-container-high: '#282a2b'
  surface-container-highest: '#333535'
  on-surface: '#e2e2e2'
  on-surface-variant: '#c2c6d8'
  inverse-surface: '#e2e2e2'
  inverse-on-surface: '#2f3131'
  outline: '#8c90a1'
  outline-variant: '#424656'
  surface-tint: '#b3c5ff'
  primary: '#b3c5ff'
  on-primary: '#002b75'
  primary-container: '#0066ff'
  on-primary-container: '#f8f7ff'
  inverse-primary: '#0054d6'
  secondary: '#f4ffc9'
  on-secondary: '#293500'
  secondary-container: '#c1ed00'
  on-secondary-container: '#546900'
  tertiary: '#c6c6cc'
  on-tertiary: '#2f3035'
  tertiary-container: '#717277'
  on-tertiary-container: '#f8f8fe'
  error: '#ffb4ab'
  on-error: '#690005'
  error-container: '#93000a'
  on-error-container: '#ffdad6'
  primary-fixed: '#dae1ff'
  primary-fixed-dim: '#b3c5ff'
  on-primary-fixed: '#001849'
  on-primary-fixed-variant: '#003fa4'
  secondary-fixed: '#c6f311'
  secondary-fixed-dim: '#add500'
  on-secondary-fixed: '#171e00'
  on-secondary-fixed-variant: '#3d4d00'
  tertiary-fixed: '#e2e2e8'
  tertiary-fixed-dim: '#c6c6cc'
  on-tertiary-fixed: '#1a1c20'
  on-tertiary-fixed-variant: '#45474b'
  background: '#121414'
  on-background: '#e2e2e2'
  surface-variant: '#333535'
typography:
  display-lg:
    fontFamily: Playfair Display
    fontSize: 48px
    fontWeight: '700'
    lineHeight: 56px
    letterSpacing: -0.02em
  display-lg-mobile:
    fontFamily: Playfair Display
    fontSize: 36px
    fontWeight: '700'
    lineHeight: 44px
    letterSpacing: -0.02em
  headline-md:
    fontFamily: Playfair Display
    fontSize: 32px
    fontWeight: '600'
    lineHeight: 40px
  headline-sm:
    fontFamily: Playfair Display
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
  title-lg:
    fontFamily: Inter
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 28px
  body-lg:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-sm:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  label-caps:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '700'
    lineHeight: 16px
    letterSpacing: 0.1em
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  container-margin: 20px
  stack-gap: 16px
  glass-padding: 24px
  section-v-space: 40px
---

## Brand & Style

The design system is engineered for high-end creative portfolios and luxury showcases. It balances the timeless elegance of editorial typography with the cutting-edge aesthetics of modern digital interfaces. 

The visual style is **Glassmorphism**, characterized by depth, translucency, and light-refracting surfaces. The experience should feel cinematic and immersive, positioning the user's content as the focal point within a dark, premium environment. Motion should be fluid and dampened, reinforcing the sense of physical weight and high-quality craftsmanship.

**Emotional Response:** Sophisticated, exclusive, visionary, and meticulous.

## Colors

The palette is anchored by a deep charcoal base to provide maximum contrast for gallery items and accents. 

- **Base:** The primary background is `#0F1115`. Avoid pure black to maintain visible depth when shadows are applied.
- **Accents:** Vibrant Blue (`#0066FF`) is used for primary actions and system-critical paths. Lime (`#D1FF26`) serves as a high-visibility disruptor for highlights, badges, or "New" indicators.
- **Surfaces:** Utilize varying opacities of White (`#FFFFFF`) for glass containers, typically ranging from 2% to 10% alpha, layered over background blurs.
- **Text:** Primary text is pure white. Secondary text uses 60% opacity white to maintain hierarchy without introducing new hues.

## Typography

This design system employs a high-contrast typographic pairing to evoke an editorial feel.

- **Headlines:** Use **Playfair Display**. It should be set with tighter letter-spacing for large display sizes to maintain a "fashion-masthead" look. Large titles should utilize the `display-lg-mobile` variant on mobile devices to ensure readability.
- **Body & UI:** Use **Inter**. This provides a clean, systematic counterpoint to the decorative headlines. It ensures high legibility for metadata, descriptions, and functional UI labels.
- **Labels:** Use `label-caps` for small identifiers like categories or timestamps to create a distinct visual rhythm.

## Layout & Spacing

The layout philosophy follows a **Fluid Grid** model with generous safe areas to let content breathe.

- **Mobile Grid:** A 4-column fluid layout with a 20px outer margin and 16px gutters.
- **Visual Rhythm:** Spacing is strictly based on an 8px scale. Use larger vertical gaps (40px+) between major content sections to emphasize the "gallery" feel.
- **Content Focus:** For portfolio items, use aspect-ratio locked containers (e.g., 4:5 or 1:1) to maintain a clean masonry or list flow.
- **Adaptation:** On larger devices (tablets), the layout expands to 8 columns, increasing the outer margins to 40px while maintaining the internal gutter at 24px for a more expansive aesthetic.

## Elevation & Depth

Depth is the cornerstone of this design system, achieved through transparency and light simulation rather than heavy drop shadows.

- **Surface Strategy:** Use `Backdrop-filter: blur(20px)` on all floating containers.
- **Borders:** Every glass element must have a thin 1px border. Use a linear gradient for the border: `rgba(255,255,255,0.15)` at the top-left to `rgba(255,255,255,0.05)` at the bottom-right. This simulates a "rim light" effect.
- **Shadows:** Use a singular, very soft "ambient" shadow for elevated cards: `0px 20px 40px rgba(0,0,0,0.4)`. Avoid harsh, high-opacity shadows.
- **Layering:** Background elements should have a lower blur (10px) while active modal layers or foreground cards use a higher blur (25px) to pull focus.

## Shapes

The shape language is sophisticated and modern. 

- **Default Corners:** Use `0.5rem` (8px) for standard UI elements like inputs and small buttons.
- **Cards & Containers:** Use `rounded-lg` (16px) for main content cards and glass panels. This creates a soft, organic feel that contrasts against the sharp serif typography.
- **Action Elements:** Primary buttons may use `rounded-xl` (24px) or full pill-shaping to distinguish them from informational containers.

## Components

- **Glass Cards:** The primary vessel for content. Must feature the backdrop blur, the "rim light" gradient border, and a subtle internal padding of 24px.
- **Buttons:** 
  - *Primary:* Solid Vibrant Blue with white `Inter` Bold text. 
  - *Secondary:* Ghost style with the 1px rim-light border and Lime text.
- **Chips/Badges:** Small, pill-shaped elements with 10% Lime background and 100% Lime text for high-contrast "New" or "Pro" tags.
- **Input Fields:** Semi-transparent dark fills (5% white) with a 1px border that glows (1px Blue outer shadow) when focused.
- **Lists:** Clean dividers using 5% white opacity lines. Metadata should be set in `body-sm` with 60% opacity.
- **Navigation:** A bottom-docked glass bar with a heavy backdrop blur. Active icons should use the Vibrant Blue color with a small glow effect.