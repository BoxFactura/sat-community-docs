SAT Community Docs
===

Este es un proyecto comunitario cuyo objetivo es documentar los web services y APIs de SAT, especialmente el de descarga masiva.

### Lo que sabemos

#### Las URLs de los web services

```
Autenticación
https://cfdidescargamasivasolicitud.clouda.sat.gob.mx/Autenticacion/Autenticacion.svc

Solicitud de descarga
https://cfdidescargamasivasolicitud.clouda.sat.gob.mx/SolicitaDescargaService.svc

Verificación
https://cfdidescargamasivasolicitud.clouda.sat.gob.mx/VerificaSolicitudDescargaService.svc

Descarga
https://cfdidescargamasiva.clouda.sat.gob.mx/DescargaMasivaService.svc
```

#### El web service es SOAP

En función a la documentación publicada, el servicio es SOAP.

#### El web service no está funcionando

Al parecer no ha sido subida la configuración requerida para el `wsdl`.

#### Se requiere la e.firma para autenticar

La e.firma anteriormente era llamada FIEL.

### Eventos

- 31/Julio - SAT modifica el servicio de Mis Cuentas, implementando un captcha para bajar una factura individual
- 8/Agosto - SAT publica [documentación incompleta](https://www.sat.gob.mx/consultas/42968/consulta-y-recuperacion-de-comprobantes-(nuevo)) de un servicio que aún no está activo.
- 9/Agosto - Servicio de Mis Cuentas vuelve al esquema anterior temporalmente.

### Contribuye

¡Crea PRs de este documento! Necesitamos el apoyo de toda la comunidad para darle sentido a este nuevo servicio que el SAT publicará.

Algunas de las guías que necesitamos:

- Cómo consumir SOAP en PHP, Ruby, Javascript, etc..
- Cómo autenticar con e.Firma.
- Una vez funcionando el webservice, una guía de cómo usar cada URL.
