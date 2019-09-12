# clickpayredeban.js
ClickPayRedebanJS es una biblioteca que permite a los desarrolladores conectar fácilmente con el API de TARJETAS DE CRÉDITO de Click Pay Redeban.

[Revise el ejemplo funcionando >](https://developers.clickpayredeban.com/docs/payments/#javascript)

## Instalación

Usted necesitará incluir jQuery y los archivos `payment_[version].min.js` y `payment_[version].min.css` dentro de su página web especificando "UTF-8" como charset.

```html
<script src="https://code.jquery.com/jquery-1.11.3.min.js" charset="UTF-8"></script>

<link href="https://cdn.clickpayredeban.com/ccapi/sdk/payment_2.0.0.min.css" rel="stylesheet" type="text/css" />
<script src="https://cdn.clickpayredeban.com/ccapi/sdk/payment_2.0.0.min.js" charset="UTF-8"></script>
```

## Uso

Para revisar ejemplos funcionales utilizando ClickPayRedebanJS, revise los [ejemplos](https://github.com/clickpayredeban/clickpayredeban.js/tree/master/examples) dentro de este proyecto.


### Utilizando el Form de ClickPayRedeban

Cualquier elemento con la clase `payment-form` será automáticamente converitdo en una entrada de tarjeta de crédito básica con fecha de vencimiento y el check de CVC.

La manera más fácil de comenzar con la ClickPayRedebanForm es insertando el siguiente pedazo de código:

```html
<div class="payment-form" id="my-card" data-capture-name="true"></div>
```
Para obtener el objeto `Card` de la `PaymentForm`, pregunte al form por su tarjeta.


```javascript
let myCard = $('#my-card');
let cardToSave = myCard.PaymentForm('card');
if(cardToSave == null){
  alert("Invalid Card Data");
}
```

Si la tarjeta (`Card`) regresada es null, el estado de error mostrará los campos que necesitan ser arreglados.

Una vez que tenga un objeto no null de la tarjeta (`Card`) del widget, usted podrá llamar [addCard](#addcard).


### Biblioteca Init

Usted deberá inicializar la biblioteca.

```javascript
/**
  * Init library
  *
  * @param env_mode `prod`, `stg`, `local` para cambiar ambiente. Por defecto es `stg`
  * @param client_app_code proporcionado por Click Pay Redeban.
  * @param client_app_key proporcionado por Click Pay Redeban.
  */
Payment.init('stg', 'CLIENT_APP_CODE', 'CLIENT_APP_KEY');
```

### addCard Añadir una Tarjeta

La función addCard convierte los datos confidenciales de una tarjeta, en un token uso que puede pasar de forma segura a su servidor, para realizar el cobro al usuario.


```javascript
/* Add Card converts sensitive card data to a single-use token which you can safely pass to your server to charge the user.
 *
 * @param uid User identifier. This is the identifier you use inside your application; you will receive it in notifications.
 * @param email Email of the user initiating the purchase. Format: Valid e-mail format.
 * @param card the Card used to create this payment token
 * @param success_callback a callback to receive the token
 * @param failure_callback a callback to receive an error
 */
Payment.addCard(uid, email, cardToSave, successHandler, errorHandler);

let successHandler = function(cardResponse) {
  console.log(cardResponse.card);
  if(cardResponse.card.status === 'valid'){
    $('#messages').html('Card Successfully Added<br>'+
                  'status: ' + cardResponse.card.status + '<br>' +
                  "Card Token: " + cardResponse.card.token + "<br>" +
                  "transaction_reference: " + cardResponse.card.transaction_reference
                );    
  }else if(cardResponse.card.status === 'review'){
    $('#messages').html('Card Under Review<br>'+
                  'status: ' + cardResponse.card.status + '<br>' +
                  "Card Token: " + cardResponse.card.token + "<br>" +
                  "transaction_reference: " + cardResponse.card.transaction_reference
                ); 
  }else if(cardResponse.card.status === 'pending'){
    $('#messages').html('Card Pending To Approve<br>'+
                  'status: ' + cardResponse.card.status + '<br>' +
                  "Card Token: " + cardResponse.card.token + "<br>" +
                  "transaction_reference: " + cardResponse.card.transaction_reference
                ); 
  }else{
    $('#messages').html('Error<br>'+
                  'status: ' + cardResponse.card.status + '<br>' +
                  "message Token: " + cardResponse.card.message + "<br>"
                ); 
  }
  submitButton.removeAttr("disabled");
  submitButton.text(submitInitialText);
};

let errorHandler = function(err) {    
  console.log(err.error);
  $('#messages').html(err.error.type);    
  submitButton.removeAttr("disabled");
  submitButton.text(submitInitialText);
};
```

El tercer argumento para el addCard es un objeto Card que contiene los campos requeridos para realizar la tokenización.


### getSessionId

El Session ID es un parámetro que ClickPay Redeban utiliza para fines del antifraude.
Llame este método si usted desea recolectar la información del dispositivo del usuario.

```javascript
let session_id = Payment.getSessionId();
```

Una vez que tenga el Session ID, usted puede pasarlo a su servidor para realizar el cargo al usuario.


## PaymentForm Referencia Completa

### Inserción Manual

Si desea alterar manualmente los campos utilizados por PaymentForm para añadir clases adicionales, el placeholder o id. Puede rellenar previamente los campos del formulario como se muestra a continuación.

Esto podría ser útil en caso de que desee procesar el formulario en otro idioma (de forma predeterminada, el formulario se representa en español), o para hacer referencia a alguna entrada por nombre o id.

Por ejemplo si desea mostrar el formulario en Inglés y añadir una clase personalizada para el _card_number_
```html
<div class="payment-form">
  <input class="card-number my-custom-class" name="card-number" placeholder="Card number">
  <input class="name" id="the-card-name-id" placeholder="Card Holders Name">
  <input class="expiry-month" name="expiry-month">
  <input class="expiry-year" name="expiry-year">
  <input class="cvc" name="cvc">
</div>
```


### Seleccionar Campos
Usted puede determinar los campos que mostrará en el formulario.

| Field                             | Description                                                      |
| :-------------------------------- | :--------------------------------------------------------------- |
| data-capture-name                 | Input para nombre del Tarjetahabiente, requerido para tokenizar  |
| data-capture-email                | Input para email del usuario                                     |
| data-capture-cellphone            | Input para teléfono celular del usuario                          |
| data-icon-colour                  | Iconos de color                                                  |
| data-use-dropdowns                | Utilice una lista desplegable para establecer la fecha de expiración de la tarjeta   |
| data-exclusive-types              | Define los tipos de tarjetas permitidos                          |
| data-invalid-card-type-message    | Define un mensaje personalizado para mostrar cuando se registre una tarjeta inválida  |

El campo 'data-use-dropdowns' puede resolver el problema que se presenta con la mascara de expiración en dispositivos móviles antiguos.

Integre en el form simplemente, como se muestra a continuación:
```html
<div class="payment-form"
id="my-card"
data-capture-name="true"
data-capture-email="true"
data-capture-cellphone="true"
data-icon-colour="#569B29"
data-use-dropdowns="true">
```

### Tipos de tarjetas específicos
Si desea especificar los tipos de tarjetas permitidos en el formulario, como Exito o Alkosto. Puede configurarlo como en el siguiente ejemplo:
Cuando una tarjeta de un tipo no permitido es capturado, el formulario se reinicia, bloqueando las entradas y mostrando un mensaje
*Tipo de tarjeta invalida para está operación.*


```html
<div class="payment-form"
id="my-card"
data-capture-name="true"
data-exclusive-types="ex,ak"
data-invalid-card-type-message="Tarjeta invalida. Por favor ingresa una tarjeta Exito / Alkosto."
>
```
Revise todos los [tipos de tarjetas](https://developers.clickpayredeban.com/api/#metodos-de-pago-tarjetas-marcas-de-tarjetas) permitidos por Click Pay Redeban.


### Leyendo los Valores

PaymentForm proporciona funcionalidad que le permite leer los valores del campo de formulario directamente con JavaScript.

Genere un elemento de PaymentForm y asigne un id único (en este ejemplo `my-card`)

```html
<div class="payment-form" id="my-card" data-capture-name="true"></div>
```
El siguiente javascript muestra cómo leer cada valor del formulario en variables locales.


```javascript
let myCard = $('#my-card');

let cardType = myCard.PaymentForm('cardType');
let name = myCard.PaymentForm('name');
let expiryMonth = myCard.PaymentForm('expiryMonth');
let expiryYear = myCard.PaymentForm('expiryYear');
let fiscalNumber = myCard.PaymentForm('fiscalNumber');
```


### Funciones

Para llamar a una función en un elemento PaymentForm, siga el patrón a continuación.
Remplace el texto 'function' con el nombre de la función que desea llamar.

```javascript
$('#my-card').PaymentForm('function')
```

Las funciones disponibles se enumeran a continuación:s
| Function          | Description                                    |
| :---------------- | :--------------------------------------------- |
| card              | Obtiene la tarjeta del objeto                  |
| cardType          | Obtiene el tipo de tarjeta que se capturó      |
| name              | Obtiene el nombre capturado                    |
| expiryMonth       | Obtiene el mes de expiración de la tarjeta     |
| expiryYear        | Obtiene el año de expiración de la tarjeta     |
| fiscalNumber      | Obtiene el número fiscal del usuario / cédula  |



#### Función CardType

La función `cardType` devolverá una cadena según el número de tarjeta ingresado. Si no se puede determinar el tipo de tarjeta, se le dará una cadena vacía.

[Marcas permitidas](https://developers.clickpayredeban.com/api/#metodos-de-pago-tarjetas-marcas-de-tarjetas)

