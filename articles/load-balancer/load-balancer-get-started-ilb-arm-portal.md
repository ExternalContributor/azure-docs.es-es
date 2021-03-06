---
title: "Primeros pasos en la creación de un equilibrador de carga interno en el Resource Manager mediante Azure Portal | Microsoft Docs"
description: "Obtenga información sobre cómo crear un equilibrador de carga interno en Resource Manager mediante el Portal de Azure"
services: load-balancer
documentationcenter: na
author: sdwheeler
manager: carmonm
editor: 
tags: azure-service-management
ms.assetid: 1ac14fb9-8d14-4892-bfe6-8bc74c48ae2c
ms.service: load-balancer
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/24/2016
ms.author: sewhee
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 616fa3b45f8b6f7f799eeacfb1f609a1031d24f5


---
# <a name="create-an-internal-load-balancer-in-the-azure-portal"></a>Creación de un equilibrador de carga interno en Azure Portal
[!INCLUDE [load-balancer-get-started-ilb-arm-selectors-include.md](../../includes/load-balancer-get-started-ilb-arm-selectors-include.md)]

[!INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/learn-about-deployment-models-rm-include.md)]

[modelo de implementación clásica](load-balancer-get-started-ilb-classic-ps.md).

[!INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## <a name="get-started-creating-an-internal-load-balancer-using-azure-portal"></a>Primeros pasos en la creación de un equilibrador de carga interno mediante el Portal de Azure
Para crear un equilibrador de carga interno desde Azure Portal, siga estos pasos.

1. Abra un explorador, navegue a [Azure Portal](http://portal.azure.com) e inicie sesión con su cuenta de Azure.
2. En la parte superior izquierda de la pantalla, haga clic en **Nuevo** > **Redes** > **Equilibrador de carga**.
3. En la hoja **Crear equilibrador de carga**, escriba el **nombre** del equilibrador de carga.
4. En **Esquema**, haga clic en **Interno**.
5. Haga clic en **Red Virtual**y, a continuación, seleccione la red virtual donde desee crear el equilibrador de carga.
   
   > [!NOTE]
   > Si no ve la red virtual que desea utilizar, compruebe la **ubicación** que utiliza para el equilibrador de carga y cámbiela según corresponda.
   > 
   > 
6. Haga clic en **Subred**y, a continuación, seleccione la subred en la que desee crear el equilibrador de carga.
7. En **Asignación de dirección IP**, haga clic en **Dinámica** o **Estática**, en función de si desea que la dirección IP del equilibrador de carga sea fija (estática) o no.
   
   > [!NOTE]
   > Si selecciona usar una dirección IP estática, tendrá que proporcionar una dirección para el equilibrador de carga.
   > 
   > 
8. En **Grupo de recursos**, especifique el nombre de un grupo de recursos nuevo para el equilibrador de carga, o haga clic en **Seleccionar existente** y seleccione un grupo de recursos existente.
9. Haga clic en **Crear**.

## <a name="configure-load-balancing-rules"></a>Configuración de las reglas de equilibrio de carga
Después de la creación del equilibrador de carga, vaya al recurso del equilibrador de carga para configurarlo.
Tiene que configurar primero un grupo de direcciones back-end y un sondeo antes de configurar una regla de equilibrio de carga.

### <a name="step-1-configure-a-backend-pool"></a>Paso 1: Configuración de un grupo back-end
1. En Azure Portal, haga clic en **Examinar** > **Equilibradores de carga** y, después, haga clic en el equilibrador de carga que ha creado anteriormente.
2. En la hoja **Configuración**, haga clic en **Grupos back-end**.
3. En la hoja **Grupos de direcciones de back-end**, haga clic en **Agregar**.
4. En la hoja **Agregar grupo de back-end**, escriba el **nombre** del grupo de back-end y haga clic en **Aceptar**.

### <a name="step-2-configure-a-probe"></a>Paso 2: Configuración de un sondeo
1. En Azure Portal, haga clic en **Examinar** > **Equilibradores de carga** y, después, haga clic en el equilibrador de carga que ha creado anteriormente.
2. En la hoja **Configuración**, haga clic en **Sondeos**.
3. En la hoja **Sondeos**, haga clic en **Agregar**.
4. En la hoja **Agregar sondeo**, escriba el **nombre** del sondeo.
5. En **Protocolo**, seleccione **HTTP** (para sitios web) o **TCP** (para otras aplicaciones basadas en TCP).
6. En **Puerto**, especifique el puerto que se utilizará al obtener acceso el sondeo.
7. En **Ruta** (solo para sondeos HTTP), especifique la ruta de acceso que usar como un sondeo.
8. En **Intervalo** , especifique la frecuencia de sondeo de la aplicación.
9. En **Umbral incorrecto**, especifique en cuántos intentos debe producirse un error para que la máquina virtual de back-end se marque como incorrecta.
10. Haga clic en **Aceptar** para crear el sondeo.

### <a name="step-3-configure-load-balancing-rules"></a>Paso 3: Configuración de las reglas de equilibrio de carga
1. En Azure Portal, haga clic en **Examinar** > **Equilibradores de carga** y, después, haga clic en el equilibrador de carga que ha creado anteriormente.
2. En la hoja **Configuración**, haga clic en **Reglas de equilibrio de carga**.
3. En la hoja **Reglas de equilibrio de carga**, haga clic en **Agregar**.
4. En la hoja **Agregar regla de equilibrio de carga**, escriba el **nombre** de la regla.
5. En **Protocolo**, seleccione **HTTP** (para sitios web) o **TCP** (para otras aplicaciones basadas en TCP).
6. En **Puerto**, especifique los puertos a los que se conectan los clientes en el equilibrador de carga.
7. En **Puerto back-end**, especifique el puerto que se utilizará para el grupo back-end (normalmente, el puerto del equilibrador de carga y el puerto back-end son iguales).
8. En **Grupo back-end**, seleccione el grupo back-end que creó anteriormente.
9. En **Persistencia de la sesión**, seleccione cómo desea que se conserven las sesiones.
10. En **Tiempo de espera de inactividad (minutos)**, especifique el tiempo de espera de inactividad.
11. En **IP flotante (Direct Server Return)**, haga clic en **Deshabilitado** o **Habilitado**.
12. Haga clic en **OK**.

## <a name="next-steps"></a>Pasos siguientes
[Configuración de un modo de distribución del equilibrador de carga](load-balancer-distribution-mode.md)

[Configuración de opciones de tiempo de espera de inactividad de TCP para el equilibrador de carga](load-balancer-tcp-idle-timeout.md)




<!--HONumber=Nov16_HO2-->


