# claude-skill-annual-report-analyzer

> **Claude Code Skill** — Analyse fondamentale institutionnelle d'un **rapport financier annuel** (RFA, Annual Report, 10-K, Form 20-F, Document d'Enregistrement Universel) pour toute société cotée — toutes bourses confondues — avec génération automatique d'une présentation HTML interactive responsive de niveau Bloomberg Terminal.

## Ce que fait ce skill

À partir d'un PDF de rapport annuel d'une société cotée (BVC/Casablanca, Euronext, LSE, NYSE, NASDAQ, etc.), le skill produit :

1. **Une note d'analyse 8 sections** (FR par défaut, EN possible, ton sceptique audit) :
   - Section 01 — Compte de résultat (P&L) — qualité du résultat, effet ciseaux
   - Section 02 — Cash-flow & BFR — pont RN→FCF, DSO/DIO/DPO
   - Section 03 — Structure financière — gearing, dette nette/EBITDA, covenants
   - Section 04 — Rentabilité & création de valeur — **ROIC vs WACC** (calibration multi-marchés)
   - Section 05 — Annexes qualitatives — opinion CAC/auditor, KAM/CAM, parties liées, hors bilan
   - Section 06 — Red Flags — mindset audit, classés par sévérité (🔴🟠🟡)
   - Section 07 — Valorisation — multiples + DCF + scénarios Bull/Base/Bear
   - Section 08 — Synthèse exécutive — recommandation, cible 12M, thèse, catalyseurs

2. **Une présentation HTML interactive responsive** :
   - Style Bloomberg Terminal dark
   - Navigation sidebar avec scroll-spy
   - Charts Chart.js (waterfall P&L, doughnut BFR, ROIC vs WACC, scénarios DCF)
   - Mobile-first (sidebar off-canvas + hamburger ≤ 768px)
   - Meta tags Open Graph pour partage WhatsApp / LinkedIn
   - Fichier unique HTML standalone, partageable via Netlify Drop / Surge / GitHub Pages

## Marchés supportés

Le skill est conçu pour fonctionner sur **toute société cotée**. Calibrations pré-encodées (WACC, devise, référentiel comptable, régulateur) :

| Marché | Devise | Référentiel | Régulateur | Source cours |
|---|---|---|---|---|
| **BVC / Maroc (MASI)** | MAD | IFRS / CGNC | AMMC | casablanca-bourse.com |
| **Euronext (Paris, Amsterdam, Bruxelles…)** | EUR | IFRS | AMF / AFM / FSMA | live.euronext.com |
| **LSE (Londres)** | GBP | IFRS UK-endorsed | FCA | londonstockexchange.com |
| **NYSE / NASDAQ** | USD | US GAAP / IFRS (20-F) | SEC | SEC EDGAR |
| **TADAWUL (Arabie Saoudite)** | SAR | IFRS | CMA SA | saudiexchange.sa |
| **EGX (Égypte)** | EGP | EAS / IFRS | FRA | egx.com.eg |
| **Autres** | — | — | — | demande de paramètres |

Pour un marché non listé, le skill demande à l'utilisateur les paramètres clés (taux sans risque, prime de risque actions, devise).

## Aperçu

Voir [`examples/adh-2025-reference.html`](examples/adh-2025-reference.html) — analyse complète d'**Addoha (ADH:CSE)** sur les comptes 2025 (~85 KB, ~2200 lignes, dark theme, fully responsive). Cas d'usage : promoteur immobilier marocain, recommandation NEUTRE biais négatif, 10 red flags identifiés, divergence majeure RN +70% vs FCF -7 MMAD.

## Installation

### Via Claude Code (skill personnel global)

```bash
git clone https://github.com/LKABAL/claude-skill-annual-report-analyzer.git ~/.claude/skills/analyse-rfa-institutionnelle
```

Le skill est immédiatement disponible dans Claude Code — il s'auto-déclenche quand vous demandez une analyse de rapport annuel.

### Vérifier l'installation

Dans Claude Code :
```
/skills
```
Vous devriez voir `analyse-rfa-institutionnelle` dans la liste.

## Utilisation

### Invocation explicite

```
/analyse-rfa-institutionnelle chemin/vers/rapport-annuel-2024.pdf
```

### Invocation naturelle

Demandez simplement à Claude Code :

- *"Analyse le RFA d'Addoha 2025"* (puis fournir le PDF)
- *"Investment note pour Sanofi sur l'exercice 2024"*
- *"Voici le 10-K d'Apple FY2024, fais-moi l'analyse institutionnelle"*
- *"Fais-moi une analyse buy-side du DEU de TotalEnergies"*

### Arguments optionnels

```
/analyse-rfa-institutionnelle <pdf-path> [--year=YYYY] [--market=BVC|EURONEXT|LSE|NYSE|...] [--out-dir=path]
```

