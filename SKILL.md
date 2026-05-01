---
name: analyse-rfa-institutionnelle
description: Produit une analyse fondamentale institutionnelle (8 sections — P&L, Cash/BFR, Structure financière, Rentabilité, Annexes qualitatives, Red Flags, Valorisation, Synthèse Bull/Bear) à partir du Rapport Financier Annuel (RFA) d'une société cotée à la Bourse de Casablanca, puis génère une présentation HTML interactive responsive de niveau Bloomberg Terminal pour gestionnaires institutionnels. Use when l'utilisateur demande "analyse RFA <ticker>", "investment note <société>", ou pointe un PDF de RFA d'une société cotée MASI/Maroc.
when_to_use: Demande d'analyse fondamentale d'un RFA de société cotée à la BVC (Casablanca Stock Exchange). Le livrable attendu est (1) une note buy-side/sell-side écrite et (2) une présentation HTML interactive partageable.
argument-hint: <pdf-rfa-path-or-ticker> [--year=YYYY] [--out-dir=path]
disable-model-invocation: false
allowed-tools: Read Write Bash Grep Glob WebFetch
---

# analyse-rfa-institutionnelle

Workflow complet : RFA PDF → Note d'analyse institutionnelle → Présentation HTML interactive responsive partageable via WhatsApp.

## Inputs

`$ARGUMENTS` =
- Chemin vers le PDF du RFA (Rapport Financier Annuel) de la société, OU
- Ticker BVC (ex: `ADI`, `IAM`, `ATW`) — dans ce cas demander à l'utilisateur le PDF
- Optionnel : `--year=YYYY` (par défaut : année la plus récente du RFA)
- Optionnel : `--out-dir=<path>` (par défaut : `<HOME>/<TICKER>-Analysis-<YEAR>/`)

## Persona à incarner

**Analyste financier senior buy-side/sell-side** avec 15+ ans d'expérience marchés émergents (notamment MENA/MASI). Mindset audit : sceptique, incisif, factuel. Pas de complaisance — recherche systématique des incohérences entre comptes (P&L vs Cash-Flow vs Bilan), des red flags comptables, des biais de communication management. Ton professionnel, rigoureux, en français.

## Workflow

### Étape 1 — Extraction des données du RFA

