# Module 11 : Pour aller plus loin - Architectures Avancées et Performance

### Objectifs pédagogiques

À la fin de cette partie, vous serez en mesure de :

* **Comprendre** en détail le hook de cycle de vie `ngOnChanges` pour réagir aux changements des `@Input`.
* **Structurer** votre application en "Features Modules" et "Shared Modules" pour une meilleure organisation.
* **Mettre en place** une stratégie de détection de changement `OnPush` pour optimiser les performances de
  l'application.
* **Connaître** l'existence des Intercepteurs HTTP pour manipuler globalement les requêtes et les réponses.

### Introduction : De l'application fonctionnelle à l'application d'exception

Votre application fonctionne. Elle est bien structurée et prête pour la production. Mais comment la faire passer au
niveau supérieur ? Comment s'assurer qu'elle reste rapide et réactive même avec des centaines de composants à l'écran ?
Comment gérer des problématiques transversales comme l'ajout d'un token d'authentification à chaque requête HTTP sans
dupliquer de code ?

Cette section vous plonge dans les techniques utilisées pour construire des applications Angular à grande échelle. Nous
allons parler d'**optimisation des performances** en contrôlant finement comment et quand Angular vérifie les
changements. Nous aborderons des **patterns d'architecture** pour organiser votre code de manière encore plus modulaire.
Enfin, nous découvrirons les **intercepteurs**, un outil puissant pour gérer les requêtes HTTP de manière globale.

Ces concepts sont la marque des développeurs et architectes Angular expérimentés.

### 1. `ngOnChanges` : Réagir aux changements d'Inputs

`ngOnInit` est appelé une seule fois. Mais que se passe-t-il si une propriété `@Input` de votre composant change *après*
l'initialisation ? C'est là qu'intervient `ngOnChanges`.

* **Quand ?** Appelé avant `ngOnInit` et à chaque fois qu'une ou plusieurs des propriétés `@Input` du composant changent
  de valeur.
* **Comment ?** Il reçoit en argument un objet `SimpleChanges` qui détaille, pour chaque `@Input` modifié, sa valeur
  précédente et sa valeur actuelle.

**Scénario :** Un composant `StockChartComponent` qui doit redessiner un graphique chaque fois que le symbole de
l'action (`stockSymbol`) fourni par le parent change.

```typescript
import {Component, Input, OnChanges, SimpleChanges} from '@angular/core';

@Component({ /* ... */})
export class StockChartComponent implements OnChanges {
    @Input() stockSymbol: string = '';

    ngOnChanges(changes: SimpleChanges): void {
        // On vérifie si la propriété 'stockSymbol' a changé
        if (changes['stockSymbol']) {
            const change = changes['stockSymbol'];
            console.log(`Le symbole a changé de '${change.previousValue}' à '${change.currentValue}'.`);

            // C'est ici qu'on lancerait la logique pour mettre à jour le graphique
            this.redrawChart(change.currentValue);
        }
    }

    private redrawChart(symbol: string): void {
        console.log(`Redessin du graphique pour ${symbol}...`);
    }
}
```

<warning>
`ngOnChanges` peut être coûteux en performance s'il est mal utilisé, car il se déclenche souvent. Une alternative plus moderne et souvent plus performante pour réagir aux changements d'un input est d'utiliser un "setter" ou de transformer l'input en `Signal` ou en `Observable`.
</warning>

### 2. Organisation du Code : Feature Modules et Shared Module

Même si les composants "standalone" ont simplifié les choses, le concept de "Module" reste un excellent pattern
d'organisation.

* **Feature Modules :** Au lieu de mettre tous vos composants liés à une fonctionnalité (ex: "gestion des produits") à
  la racine, regroupez-les dans un dossier `/products`. Ce dossier devient un "module de fonctionnalité". Si vous
  utilisez le lazy loading, vous chargerez ce module en entier.
* **Shared Module :** Vous aurez des composants, pipes et directives réutilisables partout (ex: un `ButtonComponent`, un
  `TruncatePipe`). Regroupez-les dans un dossier `/shared`. Vous pouvez même créer un `shared.module.ts` qui exporte
  tout ce beau monde, pour ne faire qu'un seul import dans les autres modules.

