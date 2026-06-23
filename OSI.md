
# CDI Oversampling Interactive Explainer

**File:** `oversampling_explorer.html`

Single-file, dependency-free webpage (vanilla HTML/CSS/JS, no build step, no libraries) that teaches the oversampling condition in coherent diffractive imaging (BCDI). Built for a physics researcher following the BYU Sandberg lab CDI crash course.

## Notation

- `a` — object size
- `λ` — wavelength
- `z` — distance
- `N` — pixel counts (same on both planes)
- `m` — `a/p_s`
- `p_s` — real-space pixel
- `P_D` — detector pixel
- `L_s` — `N·p_s` (real-space window size)
- `L_D` — `N·P_D` (detector window size)

## Physics

**⚠️ Ground truth — do not alter:**

**Reciprocity:**
- $p_s = \frac{z\lambda}{N \cdot P_D}$
- $P_D = \frac{z\lambda}{N \cdot p_s}$

**Oversampling:**
- $OS = \frac{z\lambda}{a \cdot P_D} = \frac{L_s}{a} = \frac{N}{m} > 2$ (Nyquist condition)

**Resolution at threshold:**
- $p_s|_{OS=2} = \frac{2a}{N}$

**Wavelength conversion:**
- $\lambda[\text{Å}] = \frac{12.398}{E[\text{keV}]}$

**BCDI defaults:**
- Energy: 9 keV
- Object size: 300 nm
- Detector pixel: 55 µm
- Pixel count: 256
- Distance: 0.5 m
- **Result:** OS ≈ 4.18, p_s ≈ 4.89 nm, z_min ≈ 0.24 m

## Architecture

**Layout flow:** Hero → scrollytelling section → sandbox → reference cards → footer

### Scrollytelling
- **Canvas:** Sticky `#cv` on the left, persistent during scroll
- **Steps:** Eleven `.step` cards (data-step 0–10) on the right; layer 09/10 split the resolution trade-off from the OS = 2 ceiling
- **Animation:** IntersectionObserver sets target state per step; rAF loop lerps current toward target (factor 0.085) and calls `drawScene()`
- **Accessibility:** `prefers-reduced-motion` skips the lerp

### Renderer
- **Function:** `drawScene(ctx, s)` — single renderer used by both canvases
- **State `s`:** 
  - Opacity flags: `planes`, `beam`, `obj`, `sGrid`, `dGrid`, `sLabels`, `raysR`, `raysG` (red/green ray pairs fade independently so each step can emphasize one), `strip`, `dots`, `alias`, `fringeLbl`, `osChip`
  - Values: `os`, `osGeom`, `objH`
  - Object height: $\text{objH} = \frac{L_s}{\text{osGeom}}$ (geometric padding stays consistent with OS)

### Fringe strip
- Draws true sinc² with fringe spacing = `os × strip pixel pitch`. Everything outside the central lobe (`|u| > 1`) is scaled by `STRIP_SIDE_MULT` (2) — an artificial, non-physical boost so the weak side fringes read big. The cut is at the first null (value 0 on both sides) so it stays continuous; the sinc² shape, peak and zeros are all preserved, and there is no clipping. `NSTRIP` (12) keeps the fringes large; the `zλ/a` bracket spans peak-to-first-null to stay on-screen.
- Object is an irregular faceted crystal grain (`GRAIN` polygon in `blob()`), not a smooth lobe.
- Sample dots at pixel centers
- Catmull-Rom curve through dots shows aliased reconstruction
- Fade-in only when OS < ~2.5

### Sandbox
- **Controls:** Sliders `#sE`, `#sA`, `#sP`, `#sN`, `#sZ`
- **Behavior:** `sandbox()` computes real numbers, updates readout cards + status badge, re-renders `#cv2`
- **Same renderer:** Uses `drawScene()`
- **Layout:** Sliders + readouts stacked in the left column, canvas (max-height 78vh) on the right, so the whole instrument fits one viewport
- **Live geometry:** `sandbox()` builds an `s.geom` override (`Dx`, `Ls`, `Ld`, `NV`, `NSTRIP`, `wavePer`) from the sliders using monotone compressed mappings (powers/cbrt with clamps — not to scale, correct direction). Every slider moves its figure: z moves the detector plane and stretches L_s, E changes the drawn wavelength and L_s, P_D scales the detector pixels/L_D and shrinks L_s, N changes tick and strip-pixel counts and L_D. The walkthrough omits `geom` and gets the fixed default geometry.
- **Label halos:** `label()` and `vbar()` stroke a paper-coloured halo behind text so rays passing underneath never make labels illegible.

### Visual style
- **Aesthetic:** Graph-paper (CSS grid background)
- **Color semantics:**
  - Orange = lengths
  - Amber = counts
  - Green/red = two reciprocal ray pairs
  - Blue = wave/samples
- **Fonts:** Google Fonts (Space Grotesk, IBM Plex Sans, IBM Plex Mono) with system fallbacks

## Known issues & good first tasks

- **Canvas labels:** Hand-tuned for default geometry — verify no collisions at extreme sandbox OS values (clamped 0.6–8)
- **Mobile:** (<900px) stacks sticky canvas above steps at 46vh — could use polish
- **Possible enhancements:**
  - Scroll-scrubbed (not step-snapped) transitions
  - Bragg-geometry note (BCDI measures around a Bragg peak, not forward scattering)
  - URL-param presets for sandbox state
  - Dark mode

## Guidelines

- Review file before making changes
- Keep as single, self-contained HTML file
- No external dependencies