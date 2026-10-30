# tenth-man

Un plugin de délibération multimodèle (un skill et des agents qui occupent les sièges) pour [Codex](https://developers.openai.com/codex) et [Claude Code](https://claude.com/claude-code). Utilisez-le pour un choix de conception, la revue d'une politique ou toute autre décision qui ne doit pas reposer sur la réponse d'un seul modèle. Des agents utilisant différents modèles votent et un siège est toujours réservé à la contradiction, ce qui réduit le risque que le conseil tombe d'accord tout en partageant le même angle mort.

[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (Brasil)](README.pt-BR.md) | [Deutsch](README.de.md) | [Français](README.fr.md)

Les membres votants évaluent chaque proposition. Un **dixième homme** (tenth man) distinct doit argumenter contre chaque proposition, quel que soit son propre avis, et indiquer quels éléments réfuteraient chacune de ses objections. La session principale préside trois tours, résout les préoccupations majeures et rédige un procès-verbal qui indique ce qui a été convenu, pourquoi, et ce qui reste ouvert.

Le skill présente d'abord son plan de délibération et attend votre approbation. Pour ne pas attendre, placez `fast` avant la question (par exemple `/tenth-man:council fast Devons-nous ... ?`) : la présidence affiche le plan et commence aussitôt, choisit elle-même les options par défaut et indique ces choix dans le procès-verbal. Les conclusions d'un conseil ne sont jamais appliquées automatiquement à votre projet.

## Fonctionnement

1. La présidence propose les points, les modèles, les règles, le coût et le dossier de sortie.
2. Tour 1 : les membres votent ; le dixième homme conteste chaque point.
3. Tour 2 : les membres réexaminent les préoccupations majeures et répondent aux objections les plus importantes.
4. Tour 3 : les membres votent sur le projet de procès-verbal ; le dixième homme confronte le résultat aux éléments disponibles.
5. La présidence rédige `.tenth-man/<date>-<slug>/minutes.md`.

Les votes possibles sont **accord**, **accord sous conditions**, **opposition** et **préoccupation majeure**. L'accord du conseil requiert la majorité des membres votants et qu'aucune préoccupation majeure ne subsiste. Le dixième homme ne vote pas.

## Installation pour Codex

### En tant que plugin

```sh
codex plugin marketplace add Shintaro-My/tenth-man
codex plugin add tenth-man@tenth-man
```

Pour une copie locale du dépôt, indiquez plutôt son chemin :

```sh
codex plugin marketplace add /path/to/tenth-man
codex plugin add tenth-man@tenth-man
```

Invoquez le skill du plugin en tapant `$tenth-man:council` suivi de la question, ou choisissez-le dans `/skills` :

```text
$tenth-man:council Devons-nous déplacer l'état de session de Redis vers des cookies signés ?
```

### En tant que skill autonome

Copiez `skills/council` dans un répertoire de skills Codex :

```sh
cp -r skills/council ~/.agents/skills/
```

Installé de cette façon, il s'invoque avec `$council`.

Pour un seul dépôt, copiez-le plutôt dans `<repo>/.agents/skills/`. Ce dépôt contient aussi des profils d'agents Codex pour les sièges, sans rôle prédéfini, dans `.codex/agents/` ; Codex les détecte automatiquement dans une copie locale du dépôt. Pour utiliser les profils depuis tous les dépôts :

```sh
cp .codex/agents/seat-*.toml ~/.codex/agents/
```

Le skill fonctionne aussi sans cette copie : il peut lancer des sous-agents avec les mêmes affectations de modèles explicites et poursuivre leurs fils de discussion d'un tour à l'autre. Le paquet du plugin n'installe pas automatiquement les fichiers TOML des agents personnalisés.

## Installation pour Claude Code

### En tant que plugin

```text
/plugin marketplace add Shintaro-My/tenth-man
/plugin install tenth-man@tenth-man
```

Pour tester une copie locale du dépôt :

```sh
claude --plugin-dir /path/to/tenth-man
```

Invoquez-le avec :

```text
/tenth-man:council Devons-nous déplacer l'état de session de Redis vers des cookies signés ?
```

### Installation manuelle

```sh
cp -r skills/council ~/.claude/skills/
cp agents/seat-*.md ~/.claude/agents/
```

Installée manuellement, la commande est `/council` ; les noms des agents n'ont pas le préfixe `tenth-man:`.

## Préréglages de modèles

La présidence propose `light` pour une petite question et `standard` dans les autres cas. Vous pouvez choisir un préréglage ou modifier l'attribution de n'importe quel siège lorsque vous approuvez le plan.

Le dixième homme utilise le même modèle que le membre doté des meilleures capacités de raisonnement. Ce classement repose sur des résultats mesurés, et non sur les numéros de version ; les colonnes « Dixième homme » ci-dessous le reflètent tel qu'il était en octobre 2026. Lorsque le rang d'un nouveau modèle est inconnu, la présidence vous le demande. Pour utiliser un autre modèle, indiquez-le au moment de donner votre feu vert.

### Codex

| Préréglage | Membres votants | Dixième homme | Exécutions par tour |
|---|---|---|---|
| `light` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra | GPT-6 Astra | 4 |
| `standard` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6 Astra | 5 |
| `no-astra` | GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6.1 Sol | 4 |

Ces valeurs par défaut n'utilisent que les modèles proposés par l'hôte Codex et le compte actuels. La présidence signale les modèles indisponibles et ne les remplace jamais en silence. Si l'hôte autorise moins d'exécutions simultanées que le conseil ne compte de sièges, les agents des sièges s'exécutent par vagues à partir du même brief inchangé ; aucun décompte n'a lieu avant que tous les sièges aient répondu.

Les agents personnalisés Codex correspondants sont `seat_astra`, `seat_sol`, `seat_terra`, `seat_previous_sol` et `seat_luna`, facultatif et moins coûteux. Leurs définitions TOML fixent le modèle et l'effort de raisonnement, et limitent le travail du conseil à la lecture seule.

### Claude Code

| Préréglage | Membres votants | Dixième homme | Exécutions par tour |
|---|---|---|---|
| `light` | Fable, Opus, Sonnet | Fable | 4 |
| `standard` | Fable, Opus, Sonnet, Opus 5, Opus 4.8 | Fable | 6 |
| `no-fable` | Opus, Sonnet, Opus 5 | Opus | 4 |

Chaque préréglage conserve un siège Opus même lorsque Fable est mieux classé, car Opus trouve parfois des problèmes que Fable ne voit pas.

Le plugin Claude inclut dans `agents/` cinq agents pour les sièges, sans rôle prédéfini. Un brief attribue à chaque exécution son rôle de membre ou de dixième homme.

### Ajouter un modèle Claude

Un modèle Claude plus ancien peut être sélectionné via le champ `model:` d'une définition d'agent. Copiez le fichier d'agent d'un siège, modifiez `name`, `description` et `model`, puis enregistrez-le sous `~/.claude/agents/` pour tous les projets ou sous `.claude/agents/` pour un seul projet :

```markdown
---
name: seat-opus-4-7
description: A seat in a tenth-man council, on Opus 4.7.
model: claude-opus-4-7
effort: high
tools: Read, Grep, Glob, Bash, Write
---
```

Conservez le même corps de définition, sans rôle prédéfini, que dans les fichiers d'agents fournis. La présidence peut alors affecter `seat-opus-4-7` comme membre ou comme dixième homme.

### Utiliser vos propres agents Claude

N'importe quelle définition d'agent Claude existante peut occuper un siège. La présidence commence son prompt en indiquant que, pour cette tâche, l'agent siège au conseil au lieu de faire son travail habituel, puis lui attribue le rôle et le brief commun. C'est utile pour des agents de revue fixés sur un modèle plus ancien.

### Lorsqu'un modèle Claude fixé est retiré

Si un modèle fixé n'est plus disponible, l'agent ne démarre pas. La présidence écarte ce siège et vous en informe ; elle ne remplace jamais en silence ce modèle par un autre. Supprimez ou mettez à jour le fichier d'agent du siège en fonction des modèles disponibles pour votre compte.

## Coût et prérequis

Chaque siège correspond à une exécution complète de sous-agent par tour. Sur trois tours, le préréglage `standard` de Codex utilise 15 exécutions et celui de Claude Code en utilise 18.

- Codex : une version récente avec les sous-agents activés et l'accès aux modèles sélectionnés.
- Claude Code : les outils Agent et SendMessage, les notifications de fin des agents en arrière-plan et l'accès aux modèles sélectionnés.

Le chat ChatGPT n'est actuellement pas une cible prise en charge, car il ne peut pas exécuter de sous-agents sur un modèle différent pour chaque siège.

## Pourquoi « dixième homme » ?

Le skill s'inspire du « dixième homme », une règle israélienne fictive du film *World War Z* sorti en 2013. Lorsque neuf personnes arrivent à la même conclusion à partir des mêmes informations, le dixième part du principe qu'elles se trompent et examine l'hypothèse inverse. La règle est fictive, mais des pratiques similaires existent :

- Après avoir mal interprété des signes avant-coureurs avant la guerre du Kippour de 1973, le renseignement militaire israélien a créé une fonction chargée de contester ses propres évaluations, couramment appelée *ipcha mistabra*, « le contraire est probable » en araméen.
- Le Talmud (Sanhédrin 17a) rapporte que si tous les juges votent la condamnation dans une affaire passible de la peine de mort, l'accusé est libéré ; un résultat unanime laisse penser que personne n'a correctement plaidé la défense.
- Dans les procès de canonisation catholiques, l'« avocat du diable » argumentait contre le candidat et examinait les points faibles des récits sur sa vie et des miracles qui lui étaient attribués. La fonction a été formalisée en 1587 et fortement réduite en 1983.

## Licence

MIT. Voir [LICENSE](LICENSE).
