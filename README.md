# Partial Applications y Currying en ES6

## ¿Diferencia entre *aplicación parcial* y *currificación*? 

La aplicación parcial (partial aplication) y la currificación (currying) ambas son técnicas de programasión que sirven para especificar una función que en su creación, fue planteada de manera más general o genérica.
no obstante, aunque ambas técnicas intenten cumplir un objetivo similar,, poseen sutiles diferencias que las hacen a fin de cuentas la una diferente de la otra.
Tal como podemos leer en el siguiente artículo: ( https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8#.fai0znu0q )
o en este otro ( http://raganwald.com/2013/03/07/currying-and-partial-application.html )
Podemos notar que cada uno busca cumplir un objetivo similar, pero no igual. Procederemos a explicarlos cada uno:

Antes de proceder, recordaremos algo escencial. La aridad de una funsión, se define como la cantidad de parámetros que recibe ésta. Con esto, podemos tener funsiones unarias, que son las de aridad uno. Luego binarias, terciarias, tetrarias y así siguiendo la regla de números griegos.
Esto lo damos a conocer, precisamente porque una de las diferencias entre aplicación parcial y currificación radica en la aridad de las funciones utilizadas, y las resultantes.


### Aplicación Parcial

La aplicación parcial, es cuando una función general se reduce en parámetros para poder cumplir una función más puntual.

Por ejemplo, consideremos que tenemos una función que busca en una lista genérica, los elementos de un tipo dado, y además aplica un filtro válido sobre aquel tipo. Su firma sería algo así como:

var filtrar=(lista,tipo,filtro)=> { ...}

pero ahora, queremos aplicar una aplicación parcial sobre esta función, para crear una nueva función, que haga exactamente lo mismo, pero sabemos que es solamente para filtrar strings.

En este caso, sí estamos aplicando una aplicación parcial, porque sólo reduciremos en un parámetro, y nada más. Ejemplo:

var filtrarStrings => (lista,filtro) => filtrar(lista,"String",filtro);

Ahora bien, si se fijan, en el ejemplo dado arriba, utilizamos notación con funciones lambda, al estilo de ES6. Era posible antes en ES5 hacer lo mismo, pero requería un código más engorroso y verboso que el puesto acá arriba, que es limpio y fácil de leer.
En la sección de más abajo mostraremos una comparativa de cómo se realizaba esto en antaño con ES6.

### Currying

Ahora por otra parte, tenemos la currificación, o currying.
Esta técnica, consiste en que una funsión de aridad mayor a uno, independientemente de cuánto sea esta, se transforme finalmente a una funsión de aridad uno. Esto se realiza, convirtiendo la funsión en una cadena de múltiples funsiones de aridad uno que cada una va reciviendo uno a uno los parámetros que en su origen ésta requería, y va retornando una nueva funsión, hasta que finalmente se hayan rellenado todos los parámetros y  se retorne el mismo  resultado final.
Con explicaciones es cierto que suena  complicado y confuso, e incluso de cierta manera igual que lo que vimos en la sección anterior, pero  aquí dejamos un ejemplo:

utilizaremos nuestra misma funsión filtrar del ejemplo anterior:

Primero, la volveremos a mostrar de manera tradicional:

var filtrar = (lista,tipo,filtro) => {...};

Ahora, si le aplicásemos currying de manera manual, para que se cumpliese lo dicho previamente, la funsión tiene que ser creada de la siguiente manera:

var filtrarCurrificado = (lista)=>(tipo)=>(filtro)=> {...};

Podemos notar que tenemos una seguidilla de operadores lambda (funsión flecha) que nos crean de manera encadenada una funsión tras otra que van retornando entre sí una funsión nueva, hasta que se rellenen los tres parámetros. por ejemplo, si llamamos a la función:

filtrarCurrificado(lista);

nos retornará una nueva función, que tomará por parámetro ahora el tipo que queremos aplicar. tendríamos entonces que hacer:

var filtrarConLista = filtrarCurrificado(lista);

Ahora, tenemos que entregarle el tipo:

var filtrarContipoYLista = filtroCOnLista(tipo);

Ahora, nos retornó una última funsión que está esperando el filtro a aplicar, para finalmente ordenar la lista, con el tipo que le dimos a lo largo de todos estos pasos:

var listaOrdenada = filtrarConListaYTipo(filtro);

Y listo.

Parece engorroso a simplemente vista, pero si uno crea una función desde el principio, no es necesario tampoco hacer todos estos pasos, si uno no desea crear cada uno de los, llámese subfiltros. por ejemplo, podríamos llamar directamente a la función completa. Tan sólo que donde sabemos que la primera retorna una función, la segunda ídem, y ya la tercera nos retorna la lista, pues, tenemos que entregarle a cada una de ellas su parámetro de manera separada a los demás, miren:

var listaFiltrada = filtrarCurrificado(lista)(tipo)(filtro);

Y listo.

Ahora bien, si lo notan, nosotros realizamos el currying, digamos que de izquierda a derecha. Dejando como el primer parámetro a rellenar el de más a la izquierda (lista) y como parámetro de la funsión final el de más a la derecha, (filtro). Esto no es necesario así,  uno puede crear una función currificada tanto de izquierda a derecha, o de derecha izquierda. Es más, más adelante mostraremos librerías que siguen funcionando en ES6, que son capaces de realizar estas currificaciones de manera automática para nosotros, dejándonos de manera cómoda, funciones currificadas y listas para especificar.


## Ejemplos Básicos
blablablalblabblaabl
```JavaScript
// Blabla
const insertar = (ejemplo) => ejemplo;
const sumaCurrificada = (x) => (y) => x + y;
```

## Currying en ES5 vs ES6
Lo que dice acá por ejemplo: https://gist.github.com/ryanseddon/7330082

## Implementaciones
Tanto las librerías Underscore.js como Ramda.js cuentan con funciones para precisamente la aplicación parcial de funciones. Veamos algunos ejemplos:
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

