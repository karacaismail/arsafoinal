# Radius Audit · arsam-pitch.html

**Audit tarihi:** 2026-04-30
**Kapsam:** Tek dosya — `arsam-pitch.html` (HTML + CSS + inline styles + JS).
**Yöntem:** Sadece okuma. Değişiklik yapılmadı, hiçbir mevcut davranış etkilenmedi.

---

## 1) Mevcut radius token / değişken sistemi

**Durum: yok.**

`:root` içinde tanımlı CSS custom property'ler:

| Kategori | Değişkenler |
|---|---|
| Color | `--bg-1..4`, `--soil-deep`, `--grass`, `--grass-deep`, `--gold`, `--warn`, `--t1..t4`, `--line-1..3` |
| Glass | `--glass-blur`, `--glass-blur-strong`, `--glass-bg-1..3`, `--glass-edge`, `--glass-edge-hi`, `--glass-spec`, `--glass-shade` |
| Motion | `--ease` |
| **Radius** | **— hiçbir token yok —** |

Tailwind utility class'ı `rounded-*` da hiçbir yerde kullanılmıyor (config'de utility yüklü ama tüketici yok).

---

## 2) Frekans dağılımı

Toplam **16 `border-radius` deklarasyonu**, **8 farklı değer**:

| Değer | Frekans | Kullanım yeri (özet) |
|---|---|---|
| `.5rem` | **5×** | `.btn`, `.header-cta`, `.brand-logo` (32×32), `.header-menu-toggle`, `.pulse-soft::after` |
| `.75rem` | **3×** | `.chart-box`, "Fark · Tasarruf" banner (inline), "Şeffaflık" container (inline) |
| `9999px` | **3×** | `.pill`, `.cta-fab`, n/a |
| `50%` | **2×** | `.nav-dot` (10×10 circle), `.clock-mini .dot` (6×6 circle) |
| `1rem` | **1×** | `.card` (TEMEL CONTAINER) |
| `inherit` | **1×** | `.card::before` (concentric — doğru kullanım ✓) |
| `3px` | **1×** | `::-webkit-scrollbar-thumb` (sistem) |
| `1px` | **1×** | `.header-menu-toggle span` (hamburger çizgi micro-detay) |
| `0 .5rem .5rem 0` | **1×** | Şeffaflık altı blockquote (asimetrik) |

---

## 3) Aykırı (one-off) değerler

| Değer | Yer | Yorum |
|---|---|---|
| `3px` | scrollbar-thumb | Sistem component, sadece WebKit. Felsefenin DIŞINDA kalsın. |
| `1px` | hamburger çizgisi | "Çizgi yuvarlatma", radius felsefesi değil. Konu DIŞI. |
| `0 .5rem .5rem 0` | Şeffaflık blockquote | Sol kenar düz (sol border-left:2px gold için), sağ köşeler `.5rem`. Stilistik bilinçli karar. |

Yarım/ondalıklı (`12.5px` gibi) tek seferlik anomali **yok**.

---

## 4) Component → radius eşleşmesi (semantic katman)

### Surface tier (genel cam yüzeyler)
- **`.card`** → `1rem` · **temel container** · padding 1.75-2.5rem · arkasında `::before` `border-radius:inherit` ile eş merkezli highlight var ✓
- **Banner inline** (Bootstrap "Fark · Tasarruf" + Pazar 2027 ciro banner) → `.75rem` · padding 2.5rem 1.5rem
- **Şeffaflık container (s07)** → `.75rem` · padding 2.5rem · border-left 3px accent
- **`.chart-box`** → `.75rem` · padding 1rem · contain ECharts canvas

### Control tier (etkileşim öğeleri)
- **`.btn` / `.btn-primary`** → `.5rem` · min-height 56px · padding 1rem 1.75rem
- **`.header-cta`** → `.5rem` · padding .55rem .8rem
- **`.brand-logo`** → `.5rem` · 32×32px (square shape)
- **`.header-menu-toggle`** → `.5rem` · 40×40px

