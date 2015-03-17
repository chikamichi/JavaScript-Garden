## Utilisation des objets et de leurs propriétés

En JavaScript, tout se comporte comme un objet, avec seulement deux exceptions,
qui sont [`null`](#core.undefined) et [`undefined`](#core.undefined).

    false.toString(); // 'false'
    [1, 2, 3].toString(); // '1,2,3'

    function Foo(){}
    Foo.bar = 1;
    Foo.bar; // 1

Une erreur fréquente de compréhension concernant la nature du language
consiste à penser que les nombres ne peuvent pas être utilisés comme des
objets. Cela vient en fait d'une erreur dans les parseurs JavaScript qui
tente d'interpréter la *notation point* normalement utilisée avec un objet
comme une « virgule » d'un nombre flottant.

    2.toString(); // lève une erreur du type SyntaxError

Il existe plusieurs façons de contourner ce problème.

    2..toString(); // le deuxième point est correctement parsé
    2 .toString(); // en utilisant un espace avant le point
    (2).toString(); // 2 sera évalué en premier, puis le point

### Les objets comme type de données

En JavaScript, les objets peuvent être utilisés comme des systèmes
[*clé-valeur*][1] ; pour faire simple, on les utilisera fréquement comme des
collections de propriétés nommées, auxquelles correspondent des valeurs.

En utilisant la notation littérale pour les objets, `{}`, il est possible de
créer un objet complet, qui va [hériter](#object.prototype) de `Object.prototype`
et qui n'aura aucune [propriétés propres](#object.hasownproperty).

    var foo = {}; // un nouvel objet, vide

    // un nouvel objet avec une propriété "test", de valeur 12
    var bar = {test: 12};

### Accès aux propriétés

L'accès aux propriétés d'un objet peut se faire de deux manières : avec la
notation point, ou avec des crochets.

    var foo = {name: 'kitten'}
    foo.name; // kitten
    foo['name']; // kitten

    var get = 'name';
    foo[get]; // kitten

    foo.1234; // lève une erreur du type SyntaxError
    foo['1234']; // fonctionne très bien

Les deux notations fonctionnent *grosso modo* de la même manière, à ceci près
que les crochets permettent d'utiliser des noms construits dynamiquement, ce
qui serait impossible avec la notation point (laquelle lèverait une erreur de
syntaxe).

### Suppression de propriétés

La seule manière de supprimer une propriété d'un objet est d'utiliser
l'opérateur `delete`. Assigner une valeur `undefined` ou `null` ne fera que
supprimer la *valeur* de la propriété, et pas sa *clé d'accès* sur l'objet.

    var obj = {
        bar: 1,
        foo: 2,
        baz: 3
    };
    obj.bar = undefined;
    obj.foo = null;
    delete obj.baz;

    for(var i in obj) {
        if (obj.hasOwnProperty(i)) {
            console.log(i, '' + obj[i]);
        }
    }

Le code ci-dessus affiche `bar undefined` et `foo null` — seul `baz` a été
véritablement supprimé et n'est par conséquent pas affiché.

### Notation des clés de propriétés

    var test = {
        'case': 'Je suis un mot réservé, je dois donc être défini comme une chaîne',
        delete: 'Je suis également un mot réservé… je devrais aussi' // SyntaxError
    };

Les propriétés d'un objet peuvent aussi bien être nommées avec des caractères
« pleins », qu'en utilisant une chaîne de caractères (entre guillemets). Du
fait de certaines limitations dans le parseur JavaScript avant la version
ECMAScript 5, la clé `delete` ci-dessus occasionnerai un erreur de syntaxe.

Cette erreur provient du fait que `delete` est un mot-clé réservé du language,
et s'il est utilisé comme clé d'accès pour une propriété, doit être défini avec
une chaîne de caractère pour être correctement interprété par les anciens
moteurs JavaScript.

[1]: http://en.wikipedia.org/wiki/Hashmap

