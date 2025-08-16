# Corrections des Auto-évaluations

## Introduction

Bravo d'avoir pris le temps de répondre aux questions d'auto-évaluation tout au long de ce parcours. L'auto-évaluation
est un outil puissant pour consolider ses connaissances. Ce document contient les réponses détaillées à toutes les
questions posées. Comparez-les avec vos propres réponses, non pas pour juger, mais pour comprendre et renforcer les
concepts clés.

---

## **Module 1 : Introduction - Le Pont entre Java et Angular**

### L'essentiel

1. **(Question ouverte)** La responsabilité du serveur :
    * Votre réponse devrait inclure les points suivants : Dans une architecture **multi-page (Thymeleaf)**, le serveur
      est responsable de la **génération de la vue**. Il combine les données et les templates HTML pour renvoyer une
      page HTML complète au navigateur. Dans une architecture **SPA (API REST)**, la responsabilité du serveur se limite
      à être un **fournisseur de données**. Il expose une API qui renvoie des données brutes (généralement en JSON), et
      c'est au client (Angular) de se charger de la génération de la vue.

2. **(QCM)** L'analogie pour l'Injection de Dépendances :
    * **Réponse : (c) Les Services et le système d'Injection de Dépendances.**
    * **Explication :** Le concept est une transposition directe. En Spring, on injecte un `@Service` dans un
      `@Controller`. En Angular, on injecte un service `@Injectable` dans un `@Component` via le constructeur.

3. **(QCM)** Caractéristique d'une SPA :
    * **Réponse : (c) Les interactions semblent instantanées car le contenu est mis à jour dynamiquement sans
      rechargement de page.**
    * **Explication :** C'est l'avantage principal perçu par l'utilisateur : une expérience fluide et réactive,
      similaire à une application de bureau.

4. **(Question ouverte)** L'avantage des composants "Standalone" :
    * Votre réponse devrait mentionner : une **courbe d'apprentissage plus simple** car il n'est pas nécessaire de
      comprendre les `NgModule` au début. Cela entraîne **moins de code "boilerplate"** (répétitif) et une structure de
      projet plus légère. Les dépendances d'un composant sont déclarées **explicitement** dans son décorateur, ce qui le
      rend plus facile à comprendre de manière isolée.

5. **(QCM)** Rôle du backend dans une SPA :
    * **Réponse : (b) Fournir des données brutes (généralement en JSON) via une API.**
    * **Explication :** Une fois l'application Angular chargée, le backend ne sert plus que d'API pour les données.

### Pour aller plus loin

1. **(QCM)** Avantage du SSR :
    * **Réponse : (b) Il améliore le SEO et le temps de premier affichage.**
    * **Explication :** Le SSR envoie une page HTML déjà rendue, que les moteurs de recherche peuvent facilement
      indexer (SEO) et que le navigateur peut afficher instantanément (meilleur First Contentful Paint).

2. **(Question ouverte)** Quand utiliser React plutôt qu'Angular :
    * Un bon scénario serait un projet nécessitant une **grande flexibilité** et un **prototype rapide**, où l'on
      souhaite choisir soi-même toutes les briques de l'architecture (routeur, gestion d'état...). C'est aussi un bon
      choix pour une équipe ayant une forte culture JavaScript et préférant une approche moins "cadrée" que celle d'un
      framework complet comme Angular.

3. **(QCM)** Stratégie de rendu pour un site de documentation :
    * **Réponse : (c) SSG (Static Site Generation).**
    * **Explication :** Le contenu est statique et la performance est primordiale. Le SSG pré-génère tous les fichiers
      HTML, offrant la vitesse de chargement la plus rapide possible.

4. **(Question ouverte)** L'hydratation (hydration) :
    * C'est le processus, dans une application SSR, par lequel l'application Angular (le JavaScript) "prend le contrôle"
      de la page HTML statique qui a été rendue par le serveur. Elle attache ses écouteurs d'événements et transforme la
      page statique en une SPA entièrement interactive, sans que l'utilisateur ne perçoive de transition.

5. **(QCM)** Bibliothèque de composants UI :
    * **Réponse : (c) Angular Material.**
    * **Explication :** C'est la bibliothèque de composants UI officielle, développée et maintenue par l'équipe Angular
      de Google.

---

## **Module 2 : Mise en Place de l'Environnement**

### L'essentiel {id="l-essentiel_1"}

1. **(Question ouverte)** Analogie npm / package.json / node_modules :
    * `npm` est l'outil de build et de gestion de dépendances, comme **Maven ou Gradle**.
    * `package.json` est le fichier de description du projet, qui liste les dépendances, comme le **`pom.xml`**.
    * Le dossier `node_modules` contient les "jar" (les bibliothèques) téléchargées, c'est l'équivalent du *
      *dossier `.m2/repository` local**, mais spécifique au projet.

