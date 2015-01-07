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

En vez de ir característica por característica, vamos a empezar
trabajando con un sistema cliente-servidor para hacer porras
futbolísticas con el que seguiremos trabajando más adelante. Pero
antes, una aproximación básica a redis en [el siguiente programa](https://github.com/JJ/node-app-cc/blob/master/redis.js), que
prueba las principales características trabajando con pares
variable-valor y *hashes*

```javascript
#!/usr/bin/env node

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

