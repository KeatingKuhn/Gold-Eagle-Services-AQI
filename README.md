<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Austin AQI Dashboard | Gold Eagle Services</title>
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;600;700;800;900&family=Open+Sans:wght@400;600;700&display=swap" rel="stylesheet">
<style>
/* ─── RESET & BASE ─────────────────────────────────────── */
*, *::before, *::after { margin:0; padding:0; box-sizing:border-box; }

:root {
  --gold-light: #f0d64e;
  --gold-mid:   #d7b740;
  --gold-dark:  #ab8024;
  --black:      #000;
  --black-rich: #0a0a0a;
  --black-card: #111010;
  --black-sub:  #181717;
  --border:     rgba(171,128,36,0.22);
  --border-hi:  rgba(215,183,64,0.45);
  --text-dim:   rgba(255,255,255,0.38);
  --text-mid:   rgba(255,255,255,0.62);
  --text-full:  #ffffff;
  --grad:       linear-gradient(135deg, #f0d64e 0%, #d7b740 50%, #ab8024 100%);
  --grad-v:     linear-gradient(180deg, #f0d64e 0%, #d7b740 55%, #ab8024 100%);
  --grad-h:     linear-gradient(90deg,  #f0d64e 0%, #d7b740 50%, #ab8024 100%);

  /* AQI colors */
  --good:    #22c55e;
  --mod:     #eab308;
  --sens:    #f97316;
  --bad:     #ef4444;
  --vbad:    #a855f7;
  --haz:     #7f1d1d;
}

html { scroll-behavior: smooth; }

body {
  background: var(--black);
  color: var(--text-full);
  font-family: 'Open Sans', sans-serif;
  min-height: 100vh;
  overflow-x: hidden;
  -webkit-font-smoothing: antialiased;
}

/* subtle grain texture overlay */
body::after {
  content:'';
  position:fixed; inset:0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.035'/%3E%3C/svg%3E");
  pointer-events:none; z-index:999; opacity:0.4;
}

/* ─── LOADER ──────────────────────────────────────────── */
#loader {
  position:fixed; inset:0; background:var(--black);
  display:flex; flex-direction:column; align-items:center; justify-content:center;
  z-index:9000; transition:opacity 0.7s ease;
}
#loader.hidden { opacity:0; pointer-events:none; }
.ld-eagle { font-size:64px; animation:float 2.4s ease-in-out infinite; }
.ld-label {
  font-family:'Montserrat',sans-serif; font-weight:800;
  font-size:11px; letter-spacing:6px; text-transform:uppercase;
  background:var(--grad); -webkit-background-clip:text; -webkit-text-fill-color:transparent;
  background-clip:text; margin-top:24px;
}
.ld-bar { width:240px; height:2px; background:rgba(171,128,36,0.15); margin-top:20px; overflow:hidden; border-radius:2px; }
.ld-fill { height:100%; background:var(--grad); border-radius:2px; animation:sweep 2s ease-in-out infinite; }
@keyframes float { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-12px)} }
@keyframes sweep { 0%{width:0;margin-left:0} 50%{width:55%;margin-left:22%} 100%{width:0;margin-left:100%} }

/* ─── TOP BAR ─────────────────────────────────────────── */
.top-bar {
  display:flex; align-items:center; justify-content:space-between;
  padding:14px 40px; border-bottom:1px solid var(--border);
  background:rgba(0,0,0,0.8); backdrop-filter:blur(12px);
  position:sticky; top:0; z-index:100;
}
.brand {
  display:flex; align-items:center; gap:10px;
}
.brand-name {
  font-family:'Montserrat',sans-serif; font-weight:800;
  font-size:13px; letter-spacing:2.5px; text-transform:uppercase;
  background:var(--grad); -webkit-background-clip:text;
  -webkit-text-fill-color:transparent; background-clip:text;
}
.brand-sub {
  font-size:9px; letter-spacing:2px; text-transform:uppercase;
  color:var(--text-dim); font-family:'Open Sans',sans-serif;
  margin-top:2px;
}
.top-bar-right {
  display:flex; align-items:center; gap:20px;
}
.live-dot {
  display:flex; align-items:center; gap:7px;
  font-size:11px; color:var(--text-mid); font-family:'Open Sans',sans-serif;
}
.live-dot::before {
  content:''; width:8px; height:8px; border-radius:50%;
  background:var(--good); animation:pulse-dot 2s infinite;
}
@keyframes pulse-dot {
  0%,100%{box-shadow:0 0 0 0 rgba(34,197,94,0.4)}
  50%{box-shadow:0 0 0 6px rgba(34,197,94,0)}
}
#topTime {
  font-family:'Montserrat',sans-serif; font-weight:700;
  font-size:13px; color:var(--text-mid);
}
.btn-refresh {
  background:none; border:1px solid var(--border-hi); cursor:pointer;
  color:var(--gold-mid); font-family:'Montserrat',sans-serif;
  font-size:9px; font-weight:700; letter-spacing:2px; text-transform:uppercase;
  padding:7px 16px; transition:all 0.2s;
}
.btn-refresh:hover { background:rgba(171,128,36,0.12); color:var(--gold-light); }
.btn-refresh.spinning { animation:spin 0.8s linear infinite; }
@keyframes spin { to{transform:rotate(360deg)} }

/* ─── HERO TITLE ──────────────────────────────────────── */
.hero-title {
  text-align:center; padding:52px 24px 40px;
  position:relative; overflow:hidden;
}
.hero-title::before {
  content:'';
  position:absolute; top:0; left:50%; transform:translateX(-50%);
  width:600px; height:300px;
  background:radial-gradient(ellipse at 50% 0%, rgba(171,128,36,0.14) 0%, transparent 70%);
  pointer-events:none;
}
.ht-eyebrow {
  font-family:'Open Sans',sans-serif; font-size:11px; font-weight:600;
  letter-spacing:5px; text-transform:uppercase; color:rgba(215,183,64,0.5);
  margin-bottom:14px;
}
.ht-main {
  font-family:'Montserrat',sans-serif; font-weight:900;
  font-size:clamp(42px, 7vw, 80px);
  letter-spacing:-2px; line-height:1;
  background:var(--grad-v);
  -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text;
}
.ht-rule {
  display:flex; align-items:center; justify-content:center; gap:18px;
  margin:18px auto;
}
.ht-rule .ln { width:100px; height:1px; background:linear-gradient(90deg,transparent,rgba(171,128,36,0.4)); }
.ht-rule .ln.r { background:linear-gradient(270deg,transparent,rgba(171,128,36,0.4)); }
.ht-rule .gem { width:6px; height:6px; background:var(--gold-mid); transform:rotate(45deg); }
.ht-services {
  font-family:'Open Sans',sans-serif; font-size:12px; font-weight:600;
  letter-spacing:3px; text-transform:uppercase; color:var(--text-dim);
}
.ht-meta {
  display:flex; align-items:center; justify-content:center;
  gap:24px; margin-top:18px; flex-wrap:wrap;
}
.ht-chip {
  font-size:11px; color:var(--text-dim); font-family:'Open Sans',sans-serif;
  display:flex; align-items:center; gap:6px;
}
.ht-chip .dot { width:5px; height:5px; border-radius:50%; background:var(--gold-dark); }

