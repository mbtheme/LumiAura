# Audit complet — Lumiaura.ca (version locale)

**Date :** 2 juin 2026  
**Portée :** Code source local uniquement (index.html + style.css + assets)  
**Objectif :** Identifier les vrais problèmes à corriger, priorisés par impact.

---

## 1. Résumé exécutif

Le site de Lumiaura est une page unique bien structurée avec un bon squelette technique (HTML sémantique, SVG inline, JSON-LD, responsive). **Mais il souffre de problèmes sérieux qui nuisent à la crédibilité et à la conversion.** La palette sombre-or est inadaptée au secteur de nettoyage (elle évoque le luxe/hôtellerie plutôt que la propreté/refresh). Le parcours utilisateur contient des incohérences entre les services annoncés dans le hero et ceux détaillés plus bas. Plusieurs fichiers images sont inutilisés (pollution du déploiement), la vidéo hero est manquante, et le formulaire pointe vers un Cloudflare Worker dont la fiabilité n'est pas vérifiée. **Les corrections prioritaires peuvent doubler le taux de conversion.**

---

## 2. Top problèmes critiques (priorisés)

### 🔴 CRITIQUE — P1 : Vidéo hero manquante
Le HTML référence `assets/videos/work-showcase.mp4` qui **n'existe pas** dans le dossier `assets/videos/` (dossier vide). Résultat : le hero affiche soit un écran noir, soit le poster statique. C'est la première chose que voit le visiteur. **Impact : catastrophique.** Soit tu fournis la vidéo, soit tu remplaces par une image hero de qualité.

### 🔴 CRITIQUE — P2 : Palette de couleurs inadaptée au secteur
Le site utilise un thème sombre (`--primary: #0b1a2e`, `--white: #0d1825`, `--light-bg: #0f1d30`) avec un accent or (`#c9a84c`). Ce choix évoque le luxe, la finance, ou l'hôtellerie haut de gamme. **Pour une entreprise de lavage/nettoyage à Montréal, c'est un non-sens.** Le secteur utilise du bleu clair/cyan (propreté, eau, fraîcheur), du blanc (pureté), du vert (écologie). L'or sur fond noir fait "agence de luxe", pas "entreprise de services locaux fiable". Les propriétaires résidentiels cherchent confiance et accessibilité, pas prestige.

### 🔴 CRITIQUE — P3 : Incohérence des services annoncés
Le `<meta description>` et le hero mentionnent : *"Véhicules anciennes, véhicule commercial, gouttières, haute pression, réfrigération"*.  
Les cartes de services détaillent : *Vitres résidentiel, Vitres commercial, Gouttières, Lavage de pression, Lavage de revêtement*.

**C'est contradictoire.** "Véhicules anciennes" et "réfrigération" n'apparaissent nulle part dans le corps du site. Le visiteur est perdu dès les premières secondes. Soit tu offres ces services (et tu les détailles), soit tu les retires du hero. **Cette incohérence détruit la crédibilité.**

### 🟠 IMPORTANT — P4 : Images inutilisées — pollution du déploiement
Le dossier `assets/images/` contient **30 fichiers**, dont seulement **12 sont utilisés dans le HTML**. Les fichiers orphelins suivants seront déployés inutilement (~8 Mo) :

| Fichier | Taille | Statut |
|---|---|---|
| `before-after.jpg` | 794 Ko | Non utilisé |
| `business-waterfed.jpg` | 210 Ko | Non utilisé |
| `gallery5.jpg` | 473 Ko | Non utilisé |
| `gallery6.jpg` | 52 Ko | Non utilisé |
| `hero_poster.jpg` | 61 Ko | Non utilisé (remplacé par `work-showcase-poster.jpg`) |
| `hero-window.jpg` | 794 Ko | Non utilisé |
| `ig1.jpg` à `ig6.jpg` | ~310 Ko cumul | Non utilisés |
| `img-4416.jpg` | **1,5 Mo** | Non utilisé |
| `logo.jpg` | 58 Ko | Non utilisé (logo.png est utilisé) |
| `preview2.jpg` | 223 Ko | Non utilisé |
| `gallery*_poster.jpg` (×4) | ~570 Ko cumul | Non utilisés |

**Total orphelins : ~3,6 Mo.** À supprimer avant déploiement.

