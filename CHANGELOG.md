# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Idées en discussion
- Support TADAWUL (Arabie Saoudite) avec calibration WACC SAR + spécificités CMA SA
- Support BVMT (Tunis) avec calibration TND + référentiel Système Comptable Tunisien
- Profil "Banques" : adapter S01 (PNB / cost-to-income / NIM) et S03 (CET1 / Tier 1 / RWA)
- Profil "Assurance" : adapter avec ratio combiné, marge S/P, primes acquises
- Export PDF natif (sans passer par Chrome) via Puppeteer headless
- Mode comparatif : analyser N et N-1 côte à côte dans la même prez

## [0.2.0] — 2026-05-01

### Changed
- **Élargissement du périmètre** : le skill ne se limite plus à la BVC (Casablanca). Il fonctionne désormais sur toute société cotée, tous marchés (Euronext, LSE, NYSE, NASDAQ, TADAWUL, EGX, etc.).
- Ajout d'une grille de calibration WACC multi-marchés (Rf + ERP par place)
- Section "Spécificités par marché" ajoutée dans `SKILL.md` (devise, référentiel, régulateur, source cours)
- Vocabulaire bilingue (FR + EN) pour tous les concepts financiers majeurs
- Argument `--market=` ajouté pour forcer la calibration
- Reconnaissance des référentiels comptables : IFRS, US GAAP, CGNC, French GAAP
- Conformité réglementaire : AMMC / AMF / SEC / FCA selon le marché
- Disclaimer "analyse éducative, pas conseil personnalisé" ajouté
- Repo renommé : `claude-skill-rfa-bvc-analyzer` → `claude-skill-annual-report-analyzer`

### Why
- Retour utilisateur : la méthodologie (8 sections, audit mindset, divergence P&L vs Cash, DCF) est universelle, seule la calibration locale change. Limiter le skill à la BVC le rendait artificiellement restreint.

## [0.1.0] — 2026-05-01

### Added
- **Initial release**.
- Skill `analyse-rfa-institutionnelle` pour Claude Code.
- Workflow 4 étapes : Extraction PDF → Note 8 sections → HTML interactif → Livraison.
- Méthodologie 8 sections : P&L, Cash/BFR, Structure financière, Rentabilité (ROIC vs WACC), Annexes qualitatives, Red Flags (mindset audit), Valorisation (DCF + multiples + scénarios Bull/Base/Bear), Synthèse exécutive.
- Gabarit HTML de référence : `examples/adh-2025-reference.html` (Addoha FY2025, BVC, ~85 KB, dark Bloomberg style).
- Responsive complet (sidebar off-canvas mobile + hamburger ≤ 768px, breakpoints 1100px / 768px / 380px).
- Charts Chart.js 4.4.0 : waterfall P&L, doughnut BFR, ROIC vs WACC, scénarios DCF, gauge gearing.
- Meta tags Open Graph pour partage WhatsApp / LinkedIn (génération image dynamique via tailgraph.com).
- Palette de couleurs d'accent par secteur (immobilier amber, banques bleu, télécoms purple, etc.).
- Guide d'installation et d'utilisation dans le README.
- Licence MIT.

---

[Unreleased]: https://github.com/LKABAL-BVC/claude-skill-annual-report-analyzer/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/LKABAL-BVC/claude-skill-annual-report-analyzer/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/LKABAL-BVC/claude-skill-annual-report-analyzer/releases/tag/v0.1.0
