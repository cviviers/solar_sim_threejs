# Solar‑System Simulation (Three.js)

This project is a **browser‑based, real‑time model of our Solar System**. It couples

- **Analytic celestial mechanics** (Keplerian orbits in the J2000 reference frame) with
- **GPU rendering** (three.js r162) and
- **Interactive controls** (orbit, zoom, literal/artistic size toggle)

so you can pan through a physically accurate, textured scene that updates every frame from the actual system clock.

---

## 1  Orbital theory

### 1.1  Keplerian elements at epoch J2000.0

For each planet the simulation stores the traditional six orbital elements, **heliocentric and ecliptic, equinox J2000.0 (TT = 2000‑01‑01 12:00)**.

| Element                     | Symbol | Meaning                                  | Source\*            |
| --------------------------- | ------ | ---------------------------------------- | ------------------- |
| Semi‑major axis             | *a*    | Mean distance Sun–planet (AU)            | VSOP87 (Meeus 1998) |
| Eccentricity                | *e*    | Shape of ellipse                         | VSOP87              |
| Inclination                 | *i*    | Tilt vs. ecliptic (°)                    | VSOP87              |
| Longitude of ascending node | Ω      | ℓ where orbit crosses ecliptic northward | VSOP87              |
| Argument of periapsis       | ω      | Angle periapsis–ascending node           | VSOP87              |
| Mean longitude              | *L*    | Ω + ω + *M₀* (°)                         | VSOP87              |

> \* All numeric values are from the low‑precision VSOP87 tables reproduced in Jean Meeus, ***Astronomical Algorithms***, 2nd Ed., Chapters 31–32.

From these we derive

- **Mean anomaly at J2000**:    *M₀ = L − Ω − ω*
- **Mean motion** *(rad s⁻¹)*: $ n = \sqrt{\mu/a^{3}} \quad≈\; 2π \big/\big(\sqrt{a^{3}}·365.256898·86 400\big)$ with $\mu = GM_{☉}$.  We treat $\mu$ as 1 AU³  day⁻² so the constant drops out.

### 1.2  Time‑stepping

Every frame we compute the true anomaly $ν(t)$ in three steps:

1. **Mean anomaly** $M(t) = M₀ + n·(t−t₀)$
2. **Eccentric anomaly E** via Newton–Raphson $E−e \sin E = M$
3. **Coordinates in orbital plane** $x' = a( \cos E − e), \; y' = a\sqrt{1−e^{2}}\sin E$
4. **Rotate** to the ecliptic using the ZXZ sequence: $\vec r = R_{z}(Ω)\,R_{x}(i)\,R_{z}(ω)\,[x',y',0]^{T}$

The result is multiplied by `` so 1 world‑unit ≈ 0.02 AU.

### 1.3  Limitations of analytic 2‑body orbits

- No planetary perturbations (VSOP87 L,B,R sums) — errors grow to a few arc‑minutes over decades.
- Barycentric motion of the Sun ignored (Sun fixed at origin).
- Relativistic perihelion precession ignored (Mercury’s 43″ per century).

For an educational visualisation these effects are negligible; adding a barycentric correction is a future option.

---

## 2  Rendering choices

| Feature              | Artistic                                           | Literal (toggle)             |
| -------------------- | -------------------------------------------------- | ---------------------------- |
| **Planet radii**     | Cube‑root of real R⊕ (≈ √³ compression)            | True physical R ⭢ tiny dots  |
| **Solar radius**     | 12 units                                           | 0.23 units (R☉ ≈ 0.00465 AU) |
| **Brightness model** | Constant emissive Sun; planets = Lambert + ambient | Same                         |
| **Textures**         | NASA/three.js JPGs (1‑2 k)                         | Same                         |
| **Saturn’s ring**    | Textured `RingGeometry`                            | Same                         |

> **Why non‑literal sizes by default?**  At true scale Jupiter’s disc subtends 0.024 world‑units at 1 AU — essentially sub‑pixel when you frame its orbit. The artistic mode preserves visual interest while literal mode is a click away for scale demonstrations.

---

## 3  Running locally

1. Clone or download the repo.
2. Place textures under `` (see list below).
3. Serve via any static server:

```bash
python -m http.server 8000    # or
npx serve
```

Open [http://localhost:8000](http://localhost:8000).

> Loading from `file://` will fail because WebGL blocks cross‑origin texture reads. Always use an `http://` URL or GitHub Pages.

### Required texture files

```
lensflare0.png
starfield.jpg
mercurymap.jpg
venusmap.jpg
earthmap1k.jpg
mars_1k_color.jpg
jupitermap.jpg
saturnmap.jpg
saturnringcolor.jpg
uranusmap.jpg
neptunemap.jpg
plutomap1k.jpg
```

---

## 4  Code map (ES‑modules)

```
index.html          HTML container + <script type="module">
  · Kepler solver   (keplerE)
  · Orbit maths     (planetPos)
  · Scene setup     (WebGLRenderer, OrbitControls, lights)
  · Toggle handler  (#scaleBtn click)
textures/           All image assets (see list above)
```

---

## 5  Ideas for extension

- Replace 2‑body orbits with **n‑body integrator** (e.g. IAS15) for long‑term precision.
- Add **moons** (Galilean satellites need 4‑body solutions or pre‑tabulated ephemerides).
- Implement **lighting fall‑off** and HDR bloom.
- **Labels / HUD** that shows RA/Dec, distance, and phase angle.
- **Service Worker + **`` for installable PWA planetarium.

---

### License & credits

Textures © NASA/JPL; 

> Built with **three.js r162** · by Christiaan Viviers · 2025‑07‑19

