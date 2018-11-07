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


[1]: https://www.sat.gob.mx/consultas/42968/consulta-y-recuperacion-de-comprobantes-(nuevo)
[2]: https://www.oasis-open.org/standards#wssv1.0
[3]: http://www.validacfd.com/phpbb3/viewtopic.php?f=14&t=7935&start=70#p47484
