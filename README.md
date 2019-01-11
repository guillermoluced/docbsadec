### Integracion de BSA con Decidir

Este documento explicara como integrar BSA con Decidir utilizando .NET.
El siguiente diagrama indicia el flujo de la implementacion.
```
![Diagrama de secuencia](img/bsa-decidir-secuence.png)

```
####  Requerimientos
Tanto Todopago como Decidir tiene su SDK de NET que permite utilizar los servicios requeridos. Se pueden obtener desde:
Todopago SDK: https://github.com/TodoPago/SDK-NET-BilleteraVirtualGateway
Decidir SDK: https://github.com/decidir/sdk-.net-webtx-v2 

Para operar en Todopago es necesario tener credenciales de Todopago, Nro. de Comercio (Merchant ID) y Credenciales (API Keys). 
Por parte de Decidir es necesario tener dada de alta una tienda y obtener las credenciales "Publickey" y Privatekey.

#### Servicio Transaction
El primer paso es registrar una transaccion con el servicio [Transaction](#https://github.com/TodoPago/SDK-NET-BilleteraVirtualGateway#bvg-transaction) del  SDK de Todopago. Este requiere el Merchant y API Key de Todopago.

```C#
    Dictionary<string, Object> generalData = new Dictionary<string, Object>();
    generalData.Add(ElementNames.BSA_MERCHANT, "41702");
    generalData.Add(ElementNames.BSA_SECURITY, "TODOPAGO 8A891C0676A25FBF052D1C2FFBC82DEE");
    generalData.Add(ElementNames.BSA_OPERATION_DATE_TIME, "20170308041300");
    generalData.Add(ElementNames.BSA_REMOTE_IP_ADDRESS, "127.0.0.1");

    Dictionary<string, Object> operationData = new Dictionary<string, Object>();
    operationData.Add(ElementNames.BSA_OPERATION_TYPE, "Compra");
    operationData.Add(ElementNames.BSA_OPERATION_ID, "12345");
    operationData.Add(ElementNames.BSA_CURRENCY_CODE, "032");
    operationData.Add(ElementNames.BSA_CONCEPT, "compra");
    operationData.Add(ElementNames.BSA_AMOUNT, "10,99");

    List<string> availablePaymentMethods = new List<string>();
    availablePaymentMethods.Add("1");
    availablePaymentMethods.Add("42");
    operationData.Add(ElementNames.BSA_AVAILABLE_PAYMENT_METHODS, availablePaymentMethods);

	List<string> availableBanks = new List<string>();
	availableBanks.Add("6");
	availableBanks.Add("24");
	availableBanks.Add("29");
	operationData.Add(ElementNames.BVG_AVAILABLE_BANK, availableBanks);

    Dictionary<string, Object> technicalData = new Dictionary<string, Object>();
    technicalData.Add(ElementNames.BSA_SDK, "Net");
    technicalData.Add(ElementNames.BSA_SDK_VERSION, "1.0");
    technicalData.Add(ElementNames.BSA_LANGUAGE_VERSION, "3.5");
    technicalData.Add(ElementNames.BSA_PLUGIN_VERSION, "1.0");
    technicalData.Add(ElementNames.BSA_ECOMMERCE_NAME, "Bla");
    technicalData.Add(ElementNames.BSA_ECOMMERCE_VERSION, "3.1");
    technicalData.Add(ElementNames.BSA_CM_VERSION, "2.4");

    TransactionBVG trasactionBVG = new TransactionBVG(generalData, operationData, technicalData);
```
#####  Respuesta

La respuesta tiene el atributo **publicRequestKey**, este requerido en el formulario de Todopago.

```C#
Dictionary<string, Object>()
		{  publicRequestKey = "0e6d1f45-a85e-480f-a98f-5f18cf881b9b", //string(36)
		   merchantId = "75087", //string(36)
   		   channel = "11" //string(2)
	    }
```

#### Formulario TP de pago

Luego de Transaction se debe utilizar formulario provisto por Todopago, este se puede implementar como se indica en el ejemplo. 
Para funcionar requiere ingresar en el atributo "publicKey" el **publicRequestKey** que respondió el servicio "Transaction".

##### Endpoints:
Ambientes desarrollo: https://forms.integration.todopago.com.ar/resources/TPBSAForm.js
Ambiente Produccion: https://forms.todopago.com.ar/resources/TPBSAForm.min.js

#####  Ejemplo de implementacion
```html

<html>
    <head>
        <title>Formulario de pago TP</title>
        <meta charset="UTF-8">
        <script src="https://forms.integration.todopago.com.ar/resources/TPBSAForm.js"></script>
        <link rel="stylesheet" type="text/css" href="css/styles.css">
        <script type="text/javascript">
        </script>
    </head>
    <script>
        var success = function(data) {
            console.log(data);
        };
        var error = function(data) {
            console.log(data);
        };
        var validation = function(data) {
            console.log(data);
        }
        window.TPFORMAPI.hybridForm.initBSA({
            publicKey: "requestpublickey",
            merchantAccountId: "merchant",
            callbackCustomSuccessFunction: "success",
            callbackCustomErrorFunction: "error",
            callbackValidationErrorFunction: "validation"
        });
    </script>
</html>
```
El formulario mostrara una ventana de login para ingresar el usuario de billetera de la cuenta de Todopago
[login](login-formulario-tp.png)

#####  Respuesta
Si la compra fue aprobada el formulario devolverá un JSON con la siguiente estructura.
```html
{
"ResultCode":1,
"ResultMessage":"El medio de pago se selecciono correctamente",
"Action":"accion"
"SessionId":"DB37611F-6510-2423-1223-1C4F76F04A0D",
"IdCuenta":"41703",
"Token":"4507991692027787",
"MerchantAccountId": "46523",
"BankId":"17",
"CardNumberBin": 450799,
"FourLastDigitsOfCardNumber":"7783",
"PaymentMethodID":"42",
"SecurityCodeCheck": "false",
"SelectorClaveFlag": "1",
"TokenDate": "20180427",
"TokenizationFlag": "false",
"DatosAdicionales": {
	"tipoDocumento": "DNI",
	"numeroDocumento": "45998745",
	"generoCuentaCompradora": "M",
	"nombre": "Comprador",
	"apellido": "BSA",
  	"permiteObtenerMP": false
},
"VOLATILE_ENCRYPTED_DATA": "YRfrWggICAggsF0nR6ViuAgWsPr5ouR5knIbPtkN+yntd7G6FzN/Xb8zt6+QHnoxmpTraKphZVHvxA=="
"BSA":true
} 
```
> **Nota:** Los campos queridos por decidir son el "Token" y "VOLATILE_ENCRYPTED_DATA".


####  Solicitud de Token de Pago para BSA en Decidir

Para implementar los servicios de Decidir en NET se debe descargar la ultima versión del SDK [SDK NET Decidir](https://github.com/decidir/sdk-.net-v2). Ademas es necesario ingresar las claves publicas y privadas provistas por soporte de Decidir al dar de Alta la tienda.
Luego de importarlo en el proyecto e instanciar el SDK, se debe llamar el servicio **/tokens** para obtener el token de pago de Decidir.

```C#
string privateApiKey = "92b71cf711ca41f78362a7134f87ff65";
string publicApiKey = "e9cdb99fff374b5f91da4480c8dca741";

//AMBIENTE_SANDBOX
//AMBIENTE_PRODUCCION
//Para el ambiente de desarrollo
DecidirConnector decidir = new DecidirConnector(Ambiente.AMBIENTE_SANDBOX, privateApiKey, publicApiKey);

Tokens tokensData = new Tokens();

tokens.public_token = "4507994025297787";
tokens.volatile_encrypted_data = "YRfrWggICAggsF0nR6ViuAgWsPr5ouR5knIbPtkN+yntd7G6FzN/Xb8zt6+QHnoxmpTraKphZVHvxA==";
tokens.public_request_key = "12345678";
tokens.issue_date = "123123123123";
tokens.flag_security_code = "0";
tokens.flag_tokenization = "0";
tokens.flag_selector_key = "1";
tokens.flag_pei = "1";
tokens.card_holder_name = "Horacio";
tokens.card_holder_identification = "single";
tokens.fraud_detection = "";

try
{
    PaymentResponse resultPaymentResponse = decidir.Tokens(tokensData);
}
catch (ResponseException)
{

}
```
##### Respuesta:
```C#


```
> **Nota:** El servicio Payment requiere el token generado que devuelve el campo **id** ".

#### Ejecución del Pago para BSA en Decidir

Luego de generar el Token de pago con el servicio anterior se deberá ejecutar la solicitud de pago de la siguiente manera. Ingresando en "token" el token de pago previamente generado.

##### Ejemplo:

```C#
string privateApiKey = "92b71cf711ca41f78362a7134f87ff65";
string publicApiKey = "e9cdb99fff374b5f91da4480c8dca741";

//AMBIENTE_SANDBOX
//AMBIENTE_PRODUCCION
//Para el ambiente de desarrollo
DecidirConnector decidir = new DecidirConnector(Ambiente.AMBIENTE_SANDBOX, privateApiKey, publicApiKey);

Payment payment = new Payment();

payment.site_transaction_id = "[ID DE LA TRANSACCIÓN]"; //string unico que identifica la transaccion
payment.payment_method_id = 1;
payment.token = "[TOKEN DE PAGO]"; //token de pago provisto por el servicio tokens
payment.bin = "450799";
payment.amount = 2000;
payment.currency = "ARS";
payment.installments = 1;
payment.description = "";
payment.payment_type = "single";
payment.establishment_name = "single";

try
{
    PaymentResponse resultPaymentResponse = decidir.Payment(payment);
}
catch (ResponseException)
{
}

```

##### Respuesta:
```C#


```


#### Notification Push

Permite registrar la fiscalización de una transacción. El método retorna el objeto NotificationPushBVG con el resultado de la notificación. Se utiliza de la siguiente manera:

##### Ejemplo:

```C#

BvgConnector connector = new BvgConnector(endpoint, headers);
	NotificationPushBVG notificationPushBVG = new NotificationPushBVG();

try{

notificationPushBVG = connector.notificationPush(notificationPushBVG);
Dictionary<string, Object> dic = notificationPushBVG.toDictionary();

}catch (EmptyFieldException ex){
    Console.WriteLine(ex.Message);
}catch (ResponseException ex){
    Console.WriteLine(ex.Message);
}catch (ConnectionException ex) {
    Console.WriteLine(ex.Message);
}

```
##### Respuesta:
```C#

Dictionary<string, Object>() = notificationPushBVG.toDictionary();
		{  statusCode = -1, //string(2)
           statusMessage = OK //string(2)
	    }

```

