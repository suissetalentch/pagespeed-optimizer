# PageSpeed Optimizer

Skill pour atteindre 100% sur les 4 catégories Google PageSpeed Insights.

```
┌─────────────────────────────────────────────────────────┐
│                   PageSpeed 100%                        │
├──────────────┬──────────────┬─────────────┬────────────┤
│ Performance  │Accessibilité │Best Practices│    SEO    │
│  LCP < 2.5s  │    WCAG      │  Security   │   Meta    │
│  FCP < 1.8s  │    ARIA      │  Headers    │  Schema   │
│  CLS < 0.1   │  Contrast    │   HTTPS     │ Canonical │
│  TBT < 200ms │   Labels     │             │           │
└──────────────┴──────────────┴─────────────┴────────────┘
```

## Comment ça marche

Ce skill est un **Claude Code Skill** - une extension qui ajoute des connaissances et méthodologies spécialisées. Quand invoqué, il:

1. **Détecte automatiquement** le framework du projet (Laravel, React, Next.js)
2. **Analyse** le rapport PageSpeed ou l'URL fournie
3. **Applique** les patterns d'optimisation appropriés
4. **Vérifie** les corrections

## Utilisation

### Invocation manuelle (slash command)

```bash
# Analyser une URL
/pagespeed-optimizer https://example.com

# Analyser un rapport JSON exporté
/pagespeed-optimizer ./pagespeed-report.json

# Audit complet du projet
/pagespeed-optimizer --audit

# Corriger une catégorie spécifique
/pagespeed-optimizer --fix performance
/pagespeed-optimizer --fix accessibility
```

### Invocation automatique

Le skill s'active automatiquement quand vous mentionnez:
- "PageSpeed", "Core Web Vitals", "LCP", "FCP", "CLS"
- "optimiser les performances", "améliorer le score"
- "accessibilité WCAG", "security headers"

### Exemples de prompts

```
"Mon site a un LCP de 4.5s, aide-moi à l'optimiser"

"Voici mon rapport PageSpeed, corrige les problèmes d'accessibilité"

"Ajoute les security headers manquants pour Best Practices"

"Optimise ce projet Laravel pour PageSpeed 100%"
```

## Structure des fichiers

```
pagespeed-optimizer/
├── SKILL.md              # Définition du skill et méthodologie principale
├── laravel-patterns.md   # Patterns Laravel + Vite + Alpine.js
├── react-patterns.md     # Patterns React + Vite
├── nextjs-patterns.md    # Patterns Next.js (App Router)
├── troubleshooting.md    # Solutions aux problèmes courants
├── README.md             # Ce fichier
└── LICENSE               # MIT
```

### Contenu des fichiers

| Fichier | Rôle |
|---------|------|
| `SKILL.md` | Point d'entrée, workflow en 3 étapes, checklist, patterns essentiels |
| `*-patterns.md` | Code prêt à l'emploi pour chaque framework |
| `troubleshooting.md` | Diagnostic et solutions par message d'erreur PageSpeed |

## Frameworks supportés

| Framework | Fichier | Particularités |
|-----------|---------|----------------|
| Laravel + Vite | [laravel-patterns.md](laravel-patterns.md) | Blade, middleware PHP, Alpine.js |
| React + Vite | [react-patterns.md](react-patterns.md) | Components, lazy loading, hooks |
| Next.js | [nextjs-patterns.md](nextjs-patterns.md) | App Router, Image, metadata API |

## Workflow du skill

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   ANALYZE    │────▶│     FIX      │────▶│    VERIFY    │
├──────────────┤     ├──────────────┤     ├──────────────┤
│• Detect stack│     │• Performance │     │• Build prod  │
│• Read report │     │• Accessibility│    │• Re-test     │
│• Prioritize  │     │• Best Pract. │     │• Confirm 100%│
│              │     │• SEO         │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
```

### Priorités d'optimisation

| Priorité | Impact | Exemples |
|----------|--------|----------|
| CRITICAL | >10 pts | Image LCP, ressources bloquantes |
| HIGH | 5-10 pts | Images non optimisées, preconnect manquant |
| MEDIUM | 2-5 pts | Cache headers, resource hints |
| LOW | <2 pts | Optimisations mineures |

## Fonctionnalités avancées

### Détection automatique du contexte

Le skill exécute automatiquement ces commandes au démarrage pour détecter le framework:

```bash
cat package.json    # Dépendances npm
cat composer.json   # Dépendances PHP
ls vite.config.*    # Configuration build
```

### Outils autorisés

Le skill a accès à ces outils sans demander permission:
- `Read`, `Write`, `Edit` - Modification des fichiers
- `Bash` - Commandes de build et diagnostic
- `Grep`, `Glob` - Recherche dans le code
- `WebFetch` - Récupération des rapports PageSpeed

## Installation

### Prérequis

Ce skill nécessite [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installé.

### Installation

```bash
# Cloner dans le dossier skills
git clone https://github.com/suissetalentch/pagespeed-optimizer.git \
  ~/.claude/skills/pagespeed-optimizer
```

Le skill sera disponible immédiatement dans toutes vos sessions Claude Code.

## Métriques cibles

| Métrique | Cible | Impact |
|----------|-------|--------|
| LCP (Largest Contentful Paint) | < 2.5s | Performance |
| FCP (First Contentful Paint) | < 1.8s | Performance |
| CLS (Cumulative Layout Shift) | < 0.1 | Performance |
| TBT (Total Blocking Time) | < 200ms | Performance |
| Contrast Ratio | 4.5:1 (text) | Accessibilité |
| Security Headers | 6 headers | Best Practices |

## Licence

MIT - Baptiste Chevassut
