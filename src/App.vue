<template>
  <div class="site" :data-theme="theme">
    <!-- Header -->
    <header class="header">
      <div class="wrap head-inner">
        <a href="#" class="brand">{{ t(copy.brand) }}</a>
        <div class="controls">
          <!-- <select v-model="theme" class="select" aria-label="Theme">
            <option value="ivory">Ivory</option>
            <option value="noir">Noir</option>
            <option value="fresh">Fresh</option>
          </select> -->
          <button class="lang" @click="toggleLang">{{ lang === 'ja' ? 'EN' : '日本語' }}</button>
        </div>
      </div>
    </header>

    <!-- FULL-BLEED TOP BANNER -->
    <section
      class="hero-banner"
      :style="{ backgroundImage: `url(${hero})` }"
      role="img"
      :aria-label="t(heroAlt)"
    >
      <div class="wrap hero-content card glass">
        <h1 class="h1">{{ t(copy.tagline) }}</h1>
        <p class="lead">
          {{ lang === 'ja'
            ? '髪質・骨格・日々の過ごし方に合わせたデザインで、心地よい毎日へ。'
            : 'Designs shaped by your hair, features, and day-to-day — for effortless everyday.' }}
        </p>
        <a :href="telHref" class="call">📞 {{ phone }}</a>
        <ul class="features" role="list">
          <li v-for="f in copy.features" :key="f.en"><span class="ico">{{ f.icon }}</span>{{ lang === 'ja' ? f.ja : f.en }}</li>
        </ul>
      </div>
    </section>

    <main class="main">
      <!-- Services -->
      <section id="services" class="services wrap reveal">
        <header class="sec-head">
          <h2 class="h2">{{ t(copy.servicesTitle) }}</h2>
          <p class="muted">{{ t(copy.note) }}</p>
        </header>
        <div class="cats">
          <section class="cat" v-for="cat in categories" :key="t(cat.name)">
            <h3 class="h3 cat-title"><span class="pill">{{ t(cat.name) }}</span></h3>
            <ul class="menu" role="list">
              <li class="row" v-for="it in cat.items" :key="t(it.name)">
                <div class="name">
                  <span class="n">{{ t(it.name) }}</span>
                  <span v-if="t(it.desc)" class="d"> — {{ t(it.desc) }}</span>
                </div>
                <div class="dots" aria-hidden="true"></div>
                <div class="price">{{ yen(it.price) }}</div>
              </li>
            </ul>
          </section>
        </div>
      </section>

      <!-- GALLERY (Facebook-like collage for 1–5, masonry for 6–10) -->
      <section id="gallery" class="gallery reveal">
        <div class="wrap">
          <header class="sec-head">
            <h2 class="h2">{{ t(copy.galleryTitle) }}</h2>
          </header>

          <!-- Collage for 1–5 images -->
          <div
            v-if="gallery.length <= 5"
            class="post-collage"
            :data-count="gallery.length"
          >
            <figure
              v-for="(g, i) in gallery"
              :key="g.src"
              class="tile"
              @click="openLightbox(i)"
            >
              <img
                :src="g.src"
                :alt="t(g.alt)"
                loading="lazy"
                decoding="async"
                @error="(e) => onImgError(e)"
              />
              <figcaption class="cap">{{ t(g.alt) }}</figcaption>
            </figure>
          </div>

          <!-- Masonry for 6+ images (shows ALL, phone-friendly) -->
          <div v-else class="masonry">
            <figure
              v-for="(g, i) in gallery"
              :key="g.src"
              class="m-tile"
              @click="openLightbox(i)"
            >
              <img
                :src="g.src"
                :alt="t(g.alt)"
                loading="lazy"
                decoding="async"
                @error="(e) => onImgError(e)"
              />
              <figcaption class="m-cap">{{ t(g.alt) }}</figcaption>
            </figure>
          </div>
        </div>
      </section>

      <!-- Contact -->
      <section id="contact" class="contact reveal">
        <div class="wrap contact-inner card glass">
          <div class="contact-col">
            <h2 class="h2">{{ t(copy.contactTitle) }}</h2>
            <p class="muted">
              {{ lang === 'ja'
                ? 'ご相談だけでもお気軽にお電話ください。営業時間：10:00–19:00（最終受付18:30）／火曜定休'
                : 'Call us with any questions. Hours: 10:00–19:00 (last booking 18:30) / Closed Tue' }}
            </p>
          </div>
          <a :href="telHref" class="phone">📞 {{ phone }}</a>
        </div>
      </section>
    </main>

    <footer class="footer">
      <div class="wrap foot-inner">
        <small>© 2025 {{ t(copy.brand) }}</small>
        <nav class="foot-nav">
          <a href="#services">{{ t(copy.nav.services) }}</a>
          <a href="#gallery">{{ t(copy.nav.gallery) }}</a>
          <a href="#contact">{{ t(copy.nav.contact) }}</a>
        </nav>
      </div>
    </footer>

    <!-- Sticky call on mobile -->
    <a class="sticky-call" :href="telHref">📞 {{ phone }}</a>

    <!-- LIGHTBOX (no-crop full view) -->
    <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
      <button class="lb-close" @click="closeLightbox" aria-label="Close">×</button>
      <button class="lb-nav prev" @click.stop="prev" aria-label="Previous">‹</button>
      <figure class="lb-stage">
        <img :src="gallery[lightboxIndex].src" :alt="t(gallery[lightboxIndex].alt)" />
        <figcaption class="lb-cap">{{ t(gallery[lightboxIndex].alt) }}</figcaption>
      </figure>
      <button class="lb-nav next" @click.stop="next" aria-label="Next">›</button>

      <div class="lb-thumbs" role="tablist">
        <button
          v-for="(g, i) in gallery"
          :key="'thumb-' + i"
          class="thumb"
          :class="{ on: i === lightboxIndex }"
          @click.stop="go(i)"
          :aria-selected="i === lightboxIndex ? 'true' : 'false'"
        >
          <img :src="g.src" :alt="t(g.alt)" />
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

