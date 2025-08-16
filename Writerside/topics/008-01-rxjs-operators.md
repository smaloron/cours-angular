# Module 8 : Pour aller plus loin - La Puissance des Opérateurs RxJS

### Objectifs pédagogiques

À la fin de cette partie, vous serez en mesure de :

* **Utiliser** la méthode `.pipe()` pour enchaîner des opérateurs RxJS.
* **Transformer** les données d'un flux avec l'opérateur `map`.
* **Filtrer** les données d'un flux avec l'opérateur `filter`.
* **Gérer** les erreurs HTTP de manière centralisée avec l'opérateur `catchError`.
* **Comprendre** la différence fondamentale entre `map` et `switchMap` pour gérer les appels HTTP dépendants.

### Introduction : La Chaîne de Montage de vos Données

Dans la partie "L'essentiel", nous avons récupéré des données brutes de l'API et tout le traitement se faisait dans le
`.subscribe()` de notre composant. C'est fonctionnel, mais cela peut vite devenir complexe et charger le composant de
responsabilités qui ne sont pas les siennes.

Imaginez une chaîne de montage dans une usine. Les matières premières (la réponse de l'API) arrivent d'un côté. Au lieu
de tout faire à la fin de la chaîne, on place des "opérateurs" spécialisés tout le long :

1. Un premier poste **filtre** les pièces défectueuses.
2. Un deuxième poste **transforme** chaque pièce (la peint en rouge, par exemple).
3. Un troisième poste **ajoute** un élément à chaque pièce.

Les **opérateurs RxJS** sont ces postes de travail. Ils vous permettent de construire un "pipeline" (`.pipe()`) pour
traiter, transformer, filtrer et combiner vos flux de données avant même que le composant ne reçoive le résultat final.
Votre composant reçoit alors des données "prêtes à l'emploi", ce qui le rend beaucoup plus simple et propre.

### 1. Le Pipeline : `.pipe()`

Tous les opérateurs RxJS s'utilisent à l'intérieur de la méthode `.pipe()` d'un Observable. Cette méthode prend un
nombre illimité d'opérateurs en arguments, qui seront appliqués dans l'ordre.

```typescript
import {map, filter, tap} from 'rxjs/operators';

// ...
myObservable.pipe(
    filter(data =>...), // D'abord on filtre
    map(data =>...),    // Ensuite on transforme
    tap(data =>...)     // Puis on observe
).subscribe(...);
```

### 2. Opérateurs de Transformation et de Filtrage

#### `map` : Transformer chaque valeur émise

L'opérateur `map` est probablement le plus utilisé. Il prend une valeur en entrée et retourne une nouvelle valeur
transformée. C'est exactement comme le `.map()` des tableaux en JavaScript.

**Scénario :** L'API nous retourne une liste d'utilisateurs, mais nous ne voulons que leurs noms.

```typescript
// Fichier: user.service.ts
import {map} from 'rxjs/operators';

public getUserNames(): Observable < string[] > {
    return this.http.get<User[]>(this.apiUrl).pipe(
        // map prend le tableau de User[] en entrée
        // et doit retourner un tableau de string[] en sortie.
        map(users => users.map(user => user.name))
    );
}
```

Le composant qui s'abonne à `getUserNames()` recevra directement un tableau de chaînes de caractères. Il n'a même pas
besoin de connaître l'existence de l'objet `User`.

#### `filter` : Ne laisser passer que certaines valeurs

L'opérateur `filter` ne s'applique généralement pas directement aux appels HTTP (qui n'émettent qu'une seule valeur, le
tableau complet), mais il est fondamental dans RxJS. Il est très utile pour des flux qui émettent plusieurs valeurs (
comme des événements de clic ou des WebSockets).

Pour filtrer les *éléments d'un tableau* reçu via HTTP, on combine `map` et le `.filter()` des tableaux.

**Scénario :** Récupérer uniquement les utilisateurs "actifs" d'une API (supposons qu'ils ont une propriété `isActive`).

```typescript
// Fichier: user.service.ts
import {map} from 'rxjs/operators';

public getActiveUsers(): Observable < User[] > {
    return this.http.get<User[]>(this.apiUrl).pipe(
        // map prend le tableau de User[] en entrée
        // et retourne un NOUVEAU tableau de User[] filtré.
        map(users => users.filter(user => user.isActive)) // Supposons que User a une prop isActive
    );
}
```

