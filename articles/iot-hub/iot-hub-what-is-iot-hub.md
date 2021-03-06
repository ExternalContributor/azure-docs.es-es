---
title: "Información general del Azure IoT Hub | Microsoft Docs"
description: "Información general del servicio del Centro de IoT de Azure: qué es Centro de IoT, la conectividad de dispositivos, los patrones de comunicación de Internet de las cosas y el patrón de comunicación asistida por servicios"
services: iot-hub
documentationcenter: 
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 24376318-5344-4a81-a1e6-0003ed587d53
ms.service: iot-hub
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/25/2016
ms.author: dobett
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 9e718ceadc229d7207a5be66d6d7cdd443275b4e


---
# <a name="what-is-azure-iot-hub"></a>¿Qué es el centro de IoT de Azure?
Bienvenido al Centro de IoT de Azure. En este artículo se ofrece información general sobre Azure IoT Hub y se describe por qué debería usar este servicio para implementar una solución de Internet de las cosas (IoT). El Centro de IoT de Azure es un servicio totalmente administrado que permite la comunicación bidireccional fiable y segura entre millones de dispositivos IoT y un back-end de soluciones. Centro de IoT de Azure:

* Ofrece una mensajería confiable de dispositivo a nube y de nube a dispositivo a escala.
* Habilita las comunicaciones seguras con las credenciales de seguridad de cada dispositivo y el control de acceso.
* Proporciona una supervisión exhaustiva para la conectividad de dispositivos y los eventos de administración de identidad de dispositivos.
* Incluye bibliotecas de dispositivos para las plataformas y los lenguajes más populares.

En el artículo [Comparación de IoT Hub y Event Hubs][lnk-compare] se describen las principales diferencias entre estos dos servicios y se resaltan las ventajas del uso de IoT Hub en sus soluciones de IoT.

![Centro de IoT de Azure como solución de puerta de enlace de nube en Internet de las cosas][img-architecture]

> [!NOTE]
> Para ver un análisis detallado de la arquitectura de IoT, consulte el artículo [Microsoft Azure IoT Reference Architecture][lnk-refarch] (Arquitectura de referencia de IoT de Microsoft Azure).
> 
> 

## <a name="iot-deviceconnectivity-challenges"></a>Problemas de conectividad de dispositivos IoT
El Centro de IoT y las bibliotecas de dispositivo le ayudan a superar los desafíos derivados de la conexión confiable y segura de dispositivos al back-end de soluciones. Dispositivos IoT:

* A menudo son sistemas insertados sin operador humano.
* Pueden encontrarse en ubicaciones remotas, donde el acceso físico resulta costoso.
* Es posible que solo sean accesibles a través del back-end de soluciones.
* Es posible que tengan limitaciones de recursos de procesamiento y alimentación.
* Es posible que tengan conectividad de red intermitente, lenta o costosa.
* Es posible que necesiten usar protocolos de aplicación propios, personalizados o específicos de determinados sectores.
* Pueden crearse mediante un amplio conjunto de plataformas populares de hardware y software.

Además de los requisitos anteriores, toda solución de IoT debe ser capaz de ofrecer escalabilidad, seguridad y fiabilidad. Esto da lugar a un conjunto de requisitos de conectividad cuya implementación resulta compleja y lenta si se realiza con tecnologías tradicionales como, por ejemplo, los contenedores web y los agentes de mensajería.

## <a name="why-use-azure-iot-hub"></a>¿Por qué usar el centro de IoT de Azure?
El Centro de IoT de Azure aborda las dificultades de conectividad de dispositivos de las maneras siguientes:

* **Autenticación por dispositivo y conectividad segura**. Puede aprovisionar cada dispositivo con su propia [clave de seguridad][lnk-devguide-security] para permitirle conectarse a IoT Hub. El [registro de identidades de IoT Hub][lnk-devguide-identityregistry] almacena identidades y claves de dispositivos en una solución. Un back-end de soluciones puede agregar dispositivos individuales para permitir o denegar listas que permitan controlar por completo el acceso a los dispositivos.
* **Supervisión de operaciones de conectividad del dispositivo**. Puede recibir registros de operación detallados sobre operaciones de administración de identidad de dispositivos y eventos de conectividad de dispositivos. Esta funcionalidad de supervisión permite que la solución de IoT identifique los problemas de conectividad, como los dispositivos que intentan conectarse con credenciales incorrectas, envían mensajes con demasiada frecuencia o rechazan todos los mensajes de la nube al dispositivo.
* **Amplio conjunto de bibliotecas de dispositivos**. Los [SDK de dispositivo IoT de Azure][lnk-device-sdks] están disponibles para varios lenguajes y plataformas, y son compatibles con ellos (C para muchas distribuciones de Linux, Windows y sistemas operativos en tiempo real). Los SDK de dispositivos IoT de Azure admiten lenguajes administrados como C#, Java y JavaScript.
* **Extensibilidad y protocolos de IoT**. Si la solución no puede usar las bibliotecas de dispositivos, el Centro de IoT expone un protocolo público que permite a los dispositivos usar los protocolos MQTT v3.1.1, HTTP 1.1 o AMQP 1.0 de forma nativa. También puede ampliar el Centro de IoT para ofrecer soporte para protocolos personalizados mediante:
  
  * La creación de una puerta de enlace de campo con el [SDK de puerta de enlace de IoT de Azure][lnk-gateway-sdk] que convierte su protocolo personalizado en uno de los tres protocolos compatibles con IoT Hub. 
  * La personalización de la [puerta de enlace del protocolo de IoT de Azure][protocol-gateway], un componente de código abierto que se ejecuta en la nube.
* **Escala**. El centro de IoT de Azure se puede escalar a millones de dispositivos conectados de manera simultánea y a millones de eventos por segundo.

Estas ventajas son genéricas para varios patrones de comunicación. El Centro de IoT actualmente permite implementar los siguientes patrones de comunicación específicos:

* **Ingestión de dispositivos a nube basada en eventos.**  El Centro de IoT puede recibir de manera confiable millones de eventos por segundo de los dispositivos. A continuación, puede procesarlos en la ruta de acceso activa mediante un motor procesador de eventos. También puede almacenarlos en su ruta de acceso no activa para someterlos a análisis. El Centro de IoT conserva los datos de eventos hasta siete días para garantizar un procesamiento fiable y absorber picos de carga.
* **Mensajería confiable de nube a dispositivo (o *comandos*). El back-end de la solución puede usar IoT Hub para enviar mensajes con garantía de entrega de al menos una vez a dispositivos individuales. Cada mensaje tiene una configuración de período de vida individual y el back-end puede solicitar confirmación de entrega y vencimiento. Estas confirmaciones garantizan una visibilidad completa en el ciclo de vida de un mensaje de la nube al dispositivo. Luego puede implementar la lógica de negocios que incluye las operaciones que se ejecutan en dispositivos.
* **Carga de archivos y datos de sensor en caché a la nube.**  Los dispositivos pueden cargar archivos en el Almacenamiento de Azure mediante los URI de SAS gestionados por el Centro de IoT. El Centro de IoT puede generar notificaciones cuando llegan los archivos en la nube para permitir que el back-end los procese.

## <a name="gateways"></a>Puertas de enlace
Una puerta de enlace de una solución IoT normalmente es una [puerta de enlace de protocolo][lnk-gateway] que se implementa en la nube o una [puerta de enlace de campo][lnk-field-gateway] que se implementa localmente con los dispositivos. Una puerta de enlace de protocolo realiza la traducción de protocolos, por ejemplo, de MQTT a AMQP. Una puerta de enlace de campo puede ejecutar análisis en el perímetro, tomar decisiones sujetas a limitaciones temporales para reducir la latencia, proporcionar servicios de administración del dispositivo, aplicar restricciones de privacidad y seguridad y realizar la traducción de protocolos. Ambos tipos de puerta de enlace actúan como intermediarios entre los dispositivos y el Centro de IoT.

Una puerta de enlace de campo es diferente de un dispositivo de enrutamiento de tráfico simple (como un firewall o un dispositivo de traducción de direcciones de red) porque normalmente desempeña un rol activo en la administración del acceso y del flujo de la información en su solución.

Una solución puede incluir tanto puertas de enlace de protocolo como de campo.

## <a name="how-does-iot-hub-work"></a>¿Cómo funciona el Centro de IoT?
Azure IoT Hub implementa el patrón de [comunicación asistida por servicio][lnk-service-assisted-pattern] para mediar en las interacciones entre los dispositivos y el back-end de la solución. El objetivo de la comunicación asistida por servicio es establecer rutas de acceso de comunicación bidireccional de confianza entre un sistema de control (como el Centro de IoT) y los dispositivos con una finalidad específica implementados en espacios físicos que no son de confianza. El patrón establece los principios siguientes:

* La seguridad tiene prioridad sobre el resto de las funciones.
* Los dispositivos no aceptan información de redes no solicitadas. Un dispositivo establece todas las conexiones y las rutas en modo de solo salida. Para que un dispositivo reciba comandos desde el back-end, dicho dispositivo debe iniciar una conexión con regularidad para comprobar si hay comandos pendientes de procesar.
* Los dispositivos solo deben conectarse o establecer rutas a servicios conocidos a los que están emparejados, como un Centro de IoT.
* La ruta de comunicación entre el dispositivo y el servicio o el dispositivo y puerta de enlace se protege en el nivel del protocolo de aplicaciones.
* La autenticación y la autorización de nivel de sistema se basan en las identidades de cada dispositivo. Hacen los permisos y las credenciales de acceso revocables casi al instante.
* La comunicación bidireccional de dispositivos con conexión esporádica debido a problemas de alimentación o de conectividad puede realizarse mediante el mantenimiento de comandos y notificaciones en los dispositivos hasta que uno de ellos se conecte para recibirlos. El Centro de IoT mantiene colas específicas de dispositivos para los comandos que envía.
* Los datos de carga de aplicaciones se protegen por separado para proteger el tránsito a través de las puertas de enlace a un servicio determinado.

El sector de la telefonía móvil ha usado el patrón de comunicación asistida por servicio a un gran escala para implementar servicios de notificación push como, por ejemplo, los [Servicios de notificaciones push de Windows][lnk-wns], [Google Cloud Messaging][lnk-google-messaging] y [Apple Push Notification Service][lnk-apple-push].

IoT Hub es compatible con la ruta de acceso de emparejamiento público de ExpressRoute.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de cómo puede administrar, configurar y actualizar sus dispositivos de forma remota mediante la administración de dispositivos basada en estándares de Azure IoT Hub, consulte [Introducción a la administración de dispositivos con IoT Hub][lnk-device-management].

Para implementar aplicaciones cliente que se ejecuten en una gran variedad de plataformas de hardware de dispositivos y sistemas operativos, puede usar los SDK de dispositivos IoT. Los SDK de dispositivos IoT incluyen bibliotecas que facilitan el envío de telemetría a un Centro de IoT y la recepción de comandos de nube a dispositivo. Al usar los SDK, puede elegir entre varios protocolos de red para comunicarse con el Centro de IoT. Para más información, consulte la [información acerca de los SDK de los dispositivos][lnk-device-sdks].

Para comenzar a escribir código y ejecutar algunos ejemplos, consulte el tutorial [Introducción a IoT Hub][lnk-get-started].

[img-architecture]: media/iot-hub-what-is-iot-hub/hubarchitecture.png


[lnk-get-started]: iot-hub-csharp-csharp-getstarted.md
[protocol-gateway]: https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md
[lnk-service-assisted-pattern]: http://blogs.msdn.com/b/clemensv/archive/2014/02/10/service-assisted-communication-for-connected-devices.aspx "Service Assisted Communication (Comunicación asistida por servicio), entrada de blog de Clemens Vasters"
[lnk-compare]: iot-hub-compare-event-hubs.md
[lnk-gateway]: iot-hub-protocol-gateway.md
[lnk-field-gateway]: iot-hub-devguide-endpoints.md#field-gateways
[lnk-devguide-identityregistry]: iot-hub-devguide-identity-registry.md
[lnk-devguide-security]: iot-hub-devguide-security.md
[lnk-wns]: https://msdn.microsoft.com/library/windows/apps/mt187203.aspx
[lnk-google-messaging]: https://developers.google.com/cloud-messaging/
[lnk-apple-push]: https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW9
[lnk-device-sdks]: https://github.com/Azure/azure-iot-sdks
[lnk-refarch]: http://download.microsoft.com/download/A/4/D/A4DAD253-BC21-41D3-B9D9-87D2AE6F0719/Microsoft_Azure_IoT_Reference_Architecture.pdf
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk
[lnk-device-management]: iot-hub-device-management-overview.md



<!--HONumber=Nov16_HO2-->