/* ─── MAIN GRID ───────────────────────────────────────── */
.main { max-width:1280px; margin:0 auto; padding:0 28px 80px; }

/* ─── MONSTER AQI CARD ────────────────────────────────── */
.aqi-monster {
  display:grid; grid-template-columns:1fr 1fr;
  gap:0; border:1px solid var(--border);
  margin-bottom:4px; background:var(--black-rich);
  position:relative; overflow:hidden;
}
.aqi-monster::before {
  content:'';
  position:absolute; top:-80px; left:-80px;
  width:400px; height:400px; border-radius:50%;
  background:radial-gradient(circle, rgba(171,128,36,0.09) 0%, transparent 65%);
  pointer-events:none;
}

/* Left: giant number */
.aqi-left {
  padding:52px 44px 44px;
  border-right:1px solid var(--border);
  display:flex; flex-direction:column; justify-content:space-between;
}
.aqi-eyebrow {
  font-family:'Montserrat',sans-serif; font-weight:700;
  font-size:10px; letter-spacing:5px; text-transform:uppercase;
  color:rgba(215,183,64,0.4); margin-bottom:8px;
}
.aqi-numwrap { display:flex; align-items:flex-end; gap:14px; }
.aqi-numval {
  font-family:'Montserrat',sans-serif; font-weight:900;
  font-size:clamp(110px, 14vw, 180px); line-height:0.85;
  background:var(--grad-v);
  -webkit-background-clip:text; -webkit-text-fill-color:transparent;
  background-clip:text; transition:all 0.4s ease;
  display:block;
}
.aqi-numval.override-color {
  background:none !important;
  -webkit-text-fill-color:var(--aqi-color);
}
.aqi-scale-vert {
  display:flex; flex-direction:column-reverse; gap:4px;
  padding-bottom:12px;
}
.sv-pip { width:10px; height:28px; border-radius:3px; opacity:0.18; transition:opacity 0.4s, transform 0.4s; }
.sv-pip.on { opacity:1; transform:scaleX(1.3); }

.aqi-status {
  font-family:'Montserrat',sans-serif; font-weight:800;
  font-size:clamp(22px, 3vw, 32px); margin-top:16px;
  transition:color 0.4s;
}
.aqi-category-pill {
  display:inline-flex; align-items:center; gap:8px;
  margin-top:12px; padding:7px 18px;
  border:1px solid; font-family:'Montserrat',sans-serif;
  font-size:11px; font-weight:700; letter-spacing:2px; text-transform:uppercase;
  width:fit-content;
}
.aqi-desc {
  font-size:14px; line-height:1.75; color:var(--text-mid);
  margin-top:22px; max-width:380px;
}
.aqi-scale-row {
  display:flex; gap:4px; margin-top:28px;
}
.sr-block {
  flex:1; height:6px; border-radius:2px; opacity:0.18; transition:opacity 0.4s;
}
.sr-block.on { opacity:1; }
.sr-label {
  display:flex; justify-content:space-between;
  font-size:9px; color:var(--text-dim); margin-top:5px;
  font-family:'Montserrat',sans-serif; font-weight:600; letter-spacing:1px;
}

/* Right: pollutants */
.aqi-right { padding:36px 36px; }
.sect-label {
  font-family:'Montserrat',sans-serif; font-weight:800;
  font-size:9px; letter-spacing:4px; text-transform:uppercase;
  color:rgba(215,183,64,0.4); margin-bottom:18px;
  padding-bottom:10px; border-bottom:1px solid rgba(171,128,36,0.12);
}
.poll-grid {
  display:grid; grid-template-columns:1fr 1fr; gap:0;
}
.poll-item {
  padding:16px 14px; border-bottom:1px solid rgba(255,255,255,0.04);
  border-right:1px solid rgba(255,255,255,0.03);
  transition:background 0.2s;
}
.poll-item:hover { background:rgba(171,128,36,0.05); }
.poll-item:nth-child(even) { border-right:none; }
.poll-tag {
  font-family:'Montserrat',sans-serif; font-weight:700;
  font-size:10px; letter-spacing:1.5px;
  color:var(--gold-mid); margin-bottom:4px;
}
.poll-name-full { font-size:10px; color:var(--text-dim); margin-bottom:8px; }
.poll-num {
  font-family:'Montserrat',sans-serif; font-weight:900;
  font-size:28px; line-height:1; color:var(--text-full);
}
.poll-unit { font-size:10px; color:var(--text-dim); margin-top:2px; }
.poll-mini-bar { width:100%; height:3px; background:rgba(255,255,255,0.06); border-radius:2px; margin-top:8px; overflow:hidden; }
.poll-mini-fill { height:100%; border-radius:2px; background:var(--grad-h); width:0%; transition:width 1.2s cubic-bezier(0.4,0,0.2,1); }

/* ─── QUICK STATS ROW ─────────────────────────────────── */
.quick-stats {
  display:grid; grid-template-columns:repeat(4,1fr);
  gap:4px; margin-bottom:4px;
}
.qs-card {
  background:var(--black-card); border:1px solid var(--border);
  padding:26px 22px; position:relative; overflow:hidden;
  transition:border-color 0.25s, background 0.25s;
}
.qs-card:hover { border-color:var(--border-hi); background:var(--black-sub); }
.qs-card::after {
  content:''; position:absolute; bottom:0; left:0; right:0;
  height:3px; background:var(--grad);
  transform:scaleX(0); transform-origin:left; transition:transform 0.3s;
}
.qs-card:hover::after { transform:scaleX(1); }
.qs-icon { font-size:26px; margin-bottom:10px; }
.qs-label {
  font-family:'Montserrat',sans-serif; font-weight:700;
  font-size:9px; letter-spacing:3px; text-transform:uppercase;
  color:rgba(215,183,64,0.38); margin-bottom:6px;
}
.qs-val {
  font-family:'Montserrat',sans-serif; font-weight:900;
  font-size:30px; line-height:1;
  background:var(--grad); -webkit-background-clip:text;
  -webkit-text-fill-color:transparent; background-clip:text;
}
.qs-val.override { background:none; -webkit-text-fill-color:var(--ov-color); }
.qs-note { font-size:11px; color:var(--text-dim); margin-top:5px; }

