# Project Guidelines — Centrum Zdraví Landing Page

> Instrukce pro sebe: čti tohle na začátku každé nové session nebo před tím, než začneš cokoli měnit na layoutu.

---

## 1. Struktura HTML souboru

Celá stránka žije v jednom souboru `index.html`. Pořadí je:

```
<head>
  Google Fonts (Cormorant Garamond + Jost)
  <style>
    :root { CSS proměnné (barvy) }
    html / body
    nav
    hero
    about
    services
    process
    cta-section
    contact-section
    footer
    responsive breakpointy (media queries úplně dole)
  </style>
</head>

<body>
  <nav>
  <section class="hero" id="hero">
  <section class="about" id="o-nas">
  <section class="services" id="sluzby">
  <section class="process" id="postup">
  <section class="cta-section" id="rezervace">
  <section class="contact-section" id="formular">
  <footer id="kontakt">
  <script> (formulář + fade-up animace + gradient bg)
</body>
```

**Pravidla pojmenování:**
- Sekce mají `id` česky (pro kotvy v navigaci: `#o-nas`, `#sluzby`, `#postup`, `#rezervace`)
- CSS třídy jsou anglicky (`hero`, `about`, `services`, `wave-divider`...)
- Všechny SVG ikony jsou inline v HTML, žádné externí soubory

**Brandové barvy jsou CSS proměnné v `:root`:**
```css
--sage:        #B2BDA3
--sand:        #EBDFCE
--pearl:       #F4E5D9
--peach:       #F8D3BB
--stone:       #978671
--forest:      #3D5230
--forest-dark: #2A3B22
--cream:       #F5F1EB
```

---

## 2. Pravidla pro full-width layout (od kraje ke kraji bez mezer)

### Zásadní pravidlo č. 1 — overflow-x na html I body

**Vždy hned na začátku nastav obojí:**

```css
html  { scroll-behavior: smooth; overflow-x: hidden; }
body  { overflow-x: hidden; }
```

Jen `body` nestačí — na iOS Safari se stránka přesto dá scrollovat do strany.

---

### Zásadní pravidlo č. 2 — sekce mají padding, full-bleed prvky musí z něj vyskočit

Sekce mají horizontální padding kvůli čitelnosti textu:
```css
.about    { padding: 7rem 2rem 8rem; }
.services { padding: 8rem 2rem 0; }
.process  { padding: 8rem 2rem 0; }
```

Jenže všechno uvnitř sekce (včetně vlnek) se tím zúží o `2rem` na každé straně.

**Každý prvek, který má sahat od kraje ke kraji, musí použít breakout:**

```css
.wave-divider {
  position: relative;
  width: 100vw;
  left: 50%;
  transform: translateX(-50%);
}
```

Jak to funguje:
- `width: 100vw` → vždy plná šířka viewportu, bez ohledu na parent
- `left: 50%` → posune levý okraj prvku na střed parenta
- `transform: translateX(-50%)` → posune prvek zpátky o polovinu své šířky (= 50vw)
- Výsledek: levý okraj prvku je na 0, pravý na 100vw — vždy edge to edge

Toto funguje univerzálně — pro sekce bez paddingu je výsledek nulový posun, pro sekce s paddingem se přesně vyrovná.

**Totéž pravidlo platí pro jakýkoli jiný full-bleed prvek** (background pruhy, obrázky na celou šířku, barevné pásy atd.)

---

### Zásadní pravidlo č. 3 — absolutně pozicované prvky uvnitř sekcí

Sekce s `position: relative` a absolutně pozicovaným prvkem (např. vlnka nahoře/dole):

```css
.hero { position: relative; overflow: hidden; }
```

```html
<div class="wave-divider" style="position: absolute; bottom: -1px;">
```

Inline `position: absolute` přepíše `position: relative` z CSS třídy `.wave-divider`. Transformace `left: 50%; transform: translateX(-50%)` z CSS třídy stále platí a funguje správně i pro absolutně pozicované prvky — **nemusíš přidávat `left:0;right:0`**.