/* === Banner image === */
import heroImg from "@/assets/image12.jpg";
const hero = heroImg;
const heroAlt = { ja: "スタイリストがブローしている様子", en: "Stylist blow-drying a client's hair" };

/* === Local gallery images (image1…image10) === */
import img1 from "@/assets/image1.jpg";
import img2 from "@/assets/image2.jpg";
import img3 from "@/assets/image3.jpg";
import img4 from "@/assets/image4.jpg";
import img5 from "@/assets/image5.jpg";
import img6 from "@/assets/image6.jpg";
import img7 from "@/assets/image7.jpg";
import img8 from "@/assets/image8.jpg";
import img9 from "@/assets/image9.jpg";
import img10 from "@/assets/image10.jpg";

/* ===== Language ===== */
const lang = ref("ja"); // "ja" | "en"
const t = (obj) => obj[lang.value] ?? obj.en ?? "";
const toggleLang = () => (lang.value = lang.value === "ja" ? "en" : "ja");

/* ===== Theme ===== */
const theme = ref("fresh"); // "ivory" | "noir" | "fresh"

/* ===== Content ===== */
const copy = {
  brand: { ja: "Salon SEMHAL", en: "Salon SEMHAL" },
  tagline: { ja: "静けさと丁寧さ。あなたらしい美しさを。", en: "Quiet care, tailored to you." },
  nav: { services: { ja: "メニュー", en: "Services" }, gallery: { ja: "ギャラリー", en: "Gallery" }, contact: { ja: "お問い合わせ", en: "Contact" } },
  servicesTitle: { ja: "メニューと料金", en: "Services & Prices" },
  galleryTitle: { ja: "スタイルギャラリー", en: "Style Gallery" },
  contactTitle: { ja: "ご予約・お問い合わせ", en: "Contact" },
  note: { ja: "※ 表示価格は税込。長さ・薬剤量により追加料金の場合があります。", en: "Prices include tax. Extra charges may apply for long hair or product." },
  features: [
    { icon: "✨", ja: "丁寧なカウンセリング", en: "Thoughtful consultation" },
    { icon: "🌿", ja: "頭皮/髪にやさしい施術", en: "Scalp & hair friendly" },
    { icon: "🎯", ja: "似合わせデザイン", en: "Face-flattering design" }
  ]
};