### 🟠 IMPORTANT — P5 : Format d'image incohérent et optimisation manquante
Les images de services et galerie utilisent des formats mélangés sans stratégie :
- `window-cleaning-residential.jpg` → **WebP** (187 Ko, 1066×600) ✓ bon choix
- `window-cleaning-commercial.jpg` → **JPEG baseline** (125 Ko, 1747×1139) — pas progressif
- `gallery2.jpg` → **AVIF** (52 Ko) ✓ excellent mais extension `.jpg` trompeuse
- `gallery3.jpg` → **WebP** (270 Ko, 2087×1325) — extension `.jpg` trompeuse
- `gallery4.jpg` → **PNG RGBA** (4,8 Mo, 1706×1474) — **absurde.** Un PNG de 4,8 Mo pour une image de galerie. À convertir en WebP/AVIF immédiatement.
- `logo.png` → **PNG 361 Ko** (1132×1132) — acceptable si transparence nécessaire, mais un SVG serait idéal

Les extensions `.jpg` pour des fichiers WebP/AVIF peuvent causer des problèmes de cache et de CDN. Renommer correctement ou utiliser `<picture>` avec `type="image/webp"`.

### 🟠 IMPORTANT — P6 : Logo trop grand dans la navbar
Le logo est affiché à `height: 156px` sur desktop. C'est **énorme** pour une barre de navigation. La plupart des sites professionnels utilisent entre 32 et 50 px de hauteur de logo. À 156 px, le logo domine la navbar et déséquilibre toute la hiérarchie visuelle. Sur mobile il passe à 60 px (plus raisonnable).

### 🟡 MODÉRÉ — P7 : Formulaire sans fallback
Le formulaire envoie les données via `fetch()` vers un Cloudflare Worker (`lumi-form-handler.lumiaura1.workers.dev`). Si ce worker est down, le visiteur voit juste "❌ Erreur, réessayez" et perd sa demande. **Il n'y a aucun fallback** (pas de `mailto:`, pas d'intégration Formspree/Netlify Forms, pas de message alternatif). Ajouter un fallback ou au minimum un message avec les coordonnées directes.

### 🟡 MODÉRÉ — P8 : Meta description vs contenu réel
Le `<title>` dit "Services de lavage professionnel" et le `og:description` mentionne "Véhicules anciennes, commercial, gouttières, haute pression, réfrigération". Si ces services ne sont pas tous dans le site, **Google va penaliser le CTR** car le snippet ne correspondra pas à la page.

### 🟡 MODÉRÉ — P9 : Section "Video Break" redondante
La section `.video-break` duplique la même vidéo manquante (`work-showcase.mp4`) qu'au hero. Même problème technique, et en plus c'est du contenu redondant. Si la vidéo n'existe pas, cette section affiche juste un poster statique avec un texte qui dit "Notre différence" — on pourrait mieux utiliser cet espace.

### 🟡 MODÉRÉ — P10 : Email Outlook pour une entreprise
`lumiaura1@outlook.com` est visible dans le HTML, le JSON-LD et le formulaire. Pour une entreprise professionnelle, **une adresse @outlook.com fait amateur.** Un email en `@lumiaura.ca` coûterait ~5$/mois et ferait une différence massive en perception de professionnalisme.

---

## 3. Audit esthétique détaillé

### 3.1 Palette de couleurs

| Variable | Valeur | Usage | Verdict |
|---|---|---|---|
| `--primary` | `#0b1a2e` (bleu nuit très sombre) | Top bar, footer, stats bar | Trop sombre pour le secteur |
| `--accent` | `#c9a84c` (or/moutarde) | CTA, liens, badges | Inadapté — évoque luxe, pas propreté |
| `--light-bg` | `#0f1d30` (bleu foncé) | Sections alternées | Fond quasi-noir sur un site de nettoyage |
| `--white` | `#0d1825` (pas du blanc !) | Fond body | Variable mal nommée — c'est un gris-bleu très sombre |
| `--text-dark` | `#e8e8e8` (gris clair) | Texte principal | OK pour fond sombre mais fatigue visuelle |
| `--text-medium` | `#aaaaaa` | Texte secondaire | Lisibilité acceptable |
| `--text-light` | `#666666` | Texte tertiaire | **Trop faible sur fond sombre** — contraste insuffisant |

