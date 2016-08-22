# Partial Applications y Currying en ES6

## ¿Diferencia entre *aplicación parcial* y *currificación*? 

La aplicación parcial y la currificación son técnicas de programación que sirven para especificar una función que en su creación, fue planteada de manera más genérica. Si bien se refieren a algo similar, estos conceptos poseen sutiles diferencias que las hacen a fin de cuentas diferentes.
Antes de proceder recordemos algo esencial. La **aridad** de una función se define como la cantidad de parámetros que recibe ésta. Con esto, podemos tener funciones unarias, que son las de aridad uno. Luego binarias, terciarias, tetrarias y así siguiendo la regla de números griegos. Esto lo damos a conocer precisamente porque una de las diferencias entre aplicación parcial y currificación radica en la aridad de las funciones utilizadas y de las resultantes.

### Aplicación Parcial

La aplicación parcial es cuando una función general se reduce en parámetros para poder cumplir una función más puntual. En otras palabras, se aplica parcialmente una función cuando devolvemos otra función con *algunos* de sus argumentos fijados. Por ejemplo, consideremos que tenemos una función que busca en una lista genérica, los elementos de un tipo dado, y además aplica un filtro válido sobre aquel tipo. Su firma sería algo así como:
```JavaScript
var filtrar = (lista,tipo,filtro) =>  ... ;
```
pero ahora queremos aplicar este filtro sólo para tipos, por ejemplo, string. Para eso aplicamos parcialmente la función anterior de la siguiente manera:
```JavaScript
var filtrarStrings => (lista,filtro) => filtrar(lista,"String",filtro);
```
<!--- Ahora bien, si se fijan, en el ejemplo dado arriba, utilizamos notación con funciones lambda, al estilo de ES6. Era posible antes en ES5 hacer lo mismo, pero requería un código más engorroso y verboso que el puesto acá arriba, que es limpio y fácil de leer.
En la sección de más abajo mostraremos una comparativa de cómo se realizaba esto en antaño con ES6. --->

### Currying
<!--- Ahora por otra parte, tenemos la currificación, o currying. --->
Esta técnica consiste en que una función de aridad mayor a uno se transforme finalmente en una de aridad uno. Esto se realiza convirtiendo la función en una cadena de múltiples funciones de aridad uno en la que cada una vaya recibiendo uno a uno los parámetros que en su origen ésta requería. Así, se van retornando funciones hasta que finalmente se hayan rellenado todos los parámetros y se retorne el mismo resultado final de la función original. En otras palabras, un curry es una función que irá retornando nuevas funciones (otros currys) hasta que reciba todos sus argumentos.
<!--- Con explicaciones es cierto que suena  complicado y confuso, e incluso de cierta manera igual que lo que vimos en la sección anterior, pero  aquí dejamos un ejemplo: --->
Como ejemplo, currificaremos la misma función del ejemplo anterior. Si le aplicásemos currying de manera manual, la función tendría que ser creada de la siguiente manera:
<!--- Primero, la volveremos a mostrar de manera tradicional:
var filtrar = (lista,tipo,filtro) => {...}; --->
```JavaScript
var filtrarCurrificado = (lista) => (tipo) => (filtro) => ...;
```
Podemos notar que tenemos una seguidilla de funciones que nos crean de manera encadenada una función tras otra que van retornando sucesivamente funciones nuevas hasta que se rellenen los tres parámetros. Por ejemplo la función `filtrarCurrificado(lista)` nos retornará una nueva función que toma por parámetro el tipo que queremos aplicar. Para pasarle el tipo podemos hacer
```JavaScript
var filtrarConLista = filtrarCurrificado(lista);
var filtrarConTipoYLista = filtrarConLista(tipo);
```
Ahora, nos retornó una última función que está esperando el filtro a aplicar para finalmente ordenar la lista con el tipo que le dimos a lo largo de todos estos pasos: `var listaOrdenada = filtrarConListaYTipo(filtro);`