const categories = [
  { name: { ja: "カット", en: "Cut" }, items: [
      { name: { ja: "一般カット", en: "Standard Cut" }, price: 4500, desc: { ja: "シャンプー・ブロー込", en: "Includes shampoo & blow-dry" } },
      { name: { ja: "学生カット", en: "Student Cut" }, price: 3800, desc: { ja: "中高大学生", en: "Middle/High/University" } },
      { name: { ja: "キッズカット", en: "Kids Cut" }, price: 3000, desc: { ja: "小学生以下", en: "Elementary & under" } }
    ]},
  { name: { ja: "カラー", en: "Color" }, items: [
      { name: { ja: "リタッチカラー", en: "Retouch Color" }, price: 5500, desc: { ja: "根元2cmまで", en: "Roots up to 2cm" } },
      { name: { ja: "フルカラー", en: "Full Color" }, price: 7000, desc: { ja: "ロング料金別", en: "Extra for long hair" } },
      { name: { ja: "デザインカラー", en: "Design Color" }, price: 9000, desc: { ja: "ハイライト等", en: "Highlights/Lowlights" } }
    ]},
  { name: { ja: "パーマ", en: "Perm" }, items: [
      { name: { ja: "ベーシックパーマ", en: "Basic Perm" }, price: 8000, desc: { ja: "カット別", en: "Cut not included" } },
      { name: { ja: "デジタルパーマ", en: "Digital Perm" }, price: 11000, desc: { ja: "カット別", en: "Cut not included" } }
    ]},
  { name: { ja: "ストレート", en: "Straight" }, items: [
      { name: { ja: "部分縮毛矯正", en: "Partial Straightening" }, price: 6000, desc: { ja: "前髪など", en: "Fringe/partial" } },
      { name: { ja: "全体縮毛矯正", en: "Full Straightening" }, price: 13000, desc: { ja: "カット別", en: "Cut not included" } }
    ]},
  { name: { ja: "トリートメント", en: "Treatment" }, items: [
      { name: { ja: "クイックトリートメント", en: "Quick Treatment" }, price: 2500, desc: { ja: "お手軽ケア", en: "Quick care" } },
      { name: { ja: "集中補修トリートメント", en: "Deep Repair Treatment" }, price: 4500, desc: { ja: "ダメージ毛に", en: "For damaged hair" } }
    ]},
  { name: { ja: "その他", en: "Options" }, items: [
      { name: { ja: "眉カット", en: "Brow Trim" }, price: 1000, desc: { ja: "", en: "" } },
      { name: { ja: "ヘッドスパ（15分）", en: "Head Spa (15min)" }, price: 3000, desc: { ja: "炭酸ケア", en: "Carbonated care" } },
      { name: { ja: "シャンプー＆ブロー", en: "Shampoo & Blow-dry" }, price: 2000, desc: { ja: "", en: "" } }
    ]}
];

/* Price formatter */
const yen = (v) => new Intl.NumberFormat("ja-JP", { style: "currency", currency: "JPY" }).format(v);

