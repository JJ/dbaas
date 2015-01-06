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
futbolísticas con el que seguiremos trabajando más adelante.
