---
title: "Introducción a la creación de un equilibrador de carga orientado a Internet en un modelo de implementación clásica con Cloud Services | Microsoft Docs"
description: "Obtenga información acerca de cómo crear un equilibrador de carga orientado a Internet en el modelo de implementación clásica para servicios en la nube"
services: load-balancer
documentationcenter: na
author: sdwheeler
manager: carmonm
editor: 
tags: azure-service-management
ms.assetid: 0bb16f96-56a6-429f-88f5-0de2d0136756
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/17/2016
ms.author: sewhee
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 171d5cd41d900b83c22e1db4bc514471a3d4b556

---

# <a name="get-started-creating-an-internet-facing-load-balancer-for-cloud-services"></a>Introducción a la creación de un equilibrador de carga orientado a Internet para servicios en la nube

[!INCLUDE [load-balancer-get-started-internet-classic-selectors-include.md](../../includes/load-balancer-get-started-internet-classic-selectors-include.md)]

[!INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

Este artículo trata sobre el modelo de implementación clásico. También puede [obtener información sobre cómo crear un equilibrador de carga orientado a Internet con el Administrador de recursos de Azure](load-balancer-get-started-internet-arm-cli.md).

Los servicios en la nube se configuran automáticamente con un equilibrador de carga y se pueden personalizar mediante el modelo de servicio.

## <a name="create-a-load-balancer-using-the-service-definition-file"></a>Crear un equilibrador de carga mediante el archivo de definición de servicio

Puede aprovechar el SDK de Azure para .NET 2.5 para actualizar el servicio en la nube. La configuración de extremos para los servicios en la nube se realiza en el archivo [definición de servicio](https://msdn.microsoft.com/library/azure/gg557553.aspx).csdef.

En el ejemplo siguiente se muestra cómo se configura un archivo servicedefinition.csdef para una implementación en la nube:

Mediante la comprobación del fragmento de código para el archivo .csdef generado por una implementación en la nube, puede ver el extremo externo configurado para usar puertos HTTP en los puertos 10000, 10001 y 10002.

```xml
    <ServiceDefinition name=“Tenant“>
       <WorkerRole name=“FERole” vmsize=“Small“>
    <Endpoints>
        <InputEndpoint name=“FE_External_Http” protocol=“http” port=“10000“ />
        <InputEndpoint name=“FE_External_Tcp“  protocol=“tcp“  port=“10001“ />
        <InputEndpoint name=“FE_External_Udp“  protocol=“udp“  port=“10002“ />

        <InputEndpointname=“HTTP_Probe” protocol=“http” port=“80” loadBalancerProbe=“MyProbe“ />

        <InstanceInputEndpoint name=“InstanceEP” protocol=“tcp” localPort=“80“>
           <AllocatePublicPortFrom>
              <FixedPortRange min=“10110” max=“10120“  />
           </AllocatePublicPortFrom>
        </InstanceInputEndpoint>
        <InternalEndpoint name=“FE_InternalEP_Tcp” protocol=“tcp“ />
    </Endpoints>
      </WorkerRole>
    </ServiceDefinition>
```

## <a name="check-load-balancer-health-status-for-cloud-services"></a>Comprobación del estado de mantenimiento del equilibrador de carga para servicios en la nube

A continuación se muestra un sondeo de estado:

```xml
    <LoadBalancerProbes>
        <LoadBalancerProbe name=“MyProbe” protocol=“http” path=“Probe.aspx” intervalInSeconds=“5” timeoutInSeconds=“100“ />
    </LoadBalancerProbes>
```

El equilibrador de carga combina la información del punto de conexión y la información del sondeo para crear una dirección URL en forma de http://{DIP de VM}:80/Probe.aspx que se puede usar para consultar el estado del servicio.

El servicio detecta sondeos periódicos desde la misma dirección IP. Se trata de la solicitud de sondeo de estado procedente del host del nodo donde se ejecuta la máquina virtual. El servicio debe responder con un código de estado HTTP 200 para que el equilibrador de carga asuma que el estado del servicio es correcto. Cualquier otro código de estado HTTP (por ejemplo, el 503) extrae directamente la máquina virtual de la rotación.

La definición de sondeo también controla la frecuencia del sondeo. En nuestro caso anterior, el equilibrador de carga está sondeando el extremo de cada 5 segundos. Si no se recibe una respuesta positiva durante 10 segundos (dos intervalos de sondeo), se supone que el sondeo es negativo y la máquina virtual se excluye de la rotación. De forma similar, si el servicio está fuera de la rotación y se recibe una respuesta positiva, el servicio se pone de nuevo en rotación de forma inmediata. Si el servicio fluctúa entre correcto e incorrecto, el equilibrador de carga puede decidir retrasar la reintroducción del servicio a rotación hasta que ha sido correcto durante un número de sondeos.

Compruebe el esquema de definición del [sondeo de estado](https://msdn.microsoft.com/library/azure/jj151530.aspx) para obtener más información.

## <a name="next-steps"></a>Pasos siguientes

[Introducción a la configuración de un equilibrador de carga interno](load-balancer-get-started-ilb-arm-ps.md)

[Configuración de un modo de distribución del equilibrador de carga](load-balancer-distribution-mode.md)

[Configuración de opciones de tiempo de espera de inactividad de TCP para el equilibrador de carga](load-balancer-tcp-idle-timeout.md)




<!--HONumber=Nov16_HO2-->