### Pill tier (tam yuvarlak)
- **`.pill`** → `9999px` (chip/badge) · padding .625rem 1.125rem
- **`.cta-fab`** → `9999px` (FAB circle) · 48-52×48-52px
- **`.cta-fab::before`** → `9999px` (concentric inner ring overlay)

### Decorative dot tier
- **`.nav-dot`** → `50%` · 10×10px (8×8 mobile)
- **`.clock-mini .dot`** → `50%` · 6×6px

### Asimetrik (özel)
- **Blockquote** → `0 .5rem .5rem 0` · sadece sağ köşeler

---

## 5) Nesting & concentricity gözlemi

| Outer | Outer radius | Inner element | Inner radius | Padding | Doğru concentric? |
|---|---|---|---|---|---|
| `.site-header` (no radius, fixed top) | — | `.brand-logo` (32×32) | `.5rem` | header padding .75rem | n/a |
| `.site-header` | — | `.header-cta` | `.5rem` | — | n/a |
| `.site-header` | — | `.header-menu-toggle` | `.5rem` | — | n/a |
| `.card` | `1rem` | `.card::before` | `inherit` (1rem) | inset:0 | ✓ ÇİFT (overlay perfect) |
| `.card` | `1rem` | `<p>`, `<h3>` text | — | 1.75rem | n/a (text) |
| Bootstrap money-flow card (`.card.card-glass`) | `1rem` | `<h3>` (15.000.000 ₺) | — | 1.75rem 1.5rem | n/a |
| "Hedeflediğimiz pazar payı" `.card.card-warn` | `1rem` | inline rakam `<h3>` | — | 2.5rem | n/a |
| `.btn-primary` | `.5rem` | `.pulse-soft::after` (inset:-4px) | `.5rem` | — | **⚠️ KONSENTRİK DEĞİL** |
| Şeffaflık container (`.75rem`) | `.75rem` | (içinde paragraf — text only) | — | 2.5rem | n/a |

### ⚠️ Concentricity ihlali

**`.btn-primary` + `.pulse-soft::after`:**
- Buton: `border-radius:.5rem`, `inset` yok (orijinal sınır)
- Pulse halkası: `border-radius:.5rem`, `inset:-4px` (4px DIŞARI taşar)
- **Doğru konsentrik formül:** outer halka radius = button radius + 4px = `.75rem`. Bu hâliyle buton köşesi ile halka köşesi paralel değil — radius eşit ama halka 4px daha geniş kalıyor, eğri dışarıdan keskin görünüyor.
- `radiusInner(outer, padding)` formülünün TERSİ — burada `radiusOuter(inner, expansion)` lazım: `inner_radius + expansion`.
- Etki düşük (kullanıcı detayda fark eder), **ileride düzeltilmeli**.

**`.card::before`:**
- `border-radius:inherit` ✓ — kart radius'unu bire bir alıyor, perfect overlay.
- ::before inset:0 olduğu için DIŞ-İÇ farkı yok (highlight overlay tam üstüne yapışık).
- Eğer kart padding'i içinde nested bir kart konursa o concentric matematiği yapılmamış olur (mevcut nesting'de bu yaşanmıyor).

---

## 6) Squircle / corner-shape / clip-path durumu

`grep` sonucu: **yok**.
- `corner-shape: squircle` — kullanılmıyor
- `clip-path: path(...)` veya superellipse SVG mask — yok
- `figma-squircle` veya benzeri lib — yüklü değil

Tüm köşeler **CSS `border-radius`'un dairesel arc** semantiğinde — Apple/iOS squircle'ın G2-continuity tabaklı eğrisi YOK.

---

## 7) Token gerektirecek doğal sınıflama (öneri taslağı, FAZ 1 için)

Mevcut değerlerin semantik kümeleri:

| Anlamsal isim | Mevcut değer | Component tipi | Frekans |
|---|---|---|---|
| **shell** (top-level cihaz benzeri) | `1rem` (16px) | `.card` (en büyük cam yüzey) | 1 |
| **surface** (mid container) | `.75rem` (12px) | banner, blockquote, chart-box | 3 |
| **control** (interactive) | `.5rem` (8px) | btn, brand-logo, header-cta, hamburger | 5 |
| **chip** (pill, tam yuvarlak) | `9999px` | pill, cta-fab | 3 |
| **dot** (decorative tam yuvarlak) | `50%` | nav-dot, clock-mini.dot | 2 |
| **micro** (sub-pixel) | `1px`, `3px` | scrollbar, hamburger çizgi | 2 |

→ 4 ana semantik tier (shell · surface · control · chip), + 2 utility (dot, micro). FAZ 1'de bu sınıflandırma `--radius-shell, --radius-surface, --radius-control, --radius-chip` olarak token'laşabilir.

Apple-style root mastar yaklaşımı: `--radius-root: 1rem` (shell), türevler `root×0.75 (.75rem) → surface`, `root×0.5 (.5rem) → control`, `root×0.25 (.25rem) → micro chip`. **Mevcut değerler tam bu serinin üzerinde duruyor!** Sıfırdan inşa gerekmiyor — yeniden adlandırma yeterli.

---

## 8) Risk haritası (FAZ 4 migration için)

| Component | Visual sensitivity | Migration risk | Öneri |
|---|---|---|---|
| `.cta-fab` (FAB) | Yüksek (göz hizasında) | Düşük (zaten 9999px tam circle, squircle yok) | İlk önce — squircle gerekmiyor, sadece token |
| `.pill` (teknopark chips) | Orta | Düşük | İkinci |
| `.btn-primary` | Yüksek | Orta (pulse-soft halkası ayar gerekir) | Üçüncü, halka concentric fix ile birlikte |
| `.brand-logo, .header-cta, .header-menu-toggle` | Orta | Düşük | Token ekle, davranış aynı |
| `.chart-box` | Düşük | Düşük | Token ekle |
| Banner inline (.75rem) | Yüksek (büyük yüzey) | Orta — inline style'lar manuel migrate | Sona bırak |
| `.card` | **EN YÜKSEK** | **YÜKSEK** (sahnenin yarısı) | **EN SON, feature flag arkasında** |

---

## 9) FAZ 1'e geçiş kontrol listesi

- [x] Tüm radius değerleri envantere geçti — 16 deklarasyon, 8 değer
- [x] Mevcut token sistemi tarandı — radius token YOK, kategori isimlendirmesi açık
- [x] Aykırı değerler raporlandı — sistem (3px), micro (1px), asimetrik blockquote
- [x] Concentricity ihlali tespit edildi — `.btn-primary + .pulse-soft::after` (düşük etki)
- [x] Squircle/corner-shape kullanımı — sıfır, yeni primitive ekleme alanı temiz
- [x] Migration risk haritası — `.card` en yüksek risk, FAB en düşük

**Sonuç:** FAZ 1 (token katmanı) için zemin uygun. Mevcut değerler doğal bir 4-tier semantik hiyerarşi oluşturuyor (shell .75-1rem, control .5rem, chip 9999px, dot 50%). Sıfırdan radius felsefesi tasarlamak gerekmiyor — sadece **isimlendirme + opt-in token ile sarmalama**.

**Onay bekleniyor:** FAZ 1 token katmanı (yeni `:root` değişkenleri + `radiusInner()` helper) implement edilecek mi?

---

# FAZ 1 — TOKEN KATMANI (uygulandı)

## Test planı (implementasyondan ÖNCE)

### Test-1.1 · Token export'ları — değer doğruluğu

`getComputedStyle(document.documentElement).getPropertyValue(...)` ile beklenen değerler:

| Token | Beklenen |
|---|---|
| `--radius-root` | `1rem` |
| `--radius-shell` | `1rem` (root × 1.0) |
| `--radius-surface` | `.75rem` (root × 0.75 — calc) |
| `--radius-control` | `.5rem` (root × 0.5 — calc) |
| `--radius-micro` | `.25rem` (root × 0.25 — calc) |
| `--radius-chip` | `9999px` (özel) |
| `--radius-dot` | `50%` (özel) |
| `--corner-smoothing` | `0.6` |

