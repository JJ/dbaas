Introducción a las bases de datos como servicio
=====

## Objetivos

1. Entender el concepto de DBaaS.
2. Darse de alta en algún servicio DBaaS.
3. Crear scripts simples para Redis.

## Introducción a la introducción

Como las bases de datos son, en realidad, una aplicación como otra
cualquiera,
[las bases de datos como servicio, bases de datos en la nube o *DBaaS*](http://en.wikipedia.org/wiki/Cloud_database)
encajan mejor dentro de este texto que de ningún otro sitio, aunque en
realidad no son una solución completa, sino que se tienen que combinar
con un PaaS o un IaaS para crear una aplicación. Sin embargo, es
conveniente tener conocimiento de ellas, puesto que los PaaS a que se
verán a continación las usan.

Los DBaaS ofrecen acceso tanto bases de datos clásicas, es decir, con
el lenguaje SQL, como a bases de datos *sin esquemas* o NoSQL como
Redis, CouchDB o MongoDB. También hay modelos *freemium* o gratuitos
con tarjeta de crédito, tales como
[Amazon DynamoDB](http://aws.amazon.com/es/dynamodb/?nc2=h_l3_db) o
[ClearDB, que provee servicio MySQL](https://www.cleardb.com/pricing.view). La
mayoría de los PaaS, por otro lado, ofrecen también DBaaS como
añadidos a sus plataformas; es decir, tarde o temprano se acabarán
usando.

Vamos a aprovechar que estamos hablando de nuevas bases de datos para
trabajar con [Redis](http://redis.io). Redis es una base de datos no
persistente, en memoria, de altas prestaciones, y que permite trabajar
de forma muy eficiente con estructuras de datos simples. Otros
sistemas noSQL como CouchDB o MongoDB también son bastante populares,
pero Redis se está convirtiendo en uno de los estándares emergentes y
tiene buen soporte en JavaScript, tanto en cliente como en node.

>En este momento, si no se ha hecho antes, se deberá instalar
>[Node](http://nodejs.org), un intérprete de Javascript para usar en
>servidores o en línea de órdenes. No es el objetivo de este texto
>enseñar ni node ni JavaScript, del que tendrías que tener cierto
>conocimiento previo. Si quieres aprender, hay un *cough* *cough*
>excelente
>[texto introductorio](https://www.amazon.es/dp/B00HXL8QA0?tag=atalaya-21&camp=3634&creative=24822&linkCode=as4&creativeASIN=B00HXL8QA0&adid=1TS47T651FGQTKED28CV&)
>por menos de un euro en Amazon (o libre en GitHub).

En vez de ir [característica por característica u orden por orden (que, además, son un montón)](http://redis.io/commands), vamos a empezar
trabajando con un sistema cliente-servidor para hacer porras
futbolísticas con el que seguiremos trabajando más adelante. Pero
antes, una aproximación básica a Redis en [el siguiente programa](https://github.com/JJ/node-app-cc/blob/master/redis.js), que
prueba las principales características trabajando con pares
variable-valor y *hashes*:

```javascript
var redis = require('redis');
var url = require('url');

var redisURL = url.parse(process.env.REDISCLOUD_URL);
console.log(redisURL);
var client = redis.createClient(redisURL.port, redisURL.hostname, {no_ready_check: true, auth_pass: redisURL.auth.split(":")[1]});

client.set("zape", "zipi", redis.print );
client.get("zape", function (err, reply) {
    console.log( 'Get ' );
    if ( err ) {
	console.log( err );
    } else {
	console.log(reply.toString());
    }
});
client.hset("un_foo", "bar", "baz", redis.print);
client.hset("un_foo", "quux", "zuuz", redis.print);
client.hkeys("un_foo", function (err, replies) {
    console.log( 'hkeys');
    replies.forEach(function (reply, i) {
        console.log("    " + i + ": " + reply);
    });
    console.log( "End " );
    client.end();
});
```

El programa tiene tres partes. En la primera se conecta al DBaaS que
previamente hemos tenido que crear en RedisLabs o, para el caso, en
nuestro propio ordenador. Las credenciales para acceder al sitio están
metidas en una variable de entorno, REDISCLOUD_URL. El URL de esa variable te la
habrán asignado en redislabs cuando hayas creado un recurso gratuito,
y será por el estilo de
`pub-redis-12345.us-east-1-2.3.ec2.garantiadata.com:12345`, pero
tendrás que combinarla con la clave para acceder a la base de datos de
esta forma:

	export REDISCLOUD_URL=http://daigual:esta_es_la_clave@cosas.garantiadata.com:12345

; lo que tendrás que escribir en la línea de órdenes y nunca, nunca,
dejar en el sistema de control de fuentes. 
Es un URL un tanto complejo, pero la parte principal es la que hay
detrás del `//`, de la forma `usuario:clave@dominio:puerto`. Es imprescindible
autenticarse, para que sólo uno pueda usar el recurso. En realidad, el
usuario no se usa, por eso pone `daigual`, sin embargo la clave es la
que estableceremos para el recurso cuando nos demos de alta; por
defecto, es la misma que se usa para la cuenta general, aunque puedes
establecer claves específicas para cada uno de los depósitos de
datos. Previamente a esto habrá que haber creado una *suscripción* de
Redis en "My Resources -> Manage"; hay derecho al menos a uno gratuito
por persona aunque sólo te permiten
[25 MB y 10 conexiones simultáneas](https://redislabs.com/pricing).

>Redis, de todas formas, es software libre y puedes instalarlo sin
>ninguna limitación en tu propio alojamiento si lo tienes o en tu
>IaaS.

La siguiente parte del programa es la que establece un par
variable-valor: `zipi - zape`, es decir, que asociamos a la clave
`zipi` el valor `zape`; a continuación lo recuperamos usando la forma
asíncrona habitual de node: se solicita el valor y se le pasa una
función *callback* a la que se llame cuando se haya recibido la
respuesta.

>En realidad y teniendo en cuenta que es asíncrono, no podemos
>recuperar el valor hasta que hayamos recibido el callback; es un
>error poner las órdenes de esta forma porque puede suceder que cuando
>se trate de recuperar el valor todavía no se haya establecido en el
>servidor. En este caso, sin embargo, funciona por la rapidez de
>Redis, aunque no tiene por qué funcionar en todos los casos.

El tercer bloque trabaja con un [HSET](http://redis.io/commands/HSET),
un conjunto de pares clave-valor indexados, a su vez, con una sola
clave. Redis tiene varios tipos de datos y tratándose de una base de
datos NoSQL,
[sus propios comandos para acceder a los mismos](http://openmymind.net/2011/11/8/Redis-Zero-To-Master-In-30-Minutes-Part-1/). Usamos
dos sentencias con la misma clave, `un_foo`, que será la clave del
HSET, y le asignamos dos pares variable-valor. Es una estructura de
datos un poco más compleja, que nos puede servir para almacenar las
porras más adelante. Como en el caso anterior, convendría haberlo
hecho esto de forma asíncrona, pero también, y en general (y en
Redis), también funciona de esta forma.

> Redis también permite trabajar con conjuntos usando la orden
> [SADD](http://redis.io/commands/sadd). Se trataría de varias
> variables asignadas a un solo valor (el nombre del conjunto). Crear
> un programa que cree un conjunto, el de todas las porras existentes,
> por ejemplo.

Es importante también que el cliente de Redis se cierre, como se hace
en la penúltima línea con `client.end();`. Si no, el programa queda en espera. Esa orden,
efectivamente, termina el programa (aparte del cliente de
Redis). Cualquier programa en Redis tiene que terminar de esa forma.

## Poniendo en práctica Redis en porr.io

El problema principal con Redis es rediseñar la aplicación desde
nuestra mente base-de-datos-relacional para aprovechar sus
fortalezas. Redis almacena estructuras de datos sólo indexadas por
clave. Se puede acceder a todas las claves o hacer búsquedas con
patrones. Con los resultados del ejemplo anterior se puede instalar el
cliente de redis (`sudo apt-get redis-cli`) y acceder de esta forma

```
redis-cli -h pub-redis-12345.us-east-1-2.3.ec2.garantiadata.com -p
12345 -a esta-es-la-clave
```

es decir, usando el URL anterior (que se pasa con la opción `-h` a la
línea de órdenes) y la clave que hayamos establecido (con `-a`) y
podemos hacer consultas usando las órdenes de Redis, por ejemplo:

```
pub-redis-12345.us-east-1-2.3.ec2.garantiadata.com:12345> keys *
1) "Granada-C\xc3\xb3rdoba-Liga-2018"
2) "zape"
3) "un_foo"
...
```

Aunque las claves estén almacenadas al alimón, en realidad las órdenes
que se pueden aplicar sobre ellas son diferentes: `zape` tenía
asignada una cadena, y `un_foo` un hash. Eso lo averiguamos con `type`

```
pub-redis-12345.us-east-1-2.3.ec2.garantiadata.com:12345> type "zape"
string
pub-redis-12345.us-east-1-2.3.ec2.garantiadata.com:12345> type "un_foo"
hash
```

Con esto, la estrategia de usar tablas para cosas se va un poco por
ahí. Tenemos que pensar en almacenar claves, con un criterio.
razonable, y poder recuperarlas en función del contenido de
claves. Afortunadamente, Redis es muy rápido y el hecho de que no se
puedan hacer merges realmente no importa demasiado. Es más, la
complejidad de las peticiones y el tiempo que tardan no depende del
número de claves que haya.

Como buena práctica lo que se suele hacer es usar *prefijos* separados
por `:` para distribuir las claves en diferentes "espacios de
nombres". Por ejemplo, podíamos meter todas claves referidas a porras
en el espacio `porra:` y podriamos buscarlas usando `keys
"porra:*"`. Algo así hacemos en el siguiente programa:

```
var redis = require('redis');
var url = require('url');

var apuesta = require("./Apuesta.js"),
porra = require("./Porra.js");

var redisURL = url.parse(process.env.REDISCLOUD_URL);
var client = redis.createClient(redisURL.port, redisURL.hostname, {no_ready_check: true, auth_pass: redisURL.auth.split(":")[1]});

var esta_porra = new porra.Porra("FLA", "FLU", "Premier", 1950+Math.floor(Math.random()*70) );
console.log(esta_porra);
for ( var i in esta_porra.vars() ) {
    client.hset(esta_porra.ID, "var:"+esta_porra.vars()[i], esta_porra[i], redis.print);
}

var bettors = ['UNO', 'OTRO','OTROMAS'];

for ( var i in bettors ) {
    var esta_apuesta = new apuesta.Apuesta(esta_porra, bettors[i], Math.floor(Math.random()*5), Math.floor(Math.random()*4) );
    client.hset(esta_porra.ID, "bet:"+esta_apuesta.quien, esta_apuesta.resultado());
    client.sadd(esta_porra.ID+":"+esta_apuesta.resultado(), esta_apuesta.quien,redis.print );
    
}

client.hkeys(esta_porra.ID, function (err, replies) {
    console.log( 'hkeys');
    replies.forEach(function (reply, i) {
        console.log("    " + i + ": " + reply);
    });
    console.log( "End " );
    client.end();
});

```

El
[programa, denominado obviamente `porredis.js`](https://github.com/JJ/node-app-cc/)
también se divide en varias partes. La primera parte es la conexión a
la base de datos, que es exactamente igual que en el programa
anterior. A continuación se crea una porra con elementos aleatorios
(el año) para que se cree ligeramente diferente en cada ejecución. 

> Si tenéis curiosidad de qué se trata esta porra, es del célebre
> [derby entre el Fluminense y el Flamingo](http://es.wikipedia.org/wiki/Fla-Flu),
> tradicionales rivales del estado de Río de Janeiro.

Vamos a usar un HSET para almacenar cada porra, y usamos un campo con
el prefijo `var` para cada una de las variables de la porra; como
clave usamos la propia clave de la porra. Esta clave la vamos a usar
para almacenar todo y además tiene elementos para acceder rápidamente
a todas las porras de un año o de un equipo.

Las apuestas de la porra, con tres apostadores, las generamos
aleatoriamente también en el siguiente bloque. Almacenamos las
apuestas en dos sitios. En una BD relacional esto sería anatema, pero
aquí no es un gran problema: Redis es suficientemente rápido, y se
trata de que podamos acceder rápidamente a la información. Vamos a
usar el mismo hash para almacenar los nombres de los apostantes, y
conjuntos para almacenar todos los que han apostado por un resultado
determinado. De esa forma, a partir del ID de una porra y del
resultado podemos acceder, en una sola petición, a los ganadores de la
misma, si es que los hay. Por ejemplo, se busca así todos los
resultados de una porra:

```
pub-redis-13876.us-east-1-2.3.ec2.garantiadata.com:13876> keys "FLA-FLU*1998:*"
1) "FLA-FLU-Premier-1998:4-2"
2) "FLA-FLU-Premier-1998:3-2"
```

(se puede hacer algo equivalente desde el cliente en node). Y una vez
localizado el resultado,

```
pub-redis-13876.us-east-1-2.3.ec2.garantiadata.com:13876> smembers "FLA-FLU-Premier-1998:3-2"
1) "OTRO"
2) "OTROMAS"
```

que da como afortunados ganadores a OTRO y a OTROMAS. Siempre
aciertan, los tíos.

El último bloque del programa recupera todas las apuestas que haya
almacenadas para una porra determinada, las tres que se han hecho. El
resultado será algo así:

```
Reply: 1
Reply: 1
Reply: 1
Reply: 1
Reply: 1
Reply: 1
Reply: 1
hkeys
    0: var:local
    1: var:visitante
    2: var:competition
    3: var:year
    4: bet:UNO
    5: bet:OTRO
    6: bet:OTROMAS
End 
```
Las primeras `Reply`s son el número de registros insertados. El resto
muestra las claves del *hash* que se ha creado, que serán siempre las
mismas. Por supuesto, la final del programa se cierra el cliente.

>Hacer un programa que recupere los ganadores de una porra almacenados
>en Redis.
