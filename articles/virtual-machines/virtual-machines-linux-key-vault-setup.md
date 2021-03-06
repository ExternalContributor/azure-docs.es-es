---
title: Configuración de un Almacén de claves para máquinas virtuales en Azure Resource Manager | Microsoft Docs
description: Cómo configurar un Almacén de claves para usarlo con una máquina virtual de Azure Resource Manager.
services: virtual-machines-linux
documentationcenter: ''
author: singhkays
manager: timlt
editor: ''
tags: azure-resource-manager

ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/31/2016
ms.author: singhkay

---
# Configuración de un Almacén de claves para máquinas virtuales en Azure Resource Manager
[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

modelo de implementación clásica

En la pila de Azure Resource Manager, los certificados o secretos se modelan como recursos que se proporcionan mediante el proveedor de recursos del Almacén de claves. Para más información sobre el Almacén de claves de Azure, consulte [¿Qué es el Almacén de claves de Azure?](../key-vault/key-vault-whatis.md)

Para que un Almacén de claves se utilice con máquinas virtuales de Azure Resource Manager, la propiedad *EnabledForDeployment* del Almacén de claves se debe establecer en true. Esto puede hacerlo en varios clientes.

## Uso de la CLI para configurar el Almacén de claves
Para crear un Almacén de claves mediante la interfaz de la línea de comandos (CLI), consulte [Administración del Almacén de claves mediante CLI](../key-vault/key-vault-manage-with-cli.md#create-a-key-vault).

Para la CLI, primero debe crear el almacén de claves y luego asignar la directiva de implementación. Para ello, puede usar el siguiente comando:

    azure keyvault set-policy ContosoKeyVault –enabled-for-deployment true

## Uso de plantillas para configurar el Almacén de claves
Al utilizar plantillas, debe configurar la propiedad `enabledForDeployment` como `true` para el recurso del Almacén de claves.

    {
      "type": "Microsoft.KeyVault/vaults",
      "name": "ContosoKeyVault",
      "apiVersion": "2015-06-01",
      "location": "<location-of-key-vault>",
      "properties": {
        "enabledForDeployment": "true",
        ....
        ....
      }
    }

Para otras opciones que puede configurar al crear un almacén de claves mediante plantillas, consulte [Create a key vault](https://azure.microsoft.com/documentation/templates/101-key-vault-create/) (Creación de un almacén de claves).

<!---HONumber=AcomDC_0824_2016-->