```
src/app/
├── products/
│   ├── product-list/
│   ├── product-detail/
│   └── products.routes.ts   (routes du feature module)
│
├── orders/
│   ├── ... (autre feature module)
│
└── shared/
    ├── components/
    │   └── button/
    ├── pipes/
    │   └── truncate.pipe.ts
    └── directives/
        └── highlight.directive.ts
```

### 3. Optimisation des Performances : `ChangeDetectionStrategy.OnPush`

Par défaut, Angular utilise une stratégie de détection de changement "sale" (`Default`). À chaque événement (clic,
retour de requête HTTP...), il vérifie **toute l'arborescence des composants** depuis la racine pour voir si quelque
chose a changé et doit être ré-affiché. Dans une grosse application, cela peut être lent.

`ChangeDetectionStrategy.OnPush` est une stratégie beaucoup plus performante. Un composant `OnPush` ne sera vérifié et
ré-affiché que si **l'une de ces conditions** est remplie :

1. Une de ses propriétés `@Input` change de **référence** (pas juste une modification interne de l'objet).
2. Un événement (comme un `(click)`) se déclenche **depuis son propre template** (ou celui d'un de ses enfants).
3. On utilise le pipe `async` dans son template.
4. On demande manuellement à Angular de le vérifier (via `ChangeDetectorRef`).

**Règle d'or :** Essayez de rendre un maximum de vos composants "de présentation" (ceux qui ne font qu'afficher des
données via des `@Input`) `OnPush`. Cela réduit drastiquement le travail qu'Angular doit faire.

```typescript
import {Component, Input, ChangeDetectionStrategy} from '@angular/core';

@Component({
    selector: 'app-user-avatar',
    standalone: true,
    // C'est ici que la magie opère
    changeDetection: ChangeDetectionStrategy.OnPush,
    template: `<img [src]="user.avatarUrl" alt="Avatar">`
})
export class UserAvatarComponent {
    // Ce composant ne sera mis à jour que si l'objet 'user' entier est remplacé,
    // pas si on fait user.avatarUrl = '...' depuis l'extérieur.
    @Input() user!: User;
}
```

Cette technique, combinée à des pratiques de programmation immuable (ne pas modifier les objets, mais en créer de
nouveaux), est la clé des applications Angular ultra-rapides.

### 4. Intercepteurs HTTP (`HttpInterceptor`)

Un intercepteur est un service spécial qui se place entre votre `HttpClient` et le backend. Il peut **intercepter, lire
et modifier toutes les requêtes HTTP sortantes et toutes les réponses entrantes**.

C'est un outil extraordinairement puissant pour la gestion des problématiques transversales :

* **Ajouter automatiquement un token d'authentification** à l'en-tête de chaque requête.
* **Ajouter des en-têtes communs** (ex: `Content-Type: application/json`).
* **Gérer globalement les erreurs HTTP** (ex: rediriger vers la page de login si on reçoit une erreur 401 Unauthorized).
* **Mettre en cache** certaines réponses.
* **Afficher un spinner de chargement global** au début de chaque requête et le cacher à la fin.

**Scénario :** Un intercepteur qui ajoute un token JWT à chaque requête.

```typescript
// Fichier: src/app/shared/interceptors/auth.interceptor.ts
import {HttpInterceptorFn} from '@angular/common/http';
import {inject} from '@angular/core';
import {AuthService} from '../services/auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
    const authService = inject(AuthService);
    const authToken = authService.getToken();

    // On clone la requête originale pour y ajouter un en-tête.
    // Les requêtes sont immuables.
    const authReq = req.clone({
        setHeaders: {
            Authorization: `Bearer ${authToken}`
        }
    });

    // On passe la nouvelle requête au handler suivant.
    return next(authReq);
};
```

**Mise en place (`app.config.ts`) :**

```typescript
// Fichier: src/app/app.config.ts
import {provideHttpClient, withInterceptors} from '@angular/common/http';
import {authInterceptor} from './shared/interceptors/auth.interceptor';

export const appConfig: ApplicationConfig = {
    providers: [
        // ...
        // On fournit le HttpClient avec notre intercepteur
        provideHttpClient(withInterceptors([authInterceptor]))
    ]
};
```

Désormais, chaque appel `http.get()`, `post()`, etc., passera automatiquement par cet intercepteur.

### Exercice pratique

#### Exercice 2 : Afficher un spinner de chargement global

Créez un intercepteur qui affiche un spinner de chargement pendant les appels HTTP.

1. Créez un `LoadingService` (fourni à la racine) avec un `BehaviorSubject` ou un `Signal` pour conserver l'état de
   chargement (`true`/`false`). Il doit avoir des méthodes `show()` et `hide()`.
2. Créez un `LoadingInterceptor` :

* Avant de passer la requête (`next(req)`), il doit appeler `loadingService.show()`.
* Il doit utiliser l'opérateur `finalize` dans le `.pipe()` de l'observable retourné par `next(req)`. La fonction
  `finalize` s'exécute toujours, que la requête réussisse ou échoue.
* Dans `finalize`, il doit appeler `loadingService.hide()`.

3. Configurez l'intercepteur dans `app.config.ts`.
4. Dans `AppComponent`, injectez le `LoadingService` et affichez un spinner ou un simple texte "Chargement..."
   conditionnellement, en fonction de l'état du service.

#### Correction exercice 2 {collapsible='true'}

1. **`loading.service.ts`**
   ```typescript
   import { Injectable, signal } from '@angular/core';

   @Injectable({ providedIn: 'root' })
   export class LoadingService {
     public readonly isLoading = signal(false);

     public show(): void {
       this.isLoading.set(true);
     }

     public hide(): void {
       this.isLoading.set(false);
     }
   }
   ```

2. **`loading.interceptor.ts`**
   ```typescript
   import { HttpInterceptorFn } from '@angular/common/http';
   import { inject } from '@angular/core';
   import { finalize } from 'rxjs/operators';
   import { LoadingService } from '../services/loading.service';

   export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
     const loadingService = inject(LoadingService);
     loadingService.show();
     
     return next(req).pipe(
       finalize(() => loadingService.hide())
     );
   };
   ```

3. **`app.config.ts` (ajout de l'intercepteur)**
   ```typescript
   import { provideHttpClient, withInterceptors } from '@angular/common/http';
   import { loadingInterceptor } from './shared/interceptors/loading.interceptor';
   
   // ...
   providers: [
     provideHttpClient(withInterceptors([loadingInterceptor]))
   ]
   ```

4. **`app.component.ts` et `.html`**
   ```typescript
   @Component({ /* ... */ })
   export class AppComponent {
     // On rend le service public pour pouvoir l'utiliser dans le template
     constructor(public loadingService: LoadingService) {}
   }
   ```
   ```html
   <!-- Quelque part dans app.component.html, à un niveau élevé -->
   @if(loadingService.isLoading()) {
     <div class="loading-spinner-overlay">
       <p>Chargement...</p>
     </div>
   }
   <router-outlet></router-outlet>
   ```

### Auto-évaluation

1. **(Question ouverte)** Quelle est la différence majeure entre `ngOnInit` et `ngOnChanges` ?
2. **(QCM)** Vous voulez optimiser un composant qui affiche simplement des données. Quelle stratégie de détection de
   changement est la plus performante ?
   a) `Default`
   b) `OnPush`
   c) `Optimized`
   d) `Manual`
3. **(QCM)** Quel est le rôle d'un intercepteur HTTP ?
   a) Remplacer complètement `HttpClient`.
   b) Se placer entre les appels HTTP et le backend pour les manipuler globalement.
   c) Intercepter les erreurs JavaScript dans les templates.
   d) Optimiser la taille des images.
4. **(Question ouverte)** Dans un composant `OnPush`, citez deux conditions qui déclencheront une mise à jour de sa vue.
5. **(QCM)** Dans quel dossier est-il recommandé de placer une directive personnalisée qui sera utilisée par de
   nombreux "feature modules" ?
   a) `core`
   b) `shared`
   c) `features`
   d) `common`

### Conclusion de ce chapitre

Félicitations, vous avez maintenant exploré les techniques qui distinguent les applications Angular de qualité
professionnelle. Vous savez comment optimiser les performances avec `OnPush`, comment réagir finement aux changements
avec `ngOnChanges`, et comment intercepter et manipuler globalement les flux de données avec les intercepteurs HTTP.

Ces compétences en architecture et en optimisation sont cruciales pour construire des applications qui sont non
seulement fonctionnelles, mais aussi rapides, maintenables et évolutives. Elles complètent parfaitement votre maîtrise
du framework.