/* ===== GALLERY IMAGES (10) ===== */
const gallery = [
  { src: img1,  alt: { ja: "スタイル1",  en: "Style 1" } },
  { src: img2,  alt: { ja: "スタイル2",  en: "Style 2" } },
  { src: img3,  alt: { ja: "スタイル3",  en: "Style 3" } },
  { src: img4,  alt: { ja: "スタイル4",  en: "Style 4" } },
  { src: img5,  alt: { ja: "スタイル5",  en: "Style 5" } },
  { src: img6,  alt: { ja: "スタイル6",  en: "Style 6" } },
  { src: img7,  alt: { ja: "スタイル7",  en: "Style 7" } },
  { src: img8,  alt: { ja: "スタイル8",  en: "Style 8" } },
  { src: img9,  alt: { ja: "スタイル9",  en: "Style 9" } },
  { src: img10, alt: { ja: "スタイル10", en: "Style 10" } }
];

/* ===== Lightbox ===== */
const lightboxOpen = ref(false);
const lightboxIndex = ref(0);
const openLightbox = (idx) => {
  lightboxIndex.value = idx;
  lightboxOpen.value = true;
  document.addEventListener("keydown", onKey);
};
const closeLightbox = () => {
  lightboxOpen.value = false;
  document.removeEventListener("keydown", onKey);
};
const next = () => { lightboxIndex.value = (lightboxIndex.value + 1) % gallery.length; };
const prev = () => { lightboxIndex.value = (lightboxIndex.value - 1 + gallery.length) % gallery.length; };
const go   = (i) => { lightboxIndex.value = i; };
const onKey = (e) => {
  if (e.key === "Escape") closeLightbox();
  if (e.key === "ArrowRight") next();
  if (e.key === "ArrowLeft") prev();
};

/* Graceful placeholder if any image fails */
const onImgError = (ev) => {
  ev.target.src =
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 900 600'><rect width='900' height='600' fill='%23f2f4f8'/><text x='50%25' y='50%25' text-anchor='middle' dominant-baseline='middle' font-family='Arial' font-size='24' fill='%2390a3b3'>Photo not available</text></svg>";
};

/* Phone */
const phone = "09018973275";
const telHref = computed(() => `tel:${phone}`);
</script>

<style scoped>
/* ===== Base & Theme ===== */
*{box-sizing:border-box}
html,body,:host{margin:0}
img{max-width:100%; display:block}
a{text-decoration:none; color:inherit}

