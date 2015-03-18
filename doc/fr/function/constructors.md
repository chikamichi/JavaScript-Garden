## Constructeurs

Une fois de plus, JavaScript amène à manipuler une vision un peu différente de
la notion de constructeur par rapport à ce qui existe dans d'autres languages.
En JavaScript, toute fonction appelée avec le mot-clé `new` va se comporter
comme un constructeur.

À l'intérieur du constructeur — toute fonction agissant comme tel, donc — la
valeur de `this` va référencer un objet en cours de création. Le
[prototype](#object.prototype) de ce **nouvel** objet va référencer le même
prototype que celui de la fonction qui a été utilisée comme constructeur.

Si la fonction initialement appelée n'a pas de `return` explicite, alors
`this` devient implicitement la valeur de retour, à savoir, le nouvel
objet.

    function Person(name) {
        this.name = name;
    }

    Person.prototype.logName = function() {
        console.log(this.name);
    };

    var sean = new Person();

Le code ci-dessus fait appel à la fonction `Person` en tant que constructeur,
de sorte que le nouvel objet créé lors de l'appel voit son `prototype` devenir
`Person.prototype`.

Si une instruction `return` est utilisée de façon explicite, la fonction va
retourner la valeur associée à `return` mais **seulement** si cette valeur est
un `Object`.

    function Car() {
        return 'ford';
    }
    new Car(); // Sera en fait un objet, et pas 'ford'.

    function Person() {
        this.someValue = 2;

        return {
            name: 'Charles'
        };
    }
    new Person(); // Sera l'objet {name:'Charles'}, someValue n'étant pas accessible.

En l'absence de `new` et sans `return`, la fonction ne retournera **pas** un
nouvel objet.

    function Pirate() {
        this.hasEyePatch = true; // Défini sur l'objet global !
    }
    var somePirate = Pirate(); // somePirate est undefined.

L'exemple ci-dessus peut parfois se comporter comme si tout allait pour le
mieux, mais c'est uniquement en raison d'effets de bord lié au fonctionnement
de [`this`](#function.this) en JavaScript, car bien souvent, `this` va
référencer l'objet global.

### Vers le concept d'usines à objets

En bref, pour se comporter correctement en présence du mot-clé `new`, une
fonction-constructeur se doit de retourner explicitement une valeur.

    function Robot() {
        var color = 'gray';
        return {
            getColor: function() {
                return color;
            }
        }
    }
    Robot.prototype = {
        someFunction: function() {}
    };

    new Robot();
    Robot();

Les deux appels à `Robot` vont retourner la même chose : un nouvel objet avec
une propriété `method`, en fait une [closure](#function.closures).

Il convient de noter que l'appel `new Robot()` ne va **pas** modifier le
prototype de l'objet retourné suite à cet appel (l'objet avec `getColor`).
Certes, un nouvel objet est créé avec un prototype bien particulier, mais
`Robot` ne retourne pas cet objet implicitement créé mais un autre.

Aussi, dans l'exemple ci-dessus, il n'y a pas de différence fonctionnelle
liée au fait d'utiliser ou d'omettre `new`.

### Créer de nouveaux objets avec des usines dédiées

Il est souvent recommandé de ne **pas** utiliser `new`, car un oubli du
mot-clé peut conduire à des bugs difficiles à débusquer.

Dans cette perspective, on recommandera plutôt d'utiliser une « usine »
à objet.

    function CarFactory() {
        var car = {};
        car.owner = 'nobody';

        var milesPerGallon = 2;

        car.setOwner = function(newOwner) {
            this.owner = newOwner;
        }

        car.getMPG = function() {
            return milesPerGallon;
        }

        return car;
    }

Ce type de code est assez robuste, dans le sens où un oubli de `new` n'a
aucun impact et où l'utilisation de [variables privées](#function.closures)
est simplifié, mais il y a toutefois quelques désavantages à procéder ainsi.

 1. La consommation mémoire augmente, car les méthodes ne sont plus partagées
    sur un prototype commun aux objets créés.
 2. Pour gérer l'héritage, l'usine va devoir copier toutes les méthodes d'un
    objet ou bien assigner elle-même un tel objet comme prototype, ce qui est
    coûteux en performance.
 3. Laisser tomber toute notion d'héritage juste pour gérer l'absence
    potentielle de `new` (avec de bonnes performances) revient sans doute à
    aller contre l'esprit du langage lui-même.

### En guise de conclusion

Il est vrai qu'oublier `new` est source de bugs, mais ce n'est vraiment **pas**
une raison pour se passer de toute la partie liée à l'héritage prototypal. Au
final, peu importe quelle solution est utilisée, tant qu'elle a du sens dans le
contexte particulier de votre application, et que vous vous y tenez.

