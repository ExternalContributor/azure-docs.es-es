---
title: Introducción al modelo de autenticación y seguridad de los Centros de eventos | Microsoft Docs
description: Introducción al modelo de autenticación y seguridad de Centros de eventos
services: event-hubs
documentationcenter: na
author: sethmanheim
manager: timlt
editor: ''

ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/16/2016
ms.author: sethm;clemensv

---
# <a name="event-hubs-authentication-and-security-model-overview"></a>Introducción al modelo de autenticación y seguridad de los Centros de eventos
El modelo de seguridad de los Centros de eventos cumple los siguientes requisitos:

* Solo los dispositivos que tengan credenciales válidas pueden enviar datos a un Centro de eventos.
* Un dispositivo no puede suplantar a otro dispositivo.
* Un dispositivo no autorizado puede bloquearse para que no envíe datos a un Centro de eventos.

## <a name="device-authentication"></a>Autenticación de dispositivos
El modelo de seguridad de Event Hubs se basa en una combinación de tokens de [firma de acceso compartido (SAS)](../service-bus/service-bus-shared-access-signature-authentication.md) y *publicadores de eventos*. Un publicador de eventos define un extremo virtual para un Centro de eventos. El publicador solo puede usarse para enviar mensajes a un Centro de eventos. No es posible recibir mensajes desde un publicador.

Normalmente, un Centro de eventos emplea a un publicador por dispositivo. Todos los mensajes que se envíen a cualquiera de los publicadores de un Centro de eventos se ponen en cola dentro de ese Centro de eventos. Los publicadores permiten control de acceso y limitación avanzados.

A cada dispositivo se le asigna un token único, que se carga en el dispositivo. Los tokens se producen de forma que cada token único concede acceso a un publicador único diferente. Un dispositivo que posea un token solo puede enviar a un publicador y a ningún otro. Si varios dispositivos comparten el mismo token, cada uno de estos dispositivos comparte un publicador.

Aunque no se recomienda, es posible equipar los dispositivos con tokens que concedan acceso directo a un Centro de eventos. Cualquier dispositivo que contenga un token de ese tipo puede enviar mensajes directamente a ese Centro de eventos. Ese dispositivo no estará sujeto a limitación. Además, el dispositivo no puede estar en la lista negra que impide el envío a ese Centro de eventos.

Todos los tokens se firman con una clave de SAS. Normalmente, todos los tokens se firman con la misma clave. Los dispositivos no conocen la clave; esto evita que los dispositivos produzcan tokens.

### <a name="create-the-sas-key"></a>Creación de la clave SAS
Cuando se crea un espacio de nombres de Centros de eventos, Centros de eventos de Azure genera una clave SAS de 256 bits llamada **RootManageSharedAccessKey**. Esta clave permite enviar, escuchar y administrar los derechos del espacio de nombres. Puede crear claves adicionales. Se recomienda que genere una clave que conceda permisos de envío para el Centro de eventos concreto. En el resto de este tema, se presupone que esta clave tiene el nombre `EventHubSendKey`.

En el ejemplo siguiente se crea una clave solo de envío cuando se crea el Centro de eventos:

```
// Create namespace manager.
string serviceNamespace = "YOUR_NAMESPACE";
string namespaceManageKeyName = "RootManageSharedAccessKey";
string namespaceManageKey = "YOUR_ROOT_MANAGE_SHARED_ACCESS_KEY";
Uri uri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, string.Empty);
TokenProvider td = TokenProvider.CreateSharedAccessSignatureTokenProvider(namespaceManageKeyName, namespaceManageKey);
NamespaceManager nm = new NamespaceManager(namespaceUri, namespaceManageTokenProvider);

// Create Event Hub with a SAS rule that enables sending to that Event Hub
EventHubDescription ed = new EventHubDescription("MY_EVENT_HUB") { PartitionCount = 32 };
string eventHubSendKeyName = "EventHubSendKey";
string eventHubSendKey = SharedAccessAuthorizationRule.GenerateRandomKey();
SharedAccessAuthorizationRule eventHubSendRule = new SharedAccessAuthorizationRule(eventHubSendKeyName, eventHubSendKey, new[] { AccessRights.Send });
ed.Authorization.Add(eventHubSendRule); 
nm.CreateEventHub(ed);
```

