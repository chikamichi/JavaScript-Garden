## Déclarations de fonctions et expressions

En JavaScript, les fonctions sont des objets de premier plan : une fonction
peut être traitée comme n'importe quelle autre valeur. Il est par exemple
possible d'utiliser une fonction dite *anonyme* comme callback d'une autre,
par exemple pour traiter des problèmes asynchrones.

### La déclaration `function`

    function foo() {}

La déclaration ci-dessus est soumise au processus de [hoisting](#function.scopes)
avant que l'exécution du programme ne commence, de sorte que la fonction `foo`
devient disponible *partout* à l'intérieur du scope dans lequel elle a été
définie. Il est ainsi possible de l'appeler avant d'avoir rencontré sa
définition formelle.

    foo(); // Fonctionne parfaitement, car foo est en fait créée avant
           // d'arriver à cette ligne du code, bien que la définition
           // formelle soit ci-après.
    function foo() {}

### L'expression `function`

    var foo = function() {};

Dans cet exemple, une fonction sans nom, *anonyme*, est assignée à la variable
`foo`.

    foo; // 'undefined'
    foo(); // Va lever une erreur TypeError.
    var foo = function() {};

Du fait que `var` est une déclaration qui déclenche le processus de hoisting
pour le nom de variable `foo` avant que l'exécution du code ne débute, `foo`
est déjà utilisé, d'où l'erreur lors de l'appel de la fonction du même nom.

Par contre, comme les assignations de variables ne sont gérés qu'au moment
de l'exécution, la valeur de `foo` sera bien [undefined](#core.undefined).

### Expressions faisant intervenir une fonction nommée

Autre cas un peu spécial : l'assignation d'une fonction nommée.

    var foo = function bar() {
        bar(); // Fonctionne.
    }
    bar(); // ReferenceError

Ici, `bar` n'est pas disponible dans le scope général, car la fonction n'est
en fait assignée qu'à `foo`. Par contre, à l'intérieur de `bar`, pas de
problème pour utiliser `bar`. Il faut se tourner vers le mécanisme de
[résolution de nom](#function.scopes) de JavaScript pour comprendre ce qu'il
se passe exactement : le nom d'une fonction est *toujours* disponible comme
référence à l'intérieur de la fonction elle-même.