2. **(QCM)** Commande pour démarrer le serveur :
    * **Réponse : (c) `ng serve`.**

3. **(QCM)** Fichier listant les dépendances :
    * **Réponse : (c) `package.json`.**

4. **(Question ouverte)** Avantage de `ng serve` :
    * Son principal avantage est le **rechargement à chaud (live-reloading)**. Dès qu'un fichier est sauvegardé,
      l'application est automatiquement recompilée et le navigateur se met à jour, offrant un cycle de feedback
      instantané.

5. **(QCM)** Utilisation d'un sélecteur de composant :
    * **Réponse : (b) `<app-user-card></app-user-card>`.**
    * **Explication :** Le sélecteur définit le nom de la balise HTML personnalisée pour le composant.

### Pour aller plus loin {id="pour-aller-plus-loin_1"}

1. **(QCM)** Extension VS Code essentielle :
    * **Réponse : (c) Angular Language Service.**
    * **Explication :** Elle fournit l'intelligence du framework (auto-complétion, vérification d'erreurs) directement
      dans l'éditeur, en particulier dans les templates HTML.

2. **(Question ouverte)** Linter vs. Formatter :
    * Un **linter (ESLint)** analyse la **qualité du code** et recherche les **erreurs potentielles** et le non-respect
      des bonnes pratiques (ex: variable inutilisée). Un **formatter (Prettier)** se concentre uniquement sur le **style
      et la mise en forme** du code (ex: indentation, points-virgules) pour garantir la cohérence.

3. **(QCM)** Supprimer `package-lock.json` :
    * **Réponse : (c) Parce que cela rend le build non-reproductible et peut introduire des versions de paquets
      inattendues.**
    * **Explication :** Ce fichier garantit que chaque développeur et le serveur d'intégration continue installent
      exactement les mêmes versions de chaque dépendance.

4. **(Question ouverte)** Avantage d'un projet avec ESLint/Prettier :
    * Le principal avantage est la **cohérence immédiate**. Le code respecte déjà des standards de qualité et de style,
      ce qui rend l'intégration d'un nouveau développeur plus facile, et les revues de code peuvent se concentrer sur la
      logique métier plutôt que sur le style.

5. **(QCM)** Ajouter des raccourcis de commande :
    * **Réponse : (b) `package.json`.**
    * **Explication :** Les commandes sont ajoutées dans la section `"scripts"` de ce fichier.

---
*... Et ainsi de suite pour tous les autres modules. Je vais continuer sur cette lancée.*

---

## **Module 3 : TypeScript : Un Terrain Connu pour les Javanistes**

### L'essentiel {id="l-essentiel_2"}

1. **(Question ouverte)** Principal bénéfice de TypeScript pour un Javaniste :
    * Le bénéfice principal est le **typage statique**. Cela apporte une sécurité, une robustesse, une auto-complétion
      intelligente et une détection d'erreurs en amont (à la compilation) qui sont très familières et appréciées par les
      développeurs venant d'un langage fortement typé comme Java.

2. **(QCM)** Déclarer une constante :
    * **Réponse : (d) `const PI = 3.14;`.**
    * **Explication :** `const` est utilisé pour les variables dont la référence ne sera pas réassignée.

3. **(QCM)** Manière concise d'initialiser une propriété :
    * **Réponse : (b) Utiliser un modificateur d'accès (`public`, `private`...) sur le paramètre du constructeur.**
    * **Explication :** C'est le raccourci "parameter properties" de TypeScript qui déclare et assigne la propriété en
      une seule étape.

4. **(Question ouverte)** Rôle d'une interface :
    * Comme en Java, une interface en TypeScript définit un **contrat**. Elle décrit la forme d'un objet (ses propriétés
      et méthodes) sans l'implémenter. On l'utiliserait par exemple pour définir la structure d'un objet de données
      attendu d'une API REST (ex: `interface User {...}`), garantissant que le code qui manipule ces données respecte
      bien cette structure.

5. **(QCM)** Analogue des Décorateurs :
    * **Réponse : (c) Une annotation (`@Service`, `@Controller`).**
    * **Explication :** Les deux servent à attacher des métadonnées à une classe, méthode ou propriété pour que le
      framework puisse les traiter de manière spéciale.

### Pour aller plus loin {id="pour-aller-plus-loin_2"}

1. **(QCM)** Typer une variable qui peut être `User` ou `null` :
    * **Réponse : (c) `let data: User | null;`.**
    * **Explication :** C'est la syntaxe du "type union".

