---
title: Obtenga información sobre el conector de Encode EDIFACT Message de Enterprise Integration Pack | Microsoft Docs
description: Sepa cómo usar partners con las Aplicaciones lógicas y Enterprise Integration Pack
services: logic-apps
documentationcenter: .net,nodejs,java
author: padmavc
manager: erikre
editor: ''

ms.service: logic-apps
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2016
ms.author: padmavc

---
# Introducción a Encode EDIFACT Message
Valida propiedades EDI y específicas del asociado.

## Creación de la conexión
### Requisitos previos
* Una cuenta de Azure; puede crear una [gratuita](https://azure.microsoft.com/free)
* Se requiere una cuenta de integración para usar el conector de Encode EDIFACT Message. Vea los detalles sobre cómo crear una [cuenta de integración](app-service-logic-enterprise-integration-create-integration-account.md), [asociados](app-service-logic-enterprise-integration-partners.md) y [un acuerdo EDIFACT](app-service-logic-enterprise-integration-edifact.md).

### Conecte con Encode EDIFACT Message mediante los siguientes pasos:
1. En [Creación de una aplicación lógica](app-service-logic-create-a-logic-app.md), podemos ver un ejemplo.
2. Este conector no tiene ningún desencadenador. Utilice otros desencadenadores para iniciar la aplicación lógica, como uno de solicitud. En el diseñador de aplicaciones lógicas, agregue un desencadenador y, luego, agregue una acción. Seleccione Mostrar las API administradas por Microsoft en la lista desplegable y, luego, escriba "EDIFACT" en el cuadro de búsqueda. Seleccione Encode EDIFACT Message por el nombre del acuerdo o Encode to EDIFACT Message por identidades.
   
    ![buscar EDIFACT](./media/app-service-logic-enterprise-integration-edifactorconnector/edifactdecodeimage1.png)
3. Si no ha creado anteriormente ninguna conexión a la cuenta de integración, se le pedirán los detalles de conexión.
   
    ![crear la conexión de la cuenta de integración](./media/app-service-logic-enterprise-integration-edifactorconnector/edifactencodeimage1.png)
4. Escriba los detalles de la cuenta de integración. Las propiedades con un asterisco son obligatorias.
   
   | Propiedad | Detalles |
   | --- | --- |
   | Nombre de la conexión * |Escriba cualquier nombre para la conexión. |
   | Cuenta de integración* |Escriba el nombre de la cuenta de integración. Asegúrese de que la cuenta de integración y la aplicación lógica se encuentran en la misma ubicación de Azure. |
   
    Una vez completados, los detalles de la conexión presentan un aspecto similar al siguiente:
   
    ![conexión de la cuenta de integración](./media/app-service-logic-enterprise-integration-edifactorconnector/edifactencodeimage2.png)
5. Seleccione **Crear**.
6. Observe que la conexión se ha creado en el portal.
   
    ![detalles de conexión de la cuenta de integración](./media/app-service-logic-enterprise-integration-edifactorconnector/edifactencodeimage4.png)

#### Encode EDIFACT Message por nombre del acuerdo
1. Proporcione el nombre del acuerdo de EDIFACT y el mensaje XML para codificar.
   
   ![especificar los campos obligatorios](./media/app-service-logic-enterprise-integration-edifactorconnector/edifactencodeimage6.png)

#### Encode EDIFACT Message por identidades
1. Especifique el identificador y el calificador del remitente y el identificador y el calificador del receptor tal y como están configurados en el acuerdo EDIFACT. Seleccione el mensaje XML para codificar
   
    ![especificar los campos obligatorios](./media/app-service-logic-enterprise-integration-edifactorconnector/edifactencodeimage7.png)

## EDIFACT Encode hace lo siguiente:
* Resolver el acuerdo haciendo coincidir el calificador y el identificador del remitente con el calificador y el identificador del receptor del receptor.
* Serializa el intercambio EDI, convirtiendo los mensajes codificados en XML en conjuntos de transacciones EDI en el intercambio.
* Aplica segmentos de encabezado y finalizador del conjunto de transacciones.
* Genera un número de control de intercambio, un número de control de grupo y un número de control del conjunto de transacciones para cada intercambio de salida.
* Reemplaza los separadores en los datos de carga útil.
* Valida propiedades EDI y específicas del asociado.
  * Validación del esquema de los elementos de datos del conjunto de transacciones con respecto al esquema de mensaje.
  * Validación de EDI realizada en los elementos de datos del conjunto de transacciones.
  * Validación extendida realizada en los elementos de datos del conjunto de transacciones.
* Genera un documento XML para cada conjunto de transacciones.
* Solicita una confirmación técnica o funcional (si esta opción está configurada).
  * Como confirmación técnica, el mensaje CONTRL indica la recepción de un intercambio.
  * Como confirmación funcional, el mensaje CONTRL indica la aceptación o el rechazo del intercambio, el grupo o el mensaje recibido, con una lista de errores o una funcionalidad no admitida.

## Pasos siguientes
[Más información sobre Enterprise Integration Pack](app-service-logic-enterprise-integration-overview.md "Información sobre Enterprise Integration Pack")

<!---HONumber=AcomDC_0824_2016-->