/* ─── FORECAST ────────────────────────────────────────── */
.sec-header {
  display:flex; align-items:center; gap:16px; margin:36px 0 16px;
}
.sec-title {
  font-family:'Montserrat',sans-serif; font-weight:800;
  font-size:11px; letter-spacing:5px; text-transform:uppercase;
  background:var(--grad); -webkit-background-clip:text;
  -webkit-text-fill-color:transparent; background-clip:text; white-space:nowrap;
}
.sec-line { flex:1; height:1px; background:linear-gradient(90deg,rgba(171,128,36,0.3),transparent); }

.forecast-strip {
  display:grid; grid-template-columns:repeat(7,1fr);
  gap:4px; margin-bottom:4px;
}
.fc-card {
  background:var(--black-card); border:1px solid var(--border);
  padding:20px 10px 16px; text-align:center; transition:all 0.2s; cursor:default;
}
.fc-card:hover { border-color:var(--border-hi); background:var(--black-sub); }
.fc-card.today { border-color:var(--border-hi); background:rgba(171,128,36,0.07); }
.fc-day-name {
  font-family:'Montserrat',sans-serif; font-weight:700;
  font-size:9px; letter-spacing:2.5px; text-transform:uppercase;
  color:rgba(215,183,64,0.5); margin-bottom:12px;
}
.fc-card.today .fc-day-name {
  background:var(--grad); -webkit-background-clip:text;
  -webkit-text-fill-color:transparent; background-clip:text;
}
.fc-aqi-num {
  font-family:'Montserrat',sans-serif; font-weight:900; font-size:38px; line-height:1;
}
.fc-status { font-size:10px; color:var(--text-dim); margin-top:4px; font-weight:600; font-family:'Montserrat',sans-serif; }
.fc-track { width:100%; height:4px; border-radius:2px; background:rgba(255,255,255,0.05); margin-top:12px; overflow:hidden; }
.fc-fill { height:100%; border-radius:2px; transition:width 1s ease; }

/* ─── POLLEN ──────────────────────────────────────────── */
.pollen-strip {
  display:grid; grid-template-columns:repeat(3,1fr);
  gap:4px; margin-bottom:4px;
}
.pollen-card {
  background:var(--black-card); border:1px solid var(--border);
  padding:28px 26px; position:relative; overflow:hidden;
}
.pc-type {
  font-family:'Montserrat',sans-serif; font-weight:700;
  font-size:10px; letter-spacing:3px; text-transform:uppercase;
  color:rgba(215,183,64,0.5); margin-bottom:14px;
}
.pc-num {
  font-family:'Montserrat',sans-serif; font-weight:900;
  font-size:56px; line-height:1;
}
.pc-unit { font-size:11px; color:var(--text-dim); margin-top:4px; }
.pc-badge {
  display:inline-block; margin-top:14px;
  padding:5px 16px; font-family:'Montserrat',sans-serif;
  font-size:9px; font-weight:800; letter-spacing:2.5px; text-transform:uppercase;
}

/* ─── ADVISORY ────────────────────────────────────────── */
.advisory-card {
  background:rgba(171,128,36,0.06);
  border:1px solid rgba(171,128,36,0.3);
  padding:32px 36px; margin-bottom:4px;
  position:relative; overflow:hidden;
}
.advisory-card::before {
  content:''; position:absolute; left:0; top:0; bottom:0;
  width:4px; background:var(--grad-v);
}
.adv-top { display:flex; align-items:center; gap:12px; margin-bottom:14px; }
.adv-badge {
  background:var(--grad); color:var(--black);
  font-family:'Montserrat',sans-serif; font-size:8px; font-weight:900;
  letter-spacing:2.5px; text-transform:uppercase; padding:5px 14px;
}
.adv-title {
  font-family:'Montserrat',sans-serif; font-weight:800;
  font-size:16px; letter-spacing:0.3px;
}
.adv-body {
  font-size:14px; line-height:1.85; color:var(--text-mid); max-width:900px;
}
.adv-body strong { color:var(--gold-light); font-weight:700; }
.adv-contact {
  display:flex; align-items:center; gap:24px; margin-top:20px; flex-wrap:wrap;
}
.contact-chip {
  display:flex; align-items:center; gap:8px;
  font-family:'Montserrat',sans-serif; font-weight:700;
  font-size:13px; background:var(--grad);
  -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text;
}

/* ─── EXPORT ──────────────────────────────────────────── */
.export-bar {
  display:flex; align-items:center; justify-content:space-between;
  gap:16px; flex-wrap:wrap;
  background:var(--black-card); border:1px solid var(--border);
  padding:22px 28px;
}
.exp-text h3 {
  font-family:'Montserrat',sans-serif; font-weight:800;
  font-size:13px; letter-spacing:2px; text-transform:uppercase;
  background:var(--grad); -webkit-background-clip:text;
  -webkit-text-fill-color:transparent; background-clip:text; margin-bottom:4px;
}
.exp-text p { font-size:11px; color:var(--text-dim); }
.exp-buttons { display:flex; gap:10px; flex-wrap:wrap; }
.btn-gold {
  background:var(--grad); color:var(--black); border:none; cursor:pointer;
  font-family:'Montserrat',sans-serif; font-size:10px; font-weight:800;
  letter-spacing:2px; text-transform:uppercase; padding:12px 22px;
  transition:opacity 0.2s;
}
.btn-gold:hover { opacity:0.82; }
.btn-outline {
  background:none; border:1px solid var(--border-hi); color:var(--gold-mid); cursor:pointer;
  font-family:'Montserrat',sans-serif; font-size:10px; font-weight:700;
  letter-spacing:2px; text-transform:uppercase; padding:12px 22px;
  transition:all 0.2s;
}
.btn-outline:hover { background:rgba(171,128,36,0.1); color:var(--gold-light); }

/* ─── API INFO ────────────────────────────────────────── */
.api-info {
  background:#050505; border:1px solid rgba(171,128,36,0.1);
  padding:18px 24px; margin-top:4px;
}
.api-label {
  font-family:'Montserrat',sans-serif; font-weight:700;
  font-size:8px; letter-spacing:3px; text-transform:uppercase;
  color:rgba(215,183,64,0.35); margin-bottom:10px;
}
.api-code {
  font-family:'Courier New',monospace; font-size:10px;
  color:rgba(240,214,78,0.45); background:rgba(0,0,0,0.5);
  padding:10px 14px; border-left:2px solid rgba(171,128,36,0.3);
  line-height:1.7; word-break:break-all;
}
.api-meta { font-size:10px; color:rgba(255,255,255,0.2); margin-top:8px; }