2. **(Question ouverte)** Différence entre `any` et `unknown` :
    * `any` désactive complètement la vérification de type de TypeScript. On peut faire n'importe quelle opération sur
      une variable de type `any` sans erreur de compilation. `unknown` est une alternative plus sûre : la variable peut
      contenir n'importe quoi, mais pour effectuer une opération dessus, on doit d'abord effectuer une vérification de
      type (une "garde de type" comme `typeof` ou `instanceof`).

3. **(QCM)** Signature d'une fonction générique :
    * **Réponse : (b) `function wrapInObject<T>(val: T): { value: T }`.**
    * **Explication :** `<T>` déclare le type générique, `val: T` l'utilise pour le paramètre, et `{ value: T }`
      l'utilise pour le type de retour.

4. **(Question ouverte)** Cas d'utilisation pour `Partial<T>` :
    * Un cas d'usage parfait est la fonction de **mise à jour d'un objet**. Par exemple, dans un formulaire de
      modification de profil, l'utilisateur peut ne changer que son nom ou son email. La charge utile (payload) envoyée
      à l'API peut être typée avec `Partial<User>`, car elle ne contiendra qu'une partie des propriétés de l'objet
      `User` complet.

5. **(QCM)** Garde de type pour `string | number` :
    * **Réponse : (c) `if (typeof param === "string")`.**
    * **Explication :** `typeof` est l'opérateur JavaScript/TypeScript pour obtenir le type primitif d'une variable.

---

## **Module 4 : Le Cœur d'Angular - Les Composants**

### L'essentiel {id="l-essentiel_3"}

1. **(Question ouverte)** Différence `{{ }}` et `[]` :
    * L'interpolation `{{ }}` est un sucre syntaxique pour afficher une **valeur textuelle** dans le DOM. Le property
      binding `[]` est plus puissant : il permet de lier une propriété de composant à un **attribut ou une propriété
      d'un élément DOM**, ce qui fonctionne pour des types complexes comme les booléens (`[disabled]`), les objets ou
      les tableaux, et pas seulement pour du texte.

2. **(QCM)** Soumission de formulaire :
    * **Réponse : (b) `<form (ngSubmit)="saveData()">`.**
    * **Explication :** `(ngSubmit)` est l'événement spécial d'Angular pour la soumission de formulaires, qui prévient
      le rechargement de la page.

3. **(QCM)** Import pour `[(ngModel)]` :
    * **Réponse : (d) `FormsModule`.**
    * **Explication :** `ngModel` fait partie du module des formulaires "par template", et il est souvent utilisé même
      en mode réactif pour des cas simples.

4. **(Question ouverte)** Encapsulation de style :
    * Cela signifie que les styles définis dans le fichier CSS d'un composant (ex: `user-card.component.css`) ne
      s'appliquent **qu'à ce composant**. Angular utilise des techniques pour "scoper" ces styles afin qu'ils ne "
      fuient" pas et n'affectent pas d'autres composants, garantissant ainsi leur isolation.

5. **(QCM)** Utilisation d'un sélecteur :
    * **Réponse : (c) `<app-login-form></app-login-form>`.**

### Pour aller plus loin {id="pour-aller-plus-loin_3"}

1. **(QCM)** Passer une donnée du parent à l'enfant :
    * **Réponse : (c) `@Input() product: Product;`.**
    * **Explication :** Le décorateur `@Input()` marque une propriété comme étant une "porte d'entrée" pour les données
      venant du parent.

2. **(Question ouverte)** "Props down, events up" :
    * C'est le principe du **flux de données unidirectionnel**. Les données (propriétés, "props") descendent toujours de
      l'extérieur vers l'intérieur, du **parent vers l'enfant**, via les `@Input()` et le property binding `[]`. Les
      notifications d'événements remontent toujours de l'intérieur vers l'extérieur, de l'**enfant vers le parent**, via
      les `@Output()` et l'event binding `()`. Cette pratique rend le flux de données prédictible et facile à déboguer.

3. **(QCM)** Écouter un `@Output` :
    * **Réponse : (b) `(itemSelected)="onItemSelect($event)"`.**
    * **Explication :** `(itemSelected)` est la syntaxe d'event binding, et la variable magique `$event` contient les
      données émises par l'enfant.

4. **(Question ouverte)** Cas d'utilisation pour `<ng-content>` :
    * Un exemple typique est un composant "wrapper" ou "layout" comme une carte (`<app-card>`) ou un panneau (
      `<app-panel>`). Le composant `app-card` pourrait définir un cadre, une ombre et un en-tête, mais laisser le
      contenu du corps de la carte être "projeté" par le composant parent. Cela permet de réutiliser la même structure
      de carte pour afficher du texte, des images ou d'autres composants.

