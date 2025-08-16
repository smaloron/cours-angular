# Module 7 : Pour aller plus loin - Routage Avancé, Sécurisé et Performant

### Objectifs pédagogiques

À la fin de cette partie, vous serez en mesure de :

* **Implémenter** des routes enfants (nested routes) pour créer des mises en page complexes (ex: tableau de bord avec
  des sous-sections).
* **Protéger** des routes avec des "gardes de navigation" (`CanActivate`) pour contrôler l'accès en fonction de l'
  authentification.
* **Charger** des sections entières de votre application à la demande ("lazy loading") pour améliorer drastiquement le
  temps de chargement initial.
* **Passer** des données statiques à une route via la propriété `data`.

### Introduction : De la carte routière au système de contrôle aérien

Vous avez dessiné la carte routière de votre ville-application. Les rues principales sont là, les adresses fonctionnent.
Mais que se passe-t-il lorsque votre ville devient une métropole ? Il vous faut des autoroutes à plusieurs niveaux, des
péages pour contrôler qui entre dans certains quartiers, et un système logistique qui n'apporte les marchandises que là
où elles sont nécessaires, au moment où elles le sont.

C'est précisément ce que nous allons construire. Le routage avancé n'est pas qu'une question de navigation ; c'est un
pilier de l'**architecture**, de la **sécurité** et de la **performance** de votre application. Vous apprendrez à créer
des interfaces complexes, à sécuriser des zones réservées aux administrateurs, et surtout, à rendre votre application
incroyablement rapide au premier chargement grâce au "lazy loading".

### 1. Routes Enfants (Nested Routes)

Souvent, on a une mise en page qui ne change pas, mais une partie de cette mise en page doit être dynamique. Un tableau
de bord est l'exemple parfait : le menu latéral reste, mais le contenu principal change (profil, statistiques,
paramètres...).

On peut imbriquer des routes les unes dans les autres en utilisant la propriété `children`. Le composant parent aura son
propre `<router-outlet>` pour afficher les composants enfants.

<procedure title="Mise en place de routes enfants pour un tableau de bord" id="nested-routes">
    <step>
        <b>Générer les composants :</b>
        <p>Un composant "layout" pour le tableau de bord, et deux composants pour les "pages" enfants.</p>
        <code>
        ng g c pages/dashboard/dashboard-layout
        ng g c pages/dashboard/profile
        ng g c pages/dashboard/stats
        </code>
    </step>
    <step>
        <b>Configurer les routes :</b>
        <p>Dans <code>app.routes.ts</code>, on définit une route parent avec une propriété <code>children</code>.</p>
        
```typescript
// Fichier: src/app/app.routes.ts
import { DashboardLayoutComponent } from './pages/dashboard/dashboard-layout/dashboard-layout.component';
import { ProfileComponent } from './pages/dashboard/profile/profile.component';
import { StatsComponent } from './pages/dashboard/stats/stats.component';

export const routes: Routes = [
  // ... autres routes
  { 
    path: 'dashboard', 
    component: DashboardLayoutComponent,
    children: [
      { path: '', redirectTo: 'profile', pathMatch: 'full' }, // Par défaut, on va sur le profil
      { path: 'profile', component: ProfileComponent },
      { path: 'stats', component: StatsComponent }
    ]
  },
  // ...
];
```
    
</step>
    <step>
        <b>Mettre à jour le template du parent :</b>
        <p>Le <code>DashboardLayoutComponent</code> doit avoir son propre <code>&lt;router-outlet&gt;</code> pour afficher les enfants.</p>
        
```html
<!-- Fichier: src/app/pages/dashboard/dashboard-layout/dashboard-layout.component.html -->
<h1>Tableau de Bord</h1>
<nav>
  <!-- Les liens pointent vers les routes enfants (relatives au parent) -->
  <a [routerLink]="['profile']">Mon Profil</a> | 
  <a [routerLink]="['stats']">Statistiques</a>
</nav>
<hr>
<div class="dashboard-content">
  <!-- C'est ici que ProfileComponent ou StatsComponent s'afficheront -->
  <router-outlet></router-outlet>
</div>
```
</step>

</procedure>

### 2. Gardes de Navigation (`CanActivate`) : Le Contrôle d'Accès

