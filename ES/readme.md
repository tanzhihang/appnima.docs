App/nima
========
Descubre como dar alma a tus aplicaciones con el primer LaaS en el mundo.

*Version Actual: [1.0.0]()*


Introducción
------------
Simplemente por leer este documento declara que eres un desarrollador que quiere mejorar constantemente y quiere crear proyectos cada vez más eficientes. App/nima es la primera plataforma que ofrece servicios lógicos para cualquier tipo de proyecto, da igual si quieres crear una aplicación o un site, App/nima te ayudará en ambas situaciones.

Un poco de historia, hace ya casi 3 en [**Tapquo**](http://tapquo.com) nos encontramos con un problema común en el mundo del desarrollo y no era más que cada nuevo producto que creabamos debiamos repetir una y otra vez la misma funcionalidad básica. App/nima surgio de la necesidad de ser cada vez más eficientes y de querer desarrollar únicamente el negocio implicito de nuestro nuevo producto y no tanto de la lógica horizontal:

+ Servicio OAuth 2 para la autentificación
+ Gestión de usuarios
+ Red social de usuarios
+ Servicio de mensajeria para enviar emails, SMS, llamadas y mensajes privados
+ Localizar/Geoposicionar lugares
+ Localizar/Geoposicionar a otros usuarios
+ Tener Real-time por medio de sockets
+ Notificaciones Push para las plataformas: iOS, Android o Blackberry
+ Saber como se comportan nuestros usuarios

Si por lo menos alguna vez has tenido que desarrollar alguna de estas funcionalidades en alguno de tus proyectos, eres bienvenido a nuestra plataforma ya que App/nima esta pensada para ayudarte como desarrollador. Queremos ofrecerte una plataforma centrada en facilitarte la vida, queremos el software tanto como tu y buscamos ser cada día mejores. Usa App/nima.


### Arquitectura
Seguramente en este momento tendrás dudas de como App/nima puede ayudarte a ser más eficiente pero tranquilo vamos a explicarte cómo funciona. App/nima esta basada por completo en el protocolo de autentificación [**Oauth 2**](http://oauth.net/2/) y en la serialización mediante REST-style con objetos JSON. Si nunca has utilizado este paradigma no te preocupes te explicarémos paso a paso como conectarte a la plataforma y te ofreceremos herramientas específicas como [**AppnimaJS**](http://) que te facilitarán el proceso de conexión.

Todo el backend está montado a lo largo del mundo utilizando plataformas como Amazon o Google Cloud Engine las cuales nos dan la capacidad de ofrecer siempre la mejor calidad de respuesta para nuestros usuarios. No te preocupes no tendrás que ser un *Jedy SysAdmin* ya que tu unicamente te tienes que comunicar con nuestro REST la tarea de escalabilidad corre de nuestra cuenta, tu preocupate en crear el mejor proyecto y nosotros de ofrecerte cada día una mejor plataforma.

Resumiendo App/nima es una plataforma en forma de API REST que te provee de los servicios logicos de tu proyecto, y dependiendo de la naturaleza de tu proyecto puede que no necesites ni tu propio Backend. Cada servicio lógico esta alojado en servidores diferentes para obtener la mayor escalabilidad independiente, a lo largo de la documentación conoceremos las rutas de cada servicio.


### Oauth 2
Todos los servicios de App/nima utilizan el protocolo  de autentificación [**OAuth 2**](http://oauth.net/2/) buscando la mayor compatibilidad con herramientas de terceros. Si no eres un experto en OAuth 2 te recomendamos que lo estudies ya que esta siendo a llamar el protocolo de autentificación por excelencia y empresas como Google, Facebook o Twitter lo utilizan para conectarse a sus APIs. Lee la [documentación](http://) para comenzar hoy mismo con App/nima.


### No XML, solo JSON
Sólo permitimos la serialización de datos tipo JSON. Nuestro formato es no tener ningún elemento raíz y utilizar *snake_case* para describir las claves de atributos. Esto significa que en cada petición a App/nima tienes que enviar el sufijo:

    Content-Type: application / json; charset = utf-8

Recibirás una respuesta 415 *Unsupported Media Type* si intentas utilizar otro tipo de sufijo URL.


### Usa el HTTP cache
Tienes que hacer uso de las cabeceras HTTP *freshness* para disminuir la carga de nuestros servidores (¡y aumentar la velocidad de tu aplicación!). La mayoría de las peticiones que devolvemos incluirán un *ETag* o una cabecera *Last-Modified*. Al solicitar por primera vez un recurso, almacena este valor y envianoslo en las posteriores peticiones como *If-None-Match* y *If-Modified-Since*. Si el recurso no ha cambiado, obtendrás un código 304 *Not Modified* como respuesta, que te ahorra tiempo y ancho de banda (porque ya tienes ese recurso).


### Manejando errores
Si App/nima tiene algun problema, es posible que veas un error 5xx. 500 significa que la aplicación esta totalmente caída, pero tambien puedes ver un 502 *Bad Gateway* (entrada erronea), 503 *Service Unavailable* (Servicio no disponible), o un 504 *Gateway Timeout* (límite de tiempo de espera). En todos estos casos es tu responsabilidad reintentar la petición más tarde.


Clientes
--------

Ayudanos
--------
Por favor no dudes en ponerte en contacto con nosotros si crees que puedes hacer una mejor API. Si crees que deberíamos soportar una nueva funcionalidad o si has encontrado un bug, usa GitHub issues. Haz un fork de esta documentación y mandanos tus *pulls* con las mejoras.

Para hablar con nosotros o con otros desarrolladores sobre la API, suscribete a nuestra [**lista de correo**](https://groups.google.com/forum/#!forum/appnima).


API REST
========
Autentificación
---------------
Este módulo recoge toda la funcionalidad para obtener el token para un usuario a través del sistema denominado Oauth de 2 pasos, la ruta del recurso es:

    http://appnima.com/{RECURSO}

A continuación se explica el proceso de obtención de token a través de este sistema.


#### Paso 1: GET /oauth/authorize
Se redirigirá al usuario a una url previamente mencionada y a este recurso, pasando como parámetros:
```json
`response_type` : El tipo de solicitud, deberá ser "CODE"
`client_id`     : El identificador público de tu aplicación
`scope`         : Los permisos que tendrá el token
`redirect_uri`  : La url de redirección. Será la misma que diste de alta en tu aplicación.
`state`         : Variable de estado que acompañará a la respuesta para que se pueda identificar la operación.
```

La url sería algo similar a esto:

    http://api.appnima.com/oauth2/authorize?scope=profile,push&response_type=code&client_id=519b84f0c1881dc1b3000002&redirect_uri=http://myapp.com&state=user48

El usuario verá una página en la que se le presentan los datos de la aplicación y los permisos que requiere, si lo rechaza o algo va mal, se redirigirá al usuario a la página con un campo `error` que especifica el error.

Si ha ido todo bien se redirigirá a la página con un campo `code` que especifica un código con el que continuar el proceso.

#### Paso 2: POST /oauth2/token
Una vez tengamos el `code` se solicitará el token. Para ello se llamará al recurso /oauth2/token a través de una petición POST con la cabecera basic con el id y el secret de la applicación codificados en base64 y los siguientes parámetros:
```json
    {
        grant_type: "code",
        code:       "h02h40g208ghop2envg2h9h2epne2eh092he0b2",
        client_id:  "532023h2308h2839h"
    }
```

Si ha ido todo bien retorna un `201 Created` junto con el objeto:
```json
    {
        token_type:      "bearer",
        refresh_token:   "n72c03ty202ugx2gu2u",
        access_token:    "eh024hg02g2onvev29"
    }
```


USER
----
Este módulo recoge toda la funcionalidad para incluir un usuario de tu proyecto dentro la plataforma App/nima. Para ello ten en cuenta que todas las peticiones que hagas tendrán que ir a:

    http://api.appnima.com/user/{RECURSO}

Recuerda que todas las peticiones que hagas a App/nima tienen que ir identificadas con tu `Appnima.key` o bien con el par de datos `client` y `secret`. Ahora veamos los recursos que puedes utilizar, para ello el primer parametro indica el tipo de petición (GET, POST, UPDATE, DELETE …) y el segundo parametro el nombre del recurso.


### Seguridad
#### POST /signup
Todos los usuarios de tu aplicación tienen que ser usuarios App/nima y por lo tanto lo primero que tendrás que hacer es registrarlos en la plataforma para así poder obtener su token. Para ello debes enviar los siguientes parametros:
```json
    {
        mail:       "javi@tapquo.com",
        password:   "USER_PASSWORD"
    }
```

Y opcionalmente también puedes enviar:
```json
    {
        ...
        username:   "soyjavi",
        name:       "Javi Jimenez",
        avatar:     "http://USER_AVATAR_URL"
    }
```

Si ha ido todo bien retorna un `201 Created` junto con el objeto:
```json
    {
        id:         "939349943434",
        mail:       "javi@tapquo.com",
        username:   "soyjavi",
        name:       "Javi Jimenez",
        avatar:     "http://USER_AVATAR_URL"
    }
```

#### POST /token
Una vez que tenemos un usuario registrado para tu aplicación ahora tienes que pedir el token Oauth 2 que a partir de ahora será la clave para hacer todas las peticiones a App/nima. Para ello debes enviar los siguientes parámetros con la cabecera "http authorization basic"(con el client_id:client_secret codificados en Base64):

```Authorization: basic client_id:client_secret```

```json
    {
        grant_type: "password",
        mail:       "javi@tapquo.com",
        password:   "USER_PASSWORD"
    }
```
Si ha ido todo bien retorna un `201 Created` junto con el objeto:
```json
    {
        token_type:      "bearer",
        refresh_token:   "n72c03ty202ugx2gu2u",
        access_token:    "eh024hg02g2onvev29"
    }
```

#### POST /login
Cada vez que quieras validar si el usuario tiene permisos para acceder a tu aplicación podrás usar este recurso. Únicamente necesita los parámetros:
```json
    {
        mail:       "javi@tapquo.com",
        password:   "USER_PASSWORD"
    }
```

En caso de que la validación haya sido correcta App/nima devolver un `200 Ok` con los datos del usuario


### Info
#### GET /info
Si necesitas obtener los datos del usuario debes utilizar este recurso y como estás utilizando el protocolo de autentificación OAuth 2 no es necesario que envies ningún parámetro. Solo deberás esperar a la respuesta `200 Ok` con los siguientes parámetros:
```json
    {
        _id:            28319319832
        mail:           "javi@tapquo.com",
        username:       "soyjavi",
        name:           "Javi Jimenez",
        avatar:         "http://USER_AVATAR_URL",
        bio:            "Founder & CTO at @tapquo",
        phone:          "PHONE_NUMBER",
        token:          "USER_TOKEN",
        refresh_token:  "REFRESH_TOKEN"
    }
```


#### PUT /info
Este recurso sirve para modificar los datos personales de un usuario dentro de tu aplicación, al igual que en el recurso **GET /user/info** no es necesario identificar al usuario por parámetro. Puedes enviar todos los parámetros que aparecen a continuación (aunque no es obligatorio enviarlos todos):
```json
    {
        mail:       "javi@tapquo.com",
        password:   "PASSWORD",
        username:   "soyjavi",
        name:       "Javi Jimenez",
        avatar:     "http://USER_AVATAR_URL",
        bio:        "Founder & CTO at @tapquo",
        phone:      "PHONE_NUMBER"
    }
```

En el caso de que haya ido todo bien se devolverá el código `200 OK` junto con el mismo objeto **GET /user**. En el caso de que el usuario no tenga permiso para modificar sus datos App/nima devolverá un `403 Forbidden`.


#### POST /avatar
Este recurso sirve para subir un avatar, para ello enviaremos los siguientes parámetros:
```json
    {
        avatar:       "dhsgaohgoiagangaogisah89t2h3ugb2g2b",    /* avatar data coded in base 64 */
    }
```

En el caso de que haya ido todo bien se devolverá el código `201 RESOURCE CREATED`.


### Terminal
#### POST /terminal
Este recurso sirve para que en el caso de que necesites registrar el terminal con el que se está accediendo a tu aplicación. Para ello junto con la petición tienes que enviar los parámetros:
```json
    {
        type:       "phone",    /* computer, tablet, phone, tv */
        os:         "ios",      /* windows, macos, linux, ios, android, blackberry, firefoxos, windowsphone, other */
        version:    "6.0"       /* ?.? */
    }
```

En el caso de que haya ido todo bien se devolverá el código `201 RESOURCE CREATED`.


#### GET /terminal
Si necesitas saber los terminales desde los que se ha accedido a tu aplicación simplemente tienes que utilizar este recurso y al igual que **GET /user/info** no hace falta que envies ningún parámetro. La respuesta si ha ido todo correctamente será un `200 Ok` junto con los parámetros:
```json
    [{
        _id:        "5719721071057"
        type:       "phone",
        os:         "ios",
        token:      "OHSAF9075HFQ",
        version:    "6.0"
    },
    {
        _id:        "57592807235"
        type:       "desktop",
        os:         "macos",
        token:      "89YWEOFOGH022GB",
        version:    "10.8"
    }]
```

#### PUT /terminal
Este recurso sirve para que en el caso de que necesites actualizar un terminal con el que se está accediendo a tu aplicación. Para ello junto con la petición tienes que enviar los parámetros:
```json
    {
        terminal:   "5719721071057"
        type:       "phone",    /* computer, tablet, phone, tv */
        os:         "ios",      /* windows, macos, linux, ios, android, blackberry, firefoxos, windowsphone, other */
        version:    "6.0"       /* ?.? */
    }
```

Si un usuario quiere darse de baja de tu aplicación tendrás que hacer uso de este recurso.


### Suscripciones
#### POST /subscription
Registra los e-mail de los usuarios que desean recibir información o invitación de tu aplicación. Solo necesitas pasar como parámetro la dirección e-mail:
```json
    {
        mail:       "javi@tapquo.com"
    }
```

Si el parámetro es correcto se recibe un `200 Ok` junto con el objeto:
```json
    {
        message:    'Request accepted.'
    }
```

### Soporte
#### POST /ticket
Utiliza este recurso como sistema de gestión de tickets para la resolución de las consultas e incidencias de tus usuarios. Envía como parámetro junto a la petición el texto de la consulta:
```json
    {
        question:   '[SUGGESTION] Bigger buttons'
    }
```


Network
-------
Este modulo recoge toda la funcionalidad para crear una red social dentro de tu aplicación; buscar usuarios, seguirlos (o no seguirlos, tu decides), listas de seguidores... Para ello ten en cuenta que todas las peticiones que hagas tendrán que ir a:

    http://api.appnima.com/network/{RECURSO}

Recuerda que todas las peticiones que hagas a App/nima tienen que ir identificadas con tu `Appnima.key` o bien con el par de datos `client` y `secret`. Ahora veamos los recursos que puedes utilizar, para ello el primer parámetro indica el tipo de petición (GET, POST, UPDATE, DELETE …) y el segundo parámetro el nombre del recurso.


### Relaciones
#### GET /search
Si necesitas buscar usuarios dentro de tu aplicación debes utilizar este recurso el cual únicamente recibe un único parámetro y con el cual App/nima hará todo el trabajo difícil buscando los mejores resultados:
```json
    {
        query:      "javi@tapquo.com"
    }
```

En el caso de que la respuesta haya sido satisfactoria se devolverá un `200 Ok` junto con una lista de usuarios coincidentes a esa búsqueda:
```json
    [{
        id:         120949303434,
        username:   "soyjavi",
        name:       "Javi",
        avatar:     "AVATAR_URL"
    },
    {
        id:         120949303433,
        username:   "cataflu",
        name:       "Catalina",
        avatar:     "AVATAR_URL"
    },
    {
        id:         120949303431,
        username:   "haas85",
        name:       "Iñigo",
        avatar:     "AVATAR_URL"
    }
    ]
```

#### POST /follow
Si necesitas seguir a un usuario utiliza este recurso junto con el parámetro:
```json
    {
        user:       23094392049024
    }
```

Devolverá un `200 Ok` junto con el objeto:
```json
    {
        status:     'ok'
    }
```


#### POST /unfollow
Al igual que el recurso **POST /follow** funciona de igual manera y únicamente tendrás que enviar el parámetro:
```json
    {
        user:       23094392049024
    }
```

Devolverá un `200 Ok` junto con el objeto:
```json
    {
        status:     'ok'
    }
```

#### GET /following
Funciona de igual manera que **POST /follow**  y únicamente tendrás que enviar como parámetro el usuario del que quieres conocer la lista de usuarios que esta siguiendo:
```json
    {
        user:       23094392049024
    }
```

Devolverá un `200 Ok` junto con lista de usuarios que sigue el usuario indicado:
```json
    [{
        id:         120949303434,
        username:   "soyjavi",
        name:       "Javi",
        avatar:     "AVATAR_URL"
    },
    {
        id:         120949303433,
        username:   "cataflu",
        name:       "Catalina",
        avatar:     "AVATAR_URL"
    },
    {
        id:         120949303431,
        username:   "haas85",
        name:       "Iñigo",
        avatar:     "AVATAR_URL"
    }
    ]
```

#### GET /followers
Funciona de igual manera que **GET /following**  y esta vez tendrás que enviar como parámetro el usuario del que quieres conocer la lista de usuarios que le están siguiendo:
```json
    {
        user:       23094392049024
    }
```

Devolverá un `200 Ok` junto con lista de usuarios que siguen al usuario indicado:
```json
    [{
        id:         120949303434,
        username:   "soyjavi",
        name:       "Javi",
        avatar:     "AVATAR_URL"
    },
    {
        id:         120949303433,
        username:   "cataflu",
        name:       "Catalina",
        avatar:     "AVATAR_URL"
    },
    {
        id:         120949303431,
        username:   "haas85",
        name:       "Iñigo",
        avatar:     "AVATAR_URL"
    }
    ]
```


### Estadísticas
#### GET /stats
Si quieres tener una visión general de un determinado usuario dentro de la red social de tu aplicación utiliza este recurso, que al igual que en los recursos anteriores solo necesita del id del usuario como parámetro:
```json
    {
        user:       23094392049024
    }
```

Devolverá un `200 Ok` junto con los totales de *followers* y *followings* que tiene el usuario indicado:
```json
    {
        following:  123,
        followers:  343
    }
```

#### GET /check
Este recurso sirve para saber la relación que tienes con un determinado usuario (tal vez te interese seguirlo o no) para ello tenemos que enviar el id del usuario a consultar de la siguiente manera:
```json
    {
        user:       23094392049024
    }
```

En el caso de que todo haya ido correctamente devolverá un `200 Ok` junto con el objeto que representa la relación que tiene el usuario consultado con el usuario de tu aplicación (TOKEN):
```json
    {
        following:  true,
        follower:   false
    }
```


Messenger
---------
Este módulo recoge toda la funcionalidad de mensajería: enviar e-mail, SMS y mensajes privados entre usuarios de tu misma aplicación.

Para ello ten en cuenta que todas las peticiones que hagas tendrán que ir a:

    http://api.appnima.com/messenger/{RECURSO}

Recuerda que todas las peticiones que hagas a App/nima tienen que ir identificadas con tu `Appnima.key` o bien con el par de datos `client` y `secret`. Ahora veamos los recursos que puedes utilizar, para ello el primer parámetro indica el tipo de petición (GET, POST, UPDATE, DELETE …) y el segundo parámetro el nombre del recurso.


### Mail
#### POST /mail
Con este recurso los usuarios de tu aplicación podrán enviar e-mails. Para ello debes pasar los siguientes parámentros (el campo subject es opcional):
```json
    {
        user:       23094392049024,
        subject:    "Appnima.com",
        message:    "Welcome to appnima.messenger [MAIL]"
    }
```

Si todo ha salido bien, devolverá un `201 Created` junto con el objeto:
```json
    {
        message:    'E-mail sent successfully.'
    }
```

### SMS
#### POST /sms
Con este recurso tu aplicación podrá enviar mensajes de texto a los dispositivos móviles registrados de tus usuarios. Tan solo debes enviar junto con la petición los parámetros:
```json
    {
        user:       23094392049024,
        message:    "Welcome to appnima.messenger [SMS]"
    }
```

En el caso de que la respuesta haya sido satisfactoria se devolverá un `201 Created` junto con el objeto:
```json
    {
        message:    'SMS sent successfully.'
    }
```

### Message
#### POST /message
Si lo necesitas, Appnima te provee de un sistema de mensajería interno entre los usuarios de tu aplicación. Los parámetros que necesita la petición son:
```json
    {
        user:       23094392049024,
        subject:    "Appnima.com",
        message:    "Welcome to appnima.messenger [MESSAGE]"
    }
```

El campo subject es opcional y si la petición ha salido bien se devolverá un `201 Created` junto con el objeto:
```json
    {
        message:    'Message sent successfully.'
    }
```


#### GET /message/outbox
Los usuarios de tu aplicación pueden recuperar los mensajes enviados. Para ello basta con llamar al recurso pasando como parámetro outbox.
```json
    {
        context:    outbox
    }
```

Si todo ha salido bien, devolverá un `200 Ok` junto con lista de mensajes de la bandeja indicada:
```json
    {
        _id:            120949303434,
        from:           120949303434,
        to:             120949303433,
        application     220949303432
        subject         "Appnima.com",
        body            "Welcome to appnima.messenger [MESSAGE]",
        state           SENT
    }
```

#### GET /message/inbox
Los usuarios de tu aplicación pueden recuperar los mensajes recibidos. Para ello basta con llamar al recurso pasando como parámetro inbox.

```json
    {
        context:    inbox
    }
```

Si todo ha salido bien, devolverá un `200 Ok` junto con lista de mensajes de la bandeja indicada:

```json
    {
        _id:            120949303434,
        from:           120949303434,
        to:             120949303433,
        application     220949303432
        subject         "Appnima.com",
        body            "Welcome to appnima.messenger [MESSAGE]",
        state           READ
    }
```

#### PUT /message
Para que se refleje que el usuario ha leído un mensaje o que desea eliminarlo de su sistema, basta con enviar en la petición los siguientes parámetros:
```json
    {
        message:    23094392049024,
        state:      "READ" /*READ o DELETED*/
    }
```

Si todo ha salido bien, devolverá un `200 Ok`junto con el mensaje de confirmación:
```json
    {
        message:            "Resource READ."
    }
```


Location
--------
Este módulo te permite obtener toda la funcionalidad respecto a la geolocalización de usuarios y lugares. Mediante latitud y longitud obtén los sitios en un radio determinado así como los detalles de un lugar concreto. Si tu aplicación lo requiere, también puedes registrar un sitio mediante checkins y además ofrecer a tus usuarios información sobre amigos cercanos.

Para ello ten en cuenta que todas las peticiones que hagas tendrán que ir a:

    http://api.appnima.com/location/{RECURSO}

Recuerda que todas las peticiones que hagas a App/nima tienen que ir identificadas con tu `Appnima.key` o bien con el par de datos `client` y `secret`. Ahora veamos los recursos que puedes utilizar: el primer parámetro indica el tipo de petición (GET, POST, UPDATE, DELETE …) y el segundo el nombre del recurso.

### Places
#### GET /places
Con este recurso puedes obtienes una lista de lugares alrededor de un punto en un radio determinado. Envía como parámetros la latitud, longitud y opcionalmente el radio (en metros):
```json
    {
        latitude:       "-33.9250334",
        longitude:      "18.423883499999988",
        radio:          "500"
    }
```
En el caso de que haya respuesta, se devuelve un `200 Ok` junto con una lista de lugares e información relacionada:
```json
    [{
        id:             120949303434,
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m",
        location:
            0:  -33.9242692,
            1:  18.4187029
        place_name:     "Cape Town City Centre",
        vicinity:       "Cape Town City Centre"
    },
    {
        id:             120949303435,
        created_at:     "2013-05-25T21:08:12.087Z",
        location:
            0:  -33.9249,
            1:  18.4241
        place_name:     "Wang Thai - V&A Waterfront",
        vicinity:       "Queen Victoria Street, Cape Town"
    }
    ]
```

#### GET /place
Utiliza este recurso para obtener información detallada de un sitio en concreto. Envía junto con la petición los parámetros id y reference:
```json
    {
        id:             "120949303434",
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m"
    }
```

Si la consulta ha obtenido respuesta se devuelve un `200 Ok` junto con el objeto:
```json
    {
        id:             120949303434,
        address:        "Elorduigoitia Kalea",
        country:        "ES"
        created_at:     "2013-06-03T10:02:29.951Z"
        locality:       "Mungia"
        name:           "Cruz Roja Española"
        phone:          "+34 946 74 21 51"
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m",
        created_at:     "2013-05-25T21:08:12.087Z",
        position:
            0: 43.354585
            1: -2.846662
        postal_code:    "48100"
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m",
        website:        "http://www.cruzroja.es/"
    }
```

#### POST /place
Utiliza este recurso para dar de alta un sitio. Envía junto con la petición los siguientes parámetros:
```json
    {
        name: "San Mames",
        address: "Felipe Serrate, s/n",
        locality: "Bilbao",
        postal_code: "48013",
        country: "ES",
        latitude: " ﻿43.26",
        longitude: "-2.94"
    }
```
Opcionalmente se pueden añadir los siguientes parámetros:
```json
    {
        mail: "hello@sanmames.com",
        phone: "944411445",
        website: "www.sanmames.com"
    }
```

Si la consulta ha obtenido respuesta se devuelve un `201 Created`.


### Checkins
#### POST /checkin
Los usuarios de tu aplicación pueden registrar visitas a sitios concretos. Para ello utiliza este recurso pasando como parámetros:
```json
    {
        id:             "120949303434",
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m"
    }
```

Si todo ha salido bien, se devuelve un `200 Ok` junto con el objeto:
```json
    {
        id:             120949303434,
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m",
        created_at:     "2013-05-26T20:48:05.786Z",
        latitude:       -33.9249,
        longitude:      18.4241,
        name:           "Wang Thai - V&A Waterfront",
        phone:          "+27 21 421 8702",
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m",
        vicinity:       "Queen Victoria Street, Cape Town"
    }
```

#### GET /checkin
Obtén la lista de sitios guardados por tus usuarios con este recurso. Para ello únicamente tienes que enviar el id del usuario en la petición:
```json
    {
        id:             "120949303434"
    }
```

Si ha salido todo bien, obtienes un `200 Ok` junto con el objeto:
```json
    {
        id:             120949303434,
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m",
        created_at:     "2013-05-26T20:48:05.786Z",
        latitude:       -33.9249,
        longitude:      18.4241,
        name:           "Wang Thai - V&A Waterfront",
        phone:          "+27 21 421 8702",
        reference:      "CqQBlwAAAEXdx350jL2InIRtksTkbZJ-m",
        vicinity:       "Queen Victoria Street, Cape Town"
    }
```

### Search
#### GET /friends
Proporciona a tus usuarios información sobre amigos cercanos a un punto determinado. Para ello utiliza este recurso pasando los siguientes parámetros:
```json
    {
        latitude:       "-33.9250334",
        longitude:      "18.423883499999988",
        radio:          "500"
    }
```

Si todo ha salido bien, se recibe un `200 Ok` junto con el objeto:
```json
    {
        id:         120949303431,
        username:   "haas85",
        name:       "Iñigo",
        avatar:     "AVATAR_URL"
    }
```

#### GET /people
De la misma forma que puedes mostrar a un usuarios qué amigos están a su alrededor, puedes por ejemplo proponer gente fuera de sus listas de amigos. La petición se realiza con los mismos parámetros:


```json
    {
        latitude:       "-33.9250334",
        longitude:      "18.423883499999988",
        radio:          "500"
    }
```


Socket
------
App/nima te permite trabajar con sockets donde puedes tener con salas de conversaciones. Para ello ten en cuenta que todas las peticiones que hagas tendrán que ir a:

    http://socket.appnima.com/{RECURSO}

Recuerda que todas las peticiones que hagas a App/nima tienen que ir identificadas con tu `Appnima.key` o bien con el par de datos `client` y `secret`. Ahora veamos los recursos que puedes utilizar, para ello el primer parametro indica el tipo de petición (GET, POST, UPDATE, DELETE …) y el segundo parametro el nombre del recurso.


### Comenzando
Para la comunicación en tiempo real, appnima utiliza socket.io para conectarse a las diferentes salas creadas. A continuación se exponen los siguientes métodos de interacción con ellas. Aunque recomendamos el uso de nuestra librería `Appnima.js` para esta labor.

#### open
Para crear una sala se llamará al método `open`. Esté recibirá los siguientes parametros:

* Token del usuario
* Contexto (que se usará para identificar la sala para futuras conexiones)
* Nombre de sala
* Tipo de sala
* Si queremos que la información de la sala sea persistente
* Array de los ids de los usuarios permitidos

Si todo está correcto la sala se creará y automáticamente el autor estará conectado a ella.

#### join
Un usuario puede conectarse a una sala siempre y cuando ya esté creada y tenga permiso de conexión a la misma. Para ello se llamará al método `join` y se le pasarán los siguientes parámetros:

* Token del usuario (obligatorio a no ser que sea sesión anónima)
* Contexto (identificador de la sala a la que se quiere conectar)
* Id de aplicación (en caso de sesión anónima se requiere de este campo)

#### leave
En caso de querer desconectarse de una sala se llamará al método `leave`.

#### sendmessage
Para enviar un mensaje a una sala basta con llamar al método `sendMessage` y pasarle como parámetro un objeto (el mensaje puede ser cualquier tipo de dato). Este mensaje llegará a todos los usuarios conectados a la sala, emisor incluido, se recibirá a través del listener `onMessage` y tendrá el siguiente formato:
```json
    {
        user:           {"usuario que envía el mensaje"},
        message:        {"mensaje enviado"},
        created_at:     "2013-05-23T12:01:02.736Z"
    }
```

#### sendbroadcast
Este método funciona igual que el método `sendMessage` con la única diferencia de que el emisor no recibirá el mensaje enviado.

#### allowusers
Se pueden añadir usuarios a la lista de admitidos a través del método `allowUsers`, para ello bastará con llamar a este método y pasarle un array con los ids de los usuarios a admitir.

#### disallowusers
Se pueden eliminar usuarios de la lista de admitidos a través del método `disallowUsers`, para ello bastará con llamar a este método y pasarle un array con los ids de los usuarios a eliminar de la lista de admitidos.


### Tipos y permisos
Existen distintos tipos de salas, cada una tiene unos permisos distintos dependiendo del usuario.

* Privadas: Solo se pueden conectar, leer y escribir usuarios que se encuentren en la lista de admitidos.
* Públicas: Todo el mundo se puede conectar y leer, pero solo el dueño puede publicar en ella.
* Inbox: Solo el dueño puede leer lo que se escribe, pero cualquier usuario puede ecribir.
* Aplicación: Esta sala existe en todas las aplicaciones, por lo que nadie tiene que crearla, todos pueden conectarse, leer y escribir en ella.


### Peticion HTTP
#### GET /rooms
Un usuario puede obtener el listado de salas de sockets en las que participa, para ello bastaría con llamar al recurso y se obtendría un mensaje `200 Ok` junto con un listado como el siguiente:
```json
    [{
        id:             "ba54",
        name:           "amigos",
        created_at:     "2013-05-23T12:01:02.736Z"
    },
    {
        id:             "asf9a76y2t3ub",
        name:           "appnima friends",
        created_at:     "2013-02-23T12:01:02.736Z"
    }
    ]
```


Push
----
Con este módulo puedes enviar notificaciones a los terminales registrados de los usuarios de tu aplicación.

    http://api.appnima.com/push{RECURSO}

Recuerda que todas las peticiones que hagas a App/nima tienen que ir identificadas con tu `Appnima.key` o bien con el par de datos `client` y `secret`. Ahora veamos los recursos que puedes utilizar, para ello el primer parametro indica el tipo de petición (GET, POST, UPDATE, DELETE …) y el segundo parametro el nombre del recurso.


#### POST /push
Envía notificaciones push mediante este recurso. Junto con la petición, envía los siguientes parámetros:
```json
    {
        user:       23094392049024,
        alert:      ""Texto a mostrar en la notificación",
        content":   {"title": "JSON con los campos necesarios", "text": "Hola App/nima!"}
    }
```
En caso de éxito se devolverá el código `200 Ok`.