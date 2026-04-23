<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DermaAI — Smart Skin Analysis</title>
<link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --cream: #F9F5F0;
    --sand: #EDE6DC;
    --blush: #D4A89A;
    --rose: #B5705E;
    --deep: #2C1A14;
    --warm-gray: #8A7A72;
    --white: #FFFFFF;
    --text: #1E1210;
    --muted: #7A6860;
    --border: rgba(44,26,20,0.1);
    --card-bg: rgba(255,255,255,0.72);
    --shadow: 0 4px 32px rgba(44,26,20,0.08);
    --shadow-lg: 0 16px 64px rgba(44,26,20,0.12);
  }

  html { scroll-behavior: smooth; }

  body {
    font-family: 'DM Sans', sans-serif;
    background: var(--cream);
    color: var(--text);
    overflow-x: hidden;
    -webkit-font-smoothing: antialiased;
  }

  /* ── NOISE TEXTURE OVERLAY ── */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
    pointer-events: none;
    z-index: 0;
    opacity: 0.5;
  }

  /* ── NAV ── */
  nav {
    position: fixed; top: 0; left: 0; right: 0; z-index: 100;
    display: flex; align-items: center; justify-content: space-between;
    padding: 1.1rem 2rem;
    background: rgba(249,245,240,0.85);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border-bottom: 1px solid var(--border);
  }

  .nav-logo {
    font-family: 'DM Serif Display', serif;
    font-size: 1.45rem;
    color: var(--deep);
    letter-spacing: -0.02em;
    text-decoration: none;
  }

  .nav-logo span { color: var(--rose); }

  .nav-links {
    display: flex; gap: 2rem; list-style: none;
  }

  .nav-links a {
    font-size: 0.9rem; font-weight: 500; color: var(--muted);
    text-decoration: none; transition: color 0.2s;
  }

  .nav-links a:hover { color: var(--deep); }

  .nav-cta {
    display: flex; gap: 10px; align-items: center;
  }

  .btn-ghost {
    padding: 0.52rem 1.2rem; border-radius: 999px;
    border: 1px solid var(--border); background: transparent;
    font-family: 'DM Sans', sans-serif; font-size: 0.88rem; font-weight: 500;
    color: var(--deep); cursor: pointer; transition: all 0.2s;
    text-decoration: none; display: inline-block;
  }

  .btn-ghost:hover { background: var(--sand); border-color: var(--blush); }

  .btn-primary {
    padding: 0.52rem 1.3rem; border-radius: 999px;
    border: none; background: var(--deep);
    font-family: 'DM Sans', sans-serif; font-size: 0.88rem; font-weight: 500;
    color: var(--white); cursor: pointer; transition: all 0.22s;
    text-decoration: none; display: inline-block;
  }

  .btn-primary:hover { background: var(--rose); transform: translateY(-1px); box-shadow: 0 6px 20px rgba(181,112,94,0.35); }

  .hamburger { display: none; flex-direction: column; gap: 5px; cursor: pointer; background: none; border: none; padding: 4px; }
  .hamburger span { width: 22px; height: 2px; background: var(--deep); border-radius: 2px; transition: all 0.3s; display: block; }

  /* ── MOBILE MENU ── */
  .mobile-menu {
    display: none; position: fixed; inset: 0; z-index: 99;
    background: rgba(249,245,240,0.97); backdrop-filter: blur(20px);
    flex-direction: column; align-items: center; justify-content: center; gap: 2rem;
  }

  .mobile-menu.open { display: flex; }
  .mobile-menu a { font-family: 'DM Serif Display', serif; font-size: 2rem; color: var(--deep); text-decoration: none; }
  .mobile-menu a:hover { color: var(--rose); }
  .mobile-menu .close-btn { position: absolute; top: 1.5rem; right: 2rem; background: none; border: none; font-size: 1.8rem; cursor: pointer; color: var(--deep); }

  /* ── HERO ── */
  .hero {
    min-height: 100vh; display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    padding: 8rem 2rem 5rem; text-align: center;
    position: relative; overflow: hidden;
    background:
      radial-gradient(ellipse 70% 60% at 75% 20%, rgba(212,168,154,0.28) 0%, transparent 65%),
      radial-gradient(ellipse 55% 50% at 15% 80%, rgba(200,184,162,0.22) 0%, transparent 60%),
      var(--cream);
  }

  /* botanical SVG bg */
  .hero-bg-svg {
    position: absolute; inset: 0; width: 100%; height: 100%;
    pointer-events: none; opacity: 0.07;
  }

  .hero-blob {
    position: absolute; border-radius: 50%; filter: blur(80px); opacity: 0.35; pointer-events: none;
  }

  .hero-blob-1 { width: 520px; height: 520px; background: var(--blush); top: -120px; right: -80px; animation: drift1 12s ease-in-out infinite; }
  .hero-blob-2 { width: 380px; height: 380px; background: #C8B8A2; bottom: 0; left: -80px; animation: drift2 15s ease-in-out infinite; }

  @keyframes drift1 { 0%,100%{transform:translate(0,0) scale(1)} 50%{transform:translate(-30px,30px) scale(1.05)} }
  @keyframes drift2 { 0%,100%{transform:translate(0,0) scale(1)} 50%{transform:translate(20px,-20px) scale(1.04)} }

  .hero-tag {
    display: inline-flex; align-items: center; gap: 6px;
    background: var(--white); border: 1px solid var(--border); border-radius: 999px;
    padding: 0.38rem 1rem; font-size: 0.8rem; font-weight: 500; color: var(--rose);
    margin-bottom: 1.8rem; letter-spacing: 0.04em;
    animation: fadeUp 0.6s ease both;
    box-shadow: var(--shadow);
  }

  .hero-tag::before { content:''; width:7px; height:7px; background:var(--rose); border-radius:50%; animation: pulse 2s infinite; }
  @keyframes pulse { 0%,100%{transform:scale(1);opacity:1} 50%{transform:scale(1.5);opacity:0.5} }

  h1 {
    font-family: 'DM Serif Display', serif;
    font-size: clamp(2.8rem, 7vw, 5.2rem);
    line-height: 1.08; letter-spacing: -0.03em;
    color: var(--deep); max-width: 820px;
    animation: fadeUp 0.7s 0.1s ease both;
  }

  h1 em { font-style: italic; color: var(--rose); }

  .hero-sub {
    font-size: clamp(1rem, 2.5vw, 1.18rem); font-weight: 300;
    color: var(--muted); max-width: 520px; line-height: 1.7;
    margin: 1.5rem auto 2.4rem;
    animation: fadeUp 0.7s 0.2s ease both;
  }

  .hero-actions {
    display: flex; gap: 12px; justify-content: center; flex-wrap: wrap;
    animation: fadeUp 0.7s 0.3s ease both;
  }

  .btn-hero {
    padding: 0.85rem 2.2rem; border-radius: 999px;
    font-family: 'DM Sans', sans-serif; font-size: 1rem; font-weight: 600;
    cursor: pointer; transition: all 0.22s; text-decoration: none; display: inline-block;
  }

  .btn-hero-primary {
    background: var(--deep); color: var(--white); border: none;
    box-shadow: 0 8px 28px rgba(44,26,20,0.22);
  }

  .btn-hero-primary:hover { background: var(--rose); transform: translateY(-2px); box-shadow: 0 12px 36px rgba(181,112,94,0.4); }

  .btn-hero-outline {
    background: var(--white); color: var(--deep);
    border: 1px solid var(--border);
    box-shadow: var(--shadow);
  }

  .btn-hero-outline:hover { border-color: var(--blush); transform: translateY(-2px); }

  .hero-stats {
    display: flex; gap: 3rem; margin-top: 4rem; justify-content: center; flex-wrap: wrap;
    animation: fadeUp 0.7s 0.45s ease both;
  }

  .stat { text-align: center; }
  .stat-num { font-family: 'DM Serif Display', serif; font-size: 2.1rem; color: var(--deep); }
  .stat-label { font-size: 0.82rem; color: var(--muted); font-weight: 500; margin-top: 2px; }

  @keyframes fadeUp { from{opacity:0;transform:translateY(22px)} to{opacity:1;transform:translateY(0)} }

  /* ── SECTION COMMONS ── */
  section { position: relative; z-index: 1; }

  .section-label {
    display: inline-block; font-size: 0.78rem; font-weight: 600;
    letter-spacing: 0.1em; text-transform: uppercase; color: var(--rose);
    margin-bottom: 0.7rem;
  }

  .section-title {
    font-family: 'DM Serif Display', serif;
    font-size: clamp(2rem, 5vw, 3rem);
    color: var(--deep); line-height: 1.12; letter-spacing: -0.02em;
    margin-bottom: 1rem;
  }

  .section-sub {
    font-size: 1.05rem; color: var(--muted); font-weight: 300;
    line-height: 1.7; max-width: 540px;
  }

  /* ── UPLOAD SECTION ── */
  .upload-section {
    padding: 6rem 2rem; overflow: hidden; position: relative;
    background:
      radial-gradient(ellipse 60% 80% at 90% 50%, rgba(181,112,94,0.18) 0%, transparent 60%),
      radial-gradient(ellipse 50% 60% at 10% 30%, rgba(212,168,154,0.12) 0%, transparent 55%),
      var(--deep);
  }

  .upload-bg-lines {
    position: absolute; inset: 0; pointer-events: none; overflow: hidden; opacity: 0.06;
  }

  .upload-inner {
    max-width: 1100px; margin: 0 auto;
    display: grid; grid-template-columns: 1fr 1fr; gap: 5rem; align-items: center;
  }

  .upload-text .section-label { color: var(--blush); }
  .upload-text .section-title { color: var(--white); }
  .upload-text .section-sub { color: rgba(255,255,255,0.55); max-width: 420px; }

  .upload-features { margin-top: 2rem; display: flex; flex-direction: column; gap: 1rem; }

  .feature-row {
    display: flex; align-items: flex-start; gap: 12px;
    background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.08);
    border-radius: 14px; padding: 1rem 1.2rem;
    transition: background 0.2s;
  }

  .feature-row:hover { background: rgba(255,255,255,0.09); }

  .feat-icon {
    width: 38px; height: 38px; border-radius: 10px;
    background: rgba(212,168,154,0.2); display: flex; align-items: center; justify-content: center;
    flex-shrink: 0; font-size: 1.1rem;
  }

  .feat-text h4 { font-size: 0.95rem; font-weight: 600; color: var(--white); margin-bottom: 2px; }
  .feat-text p { font-size: 0.82rem; color: rgba(255,255,255,0.5); line-height: 1.5; }

  .upload-card {
    background: var(--card-bg);
    backdrop-filter: blur(20px);
    border: 1px solid rgba(255,255,255,0.5);
    border-radius: 28px; padding: 2rem;
    box-shadow: var(--shadow-lg);
  }

  .upload-zone {
    border: 2px dashed rgba(181,112,94,0.35); border-radius: 20px;
    padding: 3rem 2rem; text-align: center; cursor: pointer;
    transition: all 0.25s; background: rgba(249,245,240,0.5);
    position: relative; overflow: hidden;
  }

  .upload-zone:hover { border-color: var(--rose); background: rgba(249,245,240,0.8); transform: scale(1.01); }

  .upload-icon { font-size: 2.8rem; margin-bottom: 0.8rem; display: block; }

  .upload-zone h3 { font-family: 'DM Serif Display', serif; font-size: 1.3rem; color: var(--deep); margin-bottom: 0.4rem; }
  .upload-zone p { font-size: 0.85rem; color: var(--muted); }

  .upload-zone input { position: absolute; inset: 0; opacity: 0; cursor: pointer; }

  .upload-divider {
    display: flex; align-items: center; gap: 1rem; margin: 1.4rem 0; color: var(--muted); font-size: 0.83rem;
  }

  .upload-divider::before, .upload-divider::after {
    content: ''; flex: 1; height: 1px; background: var(--border);
  }

  .camera-btn {
    width: 100%; padding: 0.9rem; border-radius: 14px;
    background: var(--deep); color: var(--white); border: none;
    font-family: 'DM Sans', sans-serif; font-size: 0.95rem; font-weight: 600;
    cursor: pointer; transition: all 0.22s; display: flex; align-items: center; justify-content: center; gap: 8px;
  }

  .camera-btn:hover { background: var(--rose); transform: translateY(-1px); box-shadow: 0 8px 24px rgba(181,112,94,0.35); }

  .scan-tag {
    display: flex; align-items: center; gap: 6px; margin-top: 1.2rem;
    background: rgba(181,112,94,0.08); border-radius: 10px; padding: 0.7rem 1rem;
    font-size: 0.82rem; color: var(--muted);
  }

  .scan-tag span { font-weight: 600; color: var(--rose); }

  /* ── HOW IT WORKS ── */
  .how-section {
    padding: 6rem 2rem; position: relative; overflow: hidden;
    background:
      radial-gradient(ellipse 80% 50% at 50% 100%, rgba(212,168,154,0.15) 0%, transparent 70%),
      var(--cream);
  }

  .how-bg {
    position: absolute; inset: 0; pointer-events: none;
    background-image: radial-gradient(circle, rgba(44,26,20,0.12) 1px, transparent 1px);
    background-size: 32px 32px;
    mask-image: radial-gradient(ellipse 80% 80% at 50% 50%, black 40%, transparent 100%);
    -webkit-mask-image: radial-gradient(ellipse 80% 80% at 50% 50%, black 40%, transparent 100%);
  }

  .how-inner { max-width: 1100px; margin: 0 auto; }

  .how-header { text-align: center; margin-bottom: 4rem; }

  .steps-grid {
    display: grid; grid-template-columns: repeat(3, 1fr); gap: 1.5rem;
  }

  .step-card {
    background: var(--white); border: 1px solid var(--border); border-radius: 24px;
    padding: 2rem 1.6rem; position: relative; overflow: hidden;
    transition: all 0.3s; box-shadow: var(--shadow);
  }

  .step-card:hover { transform: translateY(-5px); box-shadow: var(--shadow-lg); border-color: var(--blush); }

  .step-num {
    font-family: 'DM Serif Display', serif; font-size: 4rem;
    color: var(--sand); position: absolute; right: 1.5rem; top: 0.8rem;
    line-height: 1; user-select: none;
  }

  .step-icon { font-size: 1.8rem; margin-bottom: 1rem; }
  .step-card h3 { font-size: 1.1rem; font-weight: 600; color: var(--deep); margin-bottom: 0.5rem; }
  .step-card p { font-size: 0.88rem; color: var(--muted); line-height: 1.6; }

  /* ── AUTH SECTION ── */
  .auth-section {
    padding: 6rem 2rem; position: relative; overflow: hidden;
    background:
      radial-gradient(ellipse 60% 70% at 100% 0%, rgba(181,112,94,0.18) 0%, transparent 60%),
      radial-gradient(ellipse 50% 50% at 0% 100%, rgba(212,168,154,0.2) 0%, transparent 55%),
      var(--sand);
  }

  .auth-bg { position: absolute; inset: 0; pointer-events: none; overflow: hidden; }
  .auth-bg-shape {
    position: absolute; border-radius: 50%; border: 1px solid rgba(181,112,94,0.18);
  }

  .auth-inner {
    max-width: 1100px; margin: 0 auto;
    display: grid; grid-template-columns: 1fr 1fr; gap: 5rem; align-items: center;
  }

  .auth-card {
    background: var(--white); border-radius: 28px; padding: 2.4rem;
    box-shadow: var(--shadow-lg); border: 1px solid var(--border);
  }

  .auth-tabs {
    display: flex; background: var(--cream); border-radius: 12px; padding: 4px; margin-bottom: 1.8rem;
  }

  .auth-tab {
    flex: 1; padding: 0.55rem; text-align: center; border-radius: 9px;
    font-size: 0.88rem; font-weight: 500; cursor: pointer; border: none; background: transparent;
    font-family: 'DM Sans', sans-serif; color: var(--muted); transition: all 0.2s;
  }

  .auth-tab.active { background: var(--white); color: var(--deep); box-shadow: 0 2px 8px rgba(44,26,20,0.1); }

  .social-btns { display: flex; flex-direction: column; gap: 10px; margin-bottom: 1.5rem; }

  .social-btn {
    display: flex; align-items: center; gap: 12px; padding: 0.78rem 1.2rem;
    border: 1px solid var(--border); border-radius: 12px; background: var(--white);
    font-family: 'DM Sans', sans-serif; font-size: 0.9rem; font-weight: 500; color: var(--deep);
    cursor: pointer; transition: all 0.2s; width: 100%;
  }

  .social-btn:hover { background: var(--cream); border-color: var(--blush); transform: translateX(2px); }

  .social-btn svg { width: 20px; height: 20px; flex-shrink: 0; }

  .or-divider { display: flex; align-items: center; gap: 12px; color: var(--muted); font-size: 0.82rem; margin: 1.2rem 0; }
  .or-divider::before, .or-divider::after { content:''; flex:1; height:1px; background:var(--border); }

  .input-group { display: flex; flex-direction: column; gap: 10px; margin-bottom: 1.2rem; }

  .auth-input {
    width: 100%; padding: 0.78rem 1rem; border-radius: 12px;
    border: 1px solid var(--border); background: var(--cream);
    font-family: 'DM Sans', sans-serif; font-size: 0.9rem; color: var(--deep);
    transition: border-color 0.2s; outline: none;
  }

  .auth-input:focus { border-color: var(--rose); background: var(--white); }
  .auth-input::placeholder { color: var(--muted); }

  .btn-auth {
    width: 100%; padding: 0.85rem; border-radius: 12px; border: none;
    background: var(--deep); color: var(--white);
    font-family: 'DM Sans', sans-serif; font-size: 0.95rem; font-weight: 600;
    cursor: pointer; transition: all 0.22s;
  }

  .btn-auth:hover { background: var(--rose); box-shadow: 0 8px 24px rgba(181,112,94,0.35); }

  .auth-footer { text-align: center; font-size: 0.8rem; color: var(--muted); margin-top: 1rem; }
  .auth-footer a { color: var(--rose); text-decoration: none; font-weight: 500; }

  .auth-promo { }
  .auth-promo .section-sub { max-width: 400px; }

  .promo-list { margin-top: 2rem; display: flex; flex-direction: column; gap: 12px; }

  .promo-item {
    display: flex; align-items: center; gap: 10px;
    font-size: 0.9rem; color: var(--deep);
  }

  .promo-check {
    width: 22px; height: 22px; border-radius: 50%; background: var(--rose);
    display: flex; align-items: center; justify-content: center; flex-shrink: 0;
    color: white; font-size: 0.7rem;
  }

  /* ── REVIEWS ── */
  .reviews-section {
    padding: 6rem 2rem; overflow: hidden; position: relative;
    background:
      radial-gradient(ellipse 70% 50% at 20% 20%, rgba(212,168,154,0.2) 0%, transparent 60%),
      radial-gradient(ellipse 60% 60% at 80% 80%, rgba(181,112,94,0.12) 0%, transparent 55%),
      var(--cream);
  }

  .reviews-bg {
    position: absolute; inset: 0; pointer-events: none;
    background-image:
      linear-gradient(rgba(44,26,20,0.05) 1px, transparent 1px),
      linear-gradient(90deg, rgba(44,26,20,0.05) 1px, transparent 1px);
    background-size: 60px 60px;
    mask-image: radial-gradient(ellipse 90% 70% at 50% 50%, black 30%, transparent 100%);
    -webkit-mask-image: radial-gradient(ellipse 90% 70% at 50% 50%, black 30%, transparent 100%);
  }

  .reviews-inner { max-width: 1100px; margin: 0 auto; }

  .reviews-header { text-align: center; margin-bottom: 4rem; }

  .reviews-grid {
    display: grid; grid-template-columns: repeat(3, 1fr); gap: 1.5rem;
  }

  .review-card {
    background: var(--white); border: 1px solid var(--border); border-radius: 24px;
    padding: 1.8rem; box-shadow: var(--shadow); transition: all 0.3s;
    position: relative;
  }

  .review-card:hover { transform: translateY(-4px); box-shadow: var(--shadow-lg); }

  .review-card.featured {
    background: var(--deep); border-color: var(--deep);
  }

  .review-card.featured .review-text { color: rgba(255,255,255,0.8); }
  .review-card.featured .reviewer-name { color: var(--white); }
  .review-card.featured .reviewer-meta { color: rgba(255,255,255,0.5); }

  .stars { display: flex; gap: 3px; margin-bottom: 1.2rem; }
  .star { color: #F5A623; font-size: 0.9rem; }
  .star.dim { color: #E0C9A0; }

  .review-card.featured .star { color: var(--blush); }

  .review-text { font-size: 0.93rem; color: var(--muted); line-height: 1.7; margin-bottom: 1.4rem; }

  .reviewer { display: flex; align-items: center; gap: 10px; }

  .reviewer-avatar {
    width: 40px; height: 40px; border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    font-weight: 600; font-size: 0.9rem; flex-shrink: 0;
  }

  .reviewer-name { font-size: 0.9rem; font-weight: 600; color: var(--deep); }
  .reviewer-meta { font-size: 0.78rem; color: var(--muted); }

  .review-quote {
    font-family: 'DM Serif Display', serif;
    font-size: 3rem; line-height: 0.5; color: var(--blush);
    margin-bottom: 0.8rem; display: block;
  }

  .review-card.featured .review-quote { color: rgba(212,168,154,0.5); }

  /* ── FOOTER ── */
  footer {
    background: var(--deep); padding: 4rem 2rem 2rem;
    color: rgba(255,255,255,0.6);
  }

  .footer-inner {
    max-width: 1100px; margin: 0 auto;
    display: grid; grid-template-columns: 2fr 1fr 1fr 1fr; gap: 3rem;
    padding-bottom: 3rem; border-bottom: 1px solid rgba(255,255,255,0.08);
  }

  .footer-brand-name {
    font-family: 'DM Serif Display', serif; font-size: 1.5rem;
    color: var(--white); margin-bottom: 0.8rem;
  }

  .footer-brand-name span { color: var(--blush); }
  .footer-tagline { font-size: 0.85rem; line-height: 1.6; max-width: 260px; }

  .footer-col h5 { font-size: 0.82rem; font-weight: 600; letter-spacing: 0.08em; text-transform: uppercase; color: var(--white); margin-bottom: 1rem; }
  .footer-col ul { list-style: none; display: flex; flex-direction: column; gap: 8px; }
  .footer-col a { font-size: 0.85rem; color: rgba(255,255,255,0.55); text-decoration: none; transition: color 0.2s; }
  .footer-col a:hover { color: var(--blush); }

  .footer-bottom {
    max-width: 1100px; margin: 0 auto; padding-top: 1.8rem;
    display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 1rem;
    font-size: 0.82rem;
  }

  /* ── RESPONSIVE ── */
  @media (max-width: 900px) {
    .nav-links { display: none; }
    .nav-cta .btn-ghost { display: none; }
    .hamburger { display: flex; }

    .upload-inner, .auth-inner { grid-template-columns: 1fr; gap: 2.5rem; }
    .steps-grid { grid-template-columns: 1fr; }
    .reviews-grid { grid-template-columns: 1fr; }
    .footer-inner { grid-template-columns: 1fr 1fr; gap: 2rem; }

    .hero-stats { gap: 2rem; }
    h1 { font-size: 2.5rem; }
  }

  @media (max-width: 580px) {
    nav { padding: 1rem 1.2rem; }
    .hero { padding: 7rem 1.2rem 4rem; }
    .upload-section, .how-section, .auth-section, .reviews-section { padding: 4rem 1.2rem; }
    .footer-inner { grid-template-columns: 1fr; }
    .hero-stats { gap: 1.5rem; }
    .stat-num { font-size: 1.7rem; }
    .btn-hero { padding: 0.75rem 1.7rem; font-size: 0.9rem; }
    footer { padding: 3rem 1.2rem 1.5rem; }
  }
</style>
</head>
<body>

<!-- NAV -->
<nav>
  <a href="#" class="nav-logo">Derma<span>AI</span></a>
  <ul class="nav-links">
    <li><a href="#analyse">Analyse</a></li>
    <li><a href="#how">How it works</a></li>
    <li><a href="#reviews">Reviews</a></li>
  </ul>
  <div class="nav-cta">
    <a href="#auth" class="btn-ghost">Sign in</a>
    <a href="#auth" class="btn-primary">Get started</a>
    <button class="hamburger" onclick="toggleMenu()" aria-label="Menu">
      <span></span><span></span><span></span>
    </button>
  </div>
</nav>

<!-- MOBILE MENU -->
<div class="mobile-menu" id="mobileMenu">
  <button class="close-btn" onclick="toggleMenu()" aria-label="Close">&times;</button>
  <a href="#analyse" onclick="toggleMenu()">Analyse</a>
  <a href="#how" onclick="toggleMenu()">How it works</a>
  <a href="#reviews" onclick="toggleMenu()">Reviews</a>
  <a href="#auth" onclick="toggleMenu()">Sign in</a>
  <a href="#auth" onclick="toggleMenu()" style="color:var(--rose)">Get started →</a>
</div>

<!-- HERO -->
<section class="hero">
  <!-- botanical line art background -->
  <svg class="hero-bg-svg" viewBox="0 0 1400 900" preserveAspectRatio="xMidYMid slice" xmlns="http://www.w3.org/2000/svg" fill="none" stroke="#2C1A14" stroke-width="1">
    <!-- large leaf left -->
    <path d="M-40 800 C80 600 200 500 140 200 C120 300 80 450 -40 800Z"/>
    <path d="M-40 800 C60 650 160 520 140 200"/>
    <line x1="140" y1="200" x2="100" y2="400"/><line x1="100" y1="400" x2="60" y2="350"/><line x1="100" y1="400" x2="130" y2="360"/>
    <line x1="120" y1="300" x2="80" y2="260"/><line x1="120" y1="300" x2="150" y2="260"/>
    <!-- branch top right -->
    <path d="M1200 -20 C1180 100 1220 250 1160 400"/>
    <path d="M1160 400 C1100 350 1050 300 1000 320"/>
    <path d="M1160 400 C1200 430 1260 420 1300 390"/>
    <ellipse cx="1000" cy="305" rx="40" ry="22" transform="rotate(-25 1000 305)"/>
    <ellipse cx="1300" cy="378" rx="38" ry="20" transform="rotate(15 1300 378)"/>
    <ellipse cx="1240" cy="180" rx="32" ry="18" transform="rotate(-10 1240 180)"/>
    <!-- stem center top -->
    <path d="M700 -30 C710 80 690 180 720 300"/>
    <path d="M720 300 C680 280 650 250 620 270"/>
    <path d="M720 300 C755 285 780 255 810 270"/>
    <ellipse cx="618" cy="265" rx="36" ry="18" transform="rotate(-30 618 265)"/>
    <ellipse cx="812" cy="265" rx="36" ry="18" transform="rotate(30 812 265)"/>
    <!-- small sprigs bottom right -->
    <path d="M1350 700 C1300 650 1260 620 1220 580"/>
    <path d="M1220 580 C1200 540 1210 500 1190 470"/>
    <ellipse cx="1188" cy="462" rx="28" ry="14" transform="rotate(-40 1188 462)"/>
    <ellipse cx="1260" cy="615" rx="30" ry="14" transform="rotate(10 1260 615)"/>
    <!-- circle accents -->
    <circle cx="900" cy="750" r="60" stroke-dasharray="4 8"/>
    <circle cx="250" cy="150" r="44" stroke-dasharray="3 7"/>
    <circle cx="1100" cy="500" r="30" stroke-dasharray="3 6"/>
  </svg>
  <div class="hero-blob hero-blob-1"></div>
  <div class="hero-blob hero-blob-2"></div>

  <div class="hero-tag">AI-Powered Skin Analysis</div>

  <h1>Know your skin.<br>Deeply, <em>intelligently.</em></h1>

  <p class="hero-sub">Upload a photo or take one live — DermaAI analyses your skin in seconds, identifying conditions and recommending personalised care routines.</p>

  <div class="hero-actions">
    <button onclick="openCamera()" class="btn-hero btn-hero-primary" style="border:none;cursor:pointer;">
      📸 Analyse my skin
    </button>
    <button onclick="openHowModal()" class="btn-hero btn-hero-outline" style="border:1px solid var(--border);cursor:pointer;">
      ▶ See how it works
    </button>
  </div>

  <div class="hero-stats">
    <div class="stat"><div class="stat-num">98.4%</div><div class="stat-label">Analysis accuracy</div></div>
    <div class="stat"><div class="stat-num">240K+</div><div class="stat-label">Scans completed</div></div>
    <div class="stat"><div class="stat-num">4.9★</div><div class="stat-label">Average rating</div></div>
  </div>
</section>

<!-- UPLOAD / SCAN SECTION -->
<section class="upload-section" id="analyse">
  <!-- diagonal line grid -->
  <div class="upload-bg-lines">
    <svg width="100%" height="100%" xmlns="http://www.w3.org/2000/svg" style="position:absolute;inset:0;width:100%;height:100%">
      <defs>
        <pattern id="diag" width="40" height="40" patternUnits="userSpaceOnUse" patternTransform="rotate(45)">
          <line x1="0" y1="0" x2="0" y2="40" stroke="white" stroke-width="0.8"/>
        </pattern>
      </defs>
      <rect width="100%" height="100%" fill="url(#diag)"/>
    </svg>
  </div>
  <div class="upload-inner">
    <div class="upload-text">
      <span class="section-label">Skin Scanner</span>
      <h2 class="section-title" style="color:var(--white)">Snap or upload.<br>Results in seconds.</h2>
      <p class="section-sub">Our model is trained on over 8 million dermatological images across 60+ skin conditions. Clinical-grade analysis, right in your browser.</p>

      <div class="upload-features">
        <div class="feature-row">
          <div class="feat-icon">🔬</div>
          <div class="feat-text">
            <h4>60+ conditions detected</h4>
            <p>Acne, eczema, rosacea, hyperpigmentation and more — all identified instantly.</p>
          </div>
        </div>
        <div class="feature-row">
          <div class="feat-icon">🛡️</div>
          <div class="feat-text">
            <h4>Privacy-first processing</h4>
            <p>Your images are never stored. Analysis runs securely and is deleted immediately after.</p>
          </div>
        </div>
        <div class="feature-row">
          <div class="feat-icon">💊</div>
          <div class="feat-text">
            <h4>Personalised recommendations</h4>
            <p>Ingredient suggestions and routine steps tailored to your unique skin profile.</p>
          </div>
        </div>
      </div>
    </div>

    <div class="upload-card">
      <!-- hidden file inputs -->
      <input type="file" id="galleryInput" accept="image/*" style="display:none" aria-label="Choose photo from gallery">
      <input type="file" id="cameraInput" accept="image/*" capture="environment" style="display:none" aria-label="Take photo with camera">

      <div class="upload-zone" id="uploadZone" onclick="document.getElementById('galleryInput').click()" ondragover="handleDragOver(event)" ondrop="handleDrop(event)">
        <span class="upload-icon" id="uploadIcon">🖼️</span>
        <h3 id="uploadTitle">Drop your photo here</h3>
        <p id="uploadSubtitle">or click to choose from gallery · JPG, PNG, HEIC up to 20MB</p>
      </div>

      <div class="upload-divider">or take a live photo</div>

      <button class="camera-btn" id="cameraBtn" onclick="openCamera()">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"/><circle cx="12" cy="13" r="4"/></svg>
        Open Camera
      </button>

      <div class="scan-tag">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
        Analysis is <span>100% private</span> — images deleted after scan
      </div>
    </div>
    </div><!-- /upload-inner -->
  </section><!-- close upload-section early so modal can be outside -->

  <!-- ── CAMERA MODAL ── -->
  <div id="cameraModal" style="display:none;position:fixed;inset:0;z-index:200;background:rgba(20,10,8,0.92);align-items:center;justify-content:center;flex-direction:column;gap:16px;">
    <div style="width:min(520px,96vw);background:#1a0e0a;border-radius:24px;overflow:hidden;border:1px solid rgba(255,255,255,0.08);box-shadow:0 32px 80px rgba(0,0,0,0.5);">
      <div style="padding:1rem 1.4rem;border-bottom:1px solid rgba(255,255,255,0.07);display:flex;align-items:center;justify-content:space-between;">
        <span style="font-family:'DM Serif Display',serif;font-size:1.1rem;color:#fff;">Camera</span>
        <button onclick="closeCamera()" style="background:rgba(255,255,255,0.08);border:none;color:#fff;width:32px;height:32px;border-radius:50%;cursor:pointer;font-size:1.1rem;display:flex;align-items:center;justify-content:center;">&times;</button>
      </div>
      <div style="position:relative;background:#000;aspect-ratio:4/3;">
        <video id="cameraVideo" autoplay playsinline muted style="width:100%;height:100%;object-fit:cover;display:block;"></video>
        <div id="camPermDenied" style="display:none;position:absolute;inset:0;display:none;align-items:center;justify-content:center;flex-direction:column;gap:12px;padding:2rem;text-align:center;">
          <span style="font-size:2.5rem;">📷</span>
          <p style="color:#fff;font-size:0.95rem;line-height:1.6;">Camera access was denied.<br>Please allow camera permission in your browser settings and try again.</p>
        </div>
        <!-- scan line animation -->
        <div id="scanLine" style="position:absolute;left:0;right:0;height:2px;background:linear-gradient(90deg,transparent,rgba(212,168,154,0.8),transparent);animation:scanAnim 2.5s ease-in-out infinite;pointer-events:none;"></div>
        <!-- corner guides -->
        <div style="position:absolute;top:16px;left:16px;width:28px;height:28px;border-top:2px solid rgba(212,168,154,0.7);border-left:2px solid rgba(212,168,154,0.7);border-radius:3px 0 0 0;pointer-events:none;"></div>
        <div style="position:absolute;top:16px;right:16px;width:28px;height:28px;border-top:2px solid rgba(212,168,154,0.7);border-right:2px solid rgba(212,168,154,0.7);border-radius:0 3px 0 0;pointer-events:none;"></div>
        <div style="position:absolute;bottom:16px;left:16px;width:28px;height:28px;border-bottom:2px solid rgba(212,168,154,0.7);border-left:2px solid rgba(212,168,154,0.7);border-radius:0 0 0 3px;pointer-events:none;"></div>
        <div style="position:absolute;bottom:16px;right:16px;width:28px;height:28px;border-bottom:2px solid rgba(212,168,154,0.7);border-right:2px solid rgba(212,168,154,0.7);border-radius:0 0 3px 0;pointer-events:none;"></div>
      </div>
      <canvas id="snapCanvas" style="display:none;"></canvas>
      <div style="padding:1.2rem 1.4rem;display:flex;gap:10px;justify-content:center;align-items:center;">
        <button onclick="flipCamera()" id="flipBtn" style="padding:0.6rem 1.1rem;border-radius:999px;border:1px solid rgba(255,255,255,0.15);background:rgba(255,255,255,0.06);color:#fff;font-family:'DM Sans',sans-serif;font-size:0.85rem;cursor:pointer;">
          🔄 Flip
        </button>
        <button onclick="snapPhoto()" style="width:60px;height:60px;border-radius:50%;border:3px solid #D4A89A;background:rgba(181,112,94,0.25);cursor:pointer;display:flex;align-items:center;justify-content:center;transition:all 0.2s;" onmouseover="this.style.background='rgba(181,112,94,0.5)'" onmouseout="this.style.background='rgba(181,112,94,0.25)'">
          <div style="width:42px;height:42px;border-radius:50%;background:#D4A89A;"></div>
        </button>
        <button onclick="switchToGallery()" style="padding:0.6rem 1.1rem;border-radius:999px;border:1px solid rgba(255,255,255,0.15);background:rgba(255,255,255,0.06);color:#fff;font-family:'DM Sans',sans-serif;font-size:0.85rem;cursor:pointer;">
          🖼 Gallery
        </button>
      </div>
    </div>
  </div>

  <!-- ── IMAGE PREVIEW MODAL ── -->
  <div id="previewModal" style="display:none;position:fixed;inset:0;z-index:201;background:rgba(20,10,8,0.92);align-items:center;justify-content:center;flex-direction:column;gap:16px;">
    <div style="width:min(500px,94vw);background:#fff;border-radius:24px;overflow:hidden;box-shadow:0 32px 80px rgba(0,0,0,0.5);">
      <div style="padding:1rem 1.4rem;border-bottom:1px solid #f0e8e0;display:flex;align-items:center;justify-content:space-between;">
        <span style="font-family:'DM Serif Display',serif;font-size:1.1rem;color:#2C1A14;">Review your photo</span>
        <button onclick="closePreview()" style="background:#f5ede8;border:none;color:#2C1A14;width:32px;height:32px;border-radius:50%;cursor:pointer;font-size:1.1rem;">&times;</button>
      </div>
      <div style="position:relative;">
        <img id="previewImg" src="" alt="Skin photo preview" style="width:100%;max-height:340px;object-fit:contain;display:block;background:#f9f5f0;">
        <div id="analysingOverlay" style="display:none;position:absolute;inset:0;background:rgba(44,26,20,0.6);align-items:center;justify-content:center;flex-direction:column;gap:10px;">
          <div style="width:48px;height:48px;border:3px solid rgba(255,255,255,0.2);border-top-color:#D4A89A;border-radius:50%;animation:spin 0.8s linear infinite;"></div>
          <p style="color:#fff;font-size:0.9rem;font-weight:500;">Analysing skin...</p>
        </div>
      </div>
      <div style="padding:1.2rem 1.4rem;display:flex;gap:10px;">
        <button onclick="closePreview()" style="flex:1;padding:0.8rem;border-radius:12px;border:1px solid #e0d4cc;background:transparent;font-family:'DM Sans',sans-serif;font-size:0.9rem;color:#7A6860;cursor:pointer;">Retake</button>
        <button onclick="analysePhoto()" style="flex:2;padding:0.8rem;border-radius:12px;border:none;background:#2C1A14;font-family:'DM Sans',sans-serif;font-size:0.9rem;font-weight:600;color:#fff;cursor:pointer;transition:background 0.2s;" onmouseover="this.style.background='#B5705E'" onmouseout="this.style.background='#2C1A14'">✨ Analyse my skin</button>
      </div>
    </div>
  </div>

  <!-- ── RESULT TOAST ── -->
  <div id="resultToast" style="display:none;position:fixed;bottom:2rem;left:50%;transform:translateX(-50%);z-index:202;background:#2C1A14;color:#fff;padding:1rem 1.8rem;border-radius:999px;font-family:'DM Sans',sans-serif;font-size:0.9rem;font-weight:500;box-shadow:0 8px 32px rgba(44,26,20,0.35);white-space:nowrap;">
    ✅ Photo received — analysis complete!
  </div>

  <!-- Reopen closed sections -->
  <section style="display:none">
  </div>
</section>

<!-- HOW IT WORKS -->
<section class="how-section" id="how">
  <div class="how-bg"></div>
  <div class="how-inner">
    <div class="how-header">
      <span class="section-label">The Process</span>
      <h2 class="section-title">Three steps to clarity</h2>
      <p class="section-sub" style="margin:0 auto">From permission to personalised report — the whole process takes under 30 seconds.</p>
    </div>

    <div class="steps-grid">

      <!-- Step 1 -->
      <div class="step-card" onclick="openCamera()" style="cursor:pointer;">
        <div class="step-num">01</div>
        <div class="step-icon">📷</div>
        <h3>Open camera &amp; grant access</h3>
        <p>Tap this card or the "Analyse my skin" button. Your browser will ask for camera permission — simply allow it. DermaAI never stores your video feed.</p>
        <div style="margin-top:1.2rem;">
          <button onclick="event.stopPropagation();openCamera()" style="padding:0.55rem 1.2rem;border-radius:999px;border:none;background:var(--deep);color:#fff;font-family:'DM Sans',sans-serif;font-size:0.82rem;font-weight:600;cursor:pointer;transition:background 0.2s;" onmouseover="this.style.background='var(--rose)'" onmouseout="this.style.background='var(--deep)'">
            Open camera →
          </button>
        </div>
      </div>

      <!-- Step 2 -->
      <div class="step-card" style="cursor:default;">
        <div class="step-num">02</div>
        <div class="step-icon">🎯</div>
        <h3>Point at the area of concern</h3>
        <p>Hold your camera steady 20–30 cm from the affected area — a patch of acne, rash, or discolouration. Use the on-screen guides to centre the zone, then tap the shutter.</p>
        <div style="margin-top:1.2rem;display:flex;align-items:center;gap:8px;">
          <div style="display:flex;gap:5px;">
            <span style="display:inline-block;width:8px;height:8px;border-radius:50%;background:var(--rose);opacity:0.9;"></span>
            <span style="display:inline-block;width:8px;height:8px;border-radius:50%;background:var(--rose);opacity:0.5;"></span>
            <span style="display:inline-block;width:8px;height:8px;border-radius:50%;background:var(--rose);opacity:0.2;"></span>
          </div>
          <span style="font-size:0.78rem;color:var(--muted);">Good lighting makes results more accurate</span>
        </div>
      </div>

      <!-- Step 3 -->
      <div class="step-card" style="cursor:default;">
        <div class="step-num">03</div>
        <div class="step-icon">✨</div>
        <h3>Upload &amp; see your result</h3>
        <p>After snapping, review your photo and tap "Analyse my skin". Results appear in seconds — conditions identified, severity rated, and a personalised care routine generated.</p>
        <div style="margin-top:1.2rem;display:flex;gap:6px;flex-wrap:wrap;">
          <span style="padding:3px 10px;border-radius:999px;background:rgba(181,112,94,0.1);font-size:0.75rem;color:var(--rose);font-weight:500;">Condition ID</span>
          <span style="padding:3px 10px;border-radius:999px;background:rgba(181,112,94,0.1);font-size:0.75rem;color:var(--rose);font-weight:500;">Severity score</span>
          <span style="padding:3px 10px;border-radius:999px;background:rgba(181,112,94,0.1);font-size:0.75rem;color:var(--rose);font-weight:500;">Routine tips</span>
        </div>
      </div>

    </div>

    <!-- connector line between steps (desktop only) -->
    <div style="display:flex;justify-content:center;margin-top:2.5rem;gap:0;">
      <div style="height:2px;width:200px;background:linear-gradient(90deg,transparent,var(--blush),var(--blush),var(--blush),transparent);border-radius:99px;opacity:0.5;"></div>
    </div>

    <div style="text-align:center;margin-top:2rem;">
      <button onclick="openCamera()" style="padding:0.85rem 2.4rem;border-radius:999px;border:none;background:var(--deep);color:#fff;font-family:'DM Sans',sans-serif;font-size:1rem;font-weight:600;cursor:pointer;box-shadow:0 8px 28px rgba(44,26,20,0.18);transition:all 0.22s;" onmouseover="this.style.background='var(--rose)';this.style.transform='translateY(-2px)'" onmouseout="this.style.background='var(--deep)';this.style.transform='translateY(0)'">
        📸 Start your scan now
      </button>
    </div>

  </div>
</section>

<!-- AUTH SECTION -->
<section class="auth-section" id="auth">
  <div class="auth-bg">
    <div class="auth-bg-shape" style="width:320px;height:320px;top:-80px;right:-60px;"></div>
    <div class="auth-bg-shape" style="width:200px;height:200px;top:40px;right:80px;"></div>
    <div class="auth-bg-shape" style="width:500px;height:500px;bottom:-200px;left:-150px;"></div>
    <div class="auth-bg-shape" style="width:280px;height:280px;bottom:-60px;left:60px;"></div>
    <svg style="position:absolute;bottom:0;left:0;right:0;width:100%;opacity:0.06" viewBox="0 0 1400 120" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M0 60 C200 0 400 120 600 60 C800 0 1000 120 1200 60 C1300 30 1360 50 1400 40 L1400 120 L0 120Z" fill="#B5705E"/>
    </svg>
  </div>
  <div class="auth-inner">
    <div class="auth-promo">
      <span class="section-label">Your Account</span>
      <h2 class="section-title">Track your skin<br>over time</h2>
      <p class="section-sub">Create a free account to save your scan history, monitor improvements, and receive monthly skin health reports.</p>

      <div class="promo-list">
        <div class="promo-item"><div class="promo-check">✓</div> Save unlimited skin scans</div>
        <div class="promo-item"><div class="promo-check">✓</div> Track improvements over weeks & months</div>
        <div class="promo-item"><div class="promo-check">✓</div> Personalised product recommendations</div>
        <div class="promo-item"><div class="promo-check">✓</div> Monthly skin health digest</div>
        <div class="promo-item"><div class="promo-check">✓</div> Free forever — no credit card required</div>
      </div>
    </div>

    <div class="auth-card">
      <div class="auth-tabs">
        <button class="auth-tab active" onclick="setTab(this,'sign-up')">Create account</button>
        <button class="auth-tab" onclick="setTab(this,'sign-in')">Sign in</button>
      </div>

      <!-- Social logins -->
      <div class="social-btns">
        <button class="social-btn">
          <!-- Google -->
          <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z" fill="#4285F4"/><path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z" fill="#34A853"/><path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z" fill="#FBBC05"/><path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z" fill="#EA4335"/></svg>
          Continue with Google
        </button>
        <button class="social-btn">
          <!-- Apple -->
          <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" fill="currentColor"><path d="M18.71 19.5c-.83 1.24-1.71 2.45-3.05 2.47-1.34.03-1.77-.79-3.29-.79-1.53 0-2 .77-3.27.82-1.31.05-2.3-1.32-3.14-2.53C4.25 17 2.94 12.45 4.7 9.39c.87-1.52 2.43-2.48 4.12-2.51 1.28-.02 2.5.87 3.29.87.78 0 2.26-1.07 3.8-.91.65.03 2.47.26 3.64 1.98-.09.06-2.17 1.28-2.15 3.81.03 3.02 2.65 4.03 2.68 4.04-.03.07-.42 1.44-1.38 2.83M13 3.5c.73-.83 1.94-1.46 2.94-1.5.13 1.17-.34 2.35-1.04 3.19-.69.85-1.83 1.51-2.95 1.42-.15-1.15.41-2.35 1.05-3.11z"/></svg>
          Continue with Apple
        </button>
        <button class="social-btn">
          <!-- Microsoft -->
          <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><rect x="1" y="1" width="10" height="10" fill="#F25022"/><rect x="13" y="1" width="10" height="10" fill="#7FBA00"/><rect x="1" y="13" width="10" height="10" fill="#00A4EF"/><rect x="13" y="13" width="10" height="10" fill="#FFB900"/></svg>
          Continue with Microsoft
        </button>
      </div>

      <div class="or-divider">or use email</div>

      <div class="input-group" id="nameField">
        <input class="auth-input" type="text" placeholder="Full name">
      </div>
      <div class="input-group">
        <input class="auth-input" type="email" placeholder="Email address">
        <input class="auth-input" type="password" placeholder="Password">
      </div>

      <button class="btn-auth" id="authMainBtn">Create free account</button>

      <div class="auth-footer">
        By continuing you agree to our <a href="#">Terms</a> and <a href="#">Privacy Policy</a>
      </div>
    </div>
  </div>
</section>

<!-- REVIEWS -->
<section class="reviews-section" id="reviews">
  <div class="reviews-bg"></div>
  <!-- large decorative arc top -->
  <svg style="position:absolute;top:0;left:0;right:0;width:100%;pointer-events:none;opacity:0.06" viewBox="0 0 1400 200" preserveAspectRatio="none" xmlns="http://www.w3.org/2000/svg">
    <path d="M-100 200 C300 -60 700 -60 1100 60 C1250 100 1350 80 1500 40 L1500 0 L-100 0Z" fill="#B5705E"/>
  </svg>
  <div class="reviews-inner">
    <div class="reviews-header">
      <span class="section-label">Client Reviews</span>
      <h2 class="section-title">Loved by thousands</h2>
      <p class="section-sub" style="margin:0 auto">Real people, real results. Here's what our community has to say about DermaAI.</p>
    </div>

    <div class="reviews-grid">
      <div class="review-card featured">
        <span class="review-quote">"</span>
        <div class="stars">
          <span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span>
        </div>
        <p class="review-text">DermaAI identified my rosacea triggers before my dermatologist did. The ingredient recommendations have completely transformed my routine — my skin is the clearest it's been in years.</p>
        <div class="reviewer">
          <div class="reviewer-avatar" style="background:rgba(212,168,154,0.25);color:var(--blush)">SA</div>
          <div>
            <div class="reviewer-name" style="color:white">Shreya Agarwal</div>
            <div class="reviewer-meta">Mumbai · Using DermaAI for 8 months</div>
          </div>
        </div>
      </div>

      <div class="review-card">
        <span class="review-quote" style="color:var(--blush)">"</span>
        <div class="stars">
          <span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span>
        </div>
        <p class="review-text">I was skeptical at first, but the analysis was shockingly accurate. It caught early signs of eczema I'd been dismissing for months. Saved me an expensive dermatologist visit.</p>
        <div class="reviewer">
          <div class="reviewer-avatar" style="background:#EDE6DC;color:var(--rose)">RK</div>
          <div>
            <div class="reviewer-name">Rahul Kapoor</div>
            <div class="reviewer-meta">Bangalore · 3 months</div>
          </div>
        </div>
      </div>

      <div class="review-card">
        <span class="review-quote" style="color:var(--blush)">"</span>
        <div class="stars">
          <span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span>
        </div>
        <p class="review-text">As someone with combination skin, I've never found a routine that works. DermaAI finally broke it down zone by zone. My T-zone is under control for the first time in a decade.</p>
        <div class="reviewer">
          <div class="reviewer-avatar" style="background:#EDE6DC;color:var(--rose)">PM</div>
          <div>
            <div class="reviewer-name">Priya Mehta</div>
            <div class="reviewer-meta">Delhi · 5 months</div>
          </div>
        </div>
      </div>

      <div class="review-card">
        <span class="review-quote" style="color:var(--blush)">"</span>
        <div class="stars">
          <span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span>
        </div>
        <p class="review-text">The tracking feature is what sets this apart. Watching my hyperpigmentation fade week by week with photo comparisons is incredibly motivating. I actually stick to my routine now!</p>
        <div class="reviewer">
          <div class="reviewer-avatar" style="background:#EDE6DC;color:var(--rose)">AJ</div>
          <div>
            <div class="reviewer-name">Ananya Joshi</div>
            <div class="reviewer-meta">Pune · 6 months</div>
          </div>
        </div>
      </div>

      <div class="review-card">
        <span class="review-quote" style="color:var(--blush)">"</span>
        <div class="stars">
          <span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star dim">★</span>
        </div>
        <p class="review-text">Quick, accurate, and the privacy-first approach is reassuring. I've used other skin apps that felt intrusive — DermaAI feels trustworthy. Recommended it to my whole family.</p>
        <div class="reviewer">
          <div class="reviewer-avatar" style="background:#EDE6DC;color:var(--rose)">VS</div>
          <div>
            <div class="reviewer-name">Vikram Singh</div>
            <div class="reviewer-meta">Ahmedabad · 2 months</div>
          </div>
        </div>
      </div>

      <div class="review-card">
        <span class="review-quote" style="color:var(--blush)">"</span>
        <div class="stars">
          <span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span><span class="star">★</span>
        </div>
        <p class="review-text">I'm a skincare blogger and I've tested dozens of tools. DermaAI's ingredient analysis is genuinely the most sophisticated I've seen. The niacinamide + retinol conflict detection alone is worth it.</p>
        <div class="reviewer">
          <div class="reviewer-avatar" style="background:#EDE6DC;color:var(--rose)">KN</div>
          <div>
            <div class="reviewer-name">Kavya Nair</div>
            <div class="reviewer-meta">Chennai · 10 months</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- FOOTER -->
<footer>
  <div class="footer-inner">
    <div>
      <div class="footer-brand-name">Derma<span>AI</span></div>
      <p class="footer-tagline">AI-powered skin analysis that helps you understand and care for your skin with clinical-grade accuracy.</p>
    </div>
    <div class="footer-col">
      <h5>Product</h5>
      <ul>
        <li><a href="#">Skin scanner</a></li>
        <li><a href="#">Ingredient checker</a></li>
        <li><a href="#">Routine builder</a></li>
        <li><a href="#">Progress tracker</a></li>
      </ul>
    </div>
    <div class="footer-col">
      <h5>Company</h5>
      <ul>
        <li><a href="#">About us</a></li>
        <li><a href="#">Research</a></li>
        <li><a href="#">Blog</a></li>
        <li><a href="#">Careers</a></li>
      </ul>
    </div>
    <div class="footer-col">
      <h5>Legal</h5>
      <ul>
        <li><a href="#">Privacy policy</a></li>
        <li><a href="#">Terms of service</a></li>
        <li><a href="#">Cookie policy</a></li>
        <li><a href="#">Contact</a></li>
      </ul>
    </div>
  </div>
  <div class="footer-bottom">
    <span>© 2025 DermaAI. All rights reserved.</span>
    <span>Made with care for your skin.</span>
  </div>
</footer>

<!-- ── HOW IT WORKS WALKTHROUGH MODAL ── -->
<div id="howModal" style="display:none;position:fixed;inset:0;z-index:210;background:rgba(20,10,8,0.94);align-items:center;justify-content:center;">
  <div style="width:min(620px,96vw);background:#fff;border-radius:28px;overflow:hidden;box-shadow:0 40px 100px rgba(0,0,0,0.5);max-height:92vh;overflow-y:auto;">

    <!-- header -->
    <div style="background:var(--deep);padding:1.4rem 1.8rem;display:flex;align-items:center;justify-content:space-between;">
      <div>
        <div style="font-size:0.72rem;letter-spacing:0.1em;text-transform:uppercase;color:var(--blush);font-weight:600;margin-bottom:3px;">How it works</div>
        <div style="font-family:'DM Serif Display',serif;font-size:1.3rem;color:#fff;">Your 3-step skin scan</div>
      </div>
      <button onclick="closeHowModal()" style="background:rgba(255,255,255,0.1);border:none;color:#fff;width:36px;height:36px;border-radius:50%;cursor:pointer;font-size:1.2rem;display:flex;align-items:center;justify-content:center;">&times;</button>
    </div>

    <!-- progress bar -->
    <div style="height:3px;background:#f0e8e0;">
      <div id="howProgress" style="height:100%;background:var(--rose);transition:width 0.4s ease;width:33%;"></div>
    </div>

    <!-- steps container -->
    <div id="howSteps" style="position:relative;overflow:hidden;">

      <!-- STEP 1 -->
      <div class="how-step" id="howStep1" style="padding:2rem 2rem 1.6rem;">
        <div style="display:flex;align-items:center;gap:14px;margin-bottom:1.4rem;">
          <div style="width:52px;height:52px;border-radius:16px;background:rgba(181,112,94,0.12);display:flex;align-items:center;justify-content:center;font-size:1.6rem;flex-shrink:0;">📷</div>
          <div>
            <div style="font-size:0.72rem;font-weight:600;letter-spacing:0.08em;text-transform:uppercase;color:var(--rose);margin-bottom:2px;">Step 1 of 3</div>
            <div style="font-family:'DM Serif Display',serif;font-size:1.3rem;color:var(--deep);">Open camera &amp; grant access</div>
          </div>
        </div>

        <!-- visual explainer -->
        <div style="background:var(--deep);border-radius:18px;padding:1.4rem;margin-bottom:1.4rem;position:relative;overflow:hidden;">
          <div style="position:absolute;top:-20px;right:-20px;width:100px;height:100px;border-radius:50%;background:rgba(212,168,154,0.08);"></div>
          <!-- mock browser permission prompt -->
          <div style="background:rgba(255,255,255,0.06);border:1px solid rgba(255,255,255,0.1);border-radius:14px;padding:1rem 1.2rem;display:flex;align-items:flex-start;gap:12px;">
            <div style="width:36px;height:36px;border-radius:10px;background:rgba(212,168,154,0.15);display:flex;align-items:center;justify-content:center;font-size:1.1rem;flex-shrink:0;">📷</div>
            <div style="flex:1;">
              <div style="font-size:0.85rem;font-weight:600;color:#fff;margin-bottom:4px;">DermaAI wants to use your camera</div>
              <div style="font-size:0.78rem;color:rgba(255,255,255,0.5);line-height:1.5;margin-bottom:10px;">This lets you take a live photo for skin analysis. Your feed is never recorded or stored.</div>
              <div style="display:flex;gap:8px;">
                <div style="padding:6px 16px;border-radius:8px;background:var(--rose);color:#fff;font-size:0.8rem;font-weight:600;">Allow</div>
                <div style="padding:6px 16px;border-radius:8px;background:rgba(255,255,255,0.1);color:rgba(255,255,255,0.6);font-size:0.8rem;">Block</div>
              </div>
            </div>
          </div>
          <!-- animated arrow pointing to Allow -->
          <div style="text-align:right;margin-top:8px;">
            <span style="font-size:0.75rem;color:var(--blush);font-style:italic;">← tap Allow to continue</span>
          </div>
        </div>

        <div style="display:flex;flex-direction:column;gap:10px;margin-bottom:1.6rem;">
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">1</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">Click <strong style="color:var(--deep);">"Analyse my skin"</strong> from the homepage or the button below.</p>
          </div>
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">2</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">Your browser shows a camera permission prompt. Tap <strong style="color:var(--deep);">Allow</strong>.</p>
          </div>
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">3</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">Your live camera feed opens in a secure modal. No video is ever saved.</p>
          </div>
        </div>

        <div style="display:flex;justify-content:space-between;align-items:center;padding-top:1rem;border-top:1px solid #f0e8e0;">
          <span style="font-size:0.8rem;color:var(--muted);">1 of 3</span>
          <div style="display:flex;gap:10px;">
            <button onclick="closeHowModal();openCamera();" style="padding:0.6rem 1.4rem;border-radius:999px;border:1px solid var(--border);background:transparent;font-family:'DM Sans',sans-serif;font-size:0.88rem;color:var(--deep);cursor:pointer;">Try it now</button>
            <button onclick="howNext(2)" style="padding:0.6rem 1.4rem;border-radius:999px;border:none;background:var(--deep);font-family:'DM Sans',sans-serif;font-size:0.88rem;font-weight:600;color:#fff;cursor:pointer;transition:background 0.2s;" onmouseover="this.style.background='var(--rose)'" onmouseout="this.style.background='var(--deep)'">Next step →</button>
          </div>
        </div>
      </div>

      <!-- STEP 2 -->
      <div class="how-step" id="howStep2" style="display:none;padding:2rem 2rem 1.6rem;">
        <div style="display:flex;align-items:center;gap:14px;margin-bottom:1.4rem;">
          <div style="width:52px;height:52px;border-radius:16px;background:rgba(181,112,94,0.12);display:flex;align-items:center;justify-content:center;font-size:1.6rem;flex-shrink:0;">🎯</div>
          <div>
            <div style="font-size:0.72rem;font-weight:600;letter-spacing:0.08em;text-transform:uppercase;color:var(--rose);margin-bottom:2px;">Step 2 of 3</div>
            <div style="font-family:'DM Serif Display',serif;font-size:1.3rem;color:var(--deep);">Point at the area of concern</div>
          </div>
        </div>

        <!-- mock camera viewfinder -->
        <div style="background:#1a0a06;border-radius:18px;overflow:hidden;margin-bottom:1.4rem;aspect-ratio:16/9;position:relative;display:flex;align-items:center;justify-content:center;">
          <div style="position:absolute;inset:0;background:linear-gradient(135deg,rgba(44,26,20,0.8),rgba(44,26,20,0.4));"></div>
          <!-- face outline suggestion -->
          <div style="position:relative;z-index:1;display:flex;flex-direction:column;align-items:center;gap:8px;">
            <div style="width:100px;height:120px;border:2px dashed rgba(212,168,154,0.6);border-radius:50%;display:flex;align-items:center;justify-content:center;">
              <span style="font-size:2.5rem;opacity:0.4;">🧑</span>
            </div>
            <span style="font-size:0.75rem;color:rgba(212,168,154,0.8);font-style:italic;">Centre affected area in frame</span>
          </div>
          <!-- corner guides -->
          <div style="position:absolute;top:12px;left:12px;width:22px;height:22px;border-top:2px solid rgba(212,168,154,0.8);border-left:2px solid rgba(212,168,154,0.8);border-radius:3px 0 0 0;"></div>
          <div style="position:absolute;top:12px;right:12px;width:22px;height:22px;border-top:2px solid rgba(212,168,154,0.8);border-right:2px solid rgba(212,168,154,0.8);border-radius:0 3px 0 0;"></div>
          <div style="position:absolute;bottom:12px;left:12px;width:22px;height:22px;border-bottom:2px solid rgba(212,168,154,0.8);border-left:2px solid rgba(212,168,154,0.8);border-radius:0 0 0 3px;"></div>
          <div style="position:absolute;bottom:12px;right:12px;width:22px;height:22px;border-bottom:2px solid rgba(212,168,154,0.8);border-right:2px solid rgba(212,168,154,0.8);border-radius:0 0 3px 0;"></div>
          <!-- shutter btn preview -->
          <div style="position:absolute;bottom:16px;left:50%;transform:translateX(-50%);width:44px;height:44px;border-radius:50%;border:2px solid rgba(212,168,154,0.7);background:rgba(212,168,154,0.2);display:flex;align-items:center;justify-content:center;">
            <div style="width:30px;height:30px;border-radius:50%;background:rgba(212,168,154,0.6);"></div>
          </div>
        </div>

        <div style="display:flex;flex-direction:column;gap:10px;margin-bottom:1.6rem;">
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">1</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">Hold your device <strong style="color:var(--deep);">20–30 cm</strong> from the affected skin area.</p>
          </div>
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">2</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">Centre the rash, acne patch, or discolouration inside the guide frame.</p>
          </div>
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">3</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">Ensure good lighting — natural daylight gives the most accurate results. Tap the shutter button.</p>
          </div>
        </div>

        <div style="display:flex;justify-content:space-between;align-items:center;padding-top:1rem;border-top:1px solid #f0e8e0;">
          <button onclick="howNext(1)" style="padding:0.6rem 1.4rem;border-radius:999px;border:1px solid var(--border);background:transparent;font-family:'DM Sans',sans-serif;font-size:0.88rem;color:var(--deep);cursor:pointer;">← Back</button>
          <div style="display:flex;gap:6px;"><div style="width:7px;height:7px;border-radius:50%;background:#ddd;"></div><div style="width:7px;height:7px;border-radius:50%;background:var(--rose);"></div><div style="width:7px;height:7px;border-radius:50%;background:#ddd;"></div></div>
          <button onclick="howNext(3)" style="padding:0.6rem 1.4rem;border-radius:999px;border:none;background:var(--deep);font-family:'DM Sans',sans-serif;font-size:0.88rem;font-weight:600;color:#fff;cursor:pointer;transition:background 0.2s;" onmouseover="this.style.background='var(--rose)'" onmouseout="this.style.background='var(--deep)'">Next step →</button>
        </div>
      </div>

      <!-- STEP 3 -->
      <div class="how-step" id="howStep3" style="display:none;padding:2rem 2rem 1.6rem;">
        <div style="display:flex;align-items:center;gap:14px;margin-bottom:1.4rem;">
          <div style="width:52px;height:52px;border-radius:16px;background:rgba(181,112,94,0.12);display:flex;align-items:center;justify-content:center;font-size:1.6rem;flex-shrink:0;">✨</div>
          <div>
            <div style="font-size:0.72rem;font-weight:600;letter-spacing:0.08em;text-transform:uppercase;color:var(--rose);margin-bottom:2px;">Step 3 of 3</div>
            <div style="font-family:'DM Serif Display',serif;font-size:1.3rem;color:var(--deep);">Upload &amp; see your result</div>
          </div>
        </div>

        <!-- mock result card -->
        <div style="border:1px solid #f0e8e0;border-radius:18px;overflow:hidden;margin-bottom:1.4rem;">
          <div style="background:var(--sand);padding:0.9rem 1.2rem;display:flex;align-items:center;gap:10px;">
            <div style="width:36px;height:36px;border-radius:10px;background:var(--rose);display:flex;align-items:center;justify-content:center;font-size:1rem;">🔬</div>
            <div>
              <div style="font-size:0.82rem;font-weight:600;color:var(--deep);">Analysis complete</div>
              <div style="font-size:0.72rem;color:var(--muted);">Processed in 4.2 seconds</div>
            </div>
            <div style="margin-left:auto;padding:4px 12px;border-radius:999px;background:rgba(99,153,34,0.12);font-size:0.75rem;font-weight:600;color:#3B6D11;">98% confidence</div>
          </div>
          <div style="padding:1rem 1.2rem;display:flex;flex-direction:column;gap:10px;">
            <div style="display:flex;justify-content:space-between;align-items:center;">
              <span style="font-size:0.85rem;color:var(--deep);font-weight:500;">Condition detected</span>
              <span style="padding:3px 12px;border-radius:999px;background:rgba(181,112,94,0.12);font-size:0.78rem;color:var(--rose);font-weight:600;">Mild acne</span>
            </div>
            <div style="display:flex;justify-content:space-between;align-items:center;">
              <span style="font-size:0.85rem;color:var(--deep);font-weight:500;">Severity</span>
              <div style="display:flex;gap:3px;">
                <div style="width:18px;height:6px;border-radius:99px;background:var(--rose);"></div>
                <div style="width:18px;height:6px;border-radius:99px;background:var(--rose);"></div>
                <div style="width:18px;height:6px;border-radius:99px;background:var(--blush);opacity:0.4;"></div>
                <div style="width:18px;height:6px;border-radius:99px;background:var(--blush);opacity:0.4;"></div>
                <div style="width:18px;height:6px;border-radius:99px;background:var(--blush);opacity:0.4;"></div>
              </div>
            </div>
            <div style="border-top:1px solid #f0e8e0;padding-top:10px;">
              <div style="font-size:0.82rem;font-weight:600;color:var(--deep);margin-bottom:6px;">Recommended routine</div>
              <div style="display:flex;flex-direction:column;gap:5px;">
                <div style="font-size:0.78rem;color:var(--muted);">🌅 AM: Gentle cleanser · Niacinamide 5% · SPF 50</div>
                <div style="font-size:0.78rem;color:var(--muted);">🌙 PM: BHA exfoliant · Azelaic acid · Moisturiser</div>
              </div>
            </div>
          </div>
        </div>

        <div style="display:flex;flex-direction:column;gap:10px;margin-bottom:1.6rem;">
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">1</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">After snapping, your photo appears in a <strong style="color:var(--deep);">review screen</strong>. Confirm it's clear and centred.</p>
          </div>
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">2</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">Tap <strong style="color:var(--deep);">"Analyse my skin"</strong> — our AI scans for 60+ conditions in under 10 seconds.</p>
          </div>
          <div style="display:flex;gap:10px;align-items:flex-start;">
            <div style="width:22px;height:22px;border-radius:50%;background:var(--rose);display:flex;align-items:center;justify-content:center;color:#fff;font-size:0.7rem;flex-shrink:0;margin-top:1px;">3</div>
            <p style="font-size:0.88rem;color:var(--muted);line-height:1.6;">Your personal skin report shows the detected condition, severity rating, and a tailored routine.</p>
          </div>
        </div>

        <div style="display:flex;justify-content:space-between;align-items:center;padding-top:1rem;border-top:1px solid #f0e8e0;">
          <button onclick="howNext(2)" style="padding:0.6rem 1.4rem;border-radius:999px;border:1px solid var(--border);background:transparent;font-family:'DM Sans',sans-serif;font-size:0.88rem;color:var(--deep);cursor:pointer;">← Back</button>
          <button onclick="closeHowModal();openCamera();" style="padding:0.7rem 2rem;border-radius:999px;border:none;background:var(--rose);font-family:'DM Sans',sans-serif;font-size:0.92rem;font-weight:600;color:#fff;cursor:pointer;box-shadow:0 6px 20px rgba(181,112,94,0.35);transition:all 0.22s;" onmouseover="this.style.background='var(--deep)'" onmouseout="this.style.background='var(--rose)'">
            📸 Start my scan
          </button>
        </div>
      </div>

    </div><!-- /howSteps -->
  </div>
</div>

<script>
  /* ── NAV ── */
  function toggleMenu() {
    document.getElementById('mobileMenu').classList.toggle('open');
  }

  function setTab(el, tab) {
    document.querySelectorAll('.auth-tab').forEach(t => t.classList.remove('active'));
    el.classList.add('active');
    const nameField = document.getElementById('nameField');
    const btn = document.getElementById('authMainBtn');
    if (tab === 'sign-in') {
      nameField.style.display = 'none';
      btn.textContent = 'Sign in';
    } else {
      nameField.style.display = 'flex';
      btn.textContent = 'Create free account';
    }
  }

  /* ── SCROLL FADE ── */
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(e => {
      if (e.isIntersecting) {
        e.target.style.opacity = '1';
        e.target.style.transform = 'translateY(0)';
      }
    });
  }, { threshold: 0.12 });
  document.querySelectorAll('.step-card, .review-card, .feature-row').forEach(el => {
    el.style.opacity = '0';
    el.style.transform = 'translateY(24px)';
    el.style.transition = 'opacity 0.55s ease, transform 0.55s ease';
    observer.observe(el);
  });

  /* ── GALLERY / FILE INPUT ── */
  const galleryInput = document.getElementById('galleryInput');
  const cameraInput  = document.getElementById('cameraInput');

  galleryInput.addEventListener('change', e => {
    const file = e.target.files[0];
    if (file) loadPreview(file);
    galleryInput.value = '';
  });

  cameraInput.addEventListener('change', e => {
    const file = e.target.files[0];
    if (file) loadPreview(file);
    cameraInput.value = '';
  });

  /* drag-and-drop */
  function handleDragOver(e) {
    e.preventDefault();
    document.getElementById('uploadZone').style.borderColor = 'var(--rose)';
  }
  function handleDrop(e) {
    e.preventDefault();
    document.getElementById('uploadZone').style.borderColor = '';
    const file = e.dataTransfer.files[0];
    if (file && file.type.startsWith('image/')) loadPreview(file);
  }

  /* ── CAMERA MODAL ── */
  let stream = null;
  let facingMode = 'environment';

  async function openCamera() {
    /* On mobile browsers, getUserMedia IS available but the HTML capture
       attribute on <input> is simpler and more reliable. We try getUserMedia
       first; if it fails or isn't available we fall back to the capture input. */
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      // Fallback: native camera via input capture
      document.getElementById('cameraInput').click();
      return;
    }
    try {
      stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode, width: { ideal: 1280 }, height: { ideal: 960 } }
      });
      const modal = document.getElementById('cameraModal');
      modal.style.display = 'flex';
      document.body.style.overflow = 'hidden';
      const video = document.getElementById('cameraVideo');
      video.srcObject = stream;
      document.getElementById('camPermDenied').style.display = 'none';
      document.getElementById('scanLine').style.display = 'block';
    } catch (err) {
      if (err.name === 'NotAllowedError' || err.name === 'PermissionDeniedError') {
        // Show the modal with denial message
        const modal = document.getElementById('cameraModal');
        modal.style.display = 'flex';
        document.body.style.overflow = 'hidden';
        document.getElementById('cameraVideo').style.display = 'none';
        document.getElementById('scanLine').style.display = 'none';
        const denied = document.getElementById('camPermDenied');
        denied.style.display = 'flex';
        denied.style.position = 'relative';
        denied.style.minHeight = '220px';
      } else {
        // Any other error — fall back to native capture
        document.getElementById('cameraInput').click();
      }
    }
  }

  async function flipCamera() {
    facingMode = facingMode === 'environment' ? 'user' : 'environment';
    if (stream) {
      stream.getTracks().forEach(t => t.stop());
      try {
        stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode } });
        document.getElementById('cameraVideo').srcObject = stream;
      } catch(e) {}
    }
  }

  function snapPhoto() {
    const video  = document.getElementById('cameraVideo');
    const canvas = document.getElementById('snapCanvas');
    canvas.width  = video.videoWidth  || 640;
    canvas.height = video.videoHeight || 480;
    canvas.getContext('2d').drawImage(video, 0, 0);
    canvas.toBlob(blob => {
      closeCamera();
      loadPreview(blob, 'skin-photo.jpg');
    }, 'image/jpeg', 0.92);
  }

  function closeCamera() {
    if (stream) { stream.getTracks().forEach(t => t.stop()); stream = null; }
    document.getElementById('cameraModal').style.display = 'none';
    document.getElementById('cameraVideo').style.display = 'block';
    document.body.style.overflow = '';
  }

  function switchToGallery() {
    closeCamera();
    document.getElementById('galleryInput').click();
  }

  /* ── PREVIEW MODAL ── */
  function loadPreview(fileOrBlob, name) {
    const url = URL.createObjectURL(fileOrBlob);
    document.getElementById('previewImg').src = url;
    document.getElementById('previewModal').style.display = 'flex';
    document.getElementById('analysingOverlay').style.display = 'none';
    document.body.style.overflow = 'hidden';

    // update upload zone to show thumbnail
    const zone = document.getElementById('uploadZone');
    zone.style.backgroundImage = `url(${url})`;
    zone.style.backgroundSize = 'cover';
    zone.style.backgroundPosition = 'center';
    document.getElementById('uploadIcon').style.display = 'none';
    document.getElementById('uploadTitle').textContent = 'Photo ready';
    document.getElementById('uploadSubtitle').textContent = 'Click to change image';
  }

  function closePreview() {
    document.getElementById('previewModal').style.display = 'none';
    document.body.style.overflow = '';
  }

  function analysePhoto() {
    const overlay = document.getElementById('analysingOverlay');
    overlay.style.display = 'flex';
    // simulate analysis delay then show toast
    setTimeout(() => {
      document.getElementById('previewModal').style.display = 'none';
      document.body.style.overflow = '';
      const toast = document.getElementById('resultToast');
      toast.style.display = 'block';
      setTimeout(() => { toast.style.display = 'none'; }, 3500);
    }, 2200);
  }

  /* close modals on backdrop click */
  document.getElementById('cameraModal').addEventListener('click', function(e) {
    if (e.target === this) closeCamera();
  });
  document.getElementById('previewModal').addEventListener('click', function(e) {
    if (e.target === this) closePreview();
  });
</script>

</body>
</html>