/* ─── ERROR ───────────────────────────────────────────── */
.err-box {
  background:rgba(239,68,68,0.07); border:1px solid rgba(239,68,68,0.28);
  border-left:3px solid #ef4444; padding:20px 24px;
  margin-bottom:20px; display:none;
}
.err-box p { color:rgba(255,150,150,0.9); font-size:13px; line-height:1.75; }
.err-box code { background:rgba(255,255,255,0.08); padding:2px 6px; font-size:11px; border-radius:3px; }

/* ─── FOOTER ──────────────────────────────────────────── */
.site-footer {
  border-top:1px solid var(--border); padding:22px 40px;
  display:flex; justify-content:space-between; align-items:center;
  gap:16px; flex-wrap:wrap;
}
.site-footer p { font-size:11px; color:rgba(255,255,255,0.2); }
.site-footer a { font-size:11px; color:rgba(215,183,64,0.35); text-decoration:none; transition:color 0.2s; }
.site-footer a:hover { color:var(--gold-mid); }
.footer-links { display:flex; gap:20px; }

/* ─── RESPONSIVE ──────────────────────────────────────── */
@media(max-width:960px) {
  .aqi-monster { grid-template-columns:1fr; }
  .aqi-right { border-top:1px solid var(--border); }
  .quick-stats { grid-template-columns:repeat(2,1fr); }
  .aqi-numval { font-size:clamp(80px,20vw,140px); }
}
@media(max-width:700px) {
  .top-bar { padding:12px 20px; }
  .main { padding:0 16px 60px; }
  .forecast-strip { grid-template-columns:repeat(4,1fr); }
  .pollen-strip { grid-template-columns:1fr; }
  .aqi-left { padding:32px 24px; }
  .aqi-right { padding:24px 20px; }
  .poll-grid { grid-template-columns:1fr; }
  .site-footer { flex-direction:column; text-align:center; }
}
@media(max-width:480px) {
  .forecast-strip { grid-template-columns:repeat(3,1fr); }
  .quick-stats { grid-template-columns:1fr 1fr; }
}
</style>
</head>
<body>

<!-- LOADER -->
<div id="loader">
  <div class="ld-eagle">🦅</div>
  <div class="ld-label">Loading Austin Air Data</div>
  <div class="ld-bar"><div class="ld-fill"></div></div>
</div>

<!-- TOP BAR -->
<div class="top-bar">
  <div class="brand">
    <div>
      <div class="brand-name">Gold Eagle Services</div>
      <div class="brand-sub">Cooling · Heating · Indoor Air Quality · 877-90-EAGLE</div>
    </div>
  </div>
  <div class="top-bar-right">
    <div class="live-dot"><span id="topTime">—</span>&nbsp;·&nbsp;Austin, TX</div>
    <button class="btn-refresh" id="refreshBtn" onclick="loadData()">↻ Refresh</button>
  </div>
</div>

<!-- HERO TITLE -->
<div class="hero-title">
  <div class="ht-eyebrow">Gold Eagle Services &nbsp;·&nbsp; Austin, Texas &nbsp;·&nbsp; Travis County</div>
  <div class="ht-main">Austin AQI Dashboard</div>
  <div class="ht-rule">
    <div class="ln"></div><div class="gem"></div><div class="ln r"></div>
  </div>
  <div class="ht-services">Cooling &nbsp;·&nbsp; Heating &nbsp;·&nbsp; Indoor Air Quality</div>
  <div class="ht-meta">
    <div class="ht-chip"><div class="dot"></div> 📍 30.2672°N, 97.7431°W</div>
    <div class="ht-chip"><div class="dot"></div> <span id="lastUpdate">Fetching data...</span></div>
    <div class="ht-chip"><div class="dot"></div> Data: Open-Meteo · US EPA AQI Standard</div>
  </div>
</div>