### Test-1.2 · `radiusInner(outer, padding)` matematiği

| outer | padding | beklenen | açıklama |
|---|---|---|---|
| 16 | 8 | 8 | normal nesting |
| 16 | 24 | 0 | padding > outer → 0'a kelepçelenmeli |
| 0 | 0 | 0 | edge case — sıfır |
| 1 | 0 | 1 | padding sıfır |
| 12 | 12 | 0 | padding == outer → 0 |

### Test-1.3 · Mevcut davranış korundu (regresyon)

- `.card` → `1rem` (CSS rule değişmedi)
- `.btn`, `.header-cta`, `.brand-logo` → `.5rem`
- `.pill`, `.cta-fab` → `9999px`
- `.nav-dot`, `.clock-mini .dot` → `50%`
- ECharts render, framer/IntersectionObserver, scroll behavior — hepsi aynı

### Test-1.4 · Token kullanım korumacı

- Yeni token'lar **hiçbir mevcut component'a otomatik uygulanmadı** — sadece tanım. Migration FAZ 4'te component-by-component opt-in.

## Spec — eklenen artefaktlar

### CSS token'lar (`:root` içine paralel olarak)

```css
--radius-root:    1rem;
--radius-shell:   var(--radius-root);
--radius-surface: calc(var(--radius-root) * 0.75);
--radius-control: calc(var(--radius-root) * 0.5);
--radius-micro:   calc(var(--radius-root) * 0.25);
--radius-chip:    9999px;
--radius-dot:     50%;
--corner-smoothing: 0.6;
--radius-inner-shell-1_75: max(0px, calc(var(--radius-shell) - 1.75rem));
--radius-inner-shell-2_25: max(0px, calc(var(--radius-shell) - 2.25rem));
--radius-inner-shell-2_5:  max(0px, calc(var(--radius-shell) - 2.5rem));
```

### JS helper

```js
window.arsam = window.arsam || {};
window.arsam.radiusInner = (outer, padding) => {
  if (typeof outer !== 'number' || typeof padding !== 'number')
    throw new TypeError('radiusInner: outer and padding must be numbers');
  return Math.max(outer - padding, 0);
};
```

Birim sorumluluğu caller'da (her ikisi de `px` veya her ikisi de `rem`).

### Smoke test

`console.assert` ile 5 radiusInner test + 3 token okuma test — DevTools console'da görünür.

## Anayasa kontrolü

| Kural | Durum |
|---|---|
| Mevcut `border-radius` değerleri silinmeyecek/ezilmeyecek | ✓ |
| Paralel katman olarak ekleme | ✓ |
| Component bazında opt-in | ✓ (kimse migrate edilmedi) |
| Test-first | ✓ |
| Visual regression flag arkasında | ✓ (visual değişiklik yok zaten) |

## Eklenen / değişen dosyalar

| Dosya | Değişiklik | Etki |
|---|---|---|
| `arsam-pitch.html` | `:root` içine 11 yeni CSS değişkeni; JS başına 1 helper + 8 console.assert | 0 mevcut rule kaldırıldı/değiştirildi |
| `docs/radius-audit.md` | Bu FAZ 1 bölümü | dokümantasyon |

## FAZ 2'ye geçiş checklist

- [x] Token katmanı tanımlandı, mevcut değerlerle birebir eşleşiyor
- [x] `radiusInner()` exposed (window.arsam.radiusInner)
- [x] Smoke test bloğu yerleştirildi
- [x] Mevcut görsel davranış değişmedi (deploy doğrulamasıyla)
- [ ] FAZ 2 ön-koşul: `corner-shape: squircle` browser desteği taraması (`@supports`)
- [ ] FAZ 2 ön-koşul: SVG mask superellipse fallback path planı

**Onay bekleniyor:** FAZ 2 (Squircle primitive — `.squircle` class + opt-in `corner-shape` veya SVG mask fallback) başlatılsın mı?
