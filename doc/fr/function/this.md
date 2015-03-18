## Fonctionnement de `this`

JavaScript conceptualise la notion d'auto-référence (`this`) de façon un
peu différente de ce qui existe dans la plupart des autres languages. Il y
très exactement **cinq** façon de modifier la valeur de `this`.

### Le scope global

    this;

Lorsque `this` est utilisé dans le scope global, il va tout simplement
référencer l'objet *global* disponible par défaut.

### Lors de l'appel d'une fonction

    foo();

Dans ce cas de figure, `this` référence également l'objet *global*.

> **ES5 Note :** En mode strict, la notion d'objet global n'existe plus,
> si bien que `this` aurait ici la valeur `undefined`.

### Lors de l'appel d'une méthode

    test.foo();

Dans ce cas de figure, `this` va référencer `test`.

### Lors de l'utilisation d'un constructeur

    new foo();

Un appel de fonction précédé du mot-clé `new` donne à la fonction le rôle de
[constructeur](#function.constructors). Dans ce cas, à l'intérieur de la
fonction, `this` sera une référence vers un `Object` *en cours de création*.

### Assignation explicite de `this`

    function foo(a, b, c) {}
    
    var bar = {};
    foo.apply(bar, [1, 2, 3]); // Le tableau sera transformé en une liste
                               // d'arguments, comme ci-dessous:
    foo.call(bar, 1, 2, 3); // results in a = 1, b = 2, c = 3

En utilisant les méthodes `call` ou `apply` de `Function.prototype`, il est
possible de modifier *explicitement* la valeur de `this` à l'intérieur d'une
fonction. `this` prend comme valeur celle du premier argument de `apply` ou
`call`.

Cette technique prend le pas sur le cas présenté plus haut (appel d'une méthode).
Dans notre exemple, `this` référencera toujours `bar`.

> **Note :** Il n'est pas possible d'utiliser `this` pour s'auto-référencer à
> l'intérieur d'un `Object`. Par exemple, dans `var obj = {me: this}`, `me` ne
> deviendra pas une référence à `obj`. Il n'y a pas d'autres manières de modifier
> la valeur de `this` que les cinq techniques précédement présentées.

### Écueils courants

La plupart des techniques présentées jusqu'ici ont du sens, à l'exception
peut-être de la toute première, qui pourrait être considérée comme un
artefact du language, dans le sens où cette convention n'est en fait
**jamais** utilisée en pratique.

    Foo.method = function() {
        function test() {
            // Ici, this est l'objet global.
        }
        test();
    }

Un éceuil courant consiste à penser que `this` référence `Foo` à l'intérieur de
`test`, alors que ce n'est **pas le cas**.

Il est possible de se donner une référence à `Foo` depuis `test` en créant une
variable locale à l'intérieur de `method`, context où il est possible d'avoir
une référence à `Foo` *via* `this`.

    Foo.method = function() {
        var self = this;
        function test() {
            // Utilisez self au lieu de this.
        }
        test();
    }

`self` est un nom de variable comme un autre, d'usage relativement courant pour
désigner une référence vers le `this` du scope parent. Cette technique permet
également de facilement propager des valeurs personnalisées de `this` à travers
un programme, en utilisant des [closures](#function.closures).

Depuis ECMAScript 5, il existe la méthode `bind` qui permet d'arriver au même
résultat, en utilisant des fonctions anonymes.

    Foo.method = function() {
        var test = function() {
            // this est maintenant une référence vers Foo.
        }.bind(this);
        test();
    }

### Assignation de méthodes

Autre élément qui ne fonctionne **pas** en JavaScript : les alias de fonctions,
c'est-à-dire assigner une fonction à une variable.

    var test = someObject.methodTest;
    test();

À cause de l'assignation implicite dans un contexte global, `test` est ici une
fonction à part entière dont la valeur de `this` ne sera pas `someObject`.

Cette assignation implicite vers l'objet global peut du coup paraître assez
ennuyeuse, mais il ne s'agit en fait que d'un cas particulier d'une règle plus
général d'assignation implicite en fonction du scope courant, règle qui permet
de faire en sorte que l'[héritage prototypal](#object.prototype) fonctionne
correctement et sans surprise.

    function Foo() {}
    Foo.prototype.method = function() {};

    function Bar() {}
    Bar.prototype = Foo.prototype;

    new Bar().method();

Ici, grâce à l'assignation implicite en fonction du scope (méthode), lorsque
`method` est appelée sur une instance de `Bar`, `this` référence bien cette
instance, le comportement attendu.

