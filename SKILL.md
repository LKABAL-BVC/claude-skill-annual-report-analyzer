---
name: analyse-rfa-institutionnelle
description: Produit une analyse fondamentale institutionnelle (8 sections — P&L, Cash/BFR, Structure financière, Rentabilité, Annexes qualitatives, Red Flags, Valorisation, Synthèse Bull/Bear) à partir d'un Rapport Financier Annuel (RFA / Annual Report) d'une société cotée — sur n'importe quelle bourse (BVC/MASI, Euronext, LSE, NYSE, NASDAQ, etc.) — puis génère une présentation HTML interactive responsive de niveau Bloomberg Terminal pour gestionnaires institutionnels. Use when l'utilisateur demande "analyse RFA <ticker>", "analyse rapport annuel <société>", "investment note <société>", ou pointe un PDF de rapport annuel/10-K/Form 20-F d'une société cotée.
when_to_use: Demande d'analyse fondamentale d'un rapport financier annuel (RFA, Annual Report, 10-K, 20-F, Document d'Enregistrement Universel) d'une société cotée. Le livrable attendu est (1) une note buy-side/sell-side écrite et (2) une présentation HTML interactive partageable.
argument-hint: <pdf-rfa-path-or-ticker> [--year=YYYY] [--market=BVC|EURONEXT|LSE|NYSE|...] [--out-dir=path]
disable-model-invocation: false
allowed-tools: Read Write Bash Grep Glob WebFetch
---

# analyse-rfa-institutionnelle

Workflow complet : Rapport financier annuel PDF → Note d'analyse institutionnelle → Présentation HTML interactive responsive partageable.

**Marchés supportés** : toute société cotée. Calibration par défaut pour la BVC (Casablanca / MASI), profils additionnels pour Euronext, LSE, NYSE/NASDAQ, marchés émergents MENA. Si le marché n'est pas reconnu, le skill demande à l'utilisateur les paramètres clés (taux sans risque, prime de risque, devise).

## Inputs

`$ARGUMENTS` =
- Chemin vers le PDF du rapport annuel (RFA, Annual Report, 10-K, 20-F, URD), OU
- Ticker boursier (ex: `ADI`, `IAM`, `SAN.PA`, `AAPL`, `HSBA.L`) — dans ce cas demander à l'utilisateur le PDF
- Optionnel : `--year=YYYY` (par défaut : année la plus récente du rapport)
- Optionnel : `--market=BVC|EURONEXT|LSE|NYSE|NASDAQ|TADAWUL|EGX|...` (par défaut : déduit du ticker ou du rapport)
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
|---|---|---|---|
| BVC / Maroc (MAD) | ~3% | ~7% | Bons du Trésor 10Y |
| Euronext (EUR) | ~3% | ~5,5% | OAT 10Y / Bund 10Y |
| LSE (GBP) | ~4% | ~5,5% | Gilt 10Y |
| NYSE/NASDAQ (USD) | ~4% | ~5% | UST 10Y |
| Émergents MENA | ~5-8% | ~7-10% | + spread souverain |

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
- **Multiples actuels** : PER, P/B, EV/EBITDA, EV/CA, dividend yield, FCF yield
- **Comparables sectoriels** (locaux + internationaux pertinents)
- **DCF simplifié** : 3-5 ans de projection FCF + valeur terminale (g = 2-3%, WACC calculé en S04), sensibilité au WACC et g
- **Scénarios Bull / Base / Bear** : 3 cibles de cours avec hypothèses explicites
- **Cible 12 mois** + upside/downside vs cours actuel
- **Rendement total attendu** = upside cours + dividend yield + buyback yield

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
- **Pas de déploiement web automatique** : laisser l'utilisateur gérer le partage
- **Pas de mock/fake data** : si une donnée manque dans le rapport, le mentionner explicitement (ex: "Non publié par l'émetteur — à demander en IR")
- **Conformité réglementaire** : ne pas reformuler les données réglementées au-delà du raisonnable, citer le rapport comme source primaire (AMMC pour Maroc, AMF pour France, SEC pour USA, FCA pour UK, etc.)
- **Disclaimer** : toujours rappeler que l'analyse est éducative / informationnelle et ne constitue pas un conseil en investissement personnalisé

## Référence

- Exemple complet de livrable : `examples/adh-2025-reference.html` (Addoha FY2025, BVC, dark Bloomberg style, responsive complet, FR)