Ahora bien, podríamos llamar directamente a la función completa como `var listaFiltrada = filtrarCurrificado(lista)(tipo)(filtro);` pues estamos llamando a la misma función sucesivas veces sobre distintos argumentos.
<!---Parece engorroso a simplemente vista, pero si uno crea una función desde el principio, no es necesario tampoco hacer todos estos pasos, si uno no desea crear cada uno de los, llámese subfiltros. por ejemplo, podríamos llamar directamente a la función completa. Tan sólo que donde sabemos que la primera retorna una función, la segunda ídem, y ya la tercera nos retorna la lista, pues, tenemos que entregarle a cada una de ellas su parámetro de manera separada a los demás, miren:
var listaFiltrada = filtrarCurrificado(lista)(tipo)(filtro);
Y listo.
Ahora bien, si lo notan, nosotros realizamos el currying, digamos que de izquierda a derecha. Dejando como el primer parámetro a rellenar el de más a la izquierda (lista) y como parámetro de la funsión final el de más a la derecha, (filtro). Esto no es necesario así,  uno puede crear una función currificada tanto de izquierda a derecha, o de derecha izquierda. Es más, más adelante mostraremos librerías que siguen funcionando en ES6, que son capaces de realizar estas currificaciones de manera automática para nosotros, dejándonos de manera cómoda, funciones currificadas y listas para especificar.--->


## Currying en ES5 vs ES6
<!--- Lo que dice acá por ejemplo: https://gist.github.com/ryanseddon/7330082 --->
### ES5
Supongamos la función
```JavaScript
function formatText(tag, text) {
  return '<' + tag + '>' + text + '</' + tag + '>';
}

formatText('p', 'some text'); //=> "<p>some text</p>"
```
Una manera de aplicar parcialmente una función en ES5 es usando la función `.bind()`:
```JavaScript
var makeStrong = formatText.bind(this, 'strong');
makeStrong('this is important'); //=> "<strong>this is important</strong>"
```
Con `.bind()` podemos cambiar la aridad de la función de entrada, pero no es currying pues no retornamos funciones necesariamente de aridad 1. Lo que `.bind()` permite es aplicación parcial, fijando argumentos de la función a la que se aplica para retornar una de menor aridad. Consideremos la siguiente función: `function addTheseFourUp(a, b, c, d) { return a + b + c + d; }`. Podemos fijar algunos de sus argumentos:
```Javascript
var addTwoAndThreeAndTheseTwoUp = addTheseFourUp.bind(this, 2, 3);
addTwoAndThreeAndTheseTwoUp(1, 2); //=> 8
```
Pero si queremos currificar `addTheseFourUp()` sin librerías, una manera de hacerlo es
```Javascript
function curriedFour(a) {
  return function(b) {
    return function(c) {
      return function (d) {
        return a + b + c + d;
      }
    }
  }
}
curriedFour(1)(2)(3)(4); //=> 10
```

Para hacerlo de manera automática, independiente de la función de entrada, tendríamos que recurrir a métodos más complejos:
```JavaScript
function curry(fn) {
  var args = [].slice.call(arguments, 1);
    function curried(fnArgs) {
      if (fnArgs.length >= fn.length) {
        return fn.apply(null, fnArgs);
    }
      return function () {
        return curried(fnArgs.concat([].slice.apply(arguments)));
      };
    }
  return curried(args);
}

function mult(a, b) { return a*b; }

var multCurr = curry(mult)
var multByTwo = multCurr(2)
var twoTimesThree_v1 = multByTwo(3) //=> 6
var twoTimesThree_v2 = multCurr(2)(3) //=> 6
```

### ES6
En ECMAScript 6 podemos obtener una expresión más sencilla: 
```Javascript
function curry(fn, ...args) {
  var curried = (...fnArgs) => {
    if(fnArgs.length >= fn.length) {
      return fn.apply(this, fnArgs);
    }
    return (...cArgs) => curried([...fnArgs, ...cArgs]);
  }
  return curried(...args);
}
```

## Implementaciones

Existen variadas librerías que tienen implementados estos métodos para su ágil uso. Algunas de ellas son:

### Wu.js
En [Wu.js](https://fitzgen.github.io/wu.js/#curryable) encontramos un decorador `.curryable()` que currifica automáticamente la función a la que se lo aplicamos. Opcionalmente se le puede pasar el número máximo de argumentos para currificar como segundo argumento. En caso contrario queda por defecto como el número total de argumentos de la función `fn.length`:
```Javascript
var add = wu.curryable((a, b, c) => a + b + c);

add(1)(1)(1); //=> 3
add(1)()(1)()(1); //=> 3
add(1)(1, 1); //=> 3
add(1, 1, 1); //=> 3
```

```Javascript
var modulo = wu.curryable((divisor, dividend) => dividend % divisor);
var isOdd = modulo(2);

isOdd(6); //=> 0
isOdd(7); //=> 1

var filter = function(f, xs) {
  return xs.filter(f);
}.curryable();

var getOdds = filter(isOdd);
getOdds([1, 2, 3, 4, 5]); //=> [1, 3, 5]
```

### Underscore.js
En Underscore tenemos [una función](http://underscorejs.org/#partial) que permite la aplicación parcial (`_.partial`) de funciones de manera inmediata:
```JavaScript
var sub = (a, b) => b - a;
sub5 = _.partial(sub, 5);
sub5(20); //=> 15
```
Una ventaja de usar esta implementación es que Underscore permite la especificación de *placeholders* para usar fijar un cierto argumento:
```Javascript
subFrom20 = _.partial(subtract, _, 20);
subFrom20(5); //=> 15
```

### Ramda.js
Con [Ramda](http://ramdajs.com/docs/#curry) también podemos hacer uso del valor placeholder. Si `f(...args)` es la función a currificar y `var g = R.curry(f)`, entonces las siguientes son equivalentes:
````Javascript
g(1, 2, 3)
g(_, 2, 3)(1)
g(_, _, 3)(1)(2)
g(_, _, 3)(1, 2)
g(_, 2)(1)(3)
g(_, 2)(1, 3)
g(_, 2)(_, 3)(1)
```
Adicionalmente también se puede especificar la aridad de la función currificada al igual que en Wu, con la función `R.curryN(fn.length, fn)`.

## ...¿y para qué sirve?
Supongamos que tenemos una función `curry()`. 
A parte de los ejemplos anteriores, la currificación ayuda cuando queremos manipular data que nos llega del servidor. Al igual que las promesas, los currys devuelven contenedores (funciones en este caso) que comparten la misma interfaz y que nos permiten lidiar con ellos de manera estandarizada. Éstos tienen un mecanismo de iteración que hacen que no haga más trabajo de lo que puede hacer con los argumentos suplidos. Por ejemplo a la función `var a = curry(fn(b,c))` no tenemos que aplicarle inmediatamente el parámetro `c`. Podemos esperar aplicarlo una vez que contemos con el resultado de una promesa:
```Javascript
const nextFn = a(b);
doSomeStuff().then(() => nextFn(c));
```
Veamos un ejemplo concreto. 
Suponer que tenemos una lista de objetos y queremos mapearla bajo una función:
```Javascript
var objects = [{ id: 1 }, { id: 2 }, { id: 3 }];
objects.map((o) => o.id); //=> [1, 2, 3]
```
Una manera de hacer que el código se lea como la lógica del dominio en que estamos trabajando es con currys:
```Javascript
var map = curry((fn, value) => value.map(fn));
var getIDs = map(get('id'));

getIDs(objects) //= [1, 2, 3]
```
Esto mismo lo podemos aprovechar para tareas del mismo tipo. Si tenemos el siguiente objeto:
```Javascript
{
  "user": "hughfdjackson",
  "posts": [
    { "title": "why curry?", "contents": "..." },
    { "title": "prototypes: the short(est possible) story", "contents": "..." }
  ]
}
```
y queremos obtener el título de todos sus posteos, una manera tradicional de hacerlo sería:
```Javascript
fetchFromServer()
  .then(JSON.parse)
  .then((data) => data.posts)
  .then((posts) => posts.map((post) => post.title))
```
pero podemos expresarlo de manera más limpia y semántica con currying:
```Javascript
fetchFromServer()
  .then(JSON.parse)
  .then(get('posts'))
  .then(map(get('title')))
```
### Referencias
- [Favoring Curry](http://fr.umio.us/favoring-curry/) (Recomendado)
- [Why Curry Helps](https://hughfdjackson.com/javascript/why-curry-helps/)
- [Curry or Partial Application?](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8#.fai0znu0q)
- [Partial Application in JavaScript](http://benalman.com/news/2012/09/partial-application-in-javascript/#partial-application-vs-currying)
- [ECMAScript 6 and Currying](http://macr.ae/article/es6-and-currying.html)
- [What's the difference between Currying and Partial Application?](http://raganwald.com/2013/03/07/currying-and-partial-application.html)
- [Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)
- [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon)
