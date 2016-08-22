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

### Underscore.js
`_.partial`
```JavaScript
var subtract = (a, b) => { b - a };
sub5 = _.partial(subtract, 5);
sub5(20);
=> 15

// Using a placeholder
subFrom20 = _.partial(subtract, _, 20);
subFrom20(5);
=> 15
```

### Ramda.js

## ...¿y para qué sirve?
https://hughfdjackson.com/javascript/why-curry-helps/
http://fr.umio.us/favoring-curry/

### Referencias

1. https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8#.fai0znu0q

