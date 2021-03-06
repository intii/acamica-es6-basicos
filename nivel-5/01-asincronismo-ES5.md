# Desafíos actuales con código asíncrono

## Introducción: operaciones asíncronas
En el ambiente de un explorador web, se cuenta con un sólo thread de ejecución.
Esto significa que para mantener una interacción fluida con el usuario, este thread
no puede ser bloqueado. Con este propósito, y valiéndose de que en JavaScript las
funciones son objetos de primer orden, los exploradores exponen parte de su funcionalidad
a través de APIs asincrónicas, como XHR, User Event Listeners, entre otras.
## Callbacks - Enfoque tomado en ES5
Las operaciones asíncronas mencionadas anteriormente se manejan a través de un sistema
de eventos. Esto requiere el registro de una función que será invocada ante la llegada
de un mensaje esperado.
``` javascript
const el = document.querySelector('.btn');
el.addEventListener('click', function() {
  //handle click
}, false)
```
Ante la ocurrencia de un error durante la ejecución de la operación asíncrona, no es
posible tratarlo mediante el clásico `try/catch`, ya que la falla en cuestión está fuera
del alcance en el contexto del programa. La forma de solucionar este tipo de situaciones es
mediante el pasaje de una función extra, para manejar el caso de error.

``` javascript
getData(function(data){
  // show data
}, function(error){
  // handle error
});
```
Con este enfoque, se resuelve el problema de responder a operaciones de larga espera, sin
bloquear el thread principal, y manteniendo la aplicación interactiva.

### Combinación de callbacks

Sin embargo, a medida que la complejidad de la aplicación aumenta, esta forma de estructurar
el código comienza a presentar inconvenientes.
Un caso simple como el encadenamiento de operaciones, tomando el enfoque de callbacks, puede volverse
rápidamente complejo y poco legible.

``` javascript
getUser('user', function (user) {
     getFriendsForUser(user.name, function(friends) {
         getAvatar(friends[0], function(avatar) {
             showAvatar(avatar);
         });
     });
});
```
Otro caso de interés es cuando se requiere realizar cierta acción, cuando alguna operación
asíncrona dentro de un conjunto haya finalizado. Para esa situación, es necesario añadir
lógica adicional para poder aseverar qué fue ejecutado y qué no.

```javascript
function showUserAndFriend() {  
   let finishedSoFar = 0;

   function somethingFinished() {
      if (++finishedSoFar === 2) {
          hideSpinner();
      }
   }

   getUser('user', function(user) {
      showUser(user);
      somethingFinished();
   });

   getFriends('user', function(friends) {
     showFriend(friends[0]);
     somethingFinished();
   });
}

```
### Manejo de errores
El manejo de errores es una parte muy importante dentro de las operaciones asíncronas,
siendo los requests a través de la red los casos más comunes. En el ejemplo anterior
es posible notar que la combinación de callbacks trae complicaciones, y en el caso de
la consideración de errores, se hace aún más explícito.

```javascript
getUser('user', function(err, user) {
  if (err) {
    ui.showError(err);
  } else {
    getFriendsForUser(user, function(err, friends) {
      if (err) {
        ui.showError(err);
      } else {
        ui.showFriends(friends);
      }
    });
  }
});
```
### Desafíos

1- Crear una función `getData` que simule una llamada asincrónica, y acepte un callback para manejar la respuesta. Testear su correctitud invocándola y mostrando la respuesta por consola.

#### Solución
```javascript
function getData(callback) {
  const randomTime = (Math.floor(Math.random() * 5) + 1) * 100;
  setTimeout(function() {
    callback(randomTime);
}, randomTime);
}

getData(function(data) {
  console.log(data);
});

//10
```

2- Invocar a `getData` una segunda vez, luego de haber recibido la respuesta de haber invocado a `getData` una vez. Mostrar la respuesta por consola.

#### Solución
```javascript
getData(function(data) {
  getData(function(data2){
    console.log(data + data2);
  });
});

//20
```

3- Llamar a `getData` dos veces paralelamente. Mostrar por consola 'Se recibió una respuesta' cuando se haya recibido la respuesta de una llamada.

#### Solución
```javascript
const data1
const data2;

getData(function(data) {
  data1 = data;
  if (!data2) {
    console.log('Se recibió una respuesta');
  }
});

getData(function(data) {
  data2 = data;
  if (!data1) {
    console.log('Se recibió una respuesta');
  }
});
```
