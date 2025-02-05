---
title: "Front-end"
linkTitle: "Front-end"
description: "Coding style guide and best practices for front-end"
---

Nous utilisons **ReactJS** et tous les fichiers doivent être écrits en **Typescript**.

Le code est **linté** avec [eslint](https://eslint.org/), et **formaté** avec [prettier](https://prettier.io/).

## Nomenclature

![Diagramme de l'Infrastructure](../nomenclature-front-end.svg)

Les **applications** (osrd eex, osrd stdcm, éditeur infra, éditeur matériel) proposent des **vues** (gestion des projets, gestions des études, etc.) liées à des **modules** (projet, étude, etc.) qui contiennent les composants.

Ces **vues** sont constituées de **composants** et sous-composants <u>tous issus des modules</u>.
En plus de contenir les fichiers de **vues** des applications, elles peuvent contenir un répertoire **scripts** qui propose des scripts liés à ces vues. Les **vues** déterminent la logique et l'<u>accès au store</u>.

Les **modules** sont des collections de **composants** rattachés à un **objet** (un scénario, un matériel roulant, un TrainSchedule). Ils contiennent :
  - un répertoire *components* qui héberge <u>tous</u> les composants
  - un répertoire *styles* optionnel <u>par module</u> pour le style des composants en scss
  - un répertoire *assets* optionnel <u>par module</u> (qui contient les assets, de jeux de données par défaut par ex, spécifiques au module)
  - un fichier *reducers* optionnel <u>par module</u>
  - un fichier *types* optionnel <u>par module</u>
  - un fichier *consts* optionnel <u>par module</u>

Un répertoire **assets** (qui contient les images et autre fichiers).

Enfin, un répertoire **common** qui propose :
  - un répertoire *utils* pour les fonctions utilitaires communes à l'ensemble du projet
  - un fichier *types* pour les types communs à l'ensemble du projet
  - un fichier *consts* pour les constantes communes à l'ensemble du projet


## Principes d'implémentation
### Routage & SLUG
_Rédaction en cours_

`projects/{nom du projet}/studies/{nom de l'étude}/scenarios/{nom du scenario}`

### Styles & SCSS
> ATTENTION : en CSS/React, le scope d'une classe ne dépend pas de l'endroit où le fichier est importé mais est valide pour toute l'application. Si vous importez un fichier `scss` au fin fond d'un composant (ce que nous déconseillons fortement par ailleurs), ses classes seront disponibles pour toute l'application et peuvent donc provoquer des effets de bord.

Il est donc très recommandé de pouvoir facilement suivre l'arborescence des applications, vues, modules et composants également au sein du code SCSS, et notamment imbriquer les noms de classes pour éviter les effets de bord, le compilateur se chargera de fabriquer la hiérarchie nécessaire.

Si par exemple nous avons un composant `rollingStockSelector` qui propose une liste de matériel `rollingStockList` représentés par des cartes `rollingStockCard` contenant une image représentant le matériel roulant `rollingStockImg` nous devrions avoir la structure SCSS suivante :

```scss
.rollinStockSelector {
  .rollingStockList {
    .rollingStockCard {
      .rollingStockImg {
        width: 5rem;
        height: auto;
      }
    }
  }
}
```

Ainsi, on a la garantie que l'image contenue dans la carte de matériel roulant héritera bien des bonnes propriétés css `.rollinStockSelector.rollingStockList.rollingStockCard.rollingStockImg`.

#### CSS Modules
CSS modules allow scoping CSS styles to a specific component, thereby avoiding conflicts with global class names.

Vite natively supports CSS modules. Ensure that your CSS file has the `.module.css` extension, for example, `styles.module.css`.

##### Using CSS Modules in Components

1. **Create an SCSS file with the `.module.scss` extension**:

```css
/* MyComponent.module.scss */
.container {
  background-color: white;
}

.title {
  font-size: 24px;
  color: #333;
}
```

2. **Use the classes in your React component**:

Vite transforms classes into objects that contain hashed classes (e.g., `_container_h3d8bg`) and uses them during bundle generation, making the classes unique.

```tsx
import React from 'react';
import styles from './MyComponent.module.scss';

export function MyComponent() {
  return (
    <div className={styles.container}>
      <h1 className={styles["title"]}>My Title</h1>
    </div>
  );
};
```

For more information, you can refer to the [Vite.js documentation](https://vitejs.dev/guide/features.html#css-modules).

#### Noms de classes, utilisation de `cx()`
Les classes sont ajoutées les unes à la suite des autres, normalement, dans la propriété `className=""`.

Cependant, quand cela est nécessaire —&nbsp;tests pour l'utilisation d'une classe, concaténation, etc.&nbsp;— nous utilisons la librairie [classnames](https://github.com/JedWatson/classnames) qui préconise l'usage suivant :

```ts
<div className="rollingStockSelector">
  <div className="rollingStockList">
    <div className="rollingStockCard w-100 my-2">
      <img
        className={cx('rollingStockImg', 'm-2', 'p-1', 'bg-white', {
          valid: isValid(),
          selected: rollingStockID === selectedRollingStockID,
        })}
      />
    </div>
  </div>
</div>
```

Les classes sont **séparées** chacune dans un `string` et les opérations booléennes ou autres sont réalisées dans un objet qui retournera —&nbsp;ou pas&nbsp;— le nom de propriété comme nom de classe à utiliser dans le CSS.

### Store/Redux
Tout ce qui est *selector* est géré par la **vue** passé en props aux composants et sous-composants.

Par conséquent les appels au store en lecture et en écriture doivent être passés un niveau de la vue, en irrigant par des _props_ et _states_ les composants proposées par la vue.

### RTK
_Rédaction en cours_

Utiliser les endpoints générés à partir des fichiers `openapi.yaml` pour consommer le backend.


## Lois et éléments importants

> **Aucun composant ne doit détenir la responsabilité de mise à jour de la donnée qu'il utilise**
>
> Seules <u>les vues</u> contiennent les sélecteurs du store, donnés ensuite en props aux composants du module lié à la vue.

> **Le SCSS n'est pas scopé**
>
> Un fichier `.scss` enfoui dans l'arborescence ne vous garantit pas que les classes contenues soient seulement accessibles à cet endroit, y compris par import react (formellement interdit au passage : vous devez utiliser l'import SCSS), toutes les classes déclarées sont accessibles partout.
>
> Préférez un choix judicieux de nom de classe racine pour un module donné et utilisez l'arborescence possible dans le fichier SCSS.

> **Les liens des imports doivent être absolus au sein d'une application**
>
> Vous devez utiliser le <u>chemin complet</u> pour tous vos imports, même si le fichier à importer se trouve dans le même répertoire.

> **Les liens des imports doivent être relatifs au sein d'un module ou d'un composant**
>
> Au sein d'un module ou d'un composant, à l'instar d'une librairie, les liens d'imports doivent rester relatifs afin de permettre leur utilisation n'importe où.

## TypeScript

### import & export

We recommend using typed imports and exports.

When an `import` or `export` contains only types, indicate it with the `type` keyword.


```typescript
export type { Direction, DirectionalTrackRange as TrackRange };
```
```typescript
import type { typedEntries, ValueOf } from 'utils/types';
```

This allows to:
- Improve the performance and analysis process of the compiler and the linter.
- Make these declarations more readable; we can clearly see what we are importing.
- Avoid dependency cycles:

 ![dependency cyle](./dependency-cycle.png)

The error disappears with the `type` keyword

 ![dependency cyle](./dependency-cycle-gone.png)


- Make final bundle lighter (all types disappear at compilation)