---

## 3. Jak jsou udělané SVG vlnky mezi sekcemi

### Základní šablona vlnky

```html
<div class="wave-divider">
  <svg viewBox="0 0 1440 80" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg">
    <path d="M-50,40 C360,80 1080,0 1490,40 L1490,80 L-50,80 Z" fill="#BARVA_DALŠÍ_SEKCE"/>
  </svg>
</div>
```

**Klíčové detaily:**

| Vlastnost | Hodnota | Proč |
|-----------|---------|------|
| `preserveAspectRatio="none"` | povinné | SVG se roztáhne do rozměrů containeru bez zachování poměru stran |
| `viewBox="0 0 1440 80"` | základ | 1440 = typická desktop šířka, 80 = výška vlnky |
| Path začíná na `M-50` místo `M0` | nutné | Pokryje levý okraj i při mírném přesahu zaokrouhlení |
| Path končí na `1490` místo `1440` | nutné | Pokryje pravý okraj stejně |
| `fill` = barva NÁSLEDUJÍCÍ sekce | logika | Vlnka vizuálně patří ke stránce pod ní |

### S-křivky (přechody mezi sekcemi)

Používáme dvě varianty cesty pro S-tvar:

```
Varianta A (kopec vlevo, dolina vpravo):
M-50,40 C360,80 1080,0 1490,40 L1490,80 L-50,80 Z

Varianta B (dolina vlevo, kopec vpravo):
M-50,40 C360,0 1080,80 1490,40 L1490,80 L-50,80 Z
```

Střídej je pro hezký rytmus:
- Hero → About: Varianta A
- About → Services: Varianta B
- Services → Process: Varianta A
- Process → CTA: Varianta B

### Pozicování vlnek

**Vlnka na spodku sekce (absolutní):**
```html
<div class="wave-divider" style="position:absolute; bottom:-1px;">
```
`bottom: -1px` zabraňuje 1px tmavé čáře mezi vlnkou a další sekcí.

**Vlnka jako součást flow (relativní):**
```html
<div class="wave-divider" style="margin-top: 5rem;">
```

---

## 4. Co příště udělat jinak od začátku

### Layout setup — udělej toto jako první, ještě před psaním sekcí:

```css
/* 1. Zabránění horizontálnímu scrollu — OBOJE hned na začátku */
html { scroll-behavior: smooth; overflow-x: hidden; }
body { overflow-x: hidden; }

/* 2. Full-bleed breakout třída — definuj ji hned */
.full-bleed {
  position: relative;
  width: 100vw;
  left: 50%;
  transform: translateX(-50%);
}

/* 3. Uložit padding jako proměnnou */
:root {
  --section-padding-x: 2rem;
}
.sekce {
  padding: 8rem var(--section-padding-x) 0;
}
```

### Vlnky — nepřidávej je postupně, definuj systém:

Místo přidávání `calc(100% + 200px)` / `margin-left: -100px` (které nefunguje na mobilu) — rovnou použij `100vw + translateX` breakout a rozšířené path koordináty (`-50` až `1490`).

### Testuj na mobilu od začátku:

Po každé větší změně layoutu spusť screenshot ve viewportu `390 × 844` (iPhone 14). Horizontální overflow se jinak odhalí až na reálném zařízení.

```python
page.set_viewport_size({'width': 390, 'height': 844})
```

### Wrapper pattern pro obsah sekcí:

Místo paddingu přímo na sekci použij inner wrapper — sekce pak může být vždy 100% wide a full-bleed prvky uvnitř nepotřebují breakout:

```html
<section class="services">           <!-- padding: 0, width: 100% -->
  <div class="section-inner">        <!-- padding: 8rem 2rem, max-width: 1200px, margin: auto -->
    ... obsah ...
  </div>
  <div class="wave-divider">         <!-- přirozeně 100% wide, bez breakout -->
  </div>
</section>
```

Toto je správný pattern pro produkční projekty — ušetří všechny problémy s full-bleed prvky uvnitř paddovaných sekcí.