**Verdict global :** La palette est conçue pour un site de luxe/hôtellerie, pas pour une entreprise de services de nettoyage. Le contraste or-sur-noir est élégant mais ne communique pas "propreté", "fraîcheur", "confiance" — les émotions clés du secteur.

**Recommandation :** Pivoter vers une palette eau/propreté :
- Primaire : bleu clair/cyan (`#0ea5e9` ou `#0284c7`)
- Accent : vert émeraude (`#10b981`) ou bleu plus vif
- Fond : blanc (`#ffffff`) avec sections en gris très clair (`#f8fafc`)
- Texte : gris foncé (`#1e293b`) — inversion complète du modèle actuel

### 3.2 Typographie

- **Police :** Inter (Google Fonts) — excellent choix, moderne et lisible.
- **Poids utilisés :** 400, 500, 600, 700, 800, 900. C'est bien.
- **Taille H1 hero :** `clamp(2.4rem, 5.5vw, 3.8rem)` — bonne approche responsive.
- **Hiérarchie :** Globalement correcte. Les `h2` des sections sont bien dimensionnés avec `clamp()`.

**Problèmes :**
- Le `letter-spacing: -.03em` sur le H1 est un peu agressif. Acceptable mais au bord du trop serré.
- Les textes secondaires (`--text-medium: #aaaaaa`) sur fond sombre peuvent être difficiles à lire pour les personnes âgées (cible potentielle du résidentiel).
- L'import de font est dupliqué : une fois dans le `<head>` HTML et une fois dans le CSS via `@import`. Supprimer celui du CSS.

### 3.3 Espacement et mise en page

- **Sections :** Padding de `96px 0` sur desktop, `48px 0` sur mobile — bon rythme.
- **Container :** `max-width: 1200px` — standard, bien.
- **Grilles :** Services en 3 colonnes, stats en 4, galerie en 3 avec items "large" — cohérent.
- **Mobile :** Les media queries à 768px et 480px sont bien gérées.

**Problèmes :**
- La section `.trust-bar` a un padding de seulement `48px 0` alors que les autres sections ont `96px`. Ça crée un rythme inégal — ça semble pressé comparé au reste.
- La `.stats-bar` est collée directement après le hero sans transition visuelle claire. Le gradient aide mais c'est subtil.

### 3.4 Images — analyse technique

| Image | Format | Taille | Dimensions | Verdict |
|---|---|---|---|---|
| `logo.png` | PNG RGBA | 361 Ko | 1132×1132 | OK si transparence requise. **SVG recommandé.** |
| `hero-window.jpg` | JPEG progressive | 794 Ko | 3024×1702 | Non utilisé. Dimensions excessives. |
| `window-cleaning-residential.jpg` | WebP | 187 Ko | 1066×600 | ✓ Bon ratio taille/qualité |
| `window-cleaning-commercial.jpg` | JPEG baseline | 125 Ko | 1747×1139 | Convertir en WebP, redimensionner |
| `gallery1.jpg` | JPEG progressive | 473 Ko | 1280×853 | Acceptable mais lourd |
| `gallery2.jpg` | AVIF | 52 Ko | — | ✓ Excellent compression. Renommer `.avif` |
| `gallery3.jpg` | WebP | 270 Ko | 2087×1325 | OK mais extension trompeuse |
| `gallery4.jpg` | **PNG RGBA** | **4,8 Mo** | 1706×1474 | **🚨 CRITIQUE** — Convertir en WebP/AVIF immédiatement |
| `preview1.jpg` | JPEG progressive | 269 Ko | 1080×1440 | OK |
| `gutter-cleaning.jpg` | JPEG progressive | 92 Ko | 1000×666 | ✓ Bien optimisé (jpeg-recompress) |
| `pressure-washing.jpg` | JPEG progressive | 112 Ko | 960×500 | OK |
| `siding-cleaning.jpg` | JPEG progressive | 95 Ko | 719×735 | OK |

**Total poids images utilisées :** ~6,3 Mo (dont 4,8 Mo pour gallery4.png seul)  
**Cible après optimisation :** ~2,5 Mo max (en convertissant tout en WebP/AVIF)

### 3.5 Design mobile

- **Navbar :** Menu hamburger avec slide-in latéral — fonctionnel.
- **Hero :** Passage à `min-height: auto` — bien.
- **Grilles :** Passage correct en 1 ou 2 colonnes selon breakpoint.
- **Boutons :** Pleine largeur sur mobile — bien pour le touch.

