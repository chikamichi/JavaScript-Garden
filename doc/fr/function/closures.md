## Closures et référenceS

Une des fonctionnalités les plus puissantes de JavaScript tient dans la notion
de *closures* (aussi appelée *clôture* ou *fermeture* en français). Une closure
a ceci de particulier que son scope a à tout moment accès à son scope parent.
Mais comme, en JavaScript, la seule manière de créer un nouveau scope est de
créer une nouvelle [fonction](#function.scopes), ce cas particulier devient le
cas général : toutes les fonctions, par défaut, se comportent comme des closures.

### Émuler des variables privées

    function Counter(start) {
        var count = start;
        return {
            increment: function() {
                count++;
            },

            get: function() {
                return count;
            }
        }
    }

    var foo = Counter(4);
    foo.increment();
    foo.get(); // 5

Dans cet exemple, `counter` retourne **deux** closures : la fonction `increment`
et la fonction `get`. Ces deux fonctions ont à leur disposition une **référence**
vers le scope de `Counter`, et par conséquent, peuvent accéder à la variable
`count` qui a été définie dans ce scope « parent. »

### Explication du mécanisme de variables privées

Il est impossible en JavaScript de référencer ou d'assigner des scopes. Cela
explique pourquoi il n'est pas possible d'accèder à la variable `count` depuis
l'extérieur des fonctions/closures. La seule manière d'y accéder est d'être
à l'intérieur d'une de ces deux closures.

    var foo = new Counter(4);
    foo.hack = function() {
        count = 1337;
    };

Le code ci-dessus ne va **pas** modifier la variable `count` qui a initialement
été définie dans le scope de `Counter`, car `foo.hack` n'a pas elle-même été
définie dans ce scope. Une nouvelle variable, globale, nommée `count`, sera
ainsi définie, ou modifiée si elle existait déjà.

### Comportement des closures dans des boucles

Une erreur fréquente consiste à supposer qu'une closure à l'intérieur d'une
boucles va copier la valeur de la variable d'index de la boucle.

    for(var i = 0; i < 10; i++) {
        setTimeout(function() {
            console.log(i);
        }, 1000);
    }

Le code ci-dessus ne va **pas** afficher les chiffres `0` à `9` : on verra en
fait dix fois le chiffre `10`.

La fonction *anonyme* conserve une **référence** vers `i`, et au moment où
`console.log` est exécuté, la boucle `for` est déjà terminée, si bien que la
valeur de `i` est `10`.

Pour arriver à avoir le comportement souhaité, il est nécessaire de créer
soi-même une **copie** de la valeur de `i`.

### Le problème de référence

Pour ce faire (créer une copie explicite de `i`), le mieux consiste à utiliser
un ["wrapper" anonyme](#function.scopes).

    for(var i = 0; i < 10; i++) {
        (function(e) {
            setTimeout(function() {
                console.log(e);
            }, 1000);
        })(i);
    }

La fonction anonyme « extérieure » (le "wrapper") est appelée immédiatement
avec comme argument, `i`, de sorte que le paramètre `e` possède sa valeur
à chaque tour de boucle.

La fonction anonyme passée à `setTimeout` possède elle une référence à `e`,
car `e` est défini (en tant qu'argument) dans le scope parent. Cette valeur
de `e` n'est **pas** modifiée par la boucle.

Une autre façon de faire consisterait à retourner une fonction depuis le
wrapper, fonction ayant le même comportement que le code ci-dessus, et à
utiliser ce wrapper dans `setTimeout`.

    for(var i = 0; i < 10; i++) {
        setTimeout((function(e) {
            return function() {
                console.log(e);
            }
        })(i), 1000)
    }

`setTimeout` a été conçue pour gérer ce cas d'usage en acceptant en fait
des arguments supplémentaires, qui seront passées à la fonction de callback.

    for(var i = 0; i < 10; i++) {
        setTimeout(function(e) {
            console.log(e);
        }, 1000, i);
    }

Sachez que certains environnements d'exécution ne supportent pas cette
fonctionnalité (Internet Explorer 9 et plus anciens.)

Enfin, il est possible de mettre à contribution `.bind` pour arriver au
même résultat, en liant `this` à un context et des arguments spécifiques.

    for(var i = 0; i < 10; i++) {
        setTimeout(console.log.bind(console, i), 1000);
    }