### 3. Gestion des Erreurs avec `catchError`

Que se passe-t-il si l'appel HTTP échoue ? Au lieu de gérer l'erreur dans chaque `.subscribe()`, on peut l'intercepter
directement dans le service avec `catchError`.

Cet opérateur doit retourner un **nouvel Observable** qui servira de "plan B". Souvent, on veut juste retourner un
Observable qui émet une valeur par défaut (comme un tableau vide) et loguer l'erreur.

```typescript
// Fichier: user.service.ts
import {catchError, map} from 'rxjs/operators';
import {of} from 'rxjs'; // 'of' est une fonction qui crée un Observable à partir d'une valeur simple.

public getUsers(): Observable < User[] > {
    return this.http.get<User[]>(this.apiUrl).pipe(
        tap(users => console.log(`${users.length} utilisateurs reçus de l'API`)), // tap est utile pour les "effets de bord" comme le logging
        catchError(error => {
            console.error("Erreur interceptée dans le service :", error);
            // On retourne un Observable qui émet un tableau vide.
            // Le composant recevra un tableau vide au lieu d'une erreur.
            return of([]);
        })
    );
}
```

Avec ce code, le canal `error` du `subscribe` ne sera jamais appelé. Le composant recevra simplement `[]`, ce qui est
souvent plus simple à gérer dans le template.

### 4. `map` vs `switchMap` : L'Opérateur d'Aplatissement Clé

C'est un concept avancé mais **absolument crucial**.

**Problème :** Vous avez des appels HTTP qui dépendent les uns des autres.

1. Récupérer les informations de l'utilisateur connecté (`/me`).
2. Une fois qu'on a son ID, récupérer ses commandes (`/users/{id}/orders`).

Si on utilise `map`, on se retrouve avec un `Observable` dans un `Observable` (`Observable<Observable<Order[]>>`). C'est
un cauchemar à gérer.

```typescript
// MAUVAISE approche avec map
getUserOrders_Bad(): Observable < Observable < Order[] >> {
    return this.http.get<User>('/me').pipe(
        map(user => this.http.get<Order[]>(`/users/${user.id}/orders`))
    );
}
```

**Solution :** Les opérateurs "d'aplatissement" comme `switchMap`.
`switchMap` est conçu pour ce scénario. Il prend la valeur du premier Observable (`user`), puis attend que vous
retourniez un **nouvel Observable** (`http.get(...)`). Il va automatiquement "s'abonner" à ce nouvel Observable interne
et vous retourner directement les valeurs de celui-ci.

```typescript
// BONNE approche avec switchMap
import {switchMap} from 'rxjs/operators';

getUserOrders_Good(): Observable < Order[] > {
    return this.http.get<User>('/me').pipe(
        // user est l'objet User du premier appel
        // On doit retourner un NOUVEL observable
        switchMap(user => this.http.get<Order[]>(`/users/${user.id}/orders`))
    );
}
```

Le composant qui s'abonne à `getUserOrders_Good()` recevra directement un `Order[]`. C'est simple et propre. `switchMap`
est l'opérateur de choix pour les appels HTTP enchaînés.

### Exercice pratique

#### Exercice 2 : Afficher les Posts d'un Utilisateur Spécifique

Votre mission est d'utiliser `switchMap` pour afficher uniquement les "posts" (articles) du premier utilisateur retourné
par l'API.

1. Dans votre `UserService`, créez une méthode `getFirstUser()` qui retourne un `Observable<User>` (le premier
   utilisateur de la liste `/users`). Utilisez l'opérateur `map` pour ne prendre que `users[0]`.
2. Créez un `PostService` avec une méthode `getPostsForUser(userId: number)` qui retourne un `Observable<Post[]>` en
   appelant `https://jsonplaceholder.typicode.com/posts?userId={userId}`. (Créez une interface `Post` : `id`, `userId`,
   `title`, `body`).
3. Créez un `UserPostsComponent`.
4. Dans ce composant, injectez `UserService` et `PostService`.
5. Dans `ngOnInit`, chaînez les appels :
   a. Appelez `userService.getFirstUser()`.
   b. Utilisez `.pipe()` avec `switchMap` sur le résultat.
   c. À l'intérieur de `switchMap`, utilisez l'objet `user` reçu pour appeler `postService.getPostsForUser(user.id)`.
   d. Abonnez-vous au résultat final pour afficher les posts.