**Problèmes :**
- Le logo passe de 156 px (desktop) à 60 px (mobile). C'est un saut brutal. Sur mobile c'est mieux mais sur desktop c'est excessif.
- La top bar sur mobile affiche tout en une ligne avec `font-size: .75rem` — les informations risquent d'être illisibles sur petit écran.
- Le formulaire sur mobile : les checkboxes de services prennent beaucoup de vertical space. Considérer un `<select multiple>` ou des toggles plus compacts.

### 3.6 Comparaison secteur

Les concurrents montréalais typiques (Netto-Pro, Fenêtre Plus, etc.) utilisent :
- **Fond blanc** avec accents bleus/verts
- **Photos lumineuses** de résultats avant/après
- **Couleurs vives** qui évoquent la propreté
- **Design épuré** avec beaucoup d'espace blanc

Le site actuel est dans l'autre extrémité du spectre : sombre, doré, "premium". Pour le positionnement haut de gamme c'est cohérent en théorie, mais en pratique ça ne communique pas le bon message pour le secteur.

---

## 4. Audit parcours utilisateur (UX Flow)

### 4.1 Navigation

**Structure :** Services → Pourquoi nous → Galerie → FAQ → Avis → Contact

**Verdict :** Globalement logique. Le bouton "Devis gratuit" dans la navbar est bien placé et visible.

**Problèmes :**
- La navigation smooth scroll est implémentée mais **elle ne compense pas la hauteur de la navbar fixe**. Les ancres sont masquées derrière la navbar (~120 px). Ajouter `scroll-padding-top` ou un offset JS.
- Le menu mobile n'a pas d'overlay sombre pour fermer en cliquant à l'extérieur — l'utilisateur doit cliquer précisément sur le hamburger.

### 4.2 Hiérarchie de l'information

**Hero :**
- Badge "4.9/5 sur Google" — excellent, crée la confiance immédiatement.
- H1 "Vos vitres méritent un *regard neuf*" — bon slogan, mémorable.
- Sous-titre mentionne des services contradictoires (voir P3) — **problème majeur.**
- CTAs clairs : "Devis gratuit" + "Appeler maintenant" — bien.
- Stats (24h, 0$, 100%) — bonnes preuves rapides.

**Problème :** Le sous-titre du hero liste 5 services dont certains n'existent pas dans le reste du site. Le visiteur intelligent va se méfier.

### 4.3 Call-to-Action (CTA)

| CTA | Emplacement | Verdict |
|---|---|---|
| "Devis gratuit en 2 minutes" | Hero | ✓ Bon, clair |
| "Appeler maintenant" | Hero | ✓ Bien pour les mobiles |
| "Demander un devis" | Chaque carte service | ✓ Bon pattern |
| "Estimer mon projet" | Video break | OK mais redondant |
| "Devis gratuit en 2 minutes" | CTA banner (bas) | ✓ Bonne répétition |
| Bouton navbar "Devis gratuit" | Navigation | ✓ Bien visible |

**Verdict :** La densité de CTA est bonne (6 points de conversion). Le rythme est correct.

### 4.4 Formulaire de contact

**Structure :** Nom, Téléphone, Email, Services (checkboxes), Adresse, Détails + honeypot anti-spam.

**Points forts :**
- Honeypot `_honey` — bonne protection anti-bot.
- Aperçu en temps réel des données saisies — excellente UX, rare sur ce type de site.
- Checkboxes pour les services — clair et complet.
- Message de confidentialité sous le bouton — rassurant.

**Problèmes :**
- **Pas de validation côté client au-delà de `required`.** Le téléphone n'a pas de pattern regex. L'email n'est validé que par le type HTML (suffisant mais basique).
- **Le champ "Adresse du chantier" n'est pas `required`** — c'est probablement volontaire, mais pour un devis précis, l'adresse est essentielle.
- **Si le Cloudflare Worker échoue, zéro fallback.** Le visiteur perd tout. Ajouter un message : "En cas de problème, appelez-nous directement au 514-923-6906."
- **Les checkboxes ne sont pas marquées comme obligatoires** — le JS force la sélection d'au moins un service avec un `alert()` brut. Remplacer par une validation intégrée au formulaire.

