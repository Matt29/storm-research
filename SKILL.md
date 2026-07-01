---
name: storm-recherche
description: À utiliser quand on demande de lancer une Storm Recherche, d'utiliser le skill storm-recherche, d'appliquer la méthode STORM à un sujet, dit "storm recherche sur X" / "briefing STORM sur X" / "recherche multi-angles sur X", ou veut un briefing HTML multi-perspectives, sourcé et vérifié. Déroule un pipeline en 4 phases : cinq angles experts (Praticien, Académique, Sceptique, Économiste, Historien) -> carte des contradictions -> rapport HTML synthétisé -> peer review adversarial + vérification contre sources primaires. Idéal pour les sujets où plusieurs points de vue et des faits vérifiés comptent ; surdimensionné pour une simple recherche factuelle.
argument-hint: "[sujet à rechercher]"
---

# Storm Recherche

## Ce que fait ce skill

Transforme un sujet en un briefing HTML multi-perspectives et vérifié. Il simule cinq angles experts sur le sujet, cartographie leurs contradictions, synthétise tout dans un rapport HTML autonome, puis fait son propre peer review adversarial et vérifie chaque citation contre sa source primaire avant de livrer. La sortie est un fichier HTML unique, sans angle mort et sans claim non vérifié.

Déroule le pipeline complet de bout en bout. Ne saute aucune phase. C'est plus lourd qu'une recherche web rapide, c'est le but.

C'est une adaptation maison de la méthode STORM de Stanford (Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking, NAACL 2024). Différence assumée à toujours garder en tête : le vrai STORM découvre ses perspectives dynamiquement en analysant des articles proches ; ici le panel de cinq angles est fixé à la main. C'est un raccourci efficace, pas la méthode originale. Cette honnêteté est structurelle, elle est répétée dans le rapport, jamais gommée.

## Positionnement honest broker (non négociable)

Ce skill n'existe que si ces trois garde-fous tiennent. Sans eux, ce n'est pas une Storm Recherche.

1. **Recherche réelle uniquement.** Chaque angle et chaque citation doit remonter à une source réelle, ouverte et lue. Aucune étude, aucun chiffre, aucune URL inventés. Si un chiffre n'est pas vérifiable, il est rétrogradé ou coupé, jamais maquillé.
2. **Le panel est construit maison.** Toujours le divulguer dans le rapport. L'accord entre les angles est une hypothèse forte, pas une preuve indépendante, et surtout pas un consensus du champ.
3. **La vérification est obligatoire.** Un rapport livré sans la Phase 4 n'est pas une Storm Recherche. La bannière de vérification doit être vraie (compte réel d'inventées / corrigées / rétrogradées).

## Portabilité

Ce skill est autonome. Il dépend uniquement des outils intégrés de Claude Code (l'outil `Agent` avec l'agent `general-purpose`, `Write`, et la recherche/fetch web utilisée dans ces agents) plus `report-template.html` dans ce même dossier. Aucun script externe, API, service payant ou autre skill requis. Dépose le dossier dans n'importe quel `.claude/skills/` et ça marche.

## Langue et style

Sortie en français, vouvoiement neutre côté rapport (c'est un livrable). Ton sobre, factuel, sans survente. Pas de tirets cadratins, des virgules. Charte du rapport HTML : navy #0D1B2A, orange #F47C2B, cream #F5F5F0, police Inter. Garde le CSS du template tel quel, ne change pas de style visuel.

## Phase 0 : cadrer le sujet

1. Si `$ARGUMENTS` contient le sujet, utilise-le. Sinon, demande quoi rechercher.
2. Énonce ton interprétation du sujet en une ligne et avance. Ne pose une question de clarification que si le sujet est réellement ambigu d'une façon qui change la recherche. Par défaut, avance.
3. Identifie le **rôle du lecteur** pour que la section "action" le vise. Déduis-le du sujet et du contexte donné ; si ce n'est pas clair, demande en une ligne, ou prends par défaut "un créateur ou décideur dans ce domaine".
4. Dérive un `topic-slug` en kebab-case à partir du sujet, pour le nom de fichier.
5. Dis au lecteur que le pipeline tourne (5 angles, puis vérification). Une ligne.

## Phase 1 : cinq angles experts (agents en parallèle)

Lance **cinq agents `general-purpose` dans un seul message** pour qu'ils tournent en parallèle. Chacun reçoit le MÊME cadrage du sujet plus son propre angle. Utilise ces prompts exacts, en substituant `{SUJET}` et un `{CADRE}` d'une ligne (ton interprétation de Phase 0) :

**1. LE PRATICIEN** — `Tu es LE PRATICIEN pour : {SUJET} ({CADRE}). Tu travailles ce sujet au quotidien. Fais de la vraie recherche web (priorité aux sources récentes, études de cas, retours d'opérateurs, données terrain). Fais ressortir l'ÉCART entre ce que savent les gens de terrain et ce que ratent les académiques et les commentateurs, et les réalités pratiques (frictions de workflow, ce qui marche vraiment, là où ça casse) qu'on ignore d'habitude. Renvoie EXACTEMENT : 1) POSITION CENTRALE en 2 phrases. 2) MEILLEURE PREUVE, 3 à 5 puces, chacune avec un point de donnée / cas / source nommée concret + URL. 3) LA SEULE CHOSE qu'un praticien dirait et que personne d'autre ne dirait. Cite de vraies sources avec URL. Moins de 400 mots.`

**2. L'ACADÉMIQUE** — `Tu es L'ACADÉMIQUE pour : {SUJET} ({CADRE}). Tu te fies aux preuves évaluées par les pairs et aux tailles d'effet, pas aux anecdotes. Fais de la vraie recherche web (études évaluées par les pairs, arXiv, rapports d'universités et d'instituts de recherche, revues). Réponds : que dit VRAIMENT la preuve rigoureuse vs la croyance populaire, et où CONTREDIT-elle le hype. Renvoie EXACTEMENT : 1) POSITION CENTRALE en 2 phrases. 2) MEILLEURE PREUVE, 3 à 5 puces, chacune reliée à une étude/rapport nommé + URL avec le résultat ou l'effet réel. 3) LA SEULE CHOSE qu'un académique dirait. Signale là où la preuve est mince ou disputée, et note le statut (publié vs preprint). Moins de 400 mots.`

