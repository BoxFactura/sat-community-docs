### Autenticarse en el webservice (WS)

Según la [documentación del SAT][1], el estandar para el firmado es la
especificación [Web Services Security v1.0 (WS-Security 2004)][2], pero como
casi siempre, no respetan el estandar, solo miren los prefijos de los namespace.

IMPORTANTE: El WS **no tiene una definición de esquema**, es por eso que la
mayoría de los framework te daran un error 404 de no existencia, es necesario
enviar directamente el mensaje SOAP por método POST al WS, sigue leyendo.

Puedes ver en la [carpeta de ejemplos](Ejemplos), un mensaje SOAP request y
response, de nuevo, que afirman han sido probados en producción.

El proceso descrito a continuación es tomado desde [esta página][3] donde
afirman que funciona, vamos a validarlo.


#### URL para autenticación

- https://cfdidescargamasivasolicitud.clouda.sat.gob.mx/Autenticacion/Autenticacion.svc


#### Requisitos previos

- Debes tener tu FIEL activa y vigente.


#### Plantilla SOAP

```xml
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" xmlns:u="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">
  <s:Header>
    <o:Security s:mustUnderstand="1" xmlns:o="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">
      <u:Timestamp u:Id="_0">
        <u:Created>{DATE_CREATED}</u:Created>
        <u:Expires>{DATE_EXPIRED}</u:Expires>
      </u:Timestamp>
      <o:BinarySecurityToken u:Id="{LABEL}" ValueType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-x509-token-profile-1.0#X509v3" EncodingType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-soap-message-security-1.0#Base64Binary">{CERTIFICATE}</o:BinarySecurityToken>
      <Signature xmlns="http://www.w3.org/2000/09/xmldsig#">
        <SignedInfo>
          <CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
          <SignatureMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1" />
          <Reference URI="#_0">
            <Transforms>
              <Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
            </Transforms>
            <DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1" />
            <DigestValue></DigestValue>
          </Reference>
        </SignedInfo>
        <SignatureValue></SignatureValue>
        <KeyInfo>
          <o:SecurityTokenReference>
            <o:Reference ValueType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-x509-token-profile-1.0#X509v3" URI="#{LABEL}" />
          </o:SecurityTokenReference>
        </KeyInfo>
      </Signature>
    </o:Security>
  </s:Header>
  <s:Body>
    <Autentica xmlns="http://DescargaMasivaTerceros.gob.mx" />
  </s:Body>
</s:Envelope>
```

#### Establecer la fecha y hora de creación y expiración

En general debería ser la hora actual para la fecha de creación y 5 minutos
después para la fecha de expiración, importante, **debe ser en UTC** y
establecerlos dentro del nodo **Timestamp**. Los 5 minutos es lo que todo mundo
esta manejando, no tenemos algún fundamento.

```xml
  <u:Timestamp u:Id="_0">
    <u:Created>2018-11-07T19:38:34.000Z</u:Created>
    <u:Expires>2018-11-07T19:43:34.000Z</u:Expires>
  </u:Timestamp>
```

- Observa el atributo `u:Id`, en teoría, su valor es solo un *identificador* que
se usa dentro del nodo `Reference` como valor del atributo `URI`, que "se supone"
le indica que nodo es el origen del nodo `DigestValue`. En teoría, debería ser
cualquier texto. En los ejemplos del estandar usan: `TO`.

- Ahora el nodo se tiene que **canonizar**, no, no es volverlo santo. Se trata
de obtener un XML (o solo un nodo) con un algunas reglas, solo que estas reglas
son [bastante complejas][4]

Entonces, nuestro nodo ya canonizado debe verse así:

```
<u:Timestamp xmlns:u="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd" u:Id="_0"><u:Created>2018-11-07T19:38:34.000Z</u:Created><u:Expires>2018-11-07T19:43:34.000Z</u:Expires></u:Timestamp>
```

Nota que no hay espacios, no hay saltos de línea y se ha agregado el namespace
respectivo para el prefijo usado.

Ahora, a esta cadena, le aplicamos un hash SHA1 y lo ponemos en base64,
reemplaza **NODE** por la cadena completa del nodo arriba mostrada.

```
echo -n 'NODE' | sha1sum -b | xxd -r -p | base64
```

para obtener el valor:

```
5nYaTOpTMrMiXOJEee7SZ0tsuSw=
```

Si haces esta operación con el nodo canonizado del archivo request de ejemplo.

```
<u:Timestamp xmlns:u="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd" u:Id="_0"><u:Created>2018-09-27T19:03:58.117Z</u:Created><u:Expires>2018-09-27T19:08:58.117Z</u:Expires></u:Timestamp>
```

Deberías obtener el mismo valor mostrado en ese archivo:
```
My8tNIoupHoGgBFJ0o9ZXPK8AvI=
```

Este valor lo establecemos en nodo `DigestValue` de nuestro XML.

```
<SignedInfo>
  <CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
  <SignatureMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1" />
  <Reference URI="#_0">
    <Transforms>
      <Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
    </Transforms>
    <DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1" />
    <DigestValue>AtRnvS0kUJJnA5IJXRY7ggr91ps=</DigestValue>
  </Reference>
</SignedInfo>
```

Ahora, canizamos este nodo (`SignedInfo`)

```
<SignedInfo xmlns="http://www.w3.org/2000/09/xmldsig#"><CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"></CanonicalizationMethod><SignatureMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1"></SignatureMethod><Reference URI="#_0"><Transforms><Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"></Transform></Transforms><DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"></DigestMethod><DigestValue>5nYaTOpTMrMiXOJEee7SZ0tsuSw=</DigestValue></Reference></SignedInfo>
```

Y lo firmamos con nuestra FIEL, de nuevo, reemplaza `NODE` por la cadena anterior.

```
echo -n 'NODE' | openssl dgst -sha256 -sign fiel.pem | openssl enc -base64 -A
```

Establecemos esta firma como valor del nodo `SignatureValue`

```
<SignatureValue>XAGdaKmgosrV2g22xS/WYpxljQxoqqRW7xq4pDKefkIrwDfkjDUlfoQV7WUgT+K2M7PCNGL411CWbUEF8N+5LPUmCuNIXVVaSyn5NVkQie1CEzUkNdystmmUCLxZBgw6xtlpJ4YbWt98kILkkDifWDlHiMChOtDfM/m+UgXYj/W0b6iOiRdrHBjI1380vQhPSnFZp3Ts6Qfq5d03WcWvEwptmhA032mqy3K4aGdQAiVyJhpJYQJCWUFBqcOOIbK64WQmglSOIl90Rcph5Wvq3R3U1Xw/Tdi6gN8TCgr5ybrkxxBgT97L/3jvKzK/n9HlUzdIdtW5XncrIbRj66AfcA==</SignatureValue>
```

Por ultimo, establecemos el nodo `BinarySecurityToken`, observa que aquí
también tenemos el atributo `u:Id`, que de nuevo, es una referencia que se usa
dentro del nodo `SecurityTokenReference`. En el request de ejemplo, usan el valor
de un UUID, en teoría, podría ser cualquier etiqueta. El valor de este nodo,
será la clave pública de nuestra FIEL, es decir, el archivo CER.

```
<o:BinarySecurityToken u:Id="{LABEL}" ValueType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-x509-token-profile-1.0#X509v3" EncodingType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-soap-message-security-1.0#Base64Binary">{CERTIFICATE}</o:BinarySecurityToken>
```

En este ejemplo usamos, el número de serie del certificado, más adelante
validaremos si esto es correcto.

```
<o:BinarySecurityToken u:Id="00001000000404032438" ValueType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-x509-token-profile-1.0#X509v3" EncodingType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-soap-message-security-1.0#Base64Binary">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</o:BinarySecurityToken>
```

Esta listo nuestro mensaje SOAP para enviarse al WS.

```
curl --header "Content-Type: text/xml;charset=UTF-8" --header "SOAPAction:Autentica" --data @request.xml https://cfdidescargamasivasolicitud.clouda.sat.gob.mx/Autenticacion/Autenticacion.svc
```

Y obtenemos el error:

```
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/"><s:Body><s:Fault><faultcode xmlns:a="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">a:InvalidSecurity</faultcode><faultstring xml:lang="en-US">An error occurred when verifying security for the message.</faultstring></s:Fault></s:Body></s:Envelope>
```

Seguimos investigando.


[1]: https://www.sat.gob.mx/consultas/42968/consulta-y-recuperacion-de-comprobantes-(nuevo)
[2]: https://www.oasis-open.org/standards#wssv1.0
[3]: http://www.validacfd.com/phpbb3/viewtopic.php?f=14&t=7935&start=70#p47484
[4]: https://www.w3.org/TR/xml-c14n/