### 4.5 Éléments de confiance (preuve sociale)

| Élément | Présence | Verdict |
|---|---|---|
| Note Google 4.9/5 | Hero badge + section témoignages | ✓ Bien placé |
| Lien vers Google Reviews | Section témoignages | ✓ Excellent |
| Garantie 24h | Hero stats + Trust bar + FAQ | ✓ Bien répété |
| Assurances mentionnées | Why Us + Contact info | ✓ Rassurant |
| Nombre de clients (1000+) | Stats bar | ⚠️ Nombre animé mais pas sourcé |
| Photos de travail réel | Galerie + services | ✓ Bon mais voir problèmes images |
| Logos clients/partenaires | **Absent** | ❌ Manquant — ajouter 2-3 logos renforcerait la confiance |
| Certifications/licences | **Absent** | ❌ Mentionner les assurances par nom (ex: "Assuré par Belcourt") |

### 4.6 Ordre des sections — analyse du scroll

```
1. Hero (accroche + CTA)          ← Bien
2. Stats bar (preuves rapides)     ← Bien
3. Services (offre détaillée)      ← Bien
4. Trust bar (valeurs)             ← OK mais trop fin
5. Video break (différenciation)   ← ❌ Vidéo manquante + redondant
6. Pourquoi nous (confiance)       ← Bien
7. Galerie (preuves visuelles)     ← Bien
8. Contact / Formulaire            ← Bien placé
9. FAQ (objections)                ← ⚠️ Avant les témoignages, c'est bizarre
10. Témoignages (preuve sociale)   ← ⚠️ Trop tard dans le scroll
11. CTA banner                     ← Bien
12. Footer                         ← Complet
```

**Problème d'ordre :** Les témoignages (section 10) sont **après** la FAQ et après le formulaire. Dans un parcours de conversion, la preuve sociale doit venir **avant** le formulaire pour rassurer le visiteur AVANT qu'il s'engage à remplir ses données personnelles.

**Ordre recommandé :**
```
Hero → Stats → Services → Trust bar → Pourquoi nous → Galerie → Témoignages → FAQ → Contact → CTA → Footer
```

---

## 5. Audit technique rapide

### 5.1 Performance

| Métrique | Valeur | Verdict |
|---|---|---|
| HTML | 34 Ko | ✓ Léger, bien structuré |
| CSS | 22 Ko | ✓ Compact |
| Images utilisées | ~6,3 Mo | ❌ Trop lourd (gallery4.png = 4,8 Mo seul) |
| Images orphelines | ~3,6 Mo | ❌ À supprimer |
| JavaScript | Inline (~5 Ko) | ✓ Minimal |
| Police Inter | Google Fonts | ⚠️ Blocage render — ajouter `&display=swap` (déjà présent, OK) |
| Vidéo hero | **Manquante** | 🚨 Problème bloquant |
| Total estimated load | ~10 Mo+ | ❌ Cible : < 3 Mo après optimisation |

### 5.2 SEO de base

| Élément | Statut | Détails |
|---|---|---|
| `<title>` | ⚠️ | "Lumiaura — Services de lavage professionnel | Montréal" — OK mais mentionne des services absents |
| `<meta description>` | ❌ | Mentionne "véhicules anciennes, réfrigération" — incohérent avec le contenu |
| `og:*` tags | ⚠️ | Mêmes problèmes que meta description |
| `canonical` | ✓ | Présent |
| JSON-LD (LocalBusiness) | ✓ | Bien structuré. Vérifier que les services listés correspondent |
| `lang="fr"` | ✓ | Correct |
| `viewport` | ✓ | Correct |
| Balises H1-H3 | ⚠️ | Un seul H1 (bien). Mais le H1 ne contient pas de mot-clé fort ("lavage de vitres Montréal") |
| Alt text images | ✓ | Tous présents et descriptifs |
| `loading="lazy"` | ✓ | Bien appliqué sur les images hors hero |

### 5.3 Accessibilité

| Élément | Statut |
|---|---|
| `aria-label` sur boutons icon | ✓ `navToggle`, `heroPlayBtn` |
| Contraste texte/fond | ⚠️ `--text-light: #666666` sur fond sombre peut ne pas passer WCAG AA |
| Navigation clavier | ⚠️ Menu mobile sans gestion du focus trap |
| `<details>` FAQ | ✓ Accessible nativement |
| Form labels | ✓ Tous les champs ont des `<label>` |
| Skip navigation | ❌ Absent — ajouter un lien "Aller au contenu" |