- `--year` : année à analyser si le rapport contient plusieurs exercices (défaut : le plus récent)
- `--market` : forcer la calibration marché (défaut : déduit du ticker / rapport)
- `--out-dir` : dossier de sortie (défaut : `<HOME>/<TICKER>-Analysis-<YEAR>/`)

## Sortie

Le skill crée un dossier `<TICKER>-Analysis-<YEAR>/` contenant :
- `index.html` — la présentation interactive complète

Pour partager publiquement, plusieurs options :
- **Netlify Drop** (le plus simple) : drag-and-drop le dossier sur https://app.netlify.com/drop
- **Surge.sh** : `npx surge .` depuis le dossier
- **GitHub Pages** : push dans un repo et activer Pages
- **PDF** : ouvrir dans Chrome → Ctrl+P → Save as PDF (paysage A4)

## Personnalisation

### Couleur d'accent par secteur

Le skill choisit automatiquement une couleur d'accent visuelle cohérente avec le secteur :

| Secteur | Couleur | Hex |
|---|---|---|
| Real Estate / Immobilier | Amber | `#f59e0b` |
| Financials / Banques | Blue | `#3b82f6` |
| Tech / Télécoms | Purple | `#8b5cf6` |
| Energy / Mines | Red / Emerald | `#ef4444` / `#10b981` |
| Consumer Staples / Agro | Green | `#22c55e` |
| Insurance | Cyan | `#06b6d4` |
| Materials / Cimenterie | Slate | `#a3a3a3` |
| Healthcare / Pharma | Pink | `#ec4899` |

### Échelle de recommandations

Échelle institutionnelle classique :
- **ACHAT FORT** — upside > 25% + faible risque
- **ACHAT** — upside 10-25%
- **NEUTRE** (biais positif/négatif) — upside ±10%
- **VENTE** — downside 10-25%
- **VENTE FORTE** — downside > 25%

## Méthodologie — points clés

- **Pont RN → FCF systématique** : c'est là que se cachent la plupart des manipulations comptables (capitalisation agressive, BFR qui explose, provisions opportunistes)
- **ROIC vs WACC** : si ROIC < WACC, destruction de valeur économique (EVA négative) — message majeur
- **Effet ciseaux CA / RN** : croissance asymétrique = signal à creuser
- **Lecture systématique des KAM / CAM** : les Key/Critical Audit Matters révèlent les zones de risque réelles selon le commissaire aux comptes
- **DCF avec scénarios Bull/Base/Bear** : pas de cible unique trompeuse, fourchette de valorisation transparente

## Contraintes & garde-fous encodés

- **Aucun mock data** : si une donnée manque dans le rapport, le skill le mentionne explicitement
- **Citation systématique** des sources réglementaires (AMMC, AMF, SEC, FCA, etc.)
- **Mindset audit** : recherche systématique des incohérences P&L vs Cash-Flow vs Bilan
- **Pas de déploiement automatique** : l'utilisateur garde le contrôle du partage
- **Disclaimer** : analyse éducative / informationnelle, pas un conseil en investissement personnalisé
- **Langue par défaut** : français (adaptable EN)

## Stack technique

- **Charts** : Chart.js 4.4.0 (CDN)
- **Fonts** : System fonts + JetBrains Mono pour les chiffres
- **Pas de framework** : HTML/CSS/JS vanilla, fichier unique standalone
- **Open Graph** : génération d'image dynamique via [tailgraph.com](https://tailgraph.com)

## Limitations connues

- La qualité de l'analyse dépend de la qualité du PDF source (rapport scanné en image = extraction limitée — préférer les PDFs textuels natifs).
- Le DCF est simplifié (3-5 ans + valeur terminale) — pour valorisation complète multi-scénarios, compléter manuellement.
- Pour les sociétés très complexes (conglomérats multi-segments, banques avec compte de résultat bancaire spécifique, assurances avec ratio combiné…), le skill produit une base solide mais nécessite enrichissement sectoriel.
- Calibrations WACC fournies à titre indicatif — vérifier les conditions de marché en vigueur à la date d'analyse.

## Cas d'usage typiques

- **Buy-side** : pré-screening avant approfondissement, génération de notes courtes pour comité d'investissement
- **Sell-side** : structure de note initiation, mise à jour post-publication des comptes
- **Family office / wealth management** : pédagogie client (la prez HTML est compréhensible par un non-spécialiste)
- **Étudiants finance / CFA** : cas pratique structuré sur n'importe quelle société cotée
- **Journalisme financier** : décryptage rapide d'un rapport annuel publié

## Licence

MIT — voir [LICENSE](LICENSE).

## Auteur

**LKABAL**

## Contributions

Issues et PRs bienvenues. Pour ajouter le support fin d'autres bourses (Tunis, Alger, Nigeria, Kenya, Arabie, Inde…), ouvrir une issue d'abord avec : référentiel comptable, devise, régulateur, source de cours, calibration WACC type.

---

*Built for Claude Code — https://docs.claude.com/claude-code*