5. **(QCM)** Classe pour émettre des événements :
    * **Réponse : (c) `EventEmitter`.**

---

## **Module 5 : Directives et Pipes**

### L'essentiel {id="l-essentiel_4"}

1. **(Question ouverte)** `*ngIf="false"` vs. `display: none;` :
    * `*ngIf="false"` (ou `@if(false)`) **retire complètement l'élément du DOM**. Il n'existe plus dans l'arborescence
      de la page. `display: none;` **cache seulement l'élément**. Il est toujours présent dans le DOM, mais n'est pas
      visible. La première approche est plus performante si l'élément est complexe, car elle évite le coût de sa
      création et de sa maintenance.

2. **(QCM)** Rôle du `track` dans `@for` :
    * **Réponse : (b) À fournir un identifiant unique pour chaque élément afin d'optimiser les mises à jour du DOM.**
    * **Explication :** Cela permet à Angular de savoir quel élément a été ajouté, supprimé ou déplacé, au lieu de tout
      détruire et reconstruire, ce qui est beaucoup plus performant.

3. **(QCM)** Formater une date :
    * **Réponse : (c) `{{ birthDate | date:'dd MMMM yyyy' }}`.**
    * **Explication :** La syntaxe est `valeur | nomDuPipe:parametre1:parametre2...`.

4. **(Question ouverte)** Qu'est-ce qu'un pipe ?
    * Un pipe est une fonction de transformation simple utilisée dans les templates pour formater des données. On
      l'utiliserait plutôt qu'une méthode de composant pour des transformations pures et communes (ex: formater une
      date, une devise) car c'est plus déclaratif, plus performant (les pipes purs ne sont exécutés que si l'entrée
      change) et réutilisable sans avoir à l'importer dans la logique du composant.

5. **(QCM)** Appliquer une classe dynamiquement :
    * **Réponse : (b) `<div [ngClass]="{ 'highlight': isSelected }">`.**

### Pour aller plus loin {id="pour-aller-plus-loin_4"}

1. **(Question ouverte)** Pipe personnalisé vs. méthode de composant :
    * Il est préférable de créer un pipe personnalisé lorsqu'une logique de transformation de données est **pure** (le
      même résultat pour la même entrée) et qu'elle est **réutilisée** dans plusieurs templates. Cela centralise la
      logique, la rend testable de manière isolée et allège les composants. Une méthode est plus adaptée pour une
      transformation qui dépend de l'état interne complexe du composant.

2. **(QCM)** Pipe avec paramètre :
    * **Réponse : (c) `{{ data | myPipe:param }}`.**

3. **(QCM)** Pipe personnalisé par défaut :
    * **Réponse : (b) Pur, pour des raisons de performance.**

4. **(Question ouverte)** Rôle de `@HostListener` :
    * C'est un décorateur qui permet à une directive de s'abonner et de réagir à un événement du DOM sur son élément "
      hôte" (l'élément auquel la directive est appliquée). Ex: `@HostListener('click', ['$event'])`.

5. **(QCM)** Manipuler l'élément DOM :
    * **Réponse : (d) `ElementRef`.**
    * **Explication :** `ElementRef` est un wrapper autour de l'élément natif du DOM, qui doit être utilisé avec
      prudence.

---

## **Module 6 : Services et Injection de Dépendances**

### L'essentiel {id="l-essentiel_5"}

1. **(Question ouverte)** Responsabilité service vs. composant :
    * La responsabilité d'un **composant** est de gérer la **présentation** et l'interaction avec l'utilisateur (
      afficher des données, réagir aux clics). La responsabilité d'un **service** est de gérer la **logique métier** ou
      la communication avec des sources de données (calculs, appels API, partage d'état).

2. **(QCM)** Signification de `providedIn: 'root'` :
    * **Réponse : (b) Que le service est un singleton disponible dans toute l'application.**

3. **(QCM)** Obtenir une instance d'un service :
    * **Réponse : (c) En le déclarant comme un paramètre du constructeur avec un modificateur d'accès.**

4. **(Question ouverte)** Partager des données entre composants non liés :
    * La meilleure façon est d'utiliser un **service partagé** (fourni à la racine). Le service détient la donnée (par
      exemple, dans un `BehaviorSubject` ou un `Signal`). `ComponentA` peut écrire dans le service, et `ComponentB` peut
      lire la donnée depuis ce même service, qui agit comme un point de communication central.

5. **(QCM)** Principe de conception respecté :
    * **Réponse : (c) SRP (Single Responsibility Principle).**
    * **Explication :** Le composant se concentre sur la présentation, le service sur l'accès aux données. Chacun a une
      seule responsabilité.