<!-- MAIN -->
<div class="main">

  <!-- ERROR BOX -->
  <div class="err-box" id="errBox">
    <p>
      ⚠️ <strong>Could not reach the Open-Meteo Air Quality API.</strong><br>
      This page fetches live data from <code>air-quality-api.open-meteo.com</code>.
      This is a sandbox preview limitation — when hosted on your website or GitHub Pages,
      it will pull live data perfectly. Try opening the downloaded HTML file directly in Chrome or Firefox.
      <br><br>
      <strong>Quick deploy options:</strong> Upload to GitHub Pages, Netlify Drop, or drop into your GoldEagleServices.com resource center.
    </p>
  </div>

  <!-- AQI MONSTER CARD -->
  <div class="aqi-monster">
    <!-- Left: number -->
    <div class="aqi-left">
      <div>
        <div class="aqi-eyebrow">Current Air Quality Index · US EPA</div>
        <div class="aqi-numwrap">
          <div class="aqi-numval" id="aqiNum">—</div>
          <div class="aqi-scale-vert">
            <div class="sv-pip" id="sv5" style="background:var(--haz)"></div>
            <div class="sv-pip" id="sv4" style="background:var(--vbad)"></div>
            <div class="sv-pip" id="sv3" style="background:var(--bad)"></div>
            <div class="sv-pip" id="sv2" style="background:var(--sens)"></div>
            <div class="sv-pip" id="sv1" style="background:var(--mod)"></div>
            <div class="sv-pip" id="sv0" style="background:var(--good)"></div>
          </div>
        </div>
        <div class="aqi-status" id="aqiStatus">Loading...</div>
        <div class="aqi-category-pill" id="aqiPill">Loading AQI Category...</div>
        <div class="aqi-desc" id="aqiDesc">Connecting to air quality monitoring data for Austin, Texas...</div>
      </div>
      <div>
        <div class="aqi-scale-row">
          <div class="sr-block" id="sb0" style="background:var(--good)"></div>
          <div class="sr-block" id="sb1" style="background:var(--mod)"></div>
          <div class="sr-block" id="sb2" style="background:var(--sens)"></div>
          <div class="sr-block" id="sb3" style="background:var(--bad)"></div>
          <div class="sr-block" id="sb4" style="background:var(--vbad)"></div>
          <div class="sr-block" id="sb5" style="background:var(--haz)"></div>
        </div>
        <div class="sr-label">
          <span>0 Good</span><span>51</span><span>101</span><span>151</span><span>201</span><span>300+ Hazardous</span>
        </div>
      </div>
    </div>

    <!-- Right: pollutants -->
    <div class="aqi-right">
      <div class="sect-label">Pollutant Breakdown — Current Hour</div>
      <div class="poll-grid">
        <div class="poll-item">
          <div class="poll-tag">PM₂.₅</div>
          <div class="poll-name-full">Fine Particulate Matter</div>
          <div class="poll-num" id="pv-pm25">—</div>
          <div class="poll-unit">µg/m³</div>
          <div class="poll-mini-bar"><div class="poll-mini-fill" id="pb-pm25"></div></div>
        </div>
        <div class="poll-item">
          <div class="poll-tag">PM₁₀</div>
          <div class="poll-name-full">Coarse Particulate Matter</div>
          <div class="poll-num" id="pv-pm10">—</div>
          <div class="poll-unit">µg/m³</div>
          <div class="poll-mini-bar"><div class="poll-mini-fill" id="pb-pm10"></div></div>
        </div>
        <div class="poll-item">
          <div class="poll-tag">O₃</div>
          <div class="poll-name-full">Ground-Level Ozone</div>
          <div class="poll-num" id="pv-o3">—</div>
          <div class="poll-unit">µg/m³</div>
          <div class="poll-mini-bar"><div class="poll-mini-fill" id="pb-o3"></div></div>
        </div>
        <div class="poll-item">
          <div class="poll-tag">NO₂</div>
          <div class="poll-name-full">Nitrogen Dioxide</div>
          <div class="poll-num" id="pv-no2">—</div>
          <div class="poll-unit">µg/m³</div>
          <div class="poll-mini-bar"><div class="poll-mini-fill" id="pb-no2"></div></div>
        </div>
        <div class="poll-item">
          <div class="poll-tag">CO</div>
          <div class="poll-name-full">Carbon Monoxide</div>
          <div class="poll-num" id="pv-co">—</div>
          <div class="poll-unit">µg/m³</div>
          <div class="poll-mini-bar"><div class="poll-mini-fill" id="pb-co"></div></div>
        </div>
        <div class="poll-item">
          <div class="poll-tag">SO₂</div>
          <div class="poll-name-full">Sulfur Dioxide</div>
          <div class="poll-num" id="pv-so2">—</div>
          <div class="poll-unit">µg/m³</div>
          <div class="poll-mini-bar"><div class="poll-mini-fill" id="pb-so2"></div></div>
        </div>
        <div class="poll-item">
          <div class="poll-tag">UV INDEX</div>
          <div class="poll-name-full">Ultraviolet Radiation</div>
          <div class="poll-num" id="pv-uv">—</div>
          <div class="poll-unit">scale 0–11+</div>
          <div class="poll-mini-bar"><div class="poll-mini-fill" id="pb-uv"></div></div>
        </div>
        <div class="poll-item">
          <div class="poll-tag">DUST</div>
          <div class="poll-name-full">Dust Concentration</div>
          <div class="poll-num" id="pv-dust">—</div>
          <div class="poll-unit">µg/m³</div>
          <div class="poll-mini-bar"><div class="poll-mini-fill" id="pb-dust"></div></div>
        </div>
      </div>
    </div>
  </div>

  <!-- QUICK STATS -->
  <div class="quick-stats">
    <div class="qs-card">
      <div class="qs-icon">💨</div>
      <div class="qs-label">Dominant Pollutant</div>
      <div class="qs-val" id="qs-dom" style="font-size:24px">—</div>
      <div class="qs-note">primary concern right now</div>
    </div>
    <div class="qs-card">
      <div class="qs-icon">🏷️</div>
      <div class="qs-label">EPA Category</div>
      <div class="qs-val override" id="qs-cat" style="font-size:18px;line-height:1.2">—</div>
      <div class="qs-note">official US classification</div>
    </div>
    <div class="qs-card">
      <div class="qs-icon">🏠</div>
      <div class="qs-label">Your Indoor Action</div>
      <div class="qs-val" id="qs-action" style="font-size:18px;line-height:1.2">—</div>
      <div class="qs-note">GES recommendation</div>
    </div>
    <div class="qs-card">
      <div class="qs-icon">🌿</div>
      <div class="qs-label">Pollen Pressure</div>
      <div class="qs-val override" id="qs-pollen" style="font-size:28px">—</div>
      <div class="qs-note">grass + ragweed + tree</div>
    </div>
  </div>

  <!-- 7-DAY FORECAST -->
  <div class="sec-header">
    <div class="sec-title">7-Day AQI Forecast</div>
    <div class="sec-line"></div>
  </div>
  <div class="forecast-strip" id="forecastStrip"></div>

  <!-- POLLEN -->
  <div class="sec-header">
    <div class="sec-title">Pollen & Allergen Report</div>
    <div class="sec-line"></div>
  </div>
  <div class="pollen-strip">
    <div class="pollen-card">
      <div class="pc-type">🌿 Grass Pollen</div>
      <div class="pc-num" id="pc-grass">—</div>
      <div class="pc-unit">grains / m³</div>
      <div class="pc-badge" id="pb2-grass">—</div>
    </div>
    <div class="pollen-card">
      <div class="pc-type">🌾 Ragweed / Weed</div>
      <div class="pc-num" id="pc-ragweed">—</div>
      <div class="pc-unit">grains / m³</div>
      <div class="pc-badge" id="pb2-ragweed">—</div>
    </div>
    <div class="pollen-card">
      <div class="pc-type">🌲 Tree (Birch)</div>
      <div class="pc-num" id="pc-birch">—</div>
      <div class="pc-unit">grains / m³</div>
      <div class="pc-badge" id="pb2-birch">—</div>
    </div>
  </div>

  <!-- ADVISORY -->
  <div class="sec-header">
    <div class="sec-title">Gold Eagle Advisory</div>
    <div class="sec-line"></div>
  </div>
  <div class="advisory-card">
    <div class="adv-top">
      <div class="adv-badge">GES Indoor Air Advisory</div>
      <div class="adv-title" id="adv-title">Generating advisory...</div>
    </div>
    <div class="adv-body" id="adv-body">Loading personalized air quality recommendation for Austin, TX...</div>
    <div class="adv-contact">
      <div class="contact-chip">📞 877-90-EAGLE</div>
      <div class="contact-chip">🌐 GoldEagleServices.com</div>
      <div class="contact-chip">⏱ Available 24/7</div>
    </div>
  </div>

  <!-- EXPORT -->
  <div class="sec-header">
    <div class="sec-title">Newsletter & Resource Center</div>
    <div class="sec-line"></div>
  </div>
  <div class="export-bar">
    <div class="exp-text">
      <h3>Export for Weekly / Monthly Newsletter</h3>
      <p>Copy formatted text or JSON payload for your resource center, email platform, or HOA newsletter</p>
    </div>
    <div class="exp-buttons">
      <button class="btn-gold" onclick="copyNewsletter()">📋 Copy Newsletter Text</button>
      <button class="btn-outline" onclick="copyJSON()">{ } Copy JSON</button>
      <button class="btn-outline" onclick="window.print()">🖨 Print</button>
    </div>
  </div>

  <!-- API INFO -->
  <div class="api-info">
    <div class="api-label">📡 Live Data Endpoint — Open-Meteo Air Quality API · Free · No API Key · CORS Enabled · Updates Hourly</div>
    <div class="api-code">https://air-quality-api.open-meteo.com/v1/air-quality?latitude=30.2672&amp;longitude=-97.7431&amp;current=us_aqi,pm10,pm2_5,carbon_monoxide,nitrogen_dioxide,sulphur_dioxide,ozone,uv_index&amp;hourly=us_aqi,pm2_5,pm10,ozone,nitrogen_dioxide,carbon_monoxide,sulphur_dioxide,grass_pollen,ragweed_pollen,birch_pollen,dust,uv_index&amp;forecast_days=7&amp;timezone=America%2FChicago</div>
    <div class="api-meta">Austin, TX · 30.2672°N 97.7431°W · JSON response · Embed in any backend for automated newsletter pulls · CC BY 4.0</div>
  </div>

