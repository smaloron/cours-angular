# Module 10 : Pour aller plus loin - Opérateurs de Combinaison et d'Aplatissement

### Objectifs pédagogiques

À la fin de cette partie, vous serez en mesure de :

* **Comprendre** la problématique des "Observables d'Observables" et comment les "aplatir".
* **Maîtriser** l'opérateur `switchMap` pour gérer les appels HTTP dépendants, le cas d'usage le plus courant.
* **Différencier** les stratégies d'aplatissement : `switchMap`, `mergeMap`, `concatMap`, et `exhaustMap`.
* **Combiner** les résultats de plusieurs Observables indépendants en un seul flux avec `forkJoin` et `combineLatest`.

### Introduction : Gérer la complexité des flux asynchrones

Dans le monde réel, les flux de données sont rarement simples et isolés. Ils interagissent, dépendent les uns des
autres, ou doivent être synchronisés.

* Comment faire un appel API, et seulement après avoir reçu la réponse, en faire un deuxième avec les données du
  premier ?
* Comment lancer plusieurs appels API en parallèle et attendre que **tous** soient terminés avant d'agir ?
* Comment combiner les données d'un formulaire qui change en temps réel avec les résultats d'un appel API ?

Tenter de gérer ces scénarios avec des `.subscribe()` imbriqués mène à un code illisible et difficile à maintenir,
souvent appelé "callback hell". Les opérateurs d'aplatissement et de combinaison sont la solution de RxJS à ces
problèmes. Ils vous permettent de garder un code plat, lisible et déclaratif, même pour les scénarios asynchrones les
plus complexes.

### 1. Le Problème des "Higher-Order Observables"

Un "Higher-Order Observable" est un Observable qui émet d'autres Observables. C'est ce qui se passe lorsque vous
utilisez `map` pour lancer un appel HTTP.

```typescript
// MAUVAIS exemple : on se retrouve avec un Observable<Observable<Post[]>>
// C'est un "Higher-Order Observable"
getUserPosts()
:
Observable < Observable < Post[] >> {
    return this.http.get<User>('/api/user/1').pipe(
        map(user => this.http.get<Post[]>(`/api/posts?userId=${user.id}`))
    );
}
```

Pour utiliser le résultat, il faudrait s'abonner deux fois. C'est lourd et source d'erreurs.

```typescript
// Surtout, ne faites pas ça !
this.userService.getUserPosts().subscribe(obsPosts => {
    obsPosts.subscribe(posts => {
        this.posts = posts;
    });
});
```

La solution est d'**aplatir** ce flux : transformer `Observable<Observable<T>>` en un simple `Observable<T>`. C'est le
rôle des opérateurs `switchMap`, `mergeMap`, etc.

### 2. Les Opérateurs d'Aplatissement

Ces opérateurs s'abonnent à l'Observable interne pour vous et vous renvoient directement ses valeurs. Ils ne diffèrent
que par la manière dont ils gèrent les arrivées multiples de l'Observable source.

#### `switchMap` : La Stratégie d'Annulation

`switchMap` est le plus courant pour les requêtes déclenchées par l'utilisateur (recherche, clic...).

* **Comportement :** Dès que l'Observable source émet une nouvelle valeur, `switchMap` **se désabonne de l'Observable
  interne précédent** et s'abonne au nouveau.
* **Analogie :** Vous tapez "a", une recherche est lancée. Vous tapez "b" 100ms plus tard. La recherche pour "a" est *
  *annulée**, et une nouvelle recherche pour "ab" est lancée.
* **Cas d'usage :** Parfait pour les champs de recherche, les filtres... tout ce qui peut être déclenché rapidement et
  où seule la dernière valeur compte.

```typescript
// Exemple parfait : un champ de recherche
searchControl.valueChanges.pipe(
    debounceTime(300),
    distinctUntilChanged(),
    // Si l'utilisateur tape une nouvelle valeur, la requête HTTP précédente est annulée.
    switchMap(term => this.api.search(term))
).subscribe(results =>
...)
;
```

#### `mergeMap` (ou `flatMap`) : La Stratégie Parallèle

`mergeMap` ne se soucie pas de l'ordre. Il s'abonne à tous les Observables internes et émet leurs valeurs dès qu'elles
arrivent.