---

## 6. Recommandations par priorité

### 🔴 IMMÉDIAT (avant toute mise en ligne)

1. **Fournir la vidéo `work-showcase.mp4` OU remplacer le hero par une image statique de qualité** — C'est la première impression. Un écran noir ou un poster de 61 Ko en 464×848, c'est inacceptable.
2. **Corriger l'incohérence des services** — Aligner le hero, le meta description, les OG tags et les cartes de services. Soit tu ajoutes les sections manquantes (véhicules anciennes, réfrigération), soit tu retires ces mentions du hero.
3. **Convertir `gallery4.jpg` (PNG 4,8 Mo) en WebP/AVIF** — Cette seule image pèse plus que tout le CSS + JS combinés.
4. **Supprimer les images orphelines** (~3,6 Mo de fichiers inutilisés)
5. **Corriger les extensions trompeuses** — `gallery2.jpg` est un AVIF, `gallery3.jpg` est un WebP. Renommer ou utiliser `<picture>`.

### 🟠 COURT TERME (1-2 semaines)

6. **Changer la palette de couleurs** — Pivoter vers bleu clair/blanc/vert. C'est le changement le plus impactant pour la crédibilité secteur.
7. **Réduire la taille du logo navbar** — Passer de 156 px à ~48 px sur desktop.
8. **Réorganiser l'ordre des sections** — Témoignages avant le formulaire, FAQ après.
9. **Ajouter un overlay au menu mobile** — Pour fermer en cliquant à l'extérieur.
10. **Ajouter `scroll-padding-top`** — Pour compenser la navbar fixe lors du smooth scroll.
11. **Remplacer l'email @outlook.com par @lumiaura.ca** — Impact majeur sur la perception professionnelle.
12. **Ajouter un fallback au formulaire** — Message avec numéro de téléphone si le worker échoue.
13. **Ajouter 2-3 logos clients ou certifications** — Dans la trust bar ou le footer.

### 🟡 OPTIONNEL (amélioration progressive)

14. **Convertir le logo en SVG** — Qualité vectorielle, poids minimal.
15. **Ajouter un skip navigation link** — Pour l'accessibilité clavier.
16. **Améliorer la validation du formulaire** — Pattern regex pour téléphone, validation visuelle des checkboxes.
17. **Comprimer toutes les images en WebP/AVIF** — Cible : < 2,5 Mo total.
18. **Ajouter des animations subtiles au scroll** — Fade-in sur les sections (IntersectionObserver).
19. **Optimiser le H1 pour le SEO** — Inclure "lavage de vitres Montréal" tout en gardant le slogan.
20. **Ajouter un schéma FAQPage en JSON-LD** — Pour les rich snippets Google.
21. **Augmenter le padding de `.trust-bar`** — Passer de 48 px à 64-72 px pour un rythme plus cohérent.
22. **Remplacer `alert()` par une notification inline** — Pour la validation des services dans le formulaire.

---

## 7. Score global

| Critère | Score /10 | Commentaire |
|---|---|---|
| Structure HTML | 8/10 | Sémantique, SVG inline, JSON-LD — solide |
| CSS / Responsive | 7/10 | Bien construit mais palette inadaptée |
| Parcours utilisateur | 6/10 | Bon squelette mais ordre des sections et incohérences nuisent |
| Images / Médias | 4/10 | Vidéo manquante, formats mélangés, PNG de 4,8 Mo |
| Crédibilité secteur | 5/10 | Palette luxe ne communique pas "propreté" |
| SEO technique | 6/10 | Base OK mais meta incohérents |
| Accessibilité | 6/10 | Bien commencé mais manque skip-nav et focus trap |
| **GLOBAL** | **~5.5/10** | **Bon potentiel, problèmes correctibles** |

---

> **Bottom line :** Le site a une bonne base technique. Les 5 corrections immédiates (vidéo, cohérence services, gallery4, orphelins, extensions) prennent moins de 2 heures et résolvent 80% des problèmes bloquants. Le changement de palette est le investissement le plus important mais aussi le plus impactant — c'est ce qui transformera un site "agence de luxe confuse" en un site "entreprise de nettoyage professionnelle et fiable".
