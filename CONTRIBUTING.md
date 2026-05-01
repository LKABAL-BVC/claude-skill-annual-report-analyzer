# Contributing

Merci de l'intérêt pour ce projet. Les contributions sont bienvenues — bugs, suggestions, calibrations de nouveaux marchés, améliorations du gabarit HTML.

## Comment contribuer

### 1. Signaler un bug ou suggérer une amélioration
Ouvrir une [issue](https://github.com/LKABAL-BVC/claude-skill-annual-report-analyzer/issues) avec :
- **Titre clair** : *"[Bug] La table dette ne s'affiche pas sur mobile ≤ 380px"*
- **Étapes de reproduction** (si bug)
- **Comportement attendu vs observé**
- **Société / rapport testé** (anonymiser si confidentiel)
- **Screenshot** si UI

### 2. Proposer un nouveau marché
**Priorité actuelle** : marchés développés et structurés (Europe + Amérique du Nord + Asie-Pacifique développée).

Exemples de places visées en priorité :
- **Deutsche Börse / Xetra** (Frankfurt) — DAX
- **SIX Swiss Exchange** (Zurich) — SMI
- **Borsa Italiana** (Milan) — FTSE MIB
- **BME** (Madrid) — IBEX 35
- **Euronext Amsterdam / Bruxelles / Lisbonne / Dublin / Oslo**
- **NASDAQ Nordic** (Stockholm OMX30, Helsinki, Copenhague)
- **TSX** (Toronto)
- **ASX** (Sydney)
- **TSE** (Tokyo) — TOPIX, Nikkei
- **HKEX** (Hong Kong) — pour les sociétés non-PRC

Pour proposer une nouvelle place, ouvrir une issue avec :
- **Devise de reporting** (code ISO 4217)
- **Référentiel comptable** dominant (IFRS, US GAAP, local GAAP avec convergence IFRS)
- **Régulateur** + numéro de circulaire pertinente
- **Source de cours** (URL publique préférée)
- **Calibration WACC type** (Rf 10Y govt + ERP) avec justification (source : Damodaran, KPMG, etc.)
- **Format type du rapport annuel** (PDF, HTML, XBRL/iXBRL…)

### 3. Soumettre une Pull Request

```bash
git clone https://github.com/LKABAL-BVC/claude-skill-annual-report-analyzer.git
cd claude-skill-annual-report-analyzer
git checkout -b feat/nom-de-la-feature
# ... modifications ...
git commit -m "Add support for Tunis BVMT calibration"
git push origin feat/nom-de-la-feature
```

Puis ouvrir la PR sur GitHub avec :
- Description du changement
- Lien vers l'issue associée (si applicable)
- Test : avoir testé sur au moins 1 rapport annuel réel
- Screenshot(s) si modification UI

## Conventions

### SKILL.md
- **Frontmatter** : ne pas modifier `name`, modifier `description` avec parcimonie (impacte l'auto-déclenchement Claude Code)
- **Sections** : conserver la structure 4 étapes (Extraction → Analyse 8 sections → HTML → Livraison)
- **Calibrations marché** : ajouter dans la section *"Spécificités par marché"* + dans la grille WACC
- **Vocabulaire bilingue** : si vous ajoutez un concept, donner FR + EN (ex: *"CA / Revenue"*, *"RFA / Annual Report"*)

### HTML de référence (`examples/adh-2025-reference.html`)
- **Ne PAS** ajouter d'autres exemples dans `examples/` — un seul de référence suffit (sinon le repo grossit vite, chaque HTML fait ~80-100 KB)
- Si vous voulez partager une analyse réussie, ouvrir une issue *"Showcase"* avec un lien externe vers votre HTML hébergé
- Préserver : style dark Bloomberg, sidebar off-canvas mobile, Chart.js 4.4.0, meta OG tailgraph
- Modifications cosmétiques significatives (palette, typographie) : discuter en issue avant PR

### Méthodologie d'analyse
- **Mindset audit** : toute proposition d'ajout d'un nouveau red flag type, d'un nouveau ratio, ou d'une nouvelle section est bienvenue — justifier par un exemple concret de cas où c'aurait permis d'éviter une erreur d'investissement
- **Neutralité** : pas de biais long/short systématique, ton sceptique mais factuel
- **Pas de recommandation personnalisée** : le skill produit une analyse, jamais un conseil en investissement

## Tests

Avant d'ouvrir une PR, tester sur au moins **1 rapport annuel réel** d'une société cotée (idéalement 2 : une marocaine + une internationale). Vérifier :
- [ ] L'extraction PDF fonctionne (tous les chiffres clés présents)
- [ ] Les 8 sections sont produites
- [ ] Le HTML généré s'ouvre sans erreur console
- [ ] Le mobile responsive marche (DevTools → 375px)
- [ ] Les meta OG sont valides (https://www.opengraph.xyz/)

## Code of Conduct

Bienveillance, factualité, respect. Ce projet sert l'analyse financière éducative — pas de spam, pas de promo de tokens / signaux / conseils financiers personnalisés.

## Questions

Ouvrir une [discussion](https://github.com/LKABAL-BVC/claude-skill-annual-report-analyzer/discussions) plutôt qu'une issue pour les questions ouvertes.

---

*Maintained by LKABAL ([@LKABAL-BVC](https://github.com/LKABAL-BVC)).*