### Pour aller plus loin {id="pour-aller-plus-loin_5"}

1. **(Question ouverte)** Fournir un service au niveau composant :
    * C'est approprié quand on veut que **chaque instance d'un composant ait sa propre instance isolée du service**. Un
      exemple concret est un composant de "widget" de chat ou de compteur. Si on affiche plusieurs widgets sur la même
      page, chacun doit avoir son propre état, géré par sa propre instance du `ChatService` ou `CounterService`.

2. **(QCM)** Fournir un objet de configuration :
    * **Réponse : (b) `useValue`.**
    * **Explication :** `useValue` est conçu pour lier un token à une valeur statique déjà existante.

3. **(QCM)** Injecter une valeur via `InjectionToken` :
    * **Réponse : (d) `constructor(@Inject(MY_TOKEN) private value: any)`.**
    * **Explication :** Le décorateur `@Inject()` est nécessaire pour indiquer à l'injecteur quel token utiliser.

4. **(Question ouverte)** Utilité de `useFactory` :
    * Une factory est utile quand la **création d'un service dépend d'une logique conditionnelle ou d'autres services**.
      Par exemple, on pourrait vouloir instancier un `LocalStorageService` si on est dans le navigateur, mais un
      `InMemoryStorageService` si on est sur le serveur (en SSR). La factory peut injecter d'autres services (comme un
      `PlatformService`) pour prendre cette décision.

5. **(QCM)** Dépendance avec `@Optional` :
    * **Réponse : (c) Le composant sera créé et la propriété `service` sera `null`.**
    * **Explication :** C'est le but de `@Optional` : prévenir une erreur d'injection et fournir `null` à la place.

---

## **Module 7 : Navigation avec le Router Angular**

### L'essentiel {id="l-essentiel_6"}

1. **(Question ouverte)** Rôle de `<router-outlet>` :
    * C'est une directive qui agit comme un **placeholder** ou une "fenêtre". On la place dans un template de
      composant (généralement `AppComponent`), et c'est à cet endroit précis que le Router Angular affichera le
      composant correspondant à la route URL active.

2. **(QCM)** Créer un lien de navigation :
    * **Réponse : (d) `[routerLink]="['/about']"`.**

3. **(QCM)** Définir une route paramétrée :
    * **Réponse : (a) `path: 'books/:isbn'`.**
    * **Explication :** La syntaxe `:nom_du_parametre` définit un segment d'URL dynamique.

4. **(Question ouverte)** `routerLink` vs. `href` :
    * `href` provoque un **rechargement complet de la page** par le navigateur, ce qui détruit l'état de la SPA et est
      lent. `routerLink` intercepte le clic, demande au Router Angular de gérer la navigation **en interne sans
      rechargement de page**, en changeant simplement le composant affiché dans le `<router-outlet>`.

5. **(QCM)** Lire les paramètres de l'URL :
    * **Réponse : (c) `ActivatedRoute`.**

### Pour aller plus loin {id="pour-aller-plus-loin_6"}

1. **(Question ouverte)** Avantage du "lazy loading" :
    * Son principal avantage est la **performance au chargement initial**. Il scinde l'application en morceaux plus
      petits (chunks), et le navigateur ne télécharge que le code nécessaire pour la page de démarrage. Les autres
      fonctionnalités ne sont téléchargées que lorsque l'utilisateur y navigue, ce qui rend le premier affichage de
      l'application beaucoup plus rapide.

2. **(QCM)** Afficher des routes enfants :
    * **Réponse : (c) En plaçant un `<router-outlet>` dans son template.**
    * **Explication :** Chaque niveau de routage nécessite son propre `<router-outlet>` pour afficher ses enfants.

3. **(QCM)** Garde pour empêcher de quitter une page :
    * **Réponse : (b) `CanDeactivate`.**
    * **Explication :** `CanDeactivate` est spécifiquement conçue pour poser la question "Êtes-vous sûr de vouloir
      quitter cette page ?" avant que la navigation ne se produise.

4. **(Question ouverte)** Rôle de `CanActivate` :
    * `CanActivate` est une garde de navigation qui détermine si un utilisateur a le droit d'accéder à une route. Elle
      doit retourner `true` pour autoriser, `false` pour bloquer, ou une `UrlTree` (créée avec
      `router.createUrlTree(['/login'])`) pour rediriger l'utilisateur vers une autre page.

5. **(QCM)** Propriété pour le lazy loading :
    * **Réponse : (c) `loadChildren`.**

---

## **Module 8 : Communication avec le Backend : HttpClient et RxJS**

### L'essentiel {id="l-essentiel_7"}