1. **Read** le PDF du RFA avec l'outil `Read` (utiliser `pages` si > 10 pages).
2. **Extraire en littéral** (pas d'inférence à ce stade) :
   - **Identité** : raison sociale, ticker BVC, secteur, date de clôture, exercice analysé + N-1 (et N-2 si dispo)
   - **Compte de résultat consolidé** : CA, EBITDA (ou calculer : RE + D&A), résultat d'exploitation, résultat financier, résultat net part du groupe, part minoritaires
   - **Bilan consolidé** : actif total, immobilisations corporelles/incorporelles/financières, stocks, créances clients, trésorerie active, capitaux propres part du groupe, intérêts minoritaires, dettes financières (CT + LT), dettes fournisseurs, autres passifs
   - **Tableau de flux de trésorerie** : CAF, ΔBFR, CAPEX, FCF (à recalculer si non publié : CFO − CAPEX), dividendes versés, variation dette nette
   - **Annexes** : méthodes comptables clés (IFRS appliquées, changements), événements post-clôture, parties liées, engagements hors bilan, litiges
   - **Rapport CAC** : opinion (clean / avec réserve / refus / impossibilité), observations, points clés d'audit (KAM)
   - **Données de marché** : cours au 31/12, nombre d'actions, capitalisation boursière, dividende proposé
   - **Communication financière** : message clé du management (lettre du Président / DG)

3. **Recouper** avec sources externes si nécessaire :
   - Cours de bourse de clôture : utiliser `WebFetch` sur `https://www.casablanca-bourse.com/live-market/emetteurs/<CODE>` ou demander à l'utilisateur
   - Communiqué AMMC associé pour vérifier conformité

### Étape 2 — Analyse 8 sections (livrable n°1 : note écrite)

Produire une note structurée en **8 sections numérotées**, en français, ton institutionnel sceptique. Chaque section doit citer les chiffres exacts du RFA et **systématiquement chercher les incohérences**.

#### Section 01 — Compte de résultat (P&L)
- Évolution CA YoY (en MMAD et %), décomposition organique vs périmètre si dispo
- Marge brute, marge EBITDA, marge opérationnelle, marge nette — comparer aux 2-3 dernières années
- **Effet ciseaux** : identifier si croissance CA ≠ croissance résultat opérationnel (et pourquoi : prix vs volume, mix produits, charges fixes)
- **Qualité du résultat net** : part de l'exceptionnel, des reprises de provisions, des plus-values de cession dans le RN
- **Drapeau rouge type** : RN qui croît fortement alors que CA stagne ou décroît → vérifier la nature des produits non-récurrents

#### Section 02 — Cash-flow & BFR
- CAF, variation BFR (avec décomposition stocks/clients/fournisseurs si dispo), CFO
- CAPEX (maintenance vs croissance si distinguable), FCF = CFO − CAPEX
- **Pont RN → FCF** : expliquer l'écart (D&A, ΔBFR, provisions, CAPEX) — c'est ICI que se cachent la plupart des manipulations
- **DSO, DIO, DPO** (Days Sales/Inventory/Payables Outstanding) — évolution et benchmark sectoriel
- **Drapeau rouge type** : RN +X% mais FCF négatif/en baisse → BFR qui explose (créances qui ne rentrent pas, stocks qui s'accumulent) ou CAPEX masqué

#### Section 03 — Structure financière
- Dette nette = Dette financière brute − Trésorerie active
- Gearing = Dette nette / Capitaux propres
- Dette nette / EBITDA (covenant typique : < 3x)
- Couverture des intérêts = EBITDA / Charges financières
- Maturité de la dette (CT vs LT), nature (bancaire vs obligataire), covenants visibles
- **Liquidité** : ratio courant, ratio rapide, position cash vs CT debt
- **Drapeau rouge type** : refinancement à venir + dégradation FCF + gearing élevé = stress de liquidité

#### Section 04 — Rentabilité & création de valeur
- ROE (RN / CP moyens), ROCE = EBIT × (1−t) / Capital employé
- **ROIC vs WACC** : calculer le WACC (CAPM : Rf MAD ~3%, ERP Maroc ~7%, β sectoriel ; coût dette après IS) — comparer à ROIC
- **Si ROIC < WACC : destruction de valeur économique** (EVA négative) — c'est un message majeur
- DuPont : ROE = Marge nette × Rotation actifs × Levier financier
- Évolution sur 3-5 ans pour identifier la trajectoire

#### Section 05 — Annexes qualitatives
- Opinion CAC + KAM (Key Audit Matters) — souvent les KAM révèlent les zones de risque
- Changements de méthode comptable, options IFRS exercées
- Engagements hors bilan (cautions, garanties, leasing opérationnel résiduel)
- Litiges significatifs (avec montants si quantifiés)
- Parties liées (transactions intra-groupe, conventions réglementées)
- Événements post-clôture
- Gouvernance : indépendance du conseil, rotation CAC

#### Section 06 — Red Flags (mindset audit)
**Lister 5 à 15 red flags** classés par sévérité (🔴 critique / 🟠 modéré / 🟡 à surveiller). Format type :
- **🔴 RF#1 — Titre** : Description factuelle citant les chiffres. Implication : ...
- Cibler particulièrement : divergence P&L vs Cash, BFR anormal, provisions/reprises opportunistes, changements de méthode, transactions parties liées, litiges sous-provisionnés, engagements hors bilan masqués, communication management vs réalité comptable, qualité du carnet de commandes vs CA reconnu.

#### Section 07 — Valorisation
- **Multiples actuels** : PER, P/B, EV/EBITDA, EV/CA, dividend yield
- **Comparables sectoriels** (BVC + émergents comparables si pertinent)
- **DCF simplifié** : 3-5 ans de projection FCF + valeur terminale (g = 2-3%, WACC calculé en S04), sensibilité au WACC et g
- **Scénarios Bull / Base / Bear** : 3 cibles de cours avec hypothèses explicites
- **Cible 12 mois** + upside/downside vs cours actuel
- **Rendement total attendu** = upside cours + dividend yield

#### Section 08 — Synthèse exécutive Bull / Bear
- **Recommandation** : ACHAT FORT / ACHAT / NEUTRE (biais positif/négatif) / VENTE / VENTE FORTE
- **Cible 12 mois** : XX MAD (vs YY MAD actuel, soit ±Z%)
- **Thèse Bull (3-5 points)** : ce qui peut surprendre positivement
- **Thèse Bear (3-5 points)** : ce qui peut décevoir
- **Catalyseurs court terme** (3-6 mois) : à monitorer
- **Top 3 KPIs à surveiller** : trimestre par trimestre

### Étape 3 — Génération de la présentation HTML interactive (livrable n°2)

Utiliser `examples/adh-2025-reference.html` comme **gabarit structurel**. Adapter :
- **Identité visuelle** : ticker, nom société, secteur, accent color (voir guide ci-dessous)
- **Données** : tous les chiffres, charts, scénarios remplacés par les valeurs de la nouvelle société
- **Recommandation & rating pill** (couleur selon recommandation)
- **Meta tags Open Graph** (titre/description/image OG via `og.tailgraph.com`) pour partage WhatsApp/LinkedIn
- **Conserver** : structure responsive (sidebar off-canvas mobile + hamburger ≤ 768px), navigation scroll-spy, dark theme Bloomberg, Chart.js 4.4.0

#### Choix de l'accent color par secteur (cohérence visuelle)
- Immobilier : `#f59e0b` (amber) — comme ADH
- Banques : `#3b82f6` (blue)
- Télécoms : `#8b5cf6` (purple)
- Mines/Énergie : `#ef4444` (red) ou `#10b981` (emerald) selon positionnement
- Agro/Conso : `#22c55e` (green)
- Assurance : `#06b6d4` (cyan)
- BTP/Cimenterie : `#a3a3a3` (slate)

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
Reprendre les media queries du fichier ADH référence :
- `≤ 1100px` : sidebar plus étroite
- `≤ 768px` : sidebar off-canvas + bouton hamburger ☰ + grids → 1 colonne + KPI bar 2 colonnes + chart-wrap 240px
- `≤ 380px` : KPI bar 1 colonne

### Étape 4 — Livraison

1. **Créer le dossier** `<out-dir>` (par défaut `<HOME>/<TICKER>-Analysis-<YEAR>/`)
2. **Écrire** `index.html` (la présentation finale)
3. **Donner à l'utilisateur** :
   - Chemin local du fichier
   - Options de partage : Netlify Drop manuel (https://app.netlify.com/drop), Surge.sh, ou hébergement statique de son choix
   - Si demande PDF : proposer impression depuis Chrome (Ctrl+P → Save as PDF, paysage A4)
4. **Résumé final** en 3-5 lignes : recommandation, cible, top 2 red flags, top 2 catalyseurs

## Contraintes critiques (rappel)

- **Langue** : français de bout en bout (analyse + UI HTML) — adaptable EN si demandé
- **Tonalité** : sceptique, factuelle, pas de superlatifs commerciaux
- **Chiffres** : toujours citer la source (page du RFA si possible) ; en MMAD selon convention BVC
- **Pas de déploiement web automatique** : laisser l'utilisateur gérer le partage (stealth mode possible)
- **Pas de mock/fake data** : si une donnée manque dans le RFA, le mentionner explicitement (ex: "Non publié par l'émetteur — à demander en IR")
- **Conformité AMMC 43-12** : ne pas reformuler les données réglementées au-delà du raisonnable, citer le RFA comme source primaire

## Référence

- Exemple complet de livrable : `examples/adh-2025-reference.html` (Addoha FY2025, dark Bloomberg style, responsive complet)
