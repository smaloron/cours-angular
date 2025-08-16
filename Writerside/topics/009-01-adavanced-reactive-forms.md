# Module 9 : Pour aller plus loin - Formulaires Réactifs Avancés

### Objectifs pédagogiques

À la fin de cette partie, vous serez en mesure de :

* **Créer** des validateurs personnalisés synchrones pour des règles métier spécifiques.
* **Implémenter** des validateurs cross-champs pour valider des champs en fonction les uns des autres (ex: confirmation
  de mot de passe).
* **Gérer** des formulaires dynamiques avec `FormArray` pour des listes d'entrées (ex: ajouter plusieurs compétences,
  des numéros de téléphone...).
* **Réagir** aux changements de valeur ou de statut d'un formulaire en utilisant les observables `valueChanges` et
  `statusChanges`.

### Introduction : Du formulaire statique au formulaire intelligent

Vous savez construire un formulaire solide et bien structuré. Mais que se passe-t-il lorsque les exigences deviennent
plus complexes ?

* Comment valider qu'un nom d'utilisateur ne contient pas de caractères spéciaux interdits par votre entreprise ?
* Comment s'assurer que le champ "Confirmation du mot de passe" correspond bien au champ "Mot de passe" ?
* Comment permettre à un utilisateur d'ajouter autant d'adresses ou de numéros de téléphone qu'il le souhaite ?
* Comment déclencher une action (comme un appel API pour vérifier la disponibilité d'un email) dès que l'utilisateur
  tape dans un champ ?

C'est là que la nature "réactive" des formulaires prend tout son sens. Le modèle de formulaire que vous avez créé n'est
pas juste un objet statique ; c'est un système vivant qui émet des flux d'événements (`Observable`) auxquels vous pouvez
réagir. Cette section vous apprendra à maîtriser ces techniques avancées pour construire des formulaires véritablement
intelligents et dynamiques.

### 1. Validateurs Personnalisés

Les validateurs intégrés (`Validators.required`, `email`...) sont utiles, mais souvent, vos règles métier sont plus
spécifiques. Un validateur personnalisé n'est qu'une fonction qui prend un `FormControl` en entrée et retourne soit
`null` (si c'est valide), soit un objet d'erreur (si c'est invalide).

**Scénario :** Créer un validateur qui interdit les espaces dans un nom d'utilisateur.

```typescript
// Fichier: src/app/shared/validators/no-whitespace.validator.ts
import {AbstractControl, ValidationErrors, ValidatorFn} from '@angular/forms';

// C'est une "Factory" qui retourne notre fonction de validation.
export function noWhitespaceValidator(): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
        // On vérifie si la valeur du contrôle contient un espace.
        const hasWhitespace = (control.value || '').includes(' ');

        // Si oui, on retourne un objet d'erreur. La clé 'whitespace'
        // pourra être utilisée dans le template avec .hasError('whitespace').
        return hasWhitespace ? {'whitespace': true} : null;
    };
}
```

**Utilisation dans le composant :**

```typescript
// Fichier: register.component.ts
import {noWhitespaceValidator} from '../shared/validators/no-whitespace.validator';

// ...
this.registerForm = this.fb.group({
    username: ['', [Validators.required, noWhitespaceValidator()]],
    // ...
});
```

**Utilisation dans le template :**

```html
@if (registerForm.get('username')?.hasError('whitespace')) {
<small>Le nom d'utilisateur ne doit pas contenir d'espaces.</small>
}
```

### 2. Validateurs Cross-Champs

Parfois, la validité d'un champ dépend d'un autre (ex: confirmation de mot de passe, date de fin après date de
début...). Ce type de validateur ne s'applique pas à un `FormControl`, mais directement au `FormGroup` parent.

**Scénario :** Vérifier que deux champs de mot de passe correspondent.

```typescript
// Fichier: src/app/shared/validators/password-match.validator.ts
import {AbstractControl, ValidationErrors, ValidatorFn} from '@angular/forms';

export function passwordMatchValidator(): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
        // Ici, 'control' est le FormGroup
        const password = control.get('password');
        const confirmPassword = control.get('confirmPassword');

        // Si les champs n'existent pas ou n'ont pas encore été touchés,
        // on ne valide pas pour éviter des erreurs au démarrage.
        if (!password || !confirmPassword) {
            return null;
        }

        // Si les valeurs sont différentes, on retourne une erreur sur le FormGroup
        return password.value === confirmPassword.value ? null : {'passwordMismatch': true};
    };
}
```

**Utilisation dans le composant :** Le validateur est passé comme deuxième argument du `fb.group`.

```typescript
// Fichier: register.component.ts
import {passwordMatchValidator} from '../shared/validators/password-match.validator';

// ...
this.registerForm = this.fb.group({
    // ... autres champs
    password: ['', Validators.required],
    confirmPassword: ['', Validators.required],
}, {validators: passwordMatchValidator()}); // On l'applique ici !
```

**Utilisation dans le template :** On vérifie l'erreur sur le formulaire lui-même.

```html
@if (registerForm.hasError('passwordMismatch')) {
<div class="form-error">
    Les mots de passe ne correspondent pas.
</div>
}
```

### 3. Formulaires Dynamiques avec `FormArray`

Un `FormArray` est un type de `FormGroup` où les contrôles n'ont pas de clé nommée. C'est simplement un tableau de
`FormControl`s ou de `FormGroup`s. C'est l'outil parfait pour permettre à l'utilisateur d'ajouter ou de supprimer des
champs dynamiquement.

**Scénario :** Un formulaire de profil où l'utilisateur peut ajouter plusieurs "compétences".

```typescript
// Fichier: profile-editor.component.ts
export class ProfileEditorComponent {
    public profileForm = this.fb.group({
        name: [''],
        // On déclare un FormArray pour les compétences
        skills: this.fb.array([])
    });

    // Un "getter" pratique pour accéder facilement au FormArray depuis le template
    get skills(): FormArray {
        return this.profileForm.get('skills') as FormArray;
    }

    // Méthode pour ajouter une nouvelle compétence (un FormControl vide)
    public addSkill(): void {
        this.skills.push(this.fb.control('', Validators.required));
    }

    // Méthode pour supprimer une compétence à un index donné
    public removeSkill(index: number): void {
        this.skills.removeAt(index);
    }
}
```

**Liaison avec le template :**

```html

<form [formGroup]="profileForm">
    <!-- ... champ 'name' ... -->

    <div formArrayName="skills">
        <h3>Compétences</h3>
        <button type="button" (click)="addSkill()">+ Ajouter une compétence</button>

        <!-- On boucle sur les contrôles du FormArray -->
        @for (skill of skills.controls; track skill; let i = $index) {
        <div class="skill-item">
            <label for="skill-{{i}}">Compétence n°{{i + 1}}</label>
            <!-- Chaque input est lié à un contrôle par son index -->
            <input id="skill-{{i}}" type="text" [formControlName]="i">
            <button type="button" (click)="removeSkill(i)">Supprimer</button>
        </div>
        }
    </div>
</form>
```

### 4. Réagir aux Changements avec `valueChanges`

Chaque `FormControl`, `FormGroup` ou `FormArray` expose un `Observable` appelé `valueChanges`. Cet observable émet une
nouvelle valeur à chaque fois que l'utilisateur modifie le contenu du formulaire. C'est incroyablement puissant pour
créer des interactions en temps réel.

**Scénario :** Un champ de recherche qui lance automatiquement une recherche 300ms après que l'utilisateur a cessé de
taper.

```typescript
import {debounceTime, distinctUntilChanged, switchMap, tap} from 'rxjs/operators';

export class SearchComponent implements OnInit {
    public searchControl = new FormControl();
    public searchResults$: Observable<any[]>; // Un observable pour les résultats

    constructor(private searchService: SearchService) {
    }

    ngOnInit(): void {
        this.searchResults$ = this.searchControl.valueChanges.pipe(
            // 1. Attendre 300ms après la dernière frappe
            debounceTime(300),
            // 2. Ne pas lancer la recherche si le texte n'a pas changé
            distinctUntilChanged(),
            // 3. Afficher un indicateur de chargement
            tap(() => this.isLoading = true),
            // 4. Annuler la recherche précédente et lancer la nouvelle
            switchMap(searchTerm => this.searchService.search(searchTerm)),
            // 5. Cacher l'indicateur de chargement
            tap(() => this.isLoading = false)
        );
    }
}
```

Dans le template, on peut ensuite utiliser le pipe `async` pour afficher les résultats :
`@for(result of searchResults$ | async; ...)`

### Exercice pratique

#### Exercice 2 : Formulaire de Commande avec Items Dynamiques

Créez un formulaire de commande.

1. Le formulaire (`FormGroup`) doit avoir un champ `customerName` (`FormControl`).
2. Il doit aussi avoir un `FormArray` nommé `items` pour les articles de la commande.
3. Chaque élément du `FormArray` doit être un `FormGroup` avec deux `FormControl`s : `itemName` (string, requis) et
   `quantity` (number, requis et min(1)).
4. Implémentez les méthodes `addItem()` et `removeItem(index)` dans votre composant.
5. Dans le template, permettez à l'utilisateur d'ajouter et de supprimer des lignes d'articles.
6. **Bonus :** Créez un validateur cross-champs qui s'assure que le nom du client n'est pas "Test" si la commande
   contient un article nommé "Produit Interdit".

#### Correction exercice 2 {collapsible='true'}

**`order-form.component.ts`**

```typescript
import {Component} from '@angular/core';
import {FormBuilder, FormGroup, FormArray, Validators} from '@angular/forms';

@Component({ /* ... */})
export class OrderFormComponent {
    public orderForm: FormGroup;

    constructor(private fb: FormBuilder) {
        this.orderForm = this.fb.group({
            customerName: ['', Validators.required],
            items: this.fb.array([])
        });
    }

    get items(): FormArray {
        return this.orderForm.get('items') as FormArray;
    }

    // Crée un nouveau FormGroup pour un item
    private createItem(): FormGroup {
        return this.fb.group({
            itemName: ['', Validators.required],
            quantity: [1, [Validators.required, Validators.min(1)]]
        });
    }

    public addItem(): void {
        this.items.push(this.createItem());
    }

    public removeItem(index: number): void {
        this.items.removeAt(index);
    }

    public onSubmit(): void {
        console.log(this.orderForm.value);
    }
}
```

**`order-form.component.html`**

```html

<form [formGroup]="orderForm" (ngSubmit)="onSubmit()">
    <label for="customerName">Nom du Client</label>
    <input id="customerName" formControlName="customerName">

    <div formArrayName="items">
        <h3>Articles</h3>
        <button type="button" (click)="addItem()">+ Ajouter Article</button>

        @for (item of items.controls; track item; let i = $index) {
        <!-- [formGroupName] lie le div au FormGroup à l'index i -->
        <div [formGroupName]="i" class="item-row">
            <input type="text" formControlName="itemName" placeholder="Nom de l'article">
            <input type="number" formControlName="quantity" placeholder="Quantité">
            <button type="button" (click)="removeItem(i)">X</button>
        </div>
        }
    </div>
    <button type="submit" [disabled]="orderForm.invalid">Passer Commande</button>
</form>
```

### Auto-évaluation

---

**1. (Question ouverte)**  
Quelle est la différence entre un validateur appliqué à un `FormControl` et un validateur appliqué à un `FormGroup` ?  
Donnez un exemple pour chacun.

---

**2. (QCM)**  
Quelle classe utilisez-vous pour gérer une liste de champs de formulaire où l'utilisateur peut ajouter ou supprimer des entrées ?

- a) `FormList`
- b) `FormControlList`
- c) `FormArray`
- d) `FormCollection`

