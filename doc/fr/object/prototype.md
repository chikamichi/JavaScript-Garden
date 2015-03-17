## La notion de prototype

JavaScript ne possède pas un héritage classique, dans tous les sens du terme
(il n'y a pas de notion de classe). À la place, il utilise la notion de
prototype, et possède un héritage prototypal.

Bien que cela soit souvent considéré comme l'une des faiblesses principales du
language, ce modèle d'héritage par prototypes est en réalité plus puissant que
le modèle « classique. » Il est d'ailleurs relativement facile de construire
un modèle basé sur des classes en utilisant les prototypes comme base, alors
que l'inverse est notablement plus compliqué.

JavaScript est le seul language ubiquitaire proposant un tel modèle prototypal,
c'est pourquoi les programmeurs ont souvent besoin d'un temps d'adaptation
avant de bien saisir quels sont les différences par rapport à un modèle
classique.

La première différence majeure tient au fait que l'héritage en JavaScript est
est modélisé par une *chaîne de prototypes*.

> **Note :** l'utilisation d'un code aussi simple que `Bar.prototype = Foo.prototype`
> aura comme résultat que les deux objets partageront *le même* prototype, ce
> qui a comme conséquence immédiate qu'une modification du prototype de l'un
> entraînera la modification du prototype de l'autre. Dans bien des cas, ce
> n'est pas l'effet recherché.

    function Foo() {
        this.value = 42;
    }
    Foo.prototype = {
        method: function() {}
    };

    function Bar() {}

    // Le prototype de Bar sera une instance de Foo.
    Bar.prototype = new Foo();
    Bar.prototype.foo = 'Hello World';

    // Attention, il est important de bien assigner Bar comme étant le
    // constructeur de son propre prototype.
    Bar.prototype.constructor = Bar;

    var test = new Bar(); // création d'une instance de Bar

    // La chaîne de prototypes qui en résulte.
    test [instance of Bar]
        Bar.prototype [instance of Foo]
            { foo: 'Hello World' }
            Foo.prototype
                { method: ... }
                Object.prototype
                    { toString: ... /* etc. */ }

Dans le code ci-dessus, l'objet `test` va hériter à la fois de `Bar.prototype`
et de `Foo.prototype`, de sorte qu'il aura accès à la fonction `method` qui a
été définie sur `Foo.prototype`, et à la propriété `value` qui se trouve elle
être liée à *l'instance particulière de `Foo`* qui se trouve être son prototype.
Il est important de noter que `new Bar()` ne crée **pas** une nouvelle instance
de `Foo`, mais réutilise celle qui a été assignée à son prototype ; ainsi,
toutes les instances de `Bar` possèderont la même propriété `value`.

> **Note :** N'utilisez **pas** `Bar.prototype = Foo`, qui ferait pointer vers
> l'objet `Foo` (une fonction), et non vers le prototype de `Foo`. Si vous
> faisiez cela, la chaîne prototype lierait `Function.prototype` et non
> `Foo.prototype`, ce qui masquerait complètement `method`.

### Lecture d'une propriété

Lorsqu'on essaye d'accèder à une propriété sur un objet, JavaScript se chargera
de remonter toute la chaîne d'héritage prototypale **vers le haut**, jusqu'à
trouver une propriété qui correspond.

S'il atteint le haut de la chaîne — `Object.prototype` — sans rien trouver, la
valeur [undefined](#core.undefined) sera retournée.

### La propriété prototype

Bien que la propriété prototype soit normalement utilisée par JavaScript pour
construire une chaîne d'héritage, il reste tout à fait possible de lui
assigner n'importe quelle valeur. Toutefois, les primitives seront ignorées
dans ce cas particulier.

    function Foo() {}
    Foo.prototype = 1; // sans effet, pas d'assignation

Assigner des objets, comme l'a montré l'exemple ci-avant, fonctionne
parfaitement, ce qui permet d'ailleurs d'envisager une création dynamique
de chaînes prototypales.

### Performance

Le temps d'accès aux propriétés qui sont en fait haut placées dans la chaîne
prototypale peut avoir un impact négatif sur les performances générales, un
détail à ne pas négliger pour certaines applications critiques. Par ailleurs,
il faut se souvenir qu'essayer de lire une propriété non-existante engendre
nécessairement une traversée complète de la chaîne d'héritage.

Enfin, dans le cas d'une [itérations](#object.forinloop) sur les propriétés
d'un objet, *toutes* les propriétés qui sont définies sur le prototype
seront prises en compte dans la boucle.

### Extension des prototypes natifs

Il s'agit-là d'une mauvaise pratique pourtant souvant utilisée, du fait qu'il
est possible d'étendre `Object.prototype` ou tout autre prototype pré-fournit
par JavaScript.

Cette technique, appelée [monkey patching][1], met à mal l'*encapsulation* du
language. Bien qu'utilisée par des frameworks assez populaires tels que
[Prototype][2], il n'y a pas véritablement de bonne raison d'ajouter des
fonctionnalités non-standard aux types pré-définis.

La **seule** raison valable qui pourrait amener à faire ça consisterait à
ajouter de la rétro-compatibilité ascendante lors de la sortie d'une
nouvelle version d'un moteur JavaScript (proposant de nouvelles fonctionnalités),
par exemple pour supporter [`Array.forEach`][3] dans les anciennes versions.

### En guise de conclusion

Il est **essentiel** de comprendre le modèle d'héritage prototypal de
JavaScript avant de se lancer dans l'écriture de code complexe qui en ferait
l'usage. Garder à l'esprit la taille potentielle de la chaîne d'héritage de
vos objets, et si possible, scinder les plus longues pour éviter tout problème
de performance. Enfin, ne modifiez **jamais** les prototypes natifs, à moins
de vouloir ajouter une rétro-compatibilité ascendante.

[1]: http://en.wikipedia.org/wiki/Monkey_patch
[2]: http://prototypejs.org/
[3]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/forEach

