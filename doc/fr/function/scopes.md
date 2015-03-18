## Scopes et espaces de noms

Au cœur de JavaScript se trouve la notion de scope (parfois dénommée « portée »
en français). Bien que JavaScript vous permette d'écrire des blocs de code
entre accolades, cette syntaxe n'a **pas** pour effet de créer un scope. La seule
manière de le faire est de créer une *nouvelle fonction*.

    function test() { // Crée un scope.
        for(var i = 0; i < 10; i++) { // Ne crée pas de scope.
            // count
        }
        console.log(i); // 10
    }

> **Note :** La notation `{...}` peut être utilisé dans une assignation, une
> expression de retour ou bien comme argument de fonction, auquel cas elle
> sera interprétée comme un objet littéral. Partout ailleurs, il s'agira d'un
> bloc de code tout ce qu'il a de plus standard. Ce comportement, et d'autres
> comme par exemple [l'insertion automatique de point-virgules](#core.semicolon),
> peut être source de bugs assez subtils.

JavaScript n'a pas de notions d'espaces de nommage : tout est défini dans un
seul et unique espace de nom, global et partagé par tout le code.

Aussi, à chaque fois qu'une variable est référencée dans le code, JavaScript va
remonter toute la chaîne de scopes jusqu'à trouver cette variable. S'il atteint
le scope global sans rien trouver, une erreur `ReferenceError` est levée.

### La malédiction des variables globales

    // script A
    foo = '42';

    // script B
    var foo = '42'

Les deux scripts ci-dessus n'ont **pas** le même effet. Le script A défini une
variable `foo` dans le scope global, tandis que le script B défini `foo` dans
le scope courant.

Répétons-le : cela n'a **pas** le même effet. L'absence de `var` peut avoir
répercusions très importantes dans un programme. Voyez plutôt :

    // Nous sommes dans le scope global.
    var foo = 42;
    function test() {
        // Ici, dans un scope local.
        foo = 21;
    }
    test();
    foo; // 21

Le fait d'avoir omis `var` dans `test` lors de l'assignation de `foo` a pour
effet de modifier la valeur de la variable globale `foo`. Ce tout petit détail
peut avoir des effets désastreux dans un programme, tout en étant très
difficile à débugguer.

    // Scope global.
    var items = [/* une liste */];
    for(var i = 0; i < 10; i++) {
        subLoop();
    }

    function subLoop() {
        // scope de subLoop
        for(i = 0; i < 10; i++) { // Oubli de var!
            // Héhé, faisons ici des choses incroyables…
        }
    }

Dans cet exemple, la boucle la plus extérieure va se terminer dès le premier
appel à `subLoop`, car `subLoop` redéfinit la valeur de `i`. Pour éviter ce
problème, il faut prendre garde à bien utiliser `var` dans la seconde boucle
`for`. De façon générale, il faut **toujours** utiliser `var`, sauf si on
souhaite explicitement s'en passer pour une bonne raison.

### Variables locales

La seule façon de créer des variables purement locales en JavaScript est
d'utiliser soit des paramètres de [fonction](#function.general), soit le
mot-clé `var`.

    // Scope global.
    var foo = 1;
    var bar = 2;
    var i = 2;

    function test(i) {
        // Scope local de la fonction test.
        i = 5;

        var foo = 3;
        bar = 4;
    }
    test(10);

Les variables `foo` et `i` sont locales lorsqu'on se trouve dans le scope de la
fonction `test`, mais l'assignation de `bar` dans ce même context va modifier
la variable globale du même nom.

### Hoisting

JavaScript possède un mécanisme dit de **hoisting** pour les déclarations. En
résumé, toute expression utilisant `var` ou toute déclaration de `function`
sera en fait déplacée au début du scope auquelle elle appartient.

    bar();
    var bar = function() {};
    var someValue = 42;

    test();
    function test(data) {
        if (false) {
            goo = 1;

        } else {
            var goo = 2;
        }
        for(var i = 0; i < 100; i++) {
            var e = data[i];
        }
    }

Le code ci-dessus va être transformé avant d'être exécuté. JavaScript va
déplacer les expressions avec `var` ainsi que les déclaration avec `function`
pour les amener en faut du scope courant.

    // Les expressions utilisant var se retrouvent ici.
    var bar, someValue; // Valeur par défault de 'undefined'.

    // La déclaration de fonction a été déplacée ici.
    function test(data) {
        var goo, i, e; // De même pour ces variables. La notation {...} des
                       // tests logiques ci-dessous ne crée pas de scope.
        if (false) {
            goo = 1;
        } else {
            goo = 2;
        }
        for(i = 0; i < 100; i++) {
            e = data[i];
        }
    }

    bar(); // Lève une erreur TypeErro, car bar est toujours 'undefined'.
    someValue = 42; // Les assignations ne sont pas concernées par le hoisting.
    bar = function() {};

    test();

On peut remarquer que les variables utilisées dans le bloc `if`/`else` ont vu
leurs expressions `var` déplacées en haut du scope courant, qui se trouve être
la fonction `test`. Ce comportement peut avoir des répercusions relativement
non-intuitives sur le fonctionnement de certains programmes.

Par exemple, dans le code ci-dessus, le bloc `if` tel qu'initialement écrit
pourrait donner l'impression que la variable *globale* `goo` allait être
modifiée, mais au final, c'est la variable *locale* du même nom qui le sera, du
fait du mécanisme de hoisting.

Aussi, quelqu'un ne connaissant pas l'existance du hoisting pourait penser que le
code ci-dessous va provoquer une `ReferenceError`.

    // Vérifions si SomeImportantThing a été initialisée.
    if (!SomeImportantThing) {
        var SomeImportantThing = {};
    }

Mais on comprend bien désormais qu'il n'en sera rien, du fait de `var` qui va
être repris par le hoisting et déplacé avant le `if`, au début du scope global.

    var SomeImportantThing;

    // Du code intermédiaire pourrait tout à fait utiliser/modifier
    // SomeImportantThing…

    // Vérifions si SomeImportantThing a été initialisée.
    if (!SomeImportantThing) {
        SomeImportantThing = {};
    }

### Ordre utilisé lors de la résolution de nom

Tous les scopes connu de JavaScript, y compris le scope *global*, ont connaissance
du mot-clé [`this`](#function.this), qui est un élément du language défini dans
un scope donné comme étant une référence vers *l'objet courant*.

Par ailleurs, les scopes générés plus spécifiquement par les fonctions ont
connaissance de [`arguments`](#function.arguments), un mot-clé défini comme une
référence vers les arguments passés à une fonction donnée.

Prenons un exemple. Lorsqu'une variable nommée `foo` doit être utilisée, JavaScript
va procéder aux étapes suivantes dans l'ordre :

 1. Vérification de l'existence d'une expression `var foo` dans le scope courant.
 2. Vérification de l'existence d'un paramètre `foo`.
 3. Vérification du nom de la fonction elle-même (qui pourrait être `foo`).
 4. Passage au scope parent, et retour à l'étape **#1**.

> **Note :** La présence d'un paramètre nommé `arguments` va **empêcher** la création
> de l'objet `arguments` normalement disponible comme référence vers la liste des
> arguments.

### Espaces de noms

Le fait qu'il n'y ai qu'un seul espace de noms en JavaScript peut nous faire
nous poser la question  d'un risque de conflit de nommage. Ce problème peut
en fait être facilement géré avec l'utilisation de *"wrappers", c'est-à-dire
de fonctions encapsulantes et anonymes.

    (function() {
        // Création d'un « espace de nommage » indépendant.

        window.foo = function() {
            // Je-suis-STOP-une-closure-publique-STOP.
        };

    })(); // Exécution immédiate de la fonction anonyme !

De telles fonctions anonymes sont traitées comme des [expressions](#function.general),
de sorte que pour pouvoir les appeler, il faut d'abord les évaluer.

    ( // Évaluation de la fonction entre parenthèses…
    function() {}
    ) // puis mise à disposition de cette fonction comme valeur de retour.
    () // Appel de la-dite fonction.

Il y a plusieurs alternatives (d'un point de vue syntaxique) amenant au même résultat.

    // Voici plusieurs manières de définir-exécuter une…
    !function(){}()
    +function(){}()
    (function(){}());
    // etc.

### En guise de conclusion

On recommandera d'utiliser de façon systématique des wrappers anonymes pour
encapsuler son code dans de petits espaces de noms indépendants, de façon à se
prémunir le plus possible du risque de conflit de nommage tout en améliorant la
modularité du programme dans sa globalité.

Par ailleurs, l'utilisation de variables globales est une **mauvaise pratique**.
Si vous avez affaire à une variable globale, **dans tous les cas** c'est un signe
clair que le code a été mal architecturé et sera difficile à maintenir.

