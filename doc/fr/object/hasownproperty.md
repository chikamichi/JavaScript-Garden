## `hasOwnProperty`

Pour vérifier si une propriété d'un objet est définie sur l'objet *lui-même*,
et non plus haut dans sa [chaîne d'héritage](#object.prototype), il faut avoir
recours à la méthode `hasOwnProperty`, fournit par `Object.prototype`.

> **Note :** Vérifier qu'une propriété n'est pas `undefined` n'est **pas**
> suffisant, car il est possible que la propriété existe sur l'objet mais
> que sa *valeur* soit `undefined`.

`hasOwnProperty` est le seul outil fournit par JavaScript qui permet de gérer
les propriétés **sans** remonter la chaîne prototypale.

    // À ne pas faire: modification de Object.prototype
    Object.prototype.bar = 1;
    var foo = {goo: undefined};

    foo.bar; // 1
    'bar' in foo; // true

    foo.hasOwnProperty('bar'); // false
    foo.hasOwnProperty('goo'); // true

On voit ici que seule `hasOwnProperty` donne le bon résultat. Voyiez également
la section sur [les boucles `for in`](#object.forinloop) pour plus de détails
quant à l'utilisation de `hasOwnProperty` dans une itération sur les propriétés
d'un objet.

### `hasOwnProperty` en tant que propriété

JavaScript ne « protège » pas spécialement le nom de propriété `hasOwnProperty`,
ce qui fait qu'un objet donné pourrait très bien redéfinir une propriété avec
ce même nom. Il est donc préférable d'utiliser un `hasOwnProperty` « externe »
à l'objet pour s'assurer du bon résultat.

    var foo = {
        hasOwnProperty: function() {
            return false;
        },
        bar: 'Mystification !'
    };

    foo.hasOwnProperty('bar'); // retournera toujours false

    // Utilisation du hasOwnProperty d'un nouvel objet tout neuf, en l'appelant
    // avec le contexte foo (=> this).
    ({}).hasOwnProperty.call(foo, 'bar'); // true

    // Il est également possible d'utiliser le hasOwnProperty du prototype
    // de Object.
    Object.prototype.hasOwnProperty.call(foo, 'bar'); // true


### En guise de conclusion

`hasOwnProperty` est la **seule** manière fiable pour vérifier qu'une
propriété existe sur un objet donné. Il est recommandé d'utiliser
`hasOwnProperty` dans les itérations faites sur les propriétés d'un objet
(voir la section sur [`for in`](#object.forinloop)).