1. **(Question ouverte)** Différence `Promise` vs. `Observable` :
    * Une **Promise** ne peut émettre qu'**une seule valeur** (soit une résolution, soit un rejet) et la transaction est
      ensuite terminée. Un **Observable** est un **flux (stream)** qui peut émettre **zéro, une ou plusieurs valeurs**
      au fil du temps, et se terminer (ou non).

2. **(QCM)** Fonction pour utiliser `HttpClient` :
    * **Réponse : (c) `provideHttpClient()`.**

3. **(QCM)** Que se passe-t-il sans `.subscribe()` ?
    * **Réponse : (b) L'appel HTTP n'est jamais fait.**
    * **Explication :** Les Observables retournés par `HttpClient` sont "froids" (cold). Ils ne font rien tant que
      personne ne s'y abonne.

4. **(Question ouverte)** Les trois canaux de `subscribe` :
    * `next` : Pour traiter la donnée quand elle est émise avec succès.
    * `error` : Pour gérer une erreur si le flux en émet une.
    * `complete` : Pour exécuter une action quand le flux est terminé et qu'il n'émettra plus de valeurs.

5. **(QCM)** Type de `data` dans le `next` :
    * **Réponse : (c) `User[]`.**
    * **Explication :** Le callback `next` reçoit la valeur "déballée" de l'Observable.

### Pour aller plus loin {id="pour-aller-plus-loin_7"}

1. **(Question ouverte)** Rôle de `.pipe()` :
    * La méthode `.pipe()` permet de **chaîner des opérateurs RxJS** pour créer un "pipeline" de traitement de données.
      Chaque opérateur prend l'Observable précédent en entrée et retourne un nouvel Observable transformé, qui est passé
      à l'opérateur suivant.

2. **(QCM)** Transformer `Observable<string>` en `Observable<number>` :
    * **Réponse : (b) `map(str => str.length)`.**
    * **Explication :** `map` est l'opérateur de transformation 1-pour-1.

3. **(QCM)** Enchaîner deux appels dépendants :
    * **Réponse : (c) `switchMap`.**
    * **Explication :** `switchMap` est conçu pour "aplatir" les Observables d'Observables, ce qui est typique des
      appels dépendants.

4. **(Question ouverte)** Rôle de `catchError` :
    * `catchError` intercepte une erreur dans un flux. Pour que l'Observable ne se termine pas en erreur (ce qui
      terminerait le `subscribe`), il doit **retourner un nouvel Observable**. Souvent, on retourne un Observable qui
      émet une valeur par défaut, comme `of([])`, pour que le composant reçoive une valeur "saine" à traiter.

5. **(QCM)** Différence `map` vs. `switchMap` :
    * **Réponse : (b) `map` retourne une valeur simple, tandis que `switchMap` doit retourner un nouvel Observable.**

---

## **Module 9 : Les Formulaires Réactifs**

### L'essentiel {id="l-essentiel_8"}

1. **(Question ouverte)** Pourquoi les Formulaires Réactifs pour les Javanistes :
    * Leur approche est programmatique et centrée sur un **modèle de données défini dans le code (TS)**, ce qui est très
      similaire à la création de DTOs ou d'objets de commande en Java. La validation et la structure sont définies dans
      la logique, pas dans la vue, ce qui correspond à la séparation des préoccupations chère aux développeurs backend.

2. **(QCM)** Module pour les Formulaires Réactifs :
    * **Réponse : (d) `ReactiveFormsModule`.**

3. **(QCM)** Directive pour lier un `<input>` :
    * **Réponse : (b) `formControlName`.**

4. **(Question ouverte)** `touched` vs. `dirty` :
    * `touched` devient `true` quand l'utilisateur a **cliqué dans le champ puis en est sorti** (focus puis blur). C'est
      utile pour n'afficher les erreurs qu'après que l'utilisateur a interagi avec le champ.
    * `dirty` devient `true` dès que la **valeur du champ a été modifiée** par l'utilisateur. C'est utile pour savoir si
      un formulaire a des changements non enregistrés.

5. **(QCM)** Rôle de `FormBuilder` :
    * **Réponse : (b) Fournir une syntaxe simplifiée pour créer des instances de `FormGroup` et `FormControl`.**

### Pour aller plus loin {id="pour-aller-plus-loin_8"}

1. **(Question ouverte)** Validateur sur `FormControl` vs. `FormGroup` :
    * Un validateur sur un **`FormControl`** valide la valeur de ce **champ unique** (ex: est-ce un email valide ?). Un
      validateur sur un **`FormGroup`** valide la **relation entre plusieurs champs** du groupe (ex: les deux champs de
      mot de passe sont-ils identiques ?).

2. **(QCM)** Gérer une liste de champs :
    * **Réponse : (c) `FormArray`.**

3. **(QCM)** Réagir à chaque frappe :
    * **Réponse : (c) `valueChanges`.**

4. **(Question ouverte)** `valueChanges` sur un `FormGroup` :
    * On l'utiliserait pour des actions qui dépendent de la combinaison de plusieurs champs. Par exemple, sauvegarder
      automatiquement un brouillon du formulaire à chaque modification, ou recalculer un total ou un résumé qui dépend
      de plusieurs champs du formulaire en temps réel.

5. **(QCM)** Retour d'un validateur synchrone :
    * **Réponse : (b) Quand le champ est valide.**
    * **Explication :** S'il y a une erreur, il retourne un objet d'erreur ; s'il n'y a pas d'erreur, il retourne
      `null`.

---

## **Module 10 : RxJS - Opérateurs Essentiels**

### **L'essentiel** {id="l-essentiel_9"}

1.  **(Question ouverte)** Quel est le rôle principal de la méthode `.pipe()` sur un Observable ?
   *   La méthode `.pipe()` est le mécanisme qui permet de **chaîner des opérateurs RxJS**. Elle crée un "pipeline" (une chaîne de montage) à travers lequel les données émises par l'Observable vont passer. Chaque opérateur dans le pipe prend l'Observable résultant de l'étape précédente et retourne un nouvel Observable transformé ou filtré. Cela permet de composer des logiques de traitement de données complexes de manière lisible et déclarative, sans modifier l'Observable original.

2.  **(QCM)** Vous recevez un `Observable<string>` et vous voulez le transformer en `Observable<number>` qui émet la longueur de la chaîne. Quel opérateur utiliser ?
   *   **Réponse : (b) `map(str => str.length)`**
   *   **Explication :** `map` est l'opérateur de transformation par excellence. Il prend chaque valeur émise par le flux source (ici, une `string`) et applique une fonction pour la transformer en une nouvelle valeur (ici, un `number`).

3.  **(QCM)** Quel est le principal avantage du pipe `async` par rapport à un appel manuel à `.subscribe()` ?
   *   **Réponse : (b) Il gère automatiquement la désinscription, prévenant les fuites de mémoire.**
   *   **Explication :** C'est son avantage le plus important. En gérant `subscribe` et `unsubscribe` pour vous, il élimine l'une des sources d'erreurs les plus courantes dans les applications RxJS : les abonnements qui persistent après la destruction d'un composant.

4.  **(Question ouverte)** Vous voulez afficher un message dans la console chaque fois qu'un Observable émet une valeur, sans changer cette valeur. Quel opérateur est le plus approprié et pourquoi ?
   *   L'opérateur le plus approprié est **`tap`**.
   *   **Pourquoi :** `tap` est conçu pour exécuter des "effets de bord" (side effects), c'est-à-dire des actions qui n'affectent pas le flux de données lui-même. Il permet d'"espionner" la valeur à un point donné du pipeline, d'exécuter une action comme un `console.log`, puis de laisser la valeur continuer son chemin sans aucune modification.

5.  **(QCM)** Comment se nomme la convention de nommage pour une variable qui contient un Observable ?
   *   **Réponse : (c) On la suffixe avec un `$.`**
   *   **Explication :** C'est une convention de la communauté, très répandue et fortement recommandée, pour rendre le code plus lisible. Une variable comme `users$` indique immédiatement au lecteur qu'elle ne contient pas un simple tableau d'utilisateurs, mais un flux (Observable) d'utilisateurs.

---

### **Pour aller plus loin {id="pour-aller-plus-loin_9"}

1.  **(Question ouverte)** Expliquez la différence de comportement fondamentale entre `switchMap` et `concatMap`. Donnez un cas d'usage pour chacun.
   *   **`switchMap` :** C'est la stratégie de l'**annulation**. Dès qu'une nouvelle valeur arrive de la source, `switchMap` se désabonne de l'Observable interne précédent et s'abonne au nouveau. Seul le résultat du dernier Observable interne est émis.
      *   **Cas d'usage :** Les barres de recherche. Si l'utilisateur tape "ang", puis "angu", on veut annuler la recherche pour "ang" et ne lancer que celle pour "angu".
   *   **`concatMap` :** C'est la stratégie de la **file d'attente**. Il attend que l'Observable interne précédent soit complété avant de s'abonner au suivant. Il garantit l'ordre d'exécution.
      *   **Cas d'usage :** Les opérations d'écriture (`POST`, `PUT`) où l'ordre est critique. Si un utilisateur clique rapidement pour créer deux articles, `concatMap` s'assurera que la deuxième requête ne part que lorsque la première a réussi.