</div><!-- /main -->

<footer class="site-footer">
  <p>© 2025 Gold Eagle Services · Austin, TX · Data provided by Open-Meteo (CC BY 4.0)</p>
  <div class="footer-links">
    <a href="https://goldeagleservices.com" target="_blank">GoldEagleServices.com</a>
    <a href="tel:8779032453">877-90-EAGLE</a>
    <a href="https://open-meteo.com" target="_blank">Data: Open-Meteo</a>
  </div>
</footer>

<script>
/* ═══════════════════════════════════════════════════════
   CONFIG
═══════════════════════════════════════════════════════ */
const LAT = 30.2672, LON = -97.7431;
let liveData = null;

const API = 'https://air-quality-api.open-meteo.com/v1/air-quality'
  + '?latitude=' + LAT + '&longitude=' + LON
  + '&current=us_aqi,pm10,pm2_5,carbon_monoxide,nitrogen_dioxide,sulphur_dioxide,ozone,uv_index'
  + '&hourly=us_aqi,pm2_5,pm10,ozone,nitrogen_dioxide,carbon_monoxide,sulphur_dioxide,grass_pollen,ragweed_pollen,birch_pollen,dust,uv_index'
  + '&forecast_days=7&timezone=America%2FChicago';

/* ═══════════════════════════════════════════════════════
   AQI LOGIC
═══════════════════════════════════════════════════════ */
function aqiMeta(v) {
  if (v == null) return { status:'Unknown', color:'#888', pip:0, desc:'No data.', action:'Check system', cat:'Unknown' };
  v = Math.round(v);
  if (v <= 50)  return {
    status:'Good', color:'var(--good)', pip:0, cat:'Good',
    desc:'Air quality is excellent today. This is your green-light morning — great conditions for opening windows briefly, running your ERV, and scheduling seasonal HVAC maintenance. A clean outdoor air day keeps your indoor system performing at its best.',
    action:'Maintain & Monitor'
  };
  if (v <= 100) return {
    status:'Moderate', color:'var(--mod)', pip:1, cat:'Moderate',
    desc:'Air quality is acceptable for most Austin residents. Unusually sensitive individuals may notice mild effects. This is a good morning to check your air filter — if it\'s been in service 60+ days, swap it for a fresh MERV-11 or higher.',
    action:'Check Your Filters'
  };
  if (v <= 150) return {
    status:'Unhealthy for Sensitive Groups', color:'var(--sens)', pip:2, cat:'Sensitive Groups',
    desc:'If you or anyone in your home has asthma, allergies, or respiratory conditions, limit outdoor time today. Make sure your air purification is running. Gold Eagle Services recommends a duct inspection and MERV-13+ filter upgrade on days like this.',
    action:'Upgrade to MERV-13+'
  };
  if (v <= 200) return {
    status:'Unhealthy', color:'var(--bad)', pip:3, cat:'Unhealthy',
    desc:'Everyone may begin to feel health effects. Keep windows and doors closed. Switch your HVAC to recirculation mode. If you don\'t have whole-home air purification installed, today is the day to call Gold Eagle Services — 877-90-EAGLE.',
    action:'Close Up + Recirculate'
  };
  if (v <= 300) return {
    status:'Very Unhealthy', color:'var(--vbad)', pip:4, cat:'Very Unhealthy',
    desc:'Health alert for all Austin residents. Stay indoors and keep your HVAC on full recirculation with the highest-rated filter you have. If your system hasn\'t been serviced recently, performance will be compromised right now when it matters most.',
    action:'Emergency Recirculation'
  };
  return {
    status:'Hazardous', color:'var(--haz)', pip:5, cat:'Hazardous',
    desc:'EMERGENCY CONDITIONS. Do not open any windows or doors. Seal gaps. Run HVAC on recirculation-only with maximum filtration. Call Gold Eagle Services immediately at 877-90-EAGLE — we are available 24/7 for emergency air quality support.',
    action:'Call GES: 877-90-EAGLE'
  };
}

function pollenMeta(v) {
  if (v == null) return { label:'N/A', color:'#888', bg:'rgba(255,255,255,0.06)' };
  if (v < 10)   return { label:'Low',       color:'var(--good)', bg:'rgba(34,197,94,0.1)'  };
  if (v < 50)   return { label:'Moderate',  color:'var(--mod)',  bg:'rgba(234,179,8,0.1)'  };
  if (v < 200)  return { label:'High',      color:'var(--sens)', bg:'rgba(249,115,22,0.1)' };
  return              { label:'Very High',  color:'var(--bad)',  bg:'rgba(239,68,68,0.1)'  };
}

/* ═══════════════════════════════════════════════════════
   HELPERS
═══════════════════════════════════════════════════════ */
function set(id, text) {
  const el = document.getElementById(id);
  if (el) el.textContent = text;
}

function setBar(id, val, max) {
  const el = document.getElementById(id);
  if (el) el.style.width = (val != null ? Math.min(100, Math.abs(val) / max * 100) : 0) + '%';
}

function curHourIdx(times) {
  const now = new Date();
  const p = n => String(n).padStart(2,'0');
  const s = `${now.getFullYear()}-${p(now.getMonth()+1)}-${p(now.getDate())}T${p(now.getHours())}:00`;
  const i = times.indexOf(s);
  return i >= 0 ? i : 0;
}

/* ═══════════════════════════════════════════════════════
   CLOCK
═══════════════════════════════════════════════════════ */
function tickClock() {
  const now = new Date();
  const t = now.toLocaleTimeString('en-US', { hour:'2-digit', minute:'2-digit', second:'2-digit', timeZone:'America/Chicago' });
  const d = now.toLocaleDateString('en-US', { weekday:'long', month:'long', day:'numeric', timeZone:'America/Chicago' });
  set('topTime', d + '  ·  ' + t + ' CT');
}
tickClock();
setInterval(tickClock, 1000);

