# Weikap.play

Documentación de la API, v1.0

## Descripción

La API de Weikap.play permite la comunicación entre una aplicación o juego y la plataforma, para poder obtener y almacenar datos y así ofrecer una experiencia personalizada a cada usuario con el máximo nivel de flexibilidad para el desarrollo de las aplicaciones asociadas al sistema.

Se utiliza una convención de **versionamiento semántico** para explicitar las modificaciones a la API y así mantener la compatibilidad con las aplicaciones; es decir, todos los cambios introducidos en la versión 1 de la API se realizarán sin romper la compatibilidad.

## Autenticación

La API de Weikap.play utiliza un método de autenticación basado en OAuth2. En particular el tipo de **flujo implícito** (_implicit flow_).

Puede obtener un panorama general del modo en que funciona este tipo de flujo de autenticación en [An introduction to OAuth2](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2).

### Flujo de autenticación

Para comprender este flujo de autenticación, es necesario definir los siguientes términos:

* **Resource owner** o _dueño del recurso_, se refiere específicamente al **usuario** que posee datos personales en la plataforma; es decir, la niña o el niño que accede a un juego.
* **Resource server** corresponde a la API, que entrega los datos solicitados por el juego.
* **Client** (_cliente_) corresponde al juego o aplicación

Cada aplicación recibe un identificador o **client_id** que la individualiza y representa en la plataforma.

La aplicación debe redirigir al usuario al servidor de autenticación para su autorización.

La aplicación **debe enviar** los siguientes parámetros:

* `response_type` con el valor `token`
* `client_id` con el identificador de la aplicación
* `scope` con una lista separada por espacios con los permisos solicitados; normalmente `play`

Adicionalmente, puede enviar un parámetro `state` con un valor arbitrario. Este parámetro será copiado en la respuesta del servidor de autenticación y se utiliza como protección ante ataques CSRF. Su comprobación queda delegada al mismo cliente.

Por ejemplo:

`https://play.weikap.cl/oauth2/authorize/?response_type=token&client_id={XXXXX}&scope=play&state={RANDOM}`

Si la petición de autorización es aprobada, el usuario es redirigido a la URL registrada por la aplicación con los siguientes parámetros como parte de la URL:

* `token_type` con el valor `Bearer`
* `expires_in` con un entero que representa el tiempo de expiración del access token (p.ej: `21599` [segundos])
* `access_token` un JSON Web Token firmado con la llave privada del servidor de autorización
* `state` con el parámetro `state` enviado en la petición original; para comparar con el valor generado por la aplicación

En este flujo de autenticación, estos valores se indican normalmente como parte del _fragmento_ de la URL (`https://dominio.com/ruta/#fragmento`)

### Cómo enviar peticiones autenticadas

Luego de que el usuario haya autorizado a la aplicación, ésta puede realizar peticiones autenticadas utilizando el `access_token` recibido.

```js
var extractToken = function( hash ) {
  var match = hash.match(/access_token=([^&]+)/);
  return !!match && match[1];
};
var accessToken = extractToken( location.hash );
```

Para esto, las peticiones que la aplicación envía a la API deben adjuntar la cabecera de `Authentication: OAuth {ACCESS_TOKEN}`.

Por ejemplo, si la aplicación utiliza la librería jQuery:

```javascript
var request = $.ajax({
    url: baseUrl + '/wp-json/play/v1/user/me',
    beforeSend: function( xhr ){
        xhr.setRequestHeader('Authentication', 'OAuth '+ accessToken)
    }
});
```

O utilizando javascript puro:

```javascript
var request = new XMLHttpRequest();
request.setRequestHeader('Authentication', 'OAuth '+ accessToken);
request.open('GET', baseUrl + '/wp-json/play/v1/user/me' );
request.onload = function(){
    console.log('loaded');
};
request.onerror = function(){
    console.log(error);
};
request.send();
```

### Timeout

La aplicación debe contemplar y asegurar que el usuario no reciba errores por expiración del `access_token`, para lo cual debe tomar en consideración el límite de validez de éste indicado en el parámetro `timeout` de la respuesta de autenticación.

Por ejemplo:

```javascript
var isTimedOut = document.setTimeout(function(){
    alert('Pronto va a expirar tu sesión');
}, 3600 * 4 )
```

## Referencia del API

### Usuarios

### Juegos