#### Correction exercice 2 {collapsible='true'}

1. **`user.service.ts` mis à jour :**
   
```typescript
   import { map } from 'rxjs/operators';
   // ...
getFirstUser(): Observable<User> {
     return this.http.get<User[]>(this.apiUrl).pipe(
       map(users => users[0])
     );
}
   ```

2. **`post.service.ts` :**
   ```typescript
   // Fichier: src/app/shared/services/post.service.ts
   import { Injectable } from '@angular/core';
   import { HttpClient } from '@angular/common/http';
   import { Observable } from 'rxjs';
   
   export interface Post { id: number; userId: number; title: string; body: string; }

   @Injectable({ providedIn: 'root' })
   export class PostService {
     private apiUrl = 'https://jsonplaceholder.typicode.com/posts';
     constructor(private http: HttpClient) {}

     getPostsForUser(userId: number): Observable<Post[]> {
       return this.http.get<Post[]>(`${this.apiUrl}?userId=${userId}`);
     }
   }
   ```

3. & 4. & 5. **`user-posts.component.ts` :**
   ```typescript
   import { Component, OnInit } from '@angular/core';
   import { switchMap } from 'rxjs/operators';
   import { UserService } from '../../shared/services/user.service';
   import { PostService, Post } from '../../shared/services/post.service';

   @Component({ /* ... */ })
   export class UserPostsComponent implements OnInit {
     public posts: Post[] = [];

     constructor(
       private userService: UserService,
       private postService: PostService
     ) {}

     ngOnInit(): void {
       this.userService.getFirstUser().pipe(
         switchMap(user => {
           if (!user) {
             // Gérer le cas où il n'y a pas d'utilisateur
             return []; // Retourne un observable vide
           }
           console.log(`Premier utilisateur trouvé : ${user.name}, ID: ${user.id}`);
           return this.postService.getPostsForUser(user.id);
         })
       ).subscribe(posts => {
         this.posts = posts;
         console.log(`Posts trouvés pour le premier utilisateur :`, posts);
       });
     }
   }
   ```

### Auto-évaluation

1. **(Question ouverte)** À quoi sert la méthode `.pipe()` d'un Observable ?
2. **(QCM)** Quel opérateur utiliseriez-vous pour transformer un `Observable<User[]>` en un `Observable<number>` qui
   émet uniquement le nombre total d'utilisateurs ?
   a) `filter`
   b) `switchMap`
   c) `map`
   d) `catchError`
3. **(QCM)** Vous devez faire un premier appel pour récupérer un `id` de catégorie, puis un second appel pour récupérer
   les produits de cette catégorie. Quel opérateur est le plus adapté pour enchaîner ces deux appels ?
   a) `map`
   b) `tap`
   c) `switchMap`
   d) `merge`
4. **(Question ouverte)** Expliquez le rôle de l'opérateur `catchError`. Que doit-il retourner pour que l'Observable ne
   se termine pas en erreur ?
5. **(QCM)** Quelle est la principale différence de comportement entre `map` et `switchMap` ?
   a) `map` transforme une valeur, tandis que `switchMap` filtre une valeur.
   b) `map` retourne une valeur simple, tandis que `switchMap` doit retourner un nouvel Observable.
   c) `switchMap` est plus rapide que `map`.
   d) `map` peut être utilisé pour les erreurs, pas `switchMap`.

### Conclusion de ce chapitre

Un immense bravo ! Vous venez de débloquer le véritable super-pouvoir d'Angular : la programmation réactive avec **RxJS
**. Vous ne vous contentez plus de consommer des données ; vous les orchestrez. Vous savez maintenant construire des *
*pipelines de traitement de données** robustes et déclaratifs avec `.pipe()`, transformer, filtrer, et gérer les erreurs
de manière centralisée.

Plus important encore, vous avez compris la puissance de `switchMap` pour gérer les dépendances asynchrones, un des
scénarios les plus courants et les plus complexes du développement web.

Cette connaissance des opérateurs RxJS est ce qui distingue un développeur Angular débutant d'un développeur
expérimenté. Vous êtes maintenant prêt à aborder le dernier grand pilier d'une application : la gestion des entrées
utilisateur avec les **Formulaires Réactifs**.