Une garde est une fonction qui s'exécute avant que le Router n'active une route. Elle doit retourner `true` (la
navigation est autorisée), `false` (la navigation est bloquée), ou une `UrlTree` pour rediriger l'utilisateur.

La garde `CanActivate` est la plus courante : elle détermine si un utilisateur *peut activer* (c'est-à-dire, accéder à)
une route. C'est parfait pour la gestion de l'authentification.

**Scénario :** Notre route `/dashboard` ne doit être accessible que si l'utilisateur est connecté.

```typescript
// Fichier: src/app/shared/guards/auth.guard.ts
import {inject} from '@angular/core';
import {CanActivateFn, Router} from '@angular/router';
import {AuthService} from '../services/auth.service'; // Un service qui gère l'état de connexion

export const authGuard: CanActivateFn = (route, state) => {
    const authService = inject(AuthService);
    const router = inject(Router);

    if (authService.isLoggedIn()) {
        return true; // L'utilisateur est connecté, on autorise la navigation
    } else {
        // L'utilisateur n'est pas connecté, on le redirige vers la page de login
        console.warn("Accès refusé ! Redirection vers /login.");
        return router.createUrlTree(['/login']);
    }
};
```

On applique ensuite cette garde à notre route dans `app.routes.ts` :

```typescript
// Fichier: src/app/app.routes.ts
import {authGuard} from './shared/guards/auth.guard';
// ...

export const routes: Routes = [
    // ...
    {
        path: 'dashboard',
        component: DashboardLayoutComponent,
        canActivate: [authGuard], // On applique la garde ici !
        children: [ /* ... */]
    },
    // ...
];
```

### 3. Chargement Paresseux (Lazy Loading) : La Performance avant tout

Par défaut, quand un utilisateur charge votre application, Angular télécharge le code de **tous** les composants, même
ceux des pages qu'il ne visitera jamais. Pour une petite application, c'est acceptable. Pour une grosse, c'est un
désastre pour le temps de chargement initial.

Le **Lazy Loading** résout ce problème. Il permet de scinder votre application en "modules" qui ne seront téléchargés
par le navigateur que lorsque l'utilisateur essaiera d'accéder à une route qui en fait partie.

La syntaxe est très élégante :

```typescript
// Fichier: src/app/app.routes.ts
export const routes: Routes = [
    // ...
    {
        path: 'admin',
        // Au lieu de 'component', on utilise 'loadChildren'.
        // Angular ne chargera le code des routes d'admin que lorsque
        // l'utilisateur naviguera vers une URL commençant par /admin.
        loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
    },
    // ...
];
```

Le fichier `admin.routes.ts` contiendrait alors toutes les routes relatives à la section admin (`/dashboard`, `/users`,
etc.), comme un mini `app.routes.ts` autonome.

<tip>
**Quand utiliser le Lazy Loading ?** Toujours pour les sections majeures de votre application qui ne sont pas nécessaires au premier affichage : le back-office admin, le tableau de bord utilisateur, les fonctionnalités complexes... C'est une des optimisations les plus impactantes que vous puissiez faire.
</tip>

### Exercice pratique

#### Exercice 2 : Sécuriser un Back-Office avec une Garde et le Lazy Loading

Vous devez créer une section "admin" pour votre application.

1. Créez un `AuthService` très simple avec une propriété `loggedIn = false` et des méthodes `login()` et `logout()` pour
   changer sa valeur, et `isLoggedIn()` pour la retourner. Fournissez-le à la racine.
2. Créez une garde de navigation `authGuard` qui n'autorise l'accès que si `authService.isLoggedIn()` est `true`, et
   redirige vers `/` sinon.
3. Créez un dossier `admin` avec ses propres routes (`admin.routes.ts`) et quelques composants (ex:
   `AdminDashboardComponent`, `ManageUsersComponent`). Les routes de ce fichier seront `/` (pour le dashboard) et
   `/users` (pour la gestion).
4. Dans `app.routes.ts`, configurez la route `/admin` pour qu'elle utilise le **lazy loading** afin de charger
   `admin.routes.ts`.
5. Appliquez votre `authGuard` sur cette route `/admin`.
6. Dans votre `AppComponent`, ajoutez des boutons "Login" et "Logout" pour simuler la connexion et un lien vers
   `/admin`. Vérifiez que l'accès est bien bloqué/autorisé en fonction de l'état de connexion.

#### Correction exercice 2 {collapsible='true'}

1. **`auth.service.ts`**
   ```typescript
   import { Injectable } from '@angular/core';

   @Injectable({ providedIn: 'root' })
   export class AuthService {
     private loggedIn = false;
     login() { this.loggedIn = true; }
     logout() { this.loggedIn = false; }
     isLoggedIn() { return this.loggedIn; }
   }
   ```

2. **`auth.guard.ts`** (identique à l'exemple du cours)

3. **Dossier `admin`**
    * `ng g c admin/admin-dashboard` & `ng g c admin/manage-users`
    * Créez `src/app/admin/admin.routes.ts`:
      ```typescript
      import { Routes } from '@angular/router';
      import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';
      import { ManageUsersComponent } from './manage-users/manage-users.component';

      export const ADMIN_ROUTES: Routes = [
        { path: '', component: AdminDashboardComponent },
        { path: 'users', component: ManageUsersComponent }
      ];
      ```

4. & 5. **`app.routes.ts` mis à jour**
   ```typescript
   import { Routes } from '@angular/router';
   import { authGuard } from './shared/guards/auth.guard';

   export const routes: Routes = [
     // ... autres routes
     {
       path: 'admin',
       canActivate: [authGuard],
       loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
     }
   ];
   ```

6. **`app.component.ts` et `.html`**
   ```typescript
   // app.component.ts
   @Component({ /* ... */ })
   export class AppComponent {
     constructor(public authService: AuthService) {}
   }
   ```
   ```html
   <!-- app.component.html -->
   <nav>
     <a [routerLink]="['/admin']">Section Admin</a>
   </nav>
   <button (click)="authService.login()">Login</button>
   <button (click)="authService.logout()">Logout</button>
   <p>Connecté: {{ authService.isLoggedIn() }}</p>
   <router-outlet></router-outlet>
   ```
   Maintenant, si vous cliquez sur le lien "Section Admin" sans être "loggé", vous serez bloqué. Si vous cliquez sur "
   Login" puis sur le lien, vous accéderez au dashboard admin.

### Auto-évaluation

1. **(Question ouverte)** Quel est le principal avantage du "lazy loading" et dans quel type de scénario est-il le plus
   bénéfique ?
2. **(QCM)** Vous avez une route `/settings` avec deux sous-sections, `/settings/profile` et `/settings/account`.
   Comment le composant de la route `/settings` affiche-t-il les composants des sous-sections ?
   a) En utilisant des `@Input()` et `@Output()`.
   b) En utilisant `*ngIf`.
   c) En plaçant un `<router-outlet>` dans son template.
   d) Les routes enfants ne peuvent pas avoir de composant parent.
3. **(QCM)** Quel type de garde de navigation utiliseriez-vous pour empêcher un utilisateur de quitter une page de
   formulaire s'il a des modifications non enregistrées ?
   a) `CanActivate`
   b) `CanDeactivate`
   c) `Resolve`
   d) `CanLoad`
4. **(Question ouverte)** Expliquez le rôle d'une garde `CanActivate`. Que doit-elle retourner pour autoriser, bloquer
   ou rediriger une navigation ?
5. **(QCM)** Quelle propriété de l'objet `Route` est utilisée pour implémenter le lazy loading ?
   a) `component`
   b) `children`
   c) `loadChildren`
   d) `lazyComponent`

### Conclusion de ce chapitre

Félicitations ! Vous avez transcendé le simple rôle de dessinateur de cartes pour devenir un véritable architecte de la
navigation. Vous savez désormais construire des **mises en page complexes** avec les routes enfants, **sécuriser les
accès** de votre application avec les gardes, et, surtout, la rendre **extrêmement performante** au chargement grâce au
lazy loading.

Ces compétences sont absolument fondamentales pour la création d'applications Angular d'envergure professionnelle. Elles
garantissent une expérience utilisateur fluide, une sécurité robuste et une structure de projet saine.

Nous avons maintenant une application structurée, navigable et sécurisée. Il est temps de la connecter à votre backend
Java ! Dans le prochain module, nous allons aborder le `HttpClient` et la programmation réactive avec **RxJS** pour
effectuer des appels HTTP et gérer des données asynchrones.