* **Comportement :** Exécute tout en parallèle.
* **Analogie :** Vous cliquez 3 fois sur un bouton "Ajouter au panier". 3 requêtes partent en même temps. Les réponses
  arrivent dans le désordre, mais elles arrivent toutes.
* **Cas d'usage :** Utile pour des actions indépendantes que l'on veut exécuter sans attendre la précédente (ex: upload
  de plusieurs fichiers).

#### `concatMap` : La Stratégie Séquentielle (File d'attente)

`concatMap` attend que l'Observable interne précédent soit complété avant de s'abonner au suivant.

* **Comportement :** Traite les émissions une par une, dans l'ordre.
* **Analogie :** Vous cliquez 3 fois pour sauvegarder. La 2ème sauvegarde ne sera lancée **qu'après** la fin de la 1ère.
  La 3ème après la fin de la 2ème.
* **Cas d'usage :** Crucial pour les opérations de type `POST`, `PUT`, `DELETE` où l'ordre est important et où l'on veut
  éviter les "race conditions".

#### `exhaustMap` : La Stratégie "Ignorer"

`exhaustMap` ignore les nouvelles émissions de la source tant que son Observable interne n'est pas complété.

* **Comportement :** Ignore les nouvelles demandes tant que la précédente est en cours.
* **Analogie :** Vous cliquez sur un bouton "Login". La requête part. Vous re-cliquez frénétiquement. Tous les clics
  supplémentaires sont **ignorés** jusqu'à ce que la première requête de login soit terminée.
* **Cas d'usage :** Parfait pour les boutons de soumission de formulaire pour éviter les double-clics.

### 3. Opérateurs de Combinaison

Ces opérateurs permettent de gérer plusieurs Observables indépendants.

#### `forkJoin` : Attendre que tout soit fini

`forkJoin` est l'équivalent RxJS de `Promise.all`. Il prend un tableau d'Observables, attend que **tous** se complètent,
puis émet une seule valeur : un tableau contenant la dernière valeur de chaque Observable.

**Scénario :** Pour afficher une page, j'ai besoin des détails de l'utilisateur ET de la liste des catégories.

```typescript
import {forkJoin} from 'rxjs';

const userDetails$ = this.http.get<User>('/api/user/1');
const categories$ = this.http.get<Category[]>('/api/categories');

forkJoin([userDetails$, categories$]).subscribe(
    ([user, categories]) => {
        // Ce code ne s'exécute que lorsque les DEUX appels sont terminés.
        console.log('Utilisateur:', user);
        console.log('Catégories:', categories);
        this.user = user;
        this.categories = categories;
    }
);
```

#### `combineLatest` : La combinaison en temps réel

`combineLatest` est différent. Il émet une valeur dès que **tous** les Observables ont émis au moins une fois. Ensuite,
il émet une nouvelle valeur **à chaque fois que l'un des Observables source émet**.

**Scénario :** Afficher une liste de produits filtrée. Les filtres (`Observable` venant d'un formulaire) et la liste de
produits (`Observable` venant d'un appel HTTP) sont combinés.

```typescript
import {combineLatest} from 'rxjs';

// Observable des produits (n'émet qu'une fois)
const products$ = this.productService.getProducts();

// Observable des filtres (émet à chaque changement du formulaire)
const filters$ = this.filterForm.valueChanges;

combineLatest([products$, filters$]).pipe(
    map(([products, filters]) => {
        // Cette fonction est ré-exécutée à chaque changement de filtre
        return products.filter(p => p.category === filters.category);
    })
).subscribe(filteredProducts => this.displayProducts(filteredProducts));
```

### Exercice pratique

#### Exercice 2 : Tableau de bord agrégé

Créez un composant "Dashboard" qui a besoin de plusieurs informations pour s'afficher.

1. Dans votre `UserService`, créez une méthode `getLoggedInUser()` qui retourne les détails d'un utilisateur (simulez
   avec `http.get('.../users/1')`).
2. Dans votre `PostService`, créez une méthode `getLatestPosts()` qui retourne les 5 derniers posts (simulez avec
   `http.get('.../posts?_limit=5')`).
3. Dans votre `CommentService`, créez `getLatestComments()` (simulez avec `http.get('.../comments?_limit=5')`).
4. Dans votre `DashboardComponent`, utilisez `forkJoin` pour lancer ces trois appels en parallèle.
5. Lorsque tous les appels sont terminés, stockez les résultats dans des propriétés séparées (`user`, `posts`,
   `comments`).
6. Utilisez le pipe `async` et `@if...as` dans votre template pour afficher un message de chargement, puis les données
   une fois qu'elles sont toutes arrivées.

#### Correction exercice 2 {collapsible='true'}

1. à 3. **Services :**
   Les méthodes dans les services respectifs retourneront simplement les appels HTTP.
   Exemple pour `UserService`:

```typescript
public
getLoggedInUser()
:
Observable < User > {
    return this.http.get<User>(`${this.apiUrl}/1`);
}
```

(Idem pour les autres services)

4. à 6. **`dashboard.component.ts` et `.html`**

```typescript
import {Component} from '@angular/core';
import {forkJoin, Observable} from 'rxjs';
import {map} from 'rxjs/operators';
// ... import des services et modèles

// Un modèle pour le résultat combiné
interface DashboardData {
    user: User;
    posts: Post[];
    comments: Comment[];
}

@Component({ /* ... */})
export class DashboardComponent {
    public data$: Observable<DashboardData>;

    constructor(
        private userService: UserService,
        private postService: PostService,
        private commentService: CommentService
    ) {
        // On lance les appels avec forkJoin
        this.data$ = forkJoin({
            user: this.userService.getLoggedInUser(),
            posts: this.postService.getLatestPosts(),
            comments: this.commentService.getLatestComments()
        });
        // forkJoin peut aussi prendre un objet, c'est plus lisible !
    }
}
```

```html
<!-- dashboard.component.html -->
<h1>Tableau de Bord</h1>

@if (data$ | async; as data) {
<section>
    <h2>Bienvenue, {{ data.user.name }} !</h2>
</section>
<section>
    <h3>Derniers Posts</h3>
    <ul>
        @for(post of data.posts; track post.id) {
        <li>{{post.title}}</li>
        }
    </ul>
</section>
<section>
    <h3>Derniers Commentaires</h3>
    <ul>
        @for(comment of data.comments; track comment.id) {
        <li>{{comment.name}}</li>
        }
    </ul>
</section>
} @else {
<p>Chargement du tableau de bord...</p>
}
```

### Auto-évaluation

1. **(Question ouverte)** Expliquez la différence de comportement fondamentale entre `switchMap` et `concatMap`. Donnez
   un cas d'usage pour chacun.
2. **(QCM)** Vous voulez exécuter deux appels `GET` en parallèle et ne continuer que lorsque les deux sont terminés.
   Quel opérateur utiliser ?
   a) `combineLatest`
   b) `mergeMap`
   c) `forkJoin`
   d) `zip`
3. **(QCM)** Dans une barre de recherche, pour éviter de lancer une requête HTTP à chaque frappe de l'utilisateur,
   quelle combinaison d'opérateurs est la plus pertinente ?
   a) `tap` et `map`
   b) `debounceTime` et `switchMap`
   c) `delay` et `concatMap`
   d) `filter` et `mergeMap`
4. **(Question ouverte)** Qu'est-ce qu'un "Higher-Order Observable" et quel est le rôle des opérateurs d'aplatissement (
   `switchMap`, `mergeMap`...) par rapport à ce concept ?
5. **(QCM)** Quel opérateur est le plus adapté pour gérer les double-clics sur un bouton de soumission de formulaire ?
   a) `switchMap` (annule)
   b) `mergeMap` (exécute tout)
   c) `concatMap` (met en file d'attente)
   d) `exhaustMap` (ignore pendant l'exécution)

### Conclusion de ce chapitre

Félicitations ! Vous avez exploré les aspects les plus puissants et les plus utiles de RxJS. Vous êtes maintenant
capable de gérer des scénarios asynchrones complexes qui sont le quotidien des applications web modernes.

Vous savez comment **enchaîner des appels dépendants** avec `switchMap`, comment **paralléliser et synchroniser** des
appels avec `forkJoin`, et comment choisir la bonne **stratégie d'aplatissement** en fonction du contexte.

Cette connaissance approfondie de RxJS ne vous servira pas seulement dans Angular, mais dans tout l'écosystème
JavaScript moderne. C'est une compétence fondamentale qui vous place à un niveau de développeur senior.

Nous avons maintenant couvert l'essentiel du framework. Le dernier module abordera des sujets complémentaires mais
importants, comme le cycle de vie des composants et le build de production, pour finaliser votre parcours.