---

**3. (QCM)**  
Pour réagir à chaque frappe de l'utilisateur dans un champ de formulaire, quel `Observable` utiliseriez-vous ?

- a) `statusChanges`
- b) `onkeyup`
- c) `valueChanges`
- d) `formEvents`

---

**4. (Question ouverte)**  
Décrivez un scénario où vous utiliseriez l'observable `valueChanges` d'un `FormGroup` entier plutôt que celui d'un seul `FormControl`.

---

**5. (QCM)**  
Un validateur personnalisé synchrone doit retourner `null` dans quel cas ?

- a) Quand le champ est invalide.
- b) Quand le champ est valide.
- c) Il ne doit jamais retourner `null`.
- d) Uniquement quand le champ est vide.

---


### Conclusion de ce chapitre

Félicitations ! Vous avez atteint le plus haut niveau de maîtrise des formulaires réactifs. Vous ne vous contentez plus
de créer des formulaires ; vous construisez des systèmes de saisie de données intelligents, dynamiques et robustes.

Vous savez désormais étendre le système de validation avec vos propres **règles métier**, gérer des **relations
complexes** entre les champs, et construire des **interfaces dynamiques** avec `FormArray`. Plus important encore, vous
avez entrevu la puissance des formulaires en tant que **flux de données** avec `valueChanges`, ouvrant la porte à des
fonctionnalités en temps réel sophistiquées.

Ces compétences sont directement applicables à la majorité des défis que vous rencontrerez dans des applications
d'entreprise. Vous êtes maintenant parfaitement armé pour aborder la suite de notre programme, en commençant par les
opérateurs RxJS avancés.