2.  **(QCM)** Vous voulez exécuter deux appels `GET` en parallèle et ne continuer que lorsque les deux sont terminés. Quel opérateur utiliser ?
   *   **Réponse : (c) `forkJoin`.**
   *   **Explication :** `forkJoin` est l'équivalent de `Promise.all` pour RxJS. Il attend que tous les Observables fournis se complètent, puis il émet une seule fois un tableau avec la dernière valeur de chacun.

3.  **(QCM)** Dans une barre de recherche, pour éviter de lancer une requête HTTP à chaque frappe de l'utilisateur, quelle combinaison d'opérateurs est la plus pertinente ?
   *   **Réponse : (b) `debounceTime` et `switchMap`.**
   *   **Explication :** C'est un pattern classique. `debounceTime` attend que l'utilisateur ait fini de taper, et `switchMap` annule les requêtes précédentes pour ne lancer que la plus récente.

4.  **(Question ouverte)** Qu'est-ce qu'un "Higher-Order Observable" et quel est le rôle des opérateurs d'aplatissement (`switchMap`, `mergeMap`...) par rapport à ce concept ?
   *   Un **Higher-Order Observable** est un Observable qui émet d'autres Observables comme valeurs (ex: `Observable<Observable<T>>`). Cela se produit typiquement quand on utilise `map` pour déclencher un appel HTTP.
   *   Le rôle des **opérateurs d'aplatissement** est de résoudre ce problème. Ils s'abonnent automatiquement à l'Observable "interne" et "aplatissent" le résultat en un seul flux (`Observable<T>`), évitant ainsi les `subscribe` imbriqués et rendant le code lisible.

5.  **(QCM)** Quel opérateur est le plus adapté pour gérer les double-clics sur un bouton de soumission de formulaire ?
   *   **Réponse : (d) `exhaustMap`.**
   *   **Explication :** `exhaustMap` s'abonne à l'Observable interne (l'appel HTTP) lors du premier clic et **ignore toutes les nouvelles valeurs de la source (les clics supplémentaires)** tant que cet Observable interne n'est pas complété. C'est exactement le comportement désiré pour éviter les soumissions multiples.

---

## **Module 11 : Sujets Complémentaires**

### L'essentiel {id="l-essentiel_10"}

1. **(Question ouverte)** Appel API dans `constructor` vs. `ngOnInit` :
    * C'est une mauvaise pratique dans le `constructor` car à ce moment, les liaisons de données (comme les propriétés
      `@Input`) ne sont pas encore initialisées. `ngOnInit` est le hook de cycle de vie qui garantit que toutes les
      propriétés du composant sont en place, c'est donc l'endroit sûr et approprié pour la logique d'initialisation
      comme les appels API.

2. **(QCM)** Hook pour se désabonner :
    * **Réponse : (c) `ngOnDestroy`.**

3. **(QCM)** Commande pour le build de production :
    * **Réponse : (b) `ng build`.**
    * **Explication :** Par défaut, `ng build` crée un build de production optimisé.

4. **(Question ouverte)** Trois optimisations de `ng build` :
    * On peut citer : la **Minification** (réduction de la taille des fichiers), le **Bundling** (regroupement des
      fichiers), le **Tree Shaking** (suppression du code inutilisé), et la compilation **AOT** (Ahead-of-Time).

5. **(QCM)** Dossier pour les éléments réutilisables :
    * **Réponse : (c) `src/app/shared`.**

### Pour aller plus loin {id="pour-aller-plus-loin_10"}

1. **(Question ouverte)** `ngOnInit` vs. `ngOnChanges` :
    * `ngOnInit` est appelé **une seule fois** lors de l'initialisation du composant. `ngOnChanges` est appelé *
      *avant `ngOnInit` et à chaque fois qu'une propriété `@Input` change de valeur**, permettant de réagir à ces
      changements.

2. **(QCM)** Stratégie de détection de changement performante :
    * **Réponse : (b) `OnPush`.**

3. **(QCM)** Rôle d'un intercepteur HTTP :
    * **Réponse : (b) Se placer entre les appels HTTP et le backend pour les manipuler globalement.**

4. **(Question ouverte)** Déclencher une mise à jour en `OnPush` :
    * Deux conditions parmi :
        1. Une propriété `@Input` change de **référence**.
        2. Un événement (comme `click`) est déclenché depuis le template du composant.
        3. Le pipe `async` est utilisé dans son template et l'Observable émet.
        4. On appelle manuellement `ChangeDetectorRef.markForCheck()`.

5. **(QCM)** Dossier pour une directive réutilisable :
    * **Réponse : (b) `shared`.**