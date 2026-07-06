# Architecture

> Les **grands choix structurants**. Rempli par GENESIS ou migré par ADOPT.

## Topologie

- **Monorepo** (défaut) : `source/` et le code dans le même repo, `sync-repo: .`.
- **Détachée** : source et code dans deux repos (ex. source privée / code
  public), `sync-repo: <chemin-du-clone-code>`.

*Choix retenu et pourquoi :*

## Langages / stack

*Langages, frameworks, runtime.*

## Contraintes dures

*Confidentialité, offline, coût, latence, conformité — ce qui n'est pas
négociable.*

## Découpage en capacités

*La carte des modules (une capacité = une responsabilité cohérente). Le détail
vit dans `source/modules/`.*
