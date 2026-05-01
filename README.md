# claude-skill-rfa-bvc-analyzer

> **Claude Code Skill** — Analyse fondamentale institutionnelle d'un Rapport Financier Annuel (RFA) d'une société cotée à la **Bourse de Casablanca (BVC / MASI)**, avec génération automatique d'une présentation HTML interactive responsive de niveau Bloomberg Terminal.

## Ce que fait ce skill

À partir d'un PDF de RFA d'une société cotée marocaine, le skill produit :

1. **Une note d'analyse 8 sections** (FR, ton sceptique audit) :
   - Section 01 — Compte de résultat (P&L)
   - Section 02 — Cash-flow & BFR
   - Section 03 — Structure financière
   - Section 04 — Rentabilité & création de valeur (ROIC vs WACC)
   - Section 05 — Annexes qualitatives (opinion CAC, KAM, parties liées, hors bilan)
   - Section 06 — Red Flags (mindset audit, classés par sévérité)
   - Section 07 — Valorisation (multiples + DCF + scénarios Bull/Base/Bear)
   - Section 08 — Synthèse exécutive (recommandation, cible 12M, thèse)

2. **Une présentation HTML interactive responsive** :
   - Style Bloomberg Terminal dark
   - Navigation sidebar avec scroll-spy
   - Charts Chart.js (waterfall P&L, doughnut BFR, ROIC vs WACC, scénarios DCF)
   - Mobile-first (sidebar off-canvas + hamburger ≤ 768px)
   - Meta tags Open Graph pour partage WhatsApp / LinkedIn
   - Fichier unique HTML standalone, partageable via Netlify Drop / Surge / GitHub Pages

## Aperçu

Voir [`examples/adh-2025-reference.html`](examples/adh-2025-reference.html) — analyse complète d'Addoha (ADH:CSE) sur les comptes 2025 (~85 KB, ~2200 lignes, dark theme, fully responsive).

## Installation

### Via Claude Code (skill personnel)

```bash
git clone https://github.com/LKABAL/claude-skill-rfa-bvc-analyzer.git ~/.claude/skills/analyse-rfa-institutionnelle
```

Le skill est immédiatement disponible dans Claude Code — il s'auto-déclenche quand vous demandez une analyse RFA d'une société cotée BVC.

### Vérifier l'installation

Dans Claude Code :
```
/skills
```
Vous devriez voir `analyse-rfa-institutionnelle` dans la liste.

## Utilisation

### Invocation explicite

```
/analyse-rfa-institutionnelle chemin/vers/rfa-societe-2024.pdf
```

### Invocation naturelle

Demandez simplement à Claude Code :

- *"Analyse le RFA de IAM 2024"* (puis fournir le PDF)
- *"Investment note pour ATW sur l'exercice 2024"*
- *"Voici le rapport annuel d'ADI 2024, fais-moi l'analyse institutionnelle"*

### Arguments optionnels

```
/analyse-rfa-institutionnelle <pdf-path> [--year=YYYY] [--out-dir=path]
```

- `--year` : année à analyser si le RFA contient plusieurs exercices (défaut : le plus récent)
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

Le skill choisit automatiquement une couleur d'accent visuelle cohérente avec le secteur de la société :

| Secteur | Couleur | Hex |
|---|---|---|
| Immobilier | Amber | `#f59e0b` |
| Banques | Blue | `#3b82f6` |
| Télécoms | Purple | `#8b5cf6` |
| Mines / Énergie | Red / Emerald | `#ef4444` / `#10b981` |
| Agro / Conso | Green | `#22c55e` |
| Assurance | Cyan | `#06b6d4` |
| BTP / Cimenterie | Slate | `#a3a3a3` |

### Recommandations

Le skill suit l'échelle institutionnelle classique :
- **ACHAT FORT** — upside > 25% + faible risque
- **ACHAT** — upside 10-25%
- **NEUTRE** (biais positif/négatif) — upside ±10%
- **VENTE** — downside 10-25%
- **VENTE FORTE** — downside > 25%

## Contraintes & garde-fous encodés

- **Aucun mock data** : si une donnée manque dans le RFA, le skill le mentionne explicitement
- **Conformité AMMC 43-12** : citation systématique du RFA comme source primaire
- **Mindset audit** : recherche systématique des incohérences P&L vs Cash-Flow vs Bilan
- **Pas de déploiement automatique** : l'utilisateur garde le contrôle du partage
- **Langue par défaut** : français (adaptable)

## Contexte BVC / MASI

Ce skill est calibré pour les spécificités du marché marocain :
- Référentiel comptable IFRS pour comptes consolidés (CGNC pour comptes sociaux)
- Convention de présentation BVC (montants en MMAD)
- Cours de bourse récupérables via [casablanca-bourse.com](https://www.casablanca-bourse.com)
- WACC type marché marocain (Rf MAD ~3%, ERP Maroc ~7%, β sectoriel)
- Recommandations conformes pratique sell-side locale

## Stack technique

- **Charts** : Chart.js 4.4.0 (CDN)
- **Fonts** : System fonts + JetBrains Mono pour les chiffres
- **Pas de framework** : HTML/CSS/JS vanilla, fichier unique standalone
- **Open Graph** : génération d'image dynamique via [tailgraph.com](https://tailgraph.com)

## Limitations connues

- Skill optimisé pour les RFA des sociétés cotées BVC. Pour des sociétés non-cotées ou hors Maroc, adapter manuellement.
- La qualité de l'analyse dépend de la qualité du PDF source (RFA scanné en image = extraction limitée).
- Le DCF est simplifié (3-5 ans + valeur terminale) — pour valorisation complète, compléter manuellement.

## Licence

MIT — voir [LICENSE](LICENSE).

## Auteur

**LKABAL**

## Contributions

Issues et PRs bienvenues. Pour ajouter le support d'autres bourses (Tunis, Alger, Égypte, Nigeria…), ouvrir une issue d'abord.

---

*Built for Claude Code — https://docs.claude.com/claude-code*
