---
name: analyse-rfa-institutionnelle
description: Produit une analyse fondamentale institutionnelle (8 sections — P&L, Cash/BFR, Structure financière, Rentabilité, Annexes qualitatives, Red Flags, Valorisation, Synthèse Bull/Bear) à partir d'un Rapport Financier Annuel (RFA / Annual Report) d'une société cotée — sur n'importe quelle bourse (BVC/MASI, Euronext, LSE, NYSE, NASDAQ, etc.) — puis génère une présentation HTML interactive responsive de niveau Bloomberg Terminal pour gestionnaires institutionnels. Use when l'utilisateur demande "analyse RFA <ticker>", "analyse rapport annuel <société>", "investment note <société>", ou pointe un PDF de rapport annuel/10-K/Form 20-F d'une société cotée.
when_to_use: Demande d'analyse fondamentale d'un rapport financier annuel (RFA, Annual Report, 10-K, 20-F, Document d'Enregistrement Universel) d'une société cotée. Le livrable attendu est (1) une note buy-side/sell-side écrite et (2) une présentation HTML interactive partageable.
argument-hint: <pdf-rfa-path-or-ticker> [--year=YYYY] [--market=BVC|EURONEXT|LSE|NYSE|...] [--lang=FR|EN|BILINGUAL] [--out-dir=path]
disable-model-invocation: false
allowed-tools: Read Write Bash Grep Glob WebFetch
---

# analyse-rfa-institutionnelle

Workflow complet : Rapport financier annuel PDF → Note d'analyse institutionnelle → Présentation HTML interactive responsive partageable.

**Marchés supportés** : toute société cotée. Calibrations encodées pour les principales places développées et structurées : Euronext (Paris, Amsterdam, Bruxelles, Lisbonne, Dublin, Oslo), LSE, NYSE/NASDAQ, Deutsche Börse (Xetra), SIX Swiss Exchange, Borsa Italiana, BME, NASDAQ Nordic, TSX, ASX, TSE. Cas couvert également : BVC (Casablanca / MASI), utilisé comme exemple de référence dans `examples/`. Si le marché n'est pas reconnu, le skill demande à l'utilisateur les paramètres clés (taux sans risque, prime de risque, devise).

## Inputs

`$ARGUMENTS` =

- Chemin vers le PDF du rapport annuel (RFA, Annual Report, 10-K, 20-F, URD), OU
- Ticker boursier (ex: `ADI`, `IAM`, `SAN.PA`, `AAPL`, `HSBA.L`) — dans ce cas demander à l'utilisateur le PDF
- Optionnel : `--year=YYYY` (par défaut : année la plus récente du rapport)
- Optionnel : `--market=BVC|EURONEXT|LSE|NYSE|NASDAQ|TADAWUL|EGX|...` (par défaut : déduit du ticker ou du rapport)
- Optionnel : `--lang=FR|EN|BILINGUAL` (défaut : `FR` — `BILINGUAL` intègre un toggle FR/EN dans le HTML ; `EN` génère entièrement en anglais sans toggle)
- Optionnel : `--out-dir=<path>` (par défaut : `<HOME>/<TICKER>-Analysis-<YEAR>/`)

## Persona à incarner

**Analyste financier senior buy-side/sell-side** avec 15+ ans d'expérience marchés développés et émergents. Mindset audit : sceptique, incisif, factuel. Pas de complaisance — recherche systématique des incohérences entre comptes (P&L vs Cash-Flow vs Bilan), des red flags comptables, des biais de communication management. Ton professionnel et rigoureux.

## Workflow

### Étape 1 — Extraction des données du rapport

1. **Read** le PDF du rapport avec l'outil `Read` (utiliser `pages` si > 10 pages).
2. **Identifier le référentiel comptable** : IFRS, US GAAP, CGNC marocain, French GAAP, etc. — l'extraction et les ratios s'adaptent.
3. **Extraire en littéral** (pas d'inférence à ce stade) :
   - **Identité** : raison sociale, ticker, bourse de cotation, secteur, date de clôture, exercice analysé + N-1 (et N-2 si dispo), devise de reporting
   - **Compte de résultat consolidé** : CA / Revenue, EBITDA (ou calculer : EBIT + D&A), résultat d'exploitation / Operating Income, résultat financier, résultat net part du groupe / Net Income attributable to shareholders, intérêts minoritaires
   - **Bilan consolidé** : actif total, immobilisations corporelles/incorporelles/financières (PP&E, Intangibles, Goodwill), stocks, créances clients, trésorerie, capitaux propres part du groupe, intérêts minoritaires, dettes financières (CT + LT), dettes fournisseurs, autres passifs
   - **Tableau de flux de trésorerie** : CFO (Cash Flow from Operations), CAPEX, FCF (= CFO − CAPEX si non publié), dividendes versés, variation dette nette, rachats d'actions
   - **Annexes** : méthodes comptables clés, changements, événements post-clôture, parties liées, engagements hors bilan, litiges, segment reporting
   - **Rapport CAC / Auditor's Report** : opinion (clean / qualified / adverse / disclaimer), Key Audit Matters (KAM) / Critical Audit Matters (CAM)
   - **Données de marché** : cours au 31/12, nombre d'actions diluées, capitalisation boursière, dividende proposé, programme de rachat
   - **Communication financière** : Letter to Shareholders / Lettre du Président, MD&A / Management Report, guidance

4. **Recouper** avec sources externes si nécessaire :
   - **BVC / Maroc** : `https://www.casablanca-bourse.com/live-market/emetteurs/<CODE>` + AMMC pour conformité
   - **Euronext** : `https://live.euronext.com/`
   - **LSE** : `https://www.londonstockexchange.com/`
   - **US (NYSE/NASDAQ)** : `https://www.sec.gov/cgi-bin/browse-edgar` (10-K, 10-Q, 8-K)
   - Sinon demander à l'utilisateur ou utiliser `WebFetch` sur le site IR de la société

5. **OBLIGATOIRE — Récupérer le cours actuel de la valeur** (cours de clôture du dernier jour de bourse disponible) via `WebFetch` :
   - **BVC** : `https://www.casablanca-bourse.com/live-market/emetteurs/<CODE>` — chercher la balise prix
   - **Euronext** : `https://live.euronext.com/en/product/equities/<ISIN>-<MIC>` (ex : `FR0000120271-XPAR` pour Total)
   - **LSE** : `https://www.londonstockexchange.com/stock/<TICKER>/<COMPANY-SLUG>/quote`
   - **NYSE/NASDAQ** : `https://finance.yahoo.com/quote/<TICKER>` ou `https://www.google.com/finance/quote/<TICKER>:NASDAQ`
   - **Deutsche Börse** : `https://www.boerse-frankfurt.de/equity/<ISIN>`
   - **SIX Swiss** : `https://www.six-group.com/en/market-data/shares/explorer/<ISIN>.html`
   - **Autres** : Yahoo Finance reste le fallback universel le plus fiable
   - **Capturer** : cours, devise, date/heure de cotation, volume du jour, variation YTD
   - **Calculer** : capitalisation boursière actuelle = cours × nombre d'actions diluées (à recouper avec celle publiée par la bourse)
   - **Si la récupération échoue** (page JS-rendered, accès bloqué, ticker non trouvé) : demander explicitement le cours à l'utilisateur — **ne jamais inventer un cours**
   - **Documenter** dans le HTML : badge "Cours au [date heure]" visible dans le hero ; mentionner clairement si le cours est utilisateur-fourni vs scrappé

### Étape 2 — Analyse 8 sections (livrable n°1 : note écrite)

Produire une note structurée en **8 sections numérotées**, dans la langue demandée (FR par défaut), ton institutionnel sceptique. Chaque section doit citer les chiffres exacts du rapport et **systématiquement chercher les incohérences**.

#### Section 01 — Compte de résultat (P&L)

- Évolution CA YoY (en devise locale et %), décomposition organique vs périmètre vs change si dispo
- Marge brute, marge EBITDA, marge opérationnelle, marge nette — comparer aux 2-3 dernières années
- **Effet ciseaux** : identifier si croissance CA ≠ croissance résultat opérationnel (et pourquoi : prix vs volume, mix produits, charges fixes, FX)
- **Qualité du résultat net** : part de l'exceptionnel, des reprises de provisions, des plus-values de cession, des éléments non-cash dans le RN
- **Drapeau rouge type** : RN qui croît fortement alors que CA stagne ou décroît → vérifier la nature des produits non-récurrents

#### Section 02 — Cash-flow & BFR

- CFO, variation BFR (avec décomposition stocks/clients/fournisseurs si dispo)
- CAPEX (maintenance vs croissance si distinguable), FCF = CFO − CAPEX
- **Pont RN → FCF** : expliquer l'écart (D&A, ΔBFR, provisions, CAPEX) — c'est ICI que se cachent la plupart des manipulations
- **DSO, DIO, DPO** (Days Sales/Inventory/Payables Outstanding) — évolution et benchmark sectoriel
- **Drapeau rouge type** : RN +X% mais FCF négatif/en baisse → BFR qui explose (créances qui ne rentrent pas, stocks qui s'accumulent) ou CAPEX masqué / capitalisation agressive

#### Section 03 — Structure financière

- Dette nette = Dette financière brute − Trésorerie active
- Gearing = Dette nette / Capitaux propres
- Dette nette / EBITDA (covenant typique : < 3x à 4x selon secteur)
- Couverture des intérêts = EBITDA / Charges financières
- Maturité de la dette (CT vs LT), nature (bancaire vs obligataire vs schuldschein vs convertible), covenants visibles
- **Liquidité** : ratio courant, ratio rapide, position cash vs CT debt, lignes de crédit non tirées (RCF)
- **Drapeau rouge type** : refinancement à venir + dégradation FCF + gearing élevé = stress de liquidité

#### Section 04 — Rentabilité & création de valeur

- ROE (RN / CP moyens), ROCE = EBIT × (1−t) / Capital employé
- **ROIC vs WACC** : calculer le WACC adapté au marché — voir grille de calibration ci-dessous. Comparer à ROIC.
- **Si ROIC < WACC : destruction de valeur économique** (EVA négative) — c'est un message majeur
- DuPont : ROE = Marge nette × Rotation actifs × Levier financier
- Évolution sur 3-5 ans pour identifier la trajectoire

**Grille WACC par marché (taux sans risque + prime de risque actions)** :

| Marché | Rf (10Y govt) | ERP | Notes |
| --- | --- | --- | --- |
| BVC / Maroc (MAD) | ~3% | ~7% | Bons du Trésor 10Y |
| Euronext (EUR) | ~3% | ~5,5% | OAT 10Y / Bund 10Y |
| LSE (GBP) | ~4% | ~5,5% | Gilt 10Y |
| NYSE/NASDAQ (USD) | ~4% | ~5% | UST 10Y |
| Deutsche Börse (EUR) | ~3% | ~5,5% | Bund 10Y |
| SIX Swiss (CHF) | ~1% | ~5% | SNB 10Y |
| TSX (CAD) | ~3,5% | ~5,5% | GoC 10Y |
| ASX (AUD) | ~4% | ~6% | ACGB 10Y |
| TSE (JPY) | ~1% | ~6% | JGB 10Y |

Si les conditions de marché ont évolué, recouper via WebFetch ou demander à l'utilisateur.

#### Section 05 — Annexes qualitatives

- Opinion CAC / Auditor + KAM / CAM — souvent les KAM révèlent les zones de risque
- Changements de méthode comptable, options IFRS / US GAAP exercées
- Engagements hors bilan (cautions, garanties, leasing opérationnel résiduel pré-IFRS 16)
- Litiges significatifs (avec montants si quantifiés)
- Parties liées (transactions intra-groupe, conventions réglementées)
- Événements post-clôture (Subsequent Events)
- Gouvernance : indépendance du conseil, rotation CAC, executive compensation

#### Section 06 — Red Flags (mindset audit)

**Lister 5 à 15 red flags** classés par sévérité (🔴 critique / 🟠 modéré / 🟡 à surveiller). Format type :

- **🔴 RF#1 — Titre** : Description factuelle citant les chiffres. Implication : ...
- Cibler particulièrement : divergence P&L vs Cash, BFR anormal, provisions/reprises opportunistes, changements de méthode, transactions parties liées, litiges sous-provisionnés, engagements hors bilan masqués, communication management vs réalité comptable, qualité du carnet de commandes vs CA reconnu, capitalisation R&D agressive.

#### Section 07 — Valorisation

##### A. Multiples de marché (ancrage observable)

- PER publié + PER normatif (RN retraité des éléments non récurrents), P/B, EV/EBITDA, EV/CA, dividend yield, FCF yield
- Comparables sectoriels : locaux (même marché, même segment) + internationaux pertinents
- Un P/B < 1 signifie que le marché valorise moins que la valeur comptable liquidative : mentionner explicitement si c'est justifié (actifs dépréciés, faible ROIC) ou si c'est une opportunité

##### B. DCF — utiliser un FCF normalisé, pas le FCF spot

- **Ne jamais utiliser le FCF d'un exercice atypique** (choc BFR, CAPEX exceptionnel, première année d'une réforme comptable) comme base de projection. Calculer d'abord un FCF normalisé :
  - Option 1 : moyenne arithmétique des FCF des 3-5 dernières années
  - Option 2 : FCF normatif = NOPAT − CAPEX maintenance − ΔBFR normalisé (BFR/CA × croissance CA cible)
- Projection 3-5 ans + valeur terminale : g terminal = inflation locale + 0-1 % (pas > 3 % pour marchés émergents)
- Sensibilité systématique : tableau WACC × g (3 WACC × 3 g = 9 cellules)
- **Reverse DCF (obligatoire)** : à partir du cours de bourse actuel, résoudre pour le FCF normalisé implicite. Formuler : *"Le cours actuel implique un FCF normalisé de X MMAD — est-ce crédible ?"* Cela ancre la cible DCF sur le marché réel.

##### C. Méthode sectorielle (adapter au secteur)

- **Immobilier / Promoteur** : NAV = (valeur stock foncier + WIP + produits finis avec haircut de réalisation −15 % à −40 %) − dettes financières nettes. Comparer NAV/action vs cours. Pair sectoriels : P/NAV.
- **Banques** : P/ANC (Price-to-Adjusted Book) + Gordon Growth Model (DDM) sur dividende normalisé. RoE vs Ke comme ancre.
- **Utilities / Concessions** : DCF WACC réglementaire + Regulated Asset Base (RAB). EV/EBITDA régulé.
- **Conglomérats** : SOTP (Sum-of-the-Parts) — valoriser chaque segment séparément, sommer et appliquer une décote holding de 10-20 %.
- **Mining / Énergie** : NAV des réserves P1 au prix spot et au prix forward. EV/EBITDA cycle + EV/boe.
- **Retail / FMCG / Telecom** : EV/EBITDA + DCF avec croissance organique. Attention au CAPEX récurrent qui grève le FCF.

##### D. Décotes obligatoires (marchés émergents et small/mid cap)

- **Décote de liquidité** : appliquer −15 % à −25 % pour les small/mid cap sur marchés peu profonds (BVC, EGX, TADAWUL off-index). Critères : flottant < 20 %, volume journalier moyen < 0,5 % du capital, spread bid-ask > 1,5 %. Documenter le pourcentage retenu et sa justification.
- **Décote pays / gouvernance / ESG** : −5 % à −15 % si actionnaire de contrôle dominant sans droits minoritaires forts, ou si litiges réglementaires actifs, ou si opacité des transactions parties liées.
- **Prime OPA** (scénario Bull uniquement) : +20 % à +30 % si la société est une cible M&A plausible.

##### E. Cible pondérée multi-méthodes (obligatoire)

Construire un tableau de synthèse avec pondération explicite. Pondérations indicatives par secteur :

| Secteur | DCF | Multiples | Méthode sectorielle |
| --- | --- | --- | --- |
| Immobilier / Promoteur | 20 % | 30 % | 50 % (NAV) |
| Banques | 25 % | 25 % | 50 % (DDM/P-ANC) |
| Industrie / Conso / Telecom | 35 % | 40 % | 25 % (SOTP/EV-EBIT) |
| Utilities / Concessions | 40 % | 30 % | 30 % (RAB) |
| Mining / Énergie | 20 % | 30 % | 50 % (NAV réserves) |

Format du tableau cible :

```text
Méthode          Bear   Base   Bull   Poids
DCF (normalisé)    X      X      X    30 %
Multiples pairs    X      X      X    40 %
Méthode sect.      X      X      X    30 %
Cible brute        X      X      X   100 %
Décote liquidité  −X%    −X%    −X%    —
CIBLE FINALE       X      X      X    MAD
```

- Upside/downside Bull/Base/Bear vs cours actuel
- Rendement total attendu = upside cours + dividend yield + buyback yield
- Chart obligatoire : bar groupé Bear/Base/Bull par méthode + cible finale

#### Section 08 — Synthèse exécutive Bull / Bear

- **Recommandation** : ACHAT FORT / ACHAT / NEUTRE (biais positif/négatif) / VENTE / VENTE FORTE
- **Cible 12 mois** : XX (devise) (vs YY actuel, soit ±Z%)
- **Thèse Bull (3-5 points)** : ce qui peut surprendre positivement
- **Thèse Bear (3-5 points)** : ce qui peut décevoir
- **Catalyseurs court terme** (3-6 mois) : à monitorer
- **Top 3 KPIs à surveiller** : trimestre par trimestre

### Étape 3 — Génération de la présentation HTML interactive (livrable n°2)

Utiliser `examples/adh-2025-reference.html` comme **gabarit structurel**. Adapter :

- **Identité visuelle** : ticker, nom société, secteur, accent color (voir guide ci-dessous)
- **Données** : tous les chiffres, charts, scénarios remplacés par les valeurs de la nouvelle société, devise adaptée
- **Recommandation & rating pill** (couleur selon recommandation)
- **Meta tags Open Graph** (titre/description/image OG via `og.tailgraph.com`) pour partage WhatsApp/LinkedIn
- **Conserver** : structure responsive (sidebar off-canvas mobile + hamburger ≤ 768px), navigation scroll-spy, dark theme Bloomberg, Chart.js 4.4.0

#### Choix de l'accent color par secteur (cohérence visuelle)

- Immobilier / Real Estate : `#f59e0b` (amber)
- Banques / Financials : `#3b82f6` (blue)
- Télécoms / Tech : `#8b5cf6` (purple)
- Mines / Énergie / Oil & Gas : `#ef4444` (red) ou `#10b981` (emerald) selon positionnement
- Agro / Conso / Consumer Staples : `#22c55e` (green)
- Assurance / Insurance : `#06b6d4` (cyan)
- BTP / Cimenterie / Materials : `#a3a3a3` (slate)
- Healthcare / Pharma : `#ec4899` (pink)

#### Structure HTML (9 sections de navigation)

1. **Overview** (hero + KPI bar + résumé exécutif)
2. **01 — P&L** (waterfall CA→RN, table évolution 3 ans)
3. **02 — Cash & BFR** (pont RN→FCF, charts BFR DSO/DIO/DPO)
4. **03 — Structure financière** (gauge gearing, table dette par maturité)
5. **04 — Rentabilité** (ROIC vs WACC chart, DuPont décomposition)
6. **05 — Annexes** (cards qualitatives + opinion CAC)
7. **06 — Red Flags** (cards rouges/oranges/jaunes)
8. **07 — Valorisation** (multiples table, DCF scénarios Bull/Base/Bear)
9. **08 — Synthèse** (recommandation + thèse Bull/Bear + catalyseurs + Q&A)

#### Charts à inclure (Chart.js)

- Waterfall CA → EBITDA → EBIT → RN
- Doughnut décomposition BFR
- Line/Bar évolution marges 5 ans
- Bar comparatif ROIC vs WACC
- SVG gauge pour gearing / Net Debt-to-EBITDA
- Bar scénarios DCF Bull/Base/Bear

#### Responsive (obligatoire)

Reprendre les media queries du fichier de référence :

- `≤ 1100px` : sidebar plus étroite
- `≤ 768px` : sidebar off-canvas + bouton hamburger ☰ + grids → 1 colonne + KPI bar 2 colonnes + chart-wrap 240px
- `≤ 380px` : KPI bar 1 colonne

#### Toggle bilingue FR/EN (obligatoire si `--lang=BILINGUAL`)

Ajouter dans le `<style>` :

```css
.lang-toggle { display:flex; align-items:center; background:var(--bg-tertiary); border:1px solid var(--border-strong); border-radius:4px; padding:2px; flex-shrink:0; }
.lang-btn { background:none; border:none; color:var(--text-muted); font-family:var(--mono); font-size:11px; font-weight:700; letter-spacing:0.5px; padding:4px 9px; border-radius:3px; cursor:pointer; transition:all 0.15s; }
.lang-btn.active { background:var(--accent-soft); color:var(--accent); }
.lang-btn:hover:not(.active) { color:var(--text-primary); background:var(--bg-elevated); }
@media (max-width:768px) { .lang-toggle { display:none; } }
```

Ajouter dans `.topbar` juste avant `.rating-pill` :

```html
<div class="lang-toggle" id="langToggle" aria-label="Langue / Language">
  <button class="lang-btn active" data-lang="fr" onclick="setLang('fr')" aria-label="Français">FR</button>
  <button class="lang-btn" data-lang="en" onclick="setLang('en')" aria-label="English">EN</button>
</div>
```

**Pattern 1 — libellés UI fixes** (`data-i18n`, valeur FR dans le HTML, valeur EN injectée par JS) :

```html
<div class="kpi-bar-label" data-i18n="kpi.revenue">CA / Revenue</div>
<h2 class="section-title" data-i18n="s1.title">Compte de résultat</h2>
<span data-i18n="nav.s1">P&amp;L &amp; Performance</span>
```

**Pattern 2 — texte analytique narratif** (`data-lang`, deux blocs FR + EN) :

```html
<div data-lang="fr">
  <p class="section-subtitle">La croissance du résultat masque une détérioration cash.</p>
</div>
<div data-lang="en" style="display:none">
  <p class="section-subtitle">Earnings growth masks an underlying cash flow deterioration.</p>
</div>
```

**JS** — ajouter dans le bloc `<script>` (avant la fermeture `</script>`) :

```javascript
// ============ I18N & LANG TOGGLE ============
const i18n = {
  fr: {
    'nav.overview': 'Synthèse Exécutive', 'nav.s1': 'P&L &amp; Performance',
    'nav.s2': 'Cash &amp; BFR', 'nav.s3': 'Structure Financière',
    'nav.s4': 'Création de Valeur', 'nav.s5': 'Annexes &amp; Qualitatif',
    'nav.s6': 'Red Flags', 'nav.s7': 'Valorisation',
    'nav.s8': 'Bull vs Bear', 'nav.s9': 'Recommandation',
    'kpi.revenue': 'CA Consolidé', 'kpi.netincome': 'RN Part Groupe',
    'kpi.fcf': 'FCF Opérationnel', 'kpi.roic': 'ROIC',
    's0.title': 'Synthèse Exécutive', 's1.title': 'Compte de résultat',
    's2.title': 'Cash-Flow &amp; BFR', 's3.title': 'Structure Financière &amp; Solvabilité',
    's4.title': 'Rentabilité &amp; Création de Valeur', 's5.title': 'Analyse Qualitative &amp; Annexes',
    's6.title': 'Red Flags', 's7.title': 'Valorisation &amp; Comparables',
    's8.title': 'Bull vs Bear', 's9.title': 'Recommandation Finale',
    'rec.label': 'Recommandation', 'rec.conviction': 'Conviction analytique',
    'card.strengths': '▲ Points Forts', 'card.risks': '▼ Risques Critiques', 'card.verdict': '► Verdict',
    'flag.score': 'Score Global Red Flags'
  },
  en: {
    'nav.overview': 'Executive Summary', 'nav.s1': 'P&amp;L &amp; Performance',
    'nav.s2': 'Cash &amp; Working Capital', 'nav.s3': 'Financial Structure',
    'nav.s4': 'Value Creation', 'nav.s5': 'Qualitative &amp; Notes',
    'nav.s6': 'Red Flags', 'nav.s7': 'Valuation',
    'nav.s8': 'Bull vs Bear', 'nav.s9': 'Recommendation',
    'kpi.revenue': 'Revenue', 'kpi.netincome': 'Net Income (Group)',
    'kpi.fcf': 'Operating FCF', 'kpi.roic': 'ROIC',
    's0.title': 'Executive Summary', 's1.title': 'Income Statement',
    's2.title': 'Cash Flow &amp; Working Capital', 's3.title': 'Financial Structure &amp; Solvency',
    's4.title': 'Profitability &amp; Value Creation', 's5.title': 'Qualitative Analysis &amp; Notes',
    's6.title': 'Red Flag Detection', 's7.title': 'Valuation &amp; Comparables',
    's8.title': 'Bull vs Bear Scenarios', 's9.title': 'Final Recommendation',
    'rec.label': 'Recommendation', 'rec.conviction': 'Analytical conviction',
    'card.strengths': '▲ Key Strengths', 'card.risks': '▼ Critical Risks', 'card.verdict': '► Verdict',
    'flag.score': 'Overall Red Flag Score'
  }
};

function setLang(lang) {
  document.querySelectorAll('[data-i18n]').forEach(el => {
    const v = i18n[lang]?.[el.dataset.i18n];
    if (v !== undefined) el.innerHTML = v;
  });
  document.querySelectorAll('[data-lang]:not(.lang-btn)').forEach(el => {
    el.style.display = el.dataset.lang === lang ? '' : 'none';
  });
  document.querySelectorAll('.lang-btn').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.lang === lang);
  });
  localStorage.setItem('rfa-lang', lang);
}

document.addEventListener('DOMContentLoaded', () => {
  const saved = localStorage.getItem('rfa-lang') || 'fr';
  if (saved !== 'fr') setLang(saved);
});
```

**Charts** — pour les labels de datasets bilingues, déclarer un objet `chartLabels` et appeler `chart.update('none')` dans `setLang` :

```javascript
const chartLabels = {
  chartCA: { fr: ['CA Reporté (MMAD)', 'CA Pro-forma ancien réf.'], en: ['Reported Revenue (MAD M)', 'Pro-forma (old standard)'] }
  // ...
};
// Dans setLang(), après localStorage.setItem :
if (typeof Chart !== 'undefined') {
  Object.values(Chart.instances).forEach(chart => {
    const id = chart.canvas?.id;
    if (chartLabels[id]?.[lang]) {
      chart.data.datasets.forEach((ds, i) => {
        if (chartLabels[id][lang][i]) ds.label = chartLabels[id][lang][i];
      });
      chart.update('none');
    }
  });
}
```

Si `--lang=FR` ou `--lang=EN` (pas `BILINGUAL`) : générer le HTML **entièrement dans la langue choisie**, sans toggle, sans data-lang duals.

#### Disclaimer OBLIGATOIRE — à inclure systématiquement

**Le disclaimer suivant doit apparaître dans 3 endroits du livrable HTML**, sans exception :

**1. Dans le `<head>` en commentaire HTML :**

```html
<!--
DISCLAIMER : Ce document est produit à des fins éducatives et informationnelles uniquement.
Il ne constitue PAS un conseil en investissement, une recommandation personnalisée, une
sollicitation à acheter ou vendre des instruments financiers, ni une analyse financière
indépendante au sens réglementaire. L'auteur ne détient pas nécessairement de position
sur la valeur analysée et n'assume aucune responsabilité quant aux décisions prises sur
la base de ce document. Toute décision d'investissement doit être prise après consultation
d'un conseiller en investissement agréé et lecture intégrale des documents officiels de
l'émetteur (RFA, prospectus, communiqués réglementés).
-->
```

**2. Dans une bannière visible en haut de la première page (juste après le hero / KPI bar) :**

```html
<div class="disclaimer-banner" style="background: var(--warning-soft); border: 1px solid var(--warning); border-left: 4px solid var(--warning); padding: 12px 16px; margin: 16px 0; border-radius: 4px; font-size: 12px; color: var(--text-secondary); line-height: 1.5;">
  <strong style="color: var(--warning);">⚠️ AVERTISSEMENT</strong> — Ce document est produit à des fins <strong>éducatives et informationnelles uniquement</strong>. Il ne constitue <strong>ni un conseil en investissement, ni une recommandation personnalisée, ni une sollicitation</strong> à acheter ou vendre des instruments financiers. Les opinions exprimées reflètent l'analyse de l'auteur à la date de publication et peuvent évoluer sans préavis. Toute décision d'investissement doit être prise après consultation d'un conseiller agréé et lecture intégrale des documents officiels de l'émetteur.
</div>
```

**3. Dans le footer en pied de page (toujours visible) :**

```html
<footer class="legal-footer" style="margin-top: 48px; padding: 20px; background: var(--bg-secondary); border-top: 1px solid var(--border); font-size: 11px; color: var(--text-muted); line-height: 1.6; text-align: center;">
  <p><strong>Disclaimer légal</strong> — Document à finalité éducative et informationnelle. Ne constitue pas un conseil en investissement, une recommandation personnalisée, une sollicitation, ni une analyse financière indépendante au sens réglementaire (AMMC / AMF / SEC / FCA selon la juridiction). L'auteur n'assume aucune responsabilité quant aux décisions prises sur la base de ce document. Les performances passées ne préjugent pas des performances futures. La valeur des investissements peut fluctuer à la baisse comme à la hausse.</p>
  <p style="margin-top: 8px;">Sources : <a href="#" style="color: var(--accent);">Rapport Financier Annuel <ANNÉE> de l'émetteur</a> · Cours boursier au <DATE_HEURE_RÉCUPÉRATION> · Données de marché <SOURCE_COURS>.</p>
</footer>
```

**Adaptation EN si livrable en anglais** : remplacer le texte par sa traduction stricte ; conserver les 3 emplacements et le ton réglementaire neutre.

**JAMAIS** :

- Omettre l'un des 3 emplacements
- Réduire ou édulcorer le disclaimer
- Le mettre en très petit ou peu visible (bannière = warning color visible)
- Supprimer la mention "ne constitue pas un conseil en investissement"

### Étape 4 — Livraison

1. **Créer le dossier** `<out-dir>` (par défaut `<HOME>/<TICKER>-Analysis-<YEAR>/`)
2. **Écrire** `index.html` (la présentation finale)
3. **Donner à l'utilisateur** :
   - Chemin local du fichier
   - Options de partage : Netlify Drop manuel (https://app.netlify.com/drop), Surge.sh, GitHub Pages, ou hébergement statique de son choix
   - Si demande PDF : proposer impression depuis Chrome (Ctrl+P → Save as PDF, paysage A4)
4. **Résumé final** en 3-5 lignes : recommandation, cible, top 2 red flags, top 2 catalyseurs

## Spécificités par marché (calibrations)

### Maroc / BVC (MASI)

- Devise : MAD (montants généralement en MMAD = millions de dirhams)
- Référentiel : IFRS pour comptes consolidés, CGNC pour comptes sociaux
- Régulateur : AMMC (circulaire 03-19, conformité publication 43-12)
- Cours de bourse : casablanca-bourse.com

### France / Euronext Paris

- Devise : EUR
- Référentiel : IFRS pour comptes consolidés
- Régulateur : AMF (Document d'Enregistrement Universel = URD)
- Cours : live.euronext.com

### USA / NYSE / NASDAQ

- Devise : USD
- Référentiel : US GAAP (10-K), parfois IFRS pour foreign issuers (20-F)
- Régulateur : SEC (10-K annuel, 10-Q trimestriel, 8-K événements)
- Cours : SEC EDGAR + Yahoo Finance

### UK / LSE

- Devise : GBP (souvent en pence/GBX)
- Référentiel : IFRS (UK-endorsed)
- Régulateur : FCA
- Cours : londonstockexchange.com

### Autres marchés

Si marché non reconnu, demander à l'utilisateur : devise, référentiel comptable, régulateur, source de cours.

## Contraintes critiques (rappel)

- **Langue** : par défaut français (analyse + UI HTML), adaptable EN sur demande
- **Tonalité** : sceptique, factuelle, pas de superlatifs commerciaux
- **Chiffres** : toujours citer la source (page du rapport si possible) ; en devise locale du reporting
- **Cours actuel OBLIGATOIRE** : récupérer le dernier cours de bourse disponible via WebFetch (cf. Étape 1 §5). Afficher cours + date + heure dans le hero. Si récupération échoue, demander à l'utilisateur ; **jamais inventer**.
- **Disclaimer OBLIGATOIRE** : 3 emplacements dans le HTML (commentaire `<head>`, bannière warning visible après le hero, footer légal). Mention explicite *"ne constitue ni un conseil en investissement, ni une recommandation personnalisée, ni une sollicitation"*. Voir Étape 3 § Disclaimer pour le texte exact à reprendre.
- **Pas de déploiement web automatique** : laisser l'utilisateur gérer le partage
- **Pas de mock/fake data** : si une donnée manque dans le rapport, le mentionner explicitement (ex: "Non publié par l'émetteur — à demander en IR")
- **Conformité réglementaire** : ne pas reformuler les données réglementées au-delà du raisonnable, citer le rapport comme source primaire (AMMC pour Maroc, AMF pour France, SEC pour USA, FCA pour UK, etc.)

## Référence

- Exemple complet de livrable : `examples/adh-2025-reference.html` (Addoha FY2025, BVC, dark Bloomberg style, responsive complet, FR)
