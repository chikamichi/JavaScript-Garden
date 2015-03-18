## L'objet `arguments`

Tout scope lié à une fonction en JavaScript peut utiliser une variable un peu
spéciale, nommée `arguments`. Cette variable contient une liste de tous les
arguments qui ont été passés à la fonction.

> **Note :** Si `arguments` est défini par le programmeur dans le scope de la
> fonction, que ce soit à travers une instruction `var` ou comme un des
> arguments de la fonction, la variable spéciale `arguments` ne sera pas
> définie/modifiée.

L'objet `arguments` n'est **pas** un `Array`. Il possède certaines qualités
qu'on attendrait d'un tableau, et sémantiquement parlant, est assez proche d'un
tableau — par exemple, il possède une propriété `length` — mais il n'hérite pas
de `Array.prototype` et reste bel et bien un `Object`.

De ce fait, il n'est **pas** possible d'utiliser directement des méthodes
standards de `Array`, telles que `push`, `pop` ou `slice` : il faut d'abord
transformer `arguments` en un `Array`. Par contre, il est possible d'utiliser
`arguments` dans une boucle `for`.

### Conversion en tableau (`Array`)

Le code ci-dessous va créer un `Array` contenant les « éléments » de l'objet
`arguments`.

    Array.prototype.slice.call(arguments);

Cette conversion en tableau est un processus **lent** : à garder en mémoire
pour les applications où les performances sont critiques.

### Passer des arguments entre fonctions

Le code ci-dessous illustre la façon recommandée de passer des arguments d'une
fonction à une autre.

    function foo() {
        bar.apply(null, arguments);
    }
    function bar(a, b, c) {
        // Utilisons a, b et c…
    }

Une autre astuce consiste à utiliser conjointement `call` et `apply` pour
transformer une méthode — c'est-à-dire une fonction qui utilise une valeur
spécifique de `this`, ainsi que d'éventuels arguments — en une fonction « standard »
ne faisant usage que d'éventuels arguments (plus de `this` explicite).

    function Person(first, last) {
      this.first = first;
      this.last = last;
    }

    Person.prototype.fullname = function(joiner, options) {
      options = options || { order: "western" };
      var first = options.order === "western" ? this.first : this.last;
      var last =  options.order === "western" ? this.last  : this.first;
      return first + (joiner || " ") + last;
    };

    // Créons une version de "fullname" non-liée à un contexte particulier, qui
    // sera utilisable avec tout objet ayant reçu des propriétés "first" et "last"
    // comme premiers arguments. Ce "wrapper" n'aura pas besoin d'être modifié si
    // "fullname" (ci-avant) l'est, que ce soit en terme de nombre ou d'ordre de
    // ses arguments.
    Person.fullname = function() {
      // Résultat : Person.prototype.fullname.call(this, joiner, ..., argN);
      return Function.call.apply(Person.prototype.fullname, arguments);
    };

    var grace = new Person("Grace", "Hopper");

    // 'Grace Hopper'
    grace.fullname();

    // 'Turing, Alan'
    Person.fullname({ first: "Alan", last: "Turing" }, ", ", { order: "eastern" });


### Paramètres formels et index des arguments

La création de l'objet `arguments` a pour effet de bord la création de fonctions
"getter" et "setter" pour toutes ses propriétés ainsi que pour les paramètres
formels de la fonction à laquelle il est lié.

Résultat net : modifier la valeur d'un paramètre formel va également modifier la
propriété correspondant sur `arguments`, et inversement.

    function foo(a, b, c) {
        arguments[0] = 2;
        a; // 2

        b = 4;
        arguments[1]; // 4

        var d = c;
        d = 9;
        c; // 3
    }
    foo(1, 2, 3);

### Mythes et vérités à propos des performances

Le seul cas où `arguments` n'est pas créé est celui où une variable du même nom
est déjà déclarée dans la fonction ou définie comme paramètre formel (peu
importe si `arguments` est effectivemet utilisé ou pas dans la fonction).

Par contre, les méthodes *getter* et *setter* sont **toujours** créées. Aussi,
il n'y a pas de raison de ne pas utiliser le `arguments` implicitement créé par
JavaScript, les performances ne seront que très marginalement meilleures,
notamment dans la « vraie vie » (code applicatif et librairies) où les fonctions
font toujours bien plus que simplement accéder à `arguments`…

> **Note ES5 :** Les méthodes *getter* et *setter* ne sont pas automatiquement
> créées en mode strict.

Il existe toutefois un cas bien spécifique dans lequel les performances peuvent
être améliorées de façon très importantes, dans les moteurs JS récents : celui
où on utilise `arguments.callee`.

    function foo() {
        arguments.callee; // Utilisons l'objet fonction courant…
        arguments.callee.caller; // ainsi que l'objet fonction appelant.
    }

    function bigLoop() {
        for(var i = 0; i < 100000; i++) {
            foo(); // Normalement, passerait dans l'inliner.
        }
    }

Avec le code ci-dessus, `foo` ne sera plus traité par le mécanisme [*inliner*][1]
car on a besoin d'avoir une référence tant à `foo` elle-même qu'à la fonction où
elle a été appelée. Globalement, utiliser `caller` comme cela est une mauvaise
idée car non seulement les performances vont en pâtir, mais on y perd en
encapsulation car la fonction devient potentiellement dépendante d'un contexte
d'appel particulier.

Pour ces deux raisons, l'utilisation de `arguments.callee` et/ou de ses propriétés
telles que `caller` est **fortement déconseillé**.

> **Note ES5 :** En mode strict, `arguments.callee` est considéré comme déprécié et
> occasionnera une erreur `TypeError`.

[1]: http://en.wikipedia.org/wiki/Inlining

