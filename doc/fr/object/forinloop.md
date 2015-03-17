## La boucle `for in`

De la même manière que l'opérateur `in`, la boucle `for in` traverse
la chaîne prototypale en entier lorsqu'elle est utilisée pour boucler
sur les propriétés d'un objet.

> **Note :** La boucle `for in` va **ignorer** toute propriété ayant
> défini son attribut `enumerable` à `false` ; c'est par exemple le cas
de la propriété `length` pour les tableaux.

    // À ne pas faire : modification Object.prototype
    Object.prototype.bar = 1;

    var foo = {moo: 2};
    for(var i in foo) {
        console.log(i); // affichera à la fois bar et moo
    }

Comme il n'est pas possible de modifier ce comportement par défaut de la
boucle `for in`, il est nécessaire de filtrer les propriétés sur lesquelles
on va itérer dans la boucle. Depuis ECMAScript 3, il suffit d'utiliser
[`hasOwnProperty`](#object.hasownproperty) fournit par `Object.prototype`.

Depuis ECMAScript 5, `Object.defineProperty` peut également être utilisée en
conjonction avec `enumerable` à `false` pour ajouter des propriétés à un
objet (y compris `Object` lui-même), tout en cachant ces propriétés dans les
itérations. Dans le contexte d'ECMAScript 5, pour du code applicatif, il est
raisonnable de supposer qu'on peut se passer de `hasOwnProperty` (dont
l'utilisation est assez verbeuse), car les propriétés auront été définies
"enumerable" ou pas. Pour des librairies par contre, il est plus sage de
continuer à utiliser `hasOwnProperty`.

> **Note :** Du fait que `for in` traverse la chaîne d'héritage en entier,
> les performances se dégradent à mesure que des niveaux d'héritage sont
> ajoutés à un objet.

### Utilisation de `hasOwnProperty` pour filtrer les propriétés

    // Même foo que dans l'exemple ci-dessus.
    for(var i in foo) {
        if (foo.hasOwnProperty(i)) {
            console.log(i);
        }
    }

Cette version est la seule correcte et qui fonctionnera avec les anciennes
versions d'ECMAScript. L'utilisation de `hasOwnProperty` aura pour conséquence
que seule la propriété `moo` sera affichée. En l'absence de `hasOwnProperty`,
ce code est susceptible d'introduire des erreurs dans le cas où des prototypes
natifs, tels que `Object.prototype`, auraient été modifiés.

Dans les dernières versions d'ECMAScript, on peut définir des propriétés comme
non-énumérables avec `Object.defineProperty`, ce qui réduit le risque de prendre
en compte dans une boucle des propriétés non-souhaitées, comme cela aurait été
le cas avec `hasOwnProperty`. Toutefois, il faut quand même prendre ses
précautions avec de vieilles libraires type [Prototype][1], qui n'utilisent pas
encore toutes les nouvelles fonctionnalités d'ECMAScript, comme `defineProperty`.
Avec ce framework par exemple, les boucles `for in` qui n'utiliseraient pas
`hasOwnProperty` sont sûres de produire des bugs.

### En guise de conclusion

L'utilisation systématique de `hasOwnProperty` est recommandée pour les versions
inférieures ou égales à ECMAScript 3, et dans les librairies. Dans ces contextes
particulier, il n'est pas possible de supposer avec un bon niveau de confiance
que les prototypes natifs n'ont pas été modifiés. Depuis ECMAScript 5, il est
possible grâce à `Object.defineProperty` de masquer des propriétés dans les
itérations, ce qui permet de se passer de `hasOwnProperty` dans du code applicatif.

[1]: http://www.prototypejs.org/

