# clickpayredeban.js
ClickPayRedebanJS es una biblioteca que permite a los desarrolladores conectar fácilmente con el API de TARJETAS DE CRÉDITO de Click Pay Redeban.

[Revise el ejemplo funcionando >](https://developers.clickpayredeban.com/docs/payments/#javascript)

## Instalación

Usted necesitará incluir jQuery y los archivos `paymentez.min.js` y `paymentez.min.css` dentro de su página web especificando "UTF-8" como charset.

Para ambiente de pruebas (staging):


```html
<script src="https://code.jquery.com/jquery-1.11.3.min.js" charset="UTF-8"></script>

<link href="https://cdn.paymentez.com/js/ccapi/stg/paymentez.min.css" rel="stylesheet" type="text/css" />
<script src="https://cdn.paymentez.com/js/ccapi/stg/paymentez.min.js" charset="UTF-8"></script>
```

Para ambienete productivo:


```html
<script src="https://code.jquery.com/jquery-1.11.3.min.js" charset="UTF-8"></script>

<link href="https://cdn.paymentez.com/js/1.0.1/paymentez.min.css" rel="stylesheet" type="text/css" />
<script src="https://cdn.paymentez.com/js/1.0.1/paymentez.min.js" charset="UTF-8"></script>
```


## Uso

Para revisar ejemplos funcionales utilizando ClickPayRedebanJS, revise los [ejemplos](https://github.com/clickpayredeban/clickpayredeban.js/tree/master/examples) dentro de este proyecto.


### Utilizando el Form de ClickPayRedeban

Cualquier elemento con la clase `paymentez-form` será automáticamente converitdo en una entrada de tarjeta de crédito básica con fecha de vencimiento y el check de CVC.

La manera más fácil de comenzar con la ClickPayRedebanForm es insertando el siguiente pedazo de código:

```html
<div class="paymentez-form" id="my-card" data-capture-name="true"></div>
```
Para obtener el objeto `Card` de la `PaymentezForm`, pregunte al form por su tarjeta.


```javascript
var myCard = $('#my-card');
var cardToSave = myCard.PaymentezForm('card');
if(cardToSave == null){
  alert("Invalid Card Data");
}
```

Si la tarjeta (`Card`) regresada es null, el estado de error mostrará los campos que necesitan ser arreglados.

Una vez que tenga un objeto no null de la tarjeta (`Card`) del widget, usted podrá llamar [addCard](#addcard).


### Biblioteca Init

Usted deberá inicializar la bibliteca.

```javascript
/**
  * Init library
  *
  * @param env_mode `prod`, `stg`, `local` para cambiar ambiente. Por defecto es `stg`
  * @param client_app_code proporcionado por Click Pay Redeban.
  * @param client_app_key proporcionado por Click Pay Redeban.
  */
Paymentez.init('stg', 'CLIENT_APP_CODE', 'CLIENT_APP_KEY');
```

### addCard Añadir una Tarjeta

addCard convierte los datos confidenciales de una tarjeta, en un token de un solo uso que puede pasar de forma segura a su servidor, para realizar el cobro al usuario.


```javascript
/* Add Card converts sensitive card data to a single-use token which you can safely pass to your server to charge the user.
 *
 * @param uid User identifier. This is the identifier you use inside your application; you will receive it in notifications.
 * @param email Email of the user initiating the purchase. Format: Valid e-mail format.
 * @param card the Card used to create this payment token
 * @param success_callback a callback to receive the token
 * @param failure_callback a callback to receive an error
 */
Paymentez.addCard(uid, email, cardToSave, successHandler, errorHandler);

var successHandler = function(cardResponse) {
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

var errorHandler = function(err) {    
  console.log(err.error);
  $('#messages').html(err.error.type);    
  submitButton.removeAttr("disabled");
  submitButton.text(submitInitialText);
};
```

El tercer argumento para el addCard es un objeto Card. Una Card (tarjeta) contiene los siguientes campos:

+ number: número de la tarjeta como cadena sin ningún separador, p.e.  '4242424242424242'.
+ holder_name: nombre del tarjetahabiente.
+ expiry_month: entero que representa el mes de expiración de la tarjeta, p.e. 12.
+ expiry_year: entero que representa el año de expixración de la tarjeta, p.e. 2020.
+ cvc: códito de seguridad de la tarjeta, en cadena. p.e. '123'.


### getSessionId

El Session ID es un parámetro que ClickPay Redeban utiliza para fines del antifraude.
Llame este método si usted desea recolectar la información del dispositivo del usuario.

```javascript
var session_id = Paymentez.getSessionId();
```

Una vez que tenga el Session ID, usted puede pasarlo a su servidor para realizar el cargo al usuario.


## PaymentezForm Referencia Completa

### Inserción Manual

Si desea alterar manualmente los campos utilizados por PaymentezForm para añadir clases adicionales o para establecer el marcador de posición del campo de entrada, nombre o id. Puede rellenar previamente los campos del formulario como se muestra a continuación.

Esto podría ser útil en caso de que desee procesar el formulario en otro idioma (de forma predeterminada, el formulario se representa en español), o para hacer referencia a alguna entrada por nombre o id.

Por ejemplo si desea mostrar el formulario en Inglés y añadir una clase customizada para el _card_number_
```html
<div class="paymentez-form">
  <input class="card-number my-custom-class" name="card-number" placeholder="Card number">
  <input class="name" id="the-card-name-id" placeholder="Card Holders Name">
  <input class="expiry-month" name="expiry-month">
  <input class="expiry-year" name="expiry-year">
  <input class="cvc" name="cvc">
</div>
```


### Seleccionar Campos
Usted puede determinar los campos que mostrará en el formulario.

| Field                             | Description                                                |
| :-------------------------------- | :--------------------------------------------------------- |
| data-capture-name                 | Nombre del Tarjetahabiente                                 |
| data-capture-email                | Email del usuario                                          |
| data-capture-cellphone            | Teléfono celular del usuario                               |
| data-icon-colour                  | Iconos de color                                            |
| data-use-dropdowns                | Utilice una lista desplegable para establecer la fecha de expiración de la tarjeta   |
| data-exclusive-types              | Define los tipos de tarjetas permitidos                    |
| data-invalid-card-type-message    | Define un mensaje customizado para mostrar cuando sea una tarjeta inválida  |

El campo 'data-use-dropdowns' puede resolver el problema que se presenta con la mascara de expiración en dispositivos móviles antiguos.

Integre en el form simplemente, como sigue:
```html
<div class="paymentez-form"
id="my-card"
data-capture-name="true"
data-capture-email="true"
data-capture-cellphone="true"
data-icon-colour="#569B29"
data-use-dropdowns="true">
```

### Tipos de tarjetas específicos
Si desea especificar los tipos de tarjetas permitidos en el formulario,  como Exito o Alkosto. Puede configurarlo como en el siguiente ejemplo:
Cuando una tarjeta de un tipo no permitido es capturado, la forma se resetea, bloqueando las entradas y mostrando un mensaje
*Tipo de tarjeta invalida para está operación.*


```html
<div class="paymentez-form"
id="my-card"
data-capture-name="true"
data-exclusive-types="ex,ak"
data-invalid-card-type-message="Tarjeta invalida. Por favor ingresa una tarjeta Exito / Alkosto."
>
```
Revise todos los [tipos de tarjetas](https://developers.clickpayredeban.com/api/#metodos-de-pago-tarjetas-marcas-de-tarjetas) permitidos por Click Pay Redeban.


### Leyendo los Valores

PaymentezForm proporciona funcionalidad que le permite leer los valores del campo de formulario directamente con JavaScript. Esto puede ser útil si desea enviar los valores a través de Ajax.

Genera un elemento de PaymentezForm y dele un id único (en este ejemplo `my-card`)

```html
<div class="paymentez-form" id="my-card" data-capture-name="true"></div>
```
El siguiente javascript muestra cómo leer cada valor del formulario en variables locales.


```javascript
var myCard = $('#my-card');

var cardNumber = myCard.PaymentezForm('cardNumber');
var cardType = myCard.PaymentezForm('cardType');
var name = myCard.PaymentezForm('name');
var expiryMonth = myCard.PaymentezForm('expiryMonth');
var expiryYear = myCard.PaymentezForm('expiryYear');
var fiscalNumber = myCard.PaymentezForm('fiscalNumber');
var validationOption = myCard.PaymentezForm('validationOption');
```


### Funciones

Para llamar a una función en un elemento PaymentezForm, siga el patrón a continuación.
Remplace el texto 'function' con el nombre de la función que desea llamar.

```javascript
$('#my-card').PaymentezForm('function')
```

Las funciones disponibles se enumeran a continuación:
| Function          | Description                                    |
| :---------------- | :--------------------------------------------- |
| card              | Obtiene la tarjeta del objeto                  |
| cardType          | Obtiene el tipo de tarjeta que se capturo      |
| name              | Obtiene el nombre capturado                    |
| expiryMonth       | Obtiene el mes de expiración de la tarjeta     |
| expiryYear        | Obtiene el año de expiración de la tarjeta     |
| fiscalNumber      | Obtiene el número fiscal del usuario           |
| validationOption  | Obtiene la opción de validación                |



#### Función CardType

La función `cardType` devolverá una de las siguientes cadenas según el número de tarjeta ingresado.
Si no se puede determinar el tipo de tarjeta, se le dará una cadena vacía.


| Card Type              |
| :--------------------- |
| AMEX                   |
| Diners                 |
| Diners - Carte Blanche |
| Discover               |
| JCB                    |
| Mastercard             |
| Visa                   |
| Visa Electron          |
| Exito                  |



### Funciones Estáticas

Si solo desea realizar operaciones simples sin el formulario PaymentezForm, hay una serie de funciones estáticas proporcionadas
por la biblioteca PaymentezForm que están disponibles.

#### Tipo de Tarjeta de un Número de Tarjeta
```javascript
var cardNumber = '4242 4242 4242 4242'; // Los espacios no son importantes
var cardType = PaymentezForm.cardTypeFromNumber(cardNumber);
```

#### Limpieza y Enmascarado 
```javascript
// var formatMask = 'XXXX XXXX XXXX XXXX'; // Usted podrá manualmente definir su mascara de entrada
// var formatMask = 'XX+X X XXXX XXXX XXXX'; // Usted podrá añadir caracteres distitos al espacio en su máscara
var formatMask = PaymentezForm.CREDIT_CARD_NUMBER_VISA_MASK; // O utilizar una máscara predeterminada.
var cardNumber = '424 2424242 42   42 42';
var cardNumberWithoutSpaces = PaymentezForm.numbersOnlyString(cardNumber);
var formattedCardNumber = PaymentezForm.applyFormatMask(cardNumberWithoutSpaces, formatMask);
```

##### Mascaras

| Variable Name                                    | Mask
| :----------------------------------------------- | :------------------ |
| PaymentezForm.CREDIT_CARD_NUMBER_DEFAULT_MASK    | XXXX XXXX XXXX XXXX |
| PaymentezForm.CREDIT_CARD_NUMBER_VISA_MASK       | XXXX XXXX XXXX XXXX |
| PaymentezForm.CREDIT_CARD_NUMBER_MASTERCARD_MASK | XXXX XXXX XXXX XXXX |
| PaymentezForm.CREDIT_CARD_NUMBER_DISCOVER_MASK   | XXXX XXXX XXXX XXXX |
| PaymentezForm.CREDIT_CARD_NUMBER_JCB_MASK        | XXXX XXXX XXXX XXXX |
| PaymentezForm.CREDIT_CARD_NUMBER_AMEX_MASK       | XXXX XXXXXX XXXXX   |
| PaymentezForm.CREDIT_CARD_NUMBER_DINERS_MASK     | XXXX XXXX XXXX XX   |
| PaymentezForm.CREDIT_CARD_NUMBER_EXITO_MASK      | XXXX XXXX XXXX XXXX |



### Validación de fecha de expiración 
El mes de la fecha de expiración puede estar en el intervalo de: 1 = Enero a 12 = Diciembre
Para el caso de tarjeta 'Exito', estás no tienen fechas de expiración.

```javascript
var month = 3;
var year = 2019;
var valid = PaymentezForm.isExpiryValid(month, year);
```

El mes de expiración y el año de expiración pueden ser o enteros o cadenas.
```javascript
var month = "3";
var year = "2019";
var valid = PaymentezForm.isExpiryValid(month, year);
```

El año de expiración puede ser de 4 o 2 dígitos de longitud.
```javascript
var month = "3";
var year = "19";
var valid = PaymentezForm.isExpiryValid(month, year);
```

### Opciones de Validación de Tarjetas
Son tres las opciones de validaciónes de tarjetas

| Validation Option      | Description
| :--------------------- | :----------------------------------------------------- |
| PaymentezForm.AUTH_CVC | Valicación por cvc, la opción más común                |
| PaymentezForm.AUTH_NIP | Validación por nio (Disponible sólo para tarjetas Exito) |
| PaymentezForm.AUTH_OTP | Validación por otp (Disponible sólo para tarjetas Exito) | 