### <a name="generate-tokens"></a>Generación de tokens
Puede generar tokens con la clave de SAS. Solo debe generar un token por dispositivo. Los tokens se pueden producir con el método siguiente. Todos los tokens se generan con la clave **EventHubSendKey** . A cada token se le asigna un URI único.

```
public static string SharedAccessSignatureTokenProvider.GetSharedAccessSignature(string keyName, string sharedAccessKey, string resource, TimeSpan tokenTimeToLive)
```

Al llamar a este método, se debe especificar el URI como `//<NAMESPACE>.servicebus.windows.net/<EVENT_HUB_NAME>/publishers/<PUBLISHER_NAME>`. Para todos los tokens, el URI es idéntico, con la excepción de `PUBLISHER_NAME`, que debe ser diferente para cada token. Idealmente, `PUBLISHER_NAME` representa el identificador del dispositivo que recibe ese token.

Este método genera un token con la estructura siguiente:

```
SharedAccessSignature sr={URI}&sig={HMAC_SHA256_SIGNATURE}&se={EXPIRATION_TIME}&skn={KEY_NAME}
```

La fecha de expiración del token se especifica en segundos desde el 1 de enero de 1970. A continuación, se muestra un ejemplo de un token:

```
SharedAccessSignature sr=contoso&sig=nPzdNN%2Gli0ifrfJwaK4mkK0RqAB%2byJUlt%2bGFmBHG77A%3d&se=1403130337&skn=RootManageSharedAccessKey
```

Normalmente, los tokens tienen una duración que se parece o supera la duración del dispositivo. Si el dispositivo tiene la capacidad de obtener un nuevo token, pueden usarse tokens con una duración más corta.

### <a name="devices-sending-data"></a>Envío de datos de dispositivos
Cuando se han creado los tokens, cada dispositivo se aprovisiona con su propio token único.

Cuando el dispositivo envía datos a un Centro de eventos, el dispositivo etiqueta su token con la solicitud de envío. Para evitar que un atacante use la técnica de eavesdropping y robe el token, la comunicación entre el dispositivo y el Centro de eventos debe realizarse a través de un canal cifrado.

### <a name="blacklisting-devices"></a>Incorporación de dispositivos a la lista negra
Si un atacante roba un token, el atacante puede suplantar el dispositivo al que se ha robado el token. Al incorporar un dispositivo a la lista negra, se consigue que el dispositivo quede inutilizable hasta que se le asigne un nuevo token que use un publicador diferente.

## <a name="authentication-of-back-end-applications"></a>Autenticación de aplicaciones de back-end
Para autenticar aplicaciones de back-end que consumen los datos que generan los dispositivos, Centros de eventos emplea un modelo de seguridad que es similar al que se usa en los temas de Bus de servicio. Un grupo de consumidores de Centros de eventos equivale a una suscripción a un tema de Bus de servicio. Un cliente puede crear un grupo de consumidores si la solicitud para crear el grupo de consumidores viene acompañada de un token que concede privilegios de administración para el Centro de eventos o para el espacio de nombres al que pertenece. Se permite que un cliente pueda consumir datos de un grupo de consumidores si la solicitud de recepción está acompañada de un token que concede derechos de recepción en ese grupo de consumidores, Centro de eventos o espacio de nombres al que pertenece el Centro de eventos.

La versión actual de Bus de servicio no admite reglas SAS para suscripciones individuales. Lo mismo sucede con los grupos de consumidores de los Centros de eventos. La compatibilidad con SAS se agregará para ambas características en el futuro.

En ausencia de autenticación SAS para grupos de consumidores individuales, puede usar claves SAS para proteger todos los grupos de consumidores con una clave común. Este enfoque permite que una aplicación consuma datos desde cualquiera de los grupos de consumidores de un Centro de eventos.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre Centros de eventos, visite los siguientes temas:

* [Información general de los Centros de eventos]
* Una [solución de mensajería en cola] mediante las colas de Bus de servicio.
* Una [aplicación de ejemplo completa que usa Centros de eventos].

[Información general de los Centros de eventos]: event-hubs-overview.md
[aplicación de ejemplo completa que usa Centros de eventos]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[solución de mensajería en cola]: ../service-bus-messaging/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md




<!--HONumber=Oct16_HO2-->