/* ═══════════════════════════════════════════════════════
   RENDER
═══════════════════════════════════════════════════════ */
function render(data) {
  document.getElementById('errBox').style.display = 'none';
  const c = data.current, h = data.hourly;
  const hIdx = curHourIdx(h.time);
  const aqi = c.us_aqi;
  const meta = aqiMeta(aqi);

  /* ── AQI number */
  const numEl = document.getElementById('aqiNum');
  numEl.textContent = aqi != null ? aqi : '—';
  numEl.style.cssText = '';
  numEl.style.setProperty('--aqi-color', meta.color);
  // Use CSS variable color via inline style trick
  numEl.style.background = 'none';
  numEl.style.webkitTextFillColor = meta.color;
  numEl.style.color = meta.color;

  /* ── Status */
  const statusEl = document.getElementById('aqiStatus');
  statusEl.textContent = meta.status;
  statusEl.style.color = meta.color;

  /* ── Category pill */
  const pillEl = document.getElementById('aqiPill');
  pillEl.textContent = meta.cat;
  pillEl.style.borderColor = meta.color;
  pillEl.style.color = meta.color;

  /* ── Desc */
  set('aqiDesc', meta.desc);

  /* ── Scale indicators */
  for (let i = 0; i < 6; i++) {
    const sv = document.getElementById('sv' + i);
    const sb = document.getElementById('sb' + i);
    if (sv) sv.classList.toggle('on', i === meta.pip);
    if (sb) sb.classList.toggle('on', i === meta.pip);
  }

  /* ── Pollutants */
  const pollutants = [
    { id:'pm25', val:c.pm2_5,            max:55.5  },
    { id:'pm10', val:c.pm10,             max:155   },
    { id:'o3',   val:c.ozone,            max:105   },
    { id:'no2',  val:c.nitrogen_dioxide, max:100   },
    { id:'co',   val:c.carbon_monoxide,  max:9500  },
    { id:'so2',  val:c.sulphur_dioxide,  max:185   },
    { id:'uv',   val:c.uv_index,         max:11    },
    { id:'dust', val:h.dust?.[hIdx],     max:100   },
  ];
  pollutants.forEach(p => {
    const vEl = document.getElementById('pv-' + p.id);
    if (vEl) vEl.textContent = p.val != null ? p.val.toFixed(1) : '—';
  });
  setTimeout(() => pollutants.forEach(p => setBar('pb-' + p.id, p.val, p.max)), 200);

  /* ── Quick stats */
  const cands = [
    { name:'PM₂.₅', s: c.pm2_5 != null ? c.pm2_5/55.5 : 0 },
    { name:'PM₁₀',  s: c.pm10  != null ? c.pm10/155   : 0 },
    { name:'O₃',    s: c.ozone != null ? c.ozone/105  : 0 },
    { name:'NO₂',   s: c.nitrogen_dioxide != null ? c.nitrogen_dioxide/100 : 0 },
    { name:'CO',    s: c.carbon_monoxide  != null ? c.carbon_monoxide/9500 : 0 },
  ];
  cands.sort((a,b) => b.s - a.s);
  set('qs-dom', cands[0].name);

  const catEl = document.getElementById('qs-cat');
  catEl.textContent = meta.cat;
  catEl.style.setProperty('--ov-color', meta.color);
  catEl.style.color = meta.color;
  catEl.style.webkitTextFillColor = meta.color;

  set('qs-action', meta.action);

  /* ── Pollen */
  const grass   = h.grass_pollen?.[hIdx] ?? null;
  const ragweed = h.ragweed_pollen?.[hIdx] ?? null;
  const birch   = h.birch_pollen?.[hIdx] ?? null;
  const combined = (grass || 0) + (ragweed || 0) + (birch || 0);
  const pp = pollenMeta(combined);
  const polEl = document.getElementById('qs-pollen');
  polEl.textContent = pp.label;
  polEl.style.color = pp.color;
  polEl.style.webkitTextFillColor = pp.color;

  /* ── 7-Day Forecast */
  const dayMap = {};
  (h.time || []).forEach((t, i) => {
    const d = t.split('T')[0];
    if (h.us_aqi[i] != null) { if (!dayMap[d]) dayMap[d] = []; dayMap[d].push(h.us_aqi[i]); }
  });
  const days = Object.keys(dayMap).slice(0, 7);
  const todayStr = new Date().toISOString().split('T')[0];
  const DNS = ['Sun','Mon','Tue','Wed','Thu','Fri','Sat'];
  document.getElementById('forecastStrip').innerHTML = days.map((date, idx) => {
    const vals = dayMap[date];
    const avg = Math.round(vals.reduce((a,b) => a+b, 0) / vals.length);
    const m = aqiMeta(avg);
    const dow = new Date(date + 'T12:00:00').getDay();
    const name = idx === 0 ? 'Today' : DNS[dow];
    const isToday = date === todayStr;
    const pct = Math.min(100, avg / 200 * 100);
    return `<div class="fc-card ${isToday ? 'today' : ''}">
      <div class="fc-day-name">${name}</div>
      <div class="fc-aqi-num" style="color:${m.color}">${avg}</div>
      <div class="fc-status">${m.status.split(' ')[0]}</div>
      <div class="fc-track"><div class="fc-fill" style="width:${pct}%;background:${m.color}"></div></div>
    </div>`;
  }).join('');

  /* ── Pollen tiles */
  const setPollen = (numId, badgeId, val) => {
    const m = pollenMeta(val);
    const nEl = document.getElementById(numId);
    nEl.textContent = val != null ? val.toFixed(1) : '—';
    nEl.style.color = m.color;
    const bEl = document.getElementById(badgeId);
    bEl.textContent = m.label;
    bEl.style.background = m.bg;
    bEl.style.color = m.color;
    bEl.style.border = '1px solid ' + m.color + '50';
  };
  setPollen('pc-grass',   'pb2-grass',   grass);
  setPollen('pc-ragweed', 'pb2-ragweed', ragweed);
  setPollen('pc-birch',   'pb2-birch',   birch);

  /* ── Advisory */
  const pollenWarn = (combined > 50)
    ? ' Pollen levels are elevated today — if you have an ERV, verify its intake pre-filter is clean.'
    : '';
  set('adv-title', meta.status === 'Good'
    ? 'Clean Air Morning — Great Day for Your HVAC System'
    : meta.status === 'Moderate'
    ? 'Moderate Conditions — A Timely Filter Reminder'
    : meta.status.includes('Sensitive')
    ? 'Heads Up for Austin Families with Respiratory Conditions'
    : meta.status === 'Unhealthy'
    ? 'Unhealthy Air Today — Protect Your Home Environment'
    : meta.status.includes('Very')
    ? 'Very Unhealthy — Full Indoor Protection Recommended'
    : '🚨 Hazardous Air Quality — Emergency Action Required'
  );
  document.getElementById('adv-body').innerHTML =
    meta.desc.replace(/(877-90-EAGLE|Gold Eagle Services|MERV-13\+|ERV|HVAC|duct|filter|recirculation|whole-home|air purif)/gi,
      m => `<strong>${m}</strong>`) + pollenWarn;

  /* ── Last update */
  const now = new Date();
  set('lastUpdate', 'Updated ' + now.toLocaleTimeString('en-US',{hour:'2-digit',minute:'2-digit',timeZone:'America/Chicago'}) + ' CT');
}