.site{ --bg:#faf7f2; --ink:#171717; --muted:#6b6b6b; --line:#e9e2d9; --card:#ffffff; --accent:#7a5cfa; --accent2:#9aa6ff; }
.site[data-theme="noir"]{ --bg:#0e0e10; --ink:#f5f5f7; --muted:#a0a0a8; --line:#232329; --card:#141417; --accent:#8b7bff; --accent2:#33c3ff; }
.site[data-theme="fresh"]{ --bg:#f6fbf9; --ink:#0f1b15; --muted:#5f7a6d; --line:#d9ebe3; --card:#ffffff; --accent:#10b981; --accent2:#5eead4; }
.site{
  background: radial-gradient(900px 420px at -10% -10%, color-mix(in oklab, var(--accent) 18%, transparent), transparent 60%),
              radial-gradient(700px 340px at 110% -10%, color-mix(in oklab, var(--accent2) 16%, transparent), transparent 60%),
              var(--bg);
  color: var(--ink);
  font-family: ui-sans-serif, system-ui, -apple-system, "Noto Sans JP", Roboto, Arial;
  line-height: 1.6;
}
.wrap{ max-width: 1100px; margin: 0 auto; padding: 0 20px; }
.card{ background: var(--card); border: 1px solid var(--line); border-radius: 18px; box-shadow: 0 14px 38px rgba(0,0,0,.08); }
.glass{ backdrop-filter: blur(8px); }

/* ===== Header ===== */
.header{ position: sticky; top:0; z-index:20; backdrop-filter: blur(8px); background: color-mix(in oklab, var(--bg) 84%, transparent); border-bottom:1px solid var(--line) }
.head-inner{ display:flex; align-items:center; justify-content:space-between; gap:12px; padding:10px 0 }
.brand{
  font-size: 20px; font-weight: 800; letter-spacing: .6px;
  background: linear-gradient(90deg, var(--accent), var(--accent2));
  -webkit-background-clip:text; background-clip:text; color: transparent;
}
.controls{ display:flex; gap:8px; align-items:center }
.select,.lang{ border:1px solid var(--line); background:var(--card); color:var(--ink); border-radius:999px; padding:8px 12px; font-weight:600; cursor:pointer }

/* ===== Top Banner ===== */
.hero-banner{
  position: relative;
  min-height: 62vh;
  background-size: cover;
  background-position: center 40%;
  background-repeat: no-repeat;
  display: grid;
  place-items: end start;
  border-bottom: 1px solid var(--line);
}
.hero-banner::before{
  content:""; position:absolute; inset:0;
  background:
    linear-gradient(180deg, rgba(0,0,0,.25) 0%, rgba(0,0,0,.35) 55%, rgba(0,0,0,.35) 100%),
    linear-gradient(90deg, rgba(0,0,0,.25) 0%, rgba(0,0,0,0) 40%);
  pointer-events:none;
}
.hero-content{
  position: relative;
  margin: 24px 20px 28px;
  max-width: 720px;
  color: #fff;
  background: rgba(0,0,0,.22);
  border: 1px solid rgba(255,255,255,.22);
}
.h1{ margin:0; font-size:40px; letter-spacing:.2px; line-height:1.2; color:#fff }
.lead{ margin:10px 0 16px; color: rgba(255,255,255,.9); font-size:16px }
.call{
  display:inline-block; font-weight:800; padding:10px 14px; border-radius:12px;
  background:linear-gradient(90deg, var(--accent), var(--accent2)); color:#fff; box-shadow:0 12px 30px rgba(0,0,0,.25);
}
.features{ display:flex; gap:10px; margin:14px 0 0; padding:0; list-style:none; flex-wrap:wrap }
.features li{ display:flex; gap:8px; align-items:center; background:rgba(255,255,255,.12); border:1px solid rgba(255,255,255,.22); border-radius:999px; padding:6px 10px; font-weight:700; font-size:13px; color:#fff }
.ico{ font-size:16px }

@media (max-width: 640px){
  .hero-banner{ min-height: 54vh; background-position: center 45%; }
  .h1{ font-size: 32px; }
}

/* ===== Sections ===== */
.main{ display:grid; gap:40px; padding: 24px 0 40px }
.sec-head{ display:flex; align-items:end; justify-content:space-between; gap:12px; margin-bottom:8px }
.h2{ margin:0; font-size:28px; letter-spacing:.2px; position:relative }
.h2::after{ content:""; display:block; width:56px; height:4px; border-radius:4px; background:linear-gradient(90deg, var(--accent), var(--accent2)); margin-top:6px }
.h3{ margin:18px 0 8px; font-size:18px; letter-spacing:.2px }
.muted{ color:var(--muted); font-size:14px }

/* ===== Services (dotted leaders) ===== */
.cats{ display:grid; gap:18px }
.cat-title .pill{ display:inline-block; background:linear-gradient(90deg, var(--accent), var(--accent2)); color:#fff; padding:6px 10px; border-radius:999px; font-size:12px; letter-spacing:.4px }
.menu{ list-style:none; padding:0; margin:0; display:grid; gap:10px }
.row{
  display:grid; grid-template-columns: 1fr auto auto; align-items:baseline; gap:10px;
  padding:8px 0; border-bottom:1px dashed color-mix(in oklab, var(--line), var(--ink) 10%);
}
.name{ white-space:nowrap; overflow:hidden; text-overflow:ellipsis }
.n{ font-weight:700 }
.d{ color:var(--muted) }
.dots{ height:1px; border-bottom:1px dotted color-mix(in oklab, var(--line), var(--ink) 15%); align-self:center; margin-top:-2px }
.price{ font-weight:900; white-space:nowrap }

/* ===== Gallery: Collage styles (1–5) ===== */
.gallery{ border-top:1px solid var(--line); padding-top:28px }

.post-collage{
  display:grid;
  gap: 10px;
}

/* Base tile look */
.post-collage .tile{
  position:relative; overflow:hidden; border-radius:16px;
  background: var(--card); border:1px solid var(--line);
  box-shadow: 0 12px 28px rgba(0,0,0,.08);
  cursor: zoom-in;
}

/* Images use cover for clean collage; lightbox shows full */
.post-collage .tile img{
  width:100%; height:100%; object-fit: cover; display:block;
}

/* Captions */
.post-collage .cap{
  position:absolute; left:10px; bottom:10px; background: rgba(0,0,0,.45); color:#fff;
  padding:4px 8px; font-size:12px; border-radius:8px; opacity:0; transform: translateY(4px);
  transition: opacity .18s ease, transform .18s ease;
}
.post-collage .tile:hover .cap{ opacity:1; transform:none }

/* 1 image */
.post-collage[data-count="1"]{
  grid-template-columns: 1fr;
  grid-auto-rows: 380px;
}
.post-collage[data-count="1"] .tile{ aspect-ratio: 16/9; }

/* 2 images (side by side) */
.post-collage[data-count="2"]{
  grid-template-columns: 1fr 1fr;
  grid-auto-rows: 320px;
}
.post-collage[data-count="2"] .tile{ aspect-ratio: 4/3; }

/* 3 images (big left, two stacked right) */
.post-collage[data-count="3"]{
  grid-template-columns: 2fr 1fr;
  grid-template-rows: 1fr 1fr;
  grid-template-areas:
    "a b"
    "a c";
  height: 480px;
}
.post-collage[data-count="3"] .tile:nth-child(1){ grid-area: a }
.post-collage[data-count="3"] .tile:nth-child(2){ grid-area: b }
.post-collage[data-count="3"] .tile:nth-child(3){ grid-area: c }

/* 4 images (2x2) */
.post-collage[data-count="4"]{
  grid-template-columns: 1fr 1fr;
  grid-auto-rows: 250px;
}
.post-collage[data-count="4"] .tile{ aspect-ratio: 4/3; }

/* 5 images (big left + 2 right on top, then 2 bottom) */
.post-collage[data-count="5"]{
  grid-template-columns: 2fr 1fr 1fr;
  grid-template-rows: 1fr 1fr;
  grid-template-areas:
    "a b c"
    "a d e";
  height: 520px;
}
.post-collage[data-count="5"] .tile:nth-child(1){ grid-area: a }
.post-collage[data-count="5"] .tile:nth-child(2){ grid-area: b }
.post-collage[data-count="5"] .tile:nth-child(3){ grid-area: c }
.post-collage[data-count="5"] .tile:nth-child(4){ grid-area: d }
.post-collage[data-count="5"] .tile:nth-child(5){ grid-area: e }

/* ===== Masonry (6–10 images) — all visible, great on phones ===== */
.masonry{
  column-count: 4;
  column-gap: 12px;
}
@media (max-width: 1000px){ .masonry{ column-count: 3 } }
@media (max-width: 700px){ .masonry{ column-count: 2 } }
@media (max-width: 460px){ .masonry{ column-count: 1 } }

.masonry .m-tile{
  break-inside: avoid;
  margin: 0 0 12px;
  border-radius: 16px;
  overflow: hidden;
  background: var(--card);
  border: 1px solid var(--line);
  box-shadow: 0 12px 28px rgba(0,0,0,.08);
  cursor: zoom-in;
  position: relative;
}
.masonry .m-tile img{
  width:100%; height:auto; display:block; /* no crop in masonry */
}
.masonry .m-cap{
  position:absolute; left:10px; bottom:10px;
  background: rgba(0,0,0,.45); color:#fff; padding:4px 8px; font-size:12px; border-radius:8px;
  opacity:0; transform: translateY(4px); transition: opacity .18s ease, transform .18s ease;
}
.masonry .m-tile:hover .m-cap{ opacity:1; transform:none }

/* ===== Contact ===== */
.contact .contact-inner{ display:flex; align-items:center; justify-content:space-between; gap:12px; padding:18px }
.phone{
  display:inline-block; margin-left:auto; font-size:22px; font-weight:900;
  padding:10px 14px; border-radius:12px; border:1px solid var(--line); background:var(--card);
}

/* ===== Footer & Sticky call ===== */
.footer{ border-top:1px solid var(--line) }
.foot-inner{ display:flex; align-items:center; justify-content:space-between; padding:14px 0; color:var(--muted) }
.foot-nav a{ margin-left:14px }
.sticky-call{
  display:none; position:sticky; bottom:12px; left:0; right:0; margin:0 auto; width:fit-content;
  background:linear-gradient(90deg, var(--accent), var(--accent2)); color:#fff; font-weight:900;
  padding:10px 16px; border-radius:999px; text-decoration:none; box-shadow:0 12px 26px rgba(0,0,0,.18);
}

/* ===== Lightbox ===== */
.lightbox{
  position: fixed; inset:0; background: rgba(14,14,16,.88);
  display:grid; grid-template-rows: 1fr auto; align-items:center; justify-items:center;
  z-index:1000; animation: lbIn .18s ease;
}
@keyframes lbIn { from{ opacity:0 } to{ opacity:1 } }
.lb-stage{
  position: relative; max-width: min(92vw, 1200px); max-height: 78vh; display:grid; place-items:center; margin-top: 6vh;
}
.lb-stage img{ max-width:100%; max-height:100%; object-fit: contain; border-radius: 12px; box-shadow: 0 30px 80px rgba(0,0,0,.35) }
.lb-cap{ color:#fff; text-align:center; margin-top:10px; font-size:14px; opacity:.9 }
.lb-close{
  position:absolute; top:18px; right:18px; border:none; width:40px; height:40px; border-radius:999px;
  background: rgba(255,255,255,.92); color:#111; font-size:22px; cursor:pointer; display:grid; place-items:center;
  box-shadow: 0 8px 20px rgba(0,0,0,.22); z-index:1001;
}
.lb-nav{
  position:absolute; top: 50%; transform: translateY(-50%);
  border:none; width:44px; height:44px; border-radius:999px;
  background: rgba(255,255,255,.92); color:#111; font-size:22px; cursor:pointer; display:grid; place-items:center;
  box-shadow: 0 8px 20px rgba(0,0,0,.22);
}
.lb-nav.prev{ left: 18px }
.lb-nav.next{ right: 18px }
.lb-thumbs{
  width: 100%; max-width: 1100px; display:flex; gap:8px; overflow-x:auto; padding: 10px 16px 18px; justify-content:center;
}
.thumb{ border:none; padding:0; background:transparent; border-radius:10px; overflow:hidden; opacity:.7; cursor:pointer }
.thumb.on{ outline: 3px solid #fff; opacity: 1 }
.thumb img{ height: 56px; width: auto; display:block }

/* ===== Reveal animation ===== */
@keyframes fadeUp{ from{opacity:0; transform: translateY(8px)} to{opacity:1; transform:none} }
.reveal{ animation: fadeUp .5s ease both }
.reveal:nth-of-type(2){ animation-delay: .06s }
.reveal:nth-of-type(3){ animation-delay: .12s }

/* ===== Responsive tweaks for collage on phones ===== */
@media (max-width: 760px){
  .post-collage[data-count="3"]{
    grid-template-columns: 1fr 1fr;
    grid-template-rows: auto auto;
    grid-template-areas:
      "a a"
      "b c";
    height: auto;
  }
  .post-collage[data-count="5"]{
    grid-template-columns: 1fr 1fr;
    grid-template-rows: auto auto auto;
    grid-template-areas:
      "a a"
      "b c"
      "d e";
    height: auto;
  }
  .post-collage[data-count="1"] .tile{ aspect-ratio: 4/3; }
  .post-collage[data-count="2"]{ grid-auto-rows: 200px }
}
</style>