**3. LE SCEPTIQUE** — `Tu es LE SCEPTIQUE pour : {SUJET} ({CADRE}). Tu penses que la vue dominante est surestimée ou fausse. Construis le steelman le plus solide du scénario pessimiste. Fais de la vraie recherche web pour trouver les backlash, échecs, données contradictoires, changements réglementaires, démontages. Réponds : le contre-argument le plus fort, et ce que les partisans ignorent commodément. Renvoie EXACTEMENT : 1) POSITION CENTRALE en 2 phrases. 2) MEILLEURE PREUVE, 3 à 5 puces, chacune avec source concrète + URL. 3) LA SEULE CHOSE qu'un sceptique dirait. Sois rigoureux, pas contrarian pour le sport. Cite de vraies sources avec URL. Moins de 400 mots.`

**4. L'ÉCONOMISTE** — `Tu es L'ÉCONOMISTE pour : {SUJET} ({CADRE}). Tu suis l'argent. Fais de la vraie recherche web sur les revenus, valorisations, taille de marché, flux de financement, unit economics, incitations. Réponds : qui profite du narratif actuel, et quelles incitations financières façonnent la recherche et le hype. Renvoie EXACTEMENT : 1) POSITION CENTRALE en 2 phrases. 2) MEILLEURE PREUVE, 3 à 5 puces, chacune avec un vrai chiffre (revenu/valorisation/taille de marché/financement) + source nommée + URL. 3) LA SEULE CHOSE qu'un économiste dirait (l'insight follow-the-money). Cite de vrais chiffres avec URL. Moins de 400 mots.`

**5. L'HISTORIEN** — `Tu es L'HISTORIEN pour : {SUJET} ({CADRE}). Tu as déjà vu des cycles de disruption et tu cherches les motifs. Fais de la vraie recherche web pour trouver de vrais parallèles historiques (technologies antérieures, manies, bascules de marché). Réponds : quels parallèles collent vraiment, et ce qu'on apprend de leur déroulé (qui a gagné, qui a perdu, ce qui s'est stabilisé). Renvoie EXACTEMENT : 1) POSITION CENTRALE en 2 phrases. 2) MEILLEURE PREUVE, 3 à 5 puces, chacune un cas historique précis avec dates/résultats + URL source. 3) LA SEULE CHOSE qu'un historien dirait (le motif que personne d'autre ne voit). Cite tes sources avec URL. Moins de 400 mots.`

Quand les cinq reviennent, poste une note de 2-3 lignes dans le chat : dans quel sens ils convergent, et le désaccord le plus tranchant. Garde les briefs bruts hors du chat (les agents les ont déjà renvoyés).

## Phase 2 : cartographier les contradictions

En travaillant uniquement à partir des cinq briefs, détermine (en direct, sans agents) :

1. **Conflits directs** — là où deux angles ou plus affirment le contraire. Nomme les claims précis qui s'opposent, pas juste les thèmes.
2. **Preuve la plus forte vs la plus faible** — quel angle est le mieux étayé (hiérarchie : causal évalué par les pairs > donnée officielle > anecdote/analogie) et lequel est le plus faible, avec pourquoi.
3. **La question qui tranche** — la seule question empirique qui réglerait la plus grosse contradiction.
4. **Accord universel** — ce que chaque angle confirme, même les opposants. C'est le résultat porteur probablement vrai.
5. **L'angle mort** — ce qu'AUCUN angle n'a traité. Ça devient le "6e angle manquant" et alimente la Question de frontière.

Cette carte n'est pas un livrable séparé. C'est la matière première des enseignements du rapport (soutenu/contesté), de la connexion cachée, de l'encadré 6e angle et de la question de frontière.

## Phase 3 : synthétiser le rapport HTML

1. Lis `report-template.html` dans ce dossier de skill. Clone-le, ne reconstruis pas le CSS.
2. Remplis chaque section. Correspondance depuis les phases :
   - **Résumé 60 secondes** — niveau décideur, nuance pas titre. Commence par le fait établi, puis l'interprétation contestée.
   - **5 enseignements clés, classés par fiabilité** — les choses les plus importantes désormais connues, la plus fiable en premier. Chacun porte un score de confiance 1-10 (fixé en Phase 4) et des puces Soutenu par / Contesté par tirées de la carte des contradictions.
   - **Connexion cachée** — le lien non évident de la Phase 2 qui n'apparaît qu'en croisant les cinq angles.
   - **Hypothèse / 6e angle manquant** — l'angle mort de la Phase 2, présenté comme l'angle qui pourrait changer les conclusions.
   - **Action** — 3 à 6 mouvements précis pour le rôle du lecteur identifié en Phase 0. Précis, pas abstrait.
   - **Guide "ce que tu peux affirmer"** — sûr / réserve / éviter, rempli après la vérif de Phase 4.
   - **Question de frontière** — la seule question qui changerait tout.
   - **Références** — chaque citation avec son statut de vérification (fixé en Phase 4).
3. Écris dans `storm-reports/{topic-slug}-briefing.html` (relatif au dossier de travail courant ; crée le dossier si besoin).

## Phase 4 : peer review adversarial + vérification (à ne pas sauter)

C'est ce qui sépare une Storm Recherche d'un rapport normal. Fais-la avant de livrer.

**4a. Auto-review (en direct).** Note chacun des 5 enseignements 1-10 pour la fiabilité et justifie. Identifie le maillon faible et ce qui le vérifierait. Fais un bias check (quel angle a dominé la synthèse, qu'est-ce qui a été sous-pondéré). Nomme le 6e angle manquant. Attribue une note globale honnête.

**4b. Vérifie chaque citation (agents en parallèle).** Lance des agents `general-purpose` en un seul message, un par grappe de citations distincte (regroupe les claims liés ; ~4-6 agents). Prompt de chaque agent :

`Vérifie indépendamment une citation contre sa source PRIMAIRE. Sois sceptique, ne te fie pas aux résumés de blogs secondaires. CLAIM : {claim + chiffre cité + source nommée}. Trouve la vraie source primaire. Confirme ou corrige : titre/auteurs/source/année/URL exacts, le vrai chiffre ou effet tel que publié, l'échantillon/méthode et les limites déclarées par les auteurs, et le statut (publié vs preprint). Pour tout claim disputé, trouve la contre-source crédible la plus forte. Renvoie : VERDICT = CONFIRMÉ / PARTIELLEMENT CONFIRMÉ (liste les corrections) / NON VÉRIFIÉ / FAUX, puis la citation corrigée en une ligne, puis 2-4 puces de détails avec l'URL primaire. Moins de 280 mots.`

**4c. Applique les corrections.** Édite le rapport :
- Corrige tout chiffre, titre, date ou caractérisation faux.
- Baisse les scores de confiance là où la preuve s'est révélée mince ; rétrograde les preprints et claims disputés dans l'encadré "Signal contesté".
- Ré-attribue honnêtement les stats de sondage unique ou commandité.
- Remplis la bannière de vérification (`X inventée(s), Y corrigée(s), Z rétrogradée(s)`) et les tags de statut par citation.
- Remplis le guide "ce que tu peux affirmer" à partir des verdicts.

## Sortie

1. Livrable final : `storm-reports/{topic-slug}-briefing.html` (la v2, après vérification).
2. Ouvre-le pour le lecteur avec l'ouvreur par défaut de la plateforme : macOS `open <chemin>`, Linux `xdg-open <chemin>`, Windows `start "" <chemin>`. Si l'OS n'est pas clair, donne juste le chemin.
3. Dans le chat, donne : le chemin du fichier, le compte de vérification (`N/N vérifiées, X inventée(s), Y corrigée(s), Z rétrogradée(s)`), le seul résultat universel, la question de frontière, et le résumé du guide (ce qui est sûr à affirmer vs à éviter). Reste concis.

## Garde-fous

- **Recherche réelle uniquement.** Voir positionnement honest broker. Jamais de source inventée.
- **Le panel est construit maison.** Toujours divulgué dans le rapport. La convergence est une hypothèse, pas un consensus du champ.
- **Vérification obligatoire.** Pas de Phase 4, pas de Storm Recherche. Bannière vraie.
- **Fiabilité = qualité de preuve, pas confiance.** Note selon la hiérarchie : causal évalué par les pairs > donnée officielle/financière > sondage commandité unique > analogie > preprint.
- **Vise le lecteur, pas une personne par défaut.** L'action et le guide parlent au rôle identifié en Phase 0. Reste générique si aucun rôle n'est donné.
- **Coût.** Ça lance ~9 à 11 agents par run. C'est normal. Ne dépasse pas cinq angles ni un vérificateur par grappe de citations.
- **Design.** Charte IA IRL, navy/orange/cream, Inter. Garde le CSS du template tel quel. Ne remplace pas par un autre style visuel.