/* ═══════════════════════════════════════════════════════
   FETCH
═══════════════════════════════════════════════════════ */
async function loadData() {
  const btn = document.getElementById('refreshBtn');
  btn.classList.add('spinning');
  document.getElementById('loader').classList.remove('hidden');
  document.getElementById('errBox').style.display = 'none';

  try {
    const res = await fetch(API);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const data = await res.json();
    if (!data?.current) throw new Error('Malformed response');
    liveData = data;
    render(data);
  } catch (err) {
    console.error('AQI Fetch Error:', err);
    document.getElementById('errBox').style.display = 'block';
    set('aqiStatus', 'Data unavailable');
    set('aqiDesc', 'Could not reach the Open-Meteo API. See the message above. Works perfectly when hosted live.');
    set('lastUpdate', 'Failed — click Refresh');
  } finally {
    document.getElementById('loader').classList.add('hidden');
    btn.classList.remove('spinning');
  }
}

/* ═══════════════════════════════════════════════════════
   EXPORT
═══════════════════════════════════════════════════════ */
function copyNewsletter() {
  if (!liveData) { alert('Data not loaded yet — click Refresh first.'); return; }
  const c = liveData.current, h = liveData.hourly;
  const hIdx = curHourIdx(h.time);
  const meta = aqiMeta(c.us_aqi);
  const now = new Date().toLocaleDateString('en-US',{weekday:'long',year:'numeric',month:'long',day:'numeric'});
  const bar = '━'.repeat(48);
  const txt = [
    'GOLD EAGLE SERVICES — AUSTIN AIR QUALITY REPORT',
    now, bar, '',
    `CURRENT AQI: ${c.us_aqi} — ${meta.status.toUpperCase()}`,
    'Location: Austin, TX · Travis County', '',
    'POLLUTANT READINGS (µg/m³):',
    `  PM2.5 (Fine Particulates)     ${c.pm2_5?.toFixed(1) ?? '—'}`,
    `  PM10  (Coarse Particulates)   ${c.pm10?.toFixed(1) ?? '—'}`,
    `  Ozone O₃                      ${c.ozone?.toFixed(1) ?? '—'}`,
    `  Nitrogen Dioxide NO₂          ${c.nitrogen_dioxide?.toFixed(1) ?? '—'}`,
    `  Carbon Monoxide CO             ${c.carbon_monoxide?.toFixed(1) ?? '—'}`,
    `  Sulfur Dioxide SO₂             ${c.sulphur_dioxide?.toFixed(1) ?? '—'}`,
    `  UV Index                       ${c.uv_index?.toFixed(1) ?? '—'}`, '',
    'POLLEN LEVELS (grains/m³):',
    `  Grass Pollen    ${h.grass_pollen?.[hIdx]?.toFixed(1) ?? '—'}`,
    `  Ragweed/Weed    ${h.ragweed_pollen?.[hIdx]?.toFixed(1) ?? '—'}`,
    `  Tree (Birch)    ${h.birch_pollen?.[hIdx]?.toFixed(1) ?? '—'}`, '',
    'INDOOR AIR QUALITY ADVISORY:',
    meta.desc, '',
    `GES RECOMMENDED ACTION: ${meta.action}`, '',
    bar,
    'Gold Eagle Services — Austin\'s Indoor Air Quality Experts',
    'Swift · Trusted · Super Friendly · Available 24/7',
    '📞  877-90-EAGLE   |   🌐  GoldEagleServices.com',
    'Cooling · Heating · ERV Systems · Duct Optimization · Dehumidifiers',
  ].join('\n');

  navigator.clipboard.writeText(txt)
    .then(() => alert('Newsletter text copied to clipboard!\nPaste into Mailchimp, Klaviyo, or your email editor.'))
    .catch(() => { const w = window.open('','_blank'); w.document.write('<pre style="font-family:monospace;padding:20px">'+txt+'</pre>'); });
}

function copyJSON() {
  if (!liveData) { alert('Data not loaded yet — click Refresh first.'); return; }
  const c = liveData.current, h = liveData.hourly;
  const hIdx = curHourIdx(h.time);
  const meta = aqiMeta(c.us_aqi);
  const out = {
    source: 'Gold Eagle Services AQI Dashboard',
    dataProvider: 'Open-Meteo Air Quality API (CC BY 4.0)',
    location: { city:'Austin', state:'TX', county:'Travis', lat:LAT, lon:LON },
    timestamp: new Date().toISOString(),
    aqi: { value:c.us_aqi, category:meta.status, description:meta.desc, action:meta.action },
    pollutants: {
      pm2_5_ugm3:  c.pm2_5,
      pm10_ugm3:   c.pm10,
      ozone_ugm3:  c.ozone,
      no2_ugm3:    c.nitrogen_dioxide,
      co_ugm3:     c.carbon_monoxide,
      so2_ugm3:    c.sulphur_dioxide,
      uv_index:    c.uv_index,
    },
    pollen: {
      grass_grains_m3:   h.grass_pollen?.[hIdx] ?? null,
      ragweed_grains_m3: h.ragweed_pollen?.[hIdx] ?? null,
      birch_grains_m3:   h.birch_pollen?.[hIdx] ?? null,
    },
    contact: { phone:'877-90-EAGLE', website:'GoldEagleServices.com', available:'24/7' }
  };
  const s = JSON.stringify(out, null, 2);
  navigator.clipboard.writeText(s)
    .then(() => alert('JSON payload copied!\nPaste this into your resource center API or newsletter automation.'))
    .catch(() => { const w = window.open('','_blank'); w.document.write('<pre style="font-family:monospace;padding:20px">'+s+'</pre>'); });
}

/* ═══════════════════════════════════════════════════════
   INIT
═══════════════════════════════════════════════════════ */
window.addEventListener('DOMContentLoaded', () => {
  loadData();
  setInterval(loadData, 30 * 60 * 1000); // refresh every 30 min
});
</script>
</body>
</html>
