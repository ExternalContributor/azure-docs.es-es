---
title: Guardado y configuración del servicio Administración de API mediante Git
description: Obtenga información sobre cómo guardar y configurar la configuración del servicio de administración de API mediante Git.
services: api-management
documentationcenter: ''
author: steved0x
manager: erikre
editor: ''

ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/09/2016
ms.author: sdanie

---
# Guardado y configuración del servicio Administración de API mediante Git
> [!IMPORTANT]
> La configuración Git de Administración de API está actualmente en versión preliminar. Aunque es completa funcionalmente, es una versión preliminar porque buscamos activamente comentarios acerca de esta característica. Es posible que hagamos cambios importantes en respuesta a los comentarios de los clientes, por lo que recomendamos no depender de la característica en entornos de producción. Si tiene comentarios o preguntas, háganoslo saber en `apimgmt@microsoft.com`.
> 
> 

Cada instancia del servicio Administración de API mantiene una base de datos de configuración que contiene información sobre la configuración y los metadatos de la instancia del servicio. Es posible hacer cambios en la instancia del servicio; para ello, modifique un valor del portal para editores con un cmdlet de PowerShell o realice una llamada de API de REST. Pero esto no es todo, también puede administrar la configuración de la instancia del servicio con Git, con lo que se posibilitan escenarios de administración de servicio como los siguientes:

* Control de versiones de configuración: descargue y almacene versiones diferentes de la configuración del servicio
* Cambios masivos en la configuración: realice cambios en varios ajustes de la configuración del servicio en el repositorio local e integre los cambios de nuevo en el servidor con una sola operación
* Cadena de herramientas y flujo de trabajo familiares de Git: use las herramientas y los flujos de trabajo de Git con los que ya está familiarizado

El diagrama siguiente muestra de forma global los distintos modos de configurar la instancia del servicio de Administración de API.

![Configuración de GIT][api-management-git-configure]

Cuando hace cambios en el servicio mediante el portal para editores, los cmdlets de PowerShell o la API de REST, está administrando la base de datos de configuración del servicio mediante el punto de conexión `https://{name}.management.azure-api.net`, tal como se muestra en el lado derecho del diagrama. En el lado izquierdo del diagrama se muestra cómo administrar la configuración del servicio mediante Git y el repositorio de Git para el servicio ubicado en `https://{name}.scm.azure-api.net`.

Los pasos siguientes proporcionan una visión general sobre el proceso de administración de la instancia del servicio de Administración de API mediante Git.

1. Habilitar el acceso de Git en el servicio
2. Guardar la base de datos de configuración del servicio en el repositorio de Git
3. Clonar el repositorio de Git en el equipo local
4. Desplegar el repositorio más reciente en el equipo local, y confirmar e insertar los cambios de nuevo en el repositorio
5. Implementar los cambios desde el repositorio en la base de datos de configuración del servicio

Este artículo describe cómo habilitar y usar Git para administrar la configuración del servicio y sirve como referencia para los archivos y las carpetas del repositorio Git.

## Para habilitar el acceso de Git
Puede ver rápidamente el estado de la configuración Git en el icono de Git, en la esquina superior derecha del portal del editor. En este ejemplo, el acceso de Git no todavía se ha habilitado.

![Estado de Git][api-management-git-icon-enable]

Para ver y configurar las opciones de configuración Git, puede hacer clic en el icono de Git o haga clic en el menú **Seguridad** y vaya a la pestaña **Repositorio de configuraciones**.

![Habilitar GIT][api-management-enable-git]

Para habilitar el acceso de Git, active la casilla **Habilitar acceso de Git**.

Tras unos instantes, el cambio se guarda y se muestra un mensaje de confirmación. Tenga en cuenta que el icono de Git cambia de color para indicar que el acceso de Git está habilitado, y ahora el mensaje de estado indica que hay cambios sin guardar en el repositorio. Esto es porque la base de datos de configuración del servicio de Administración de API aún no se ha guardado en el repositorio.

![GIT habilitado][api-management-git-enabled]

> [!IMPORTANT]
> Los secretos que no se definan como propiedades se almacenarán en el repositorio y permanecerán en su historial hasta que deshabilite y vuelva a habilitar el acceso de Git. Las propiedades proporcionan un lugar seguro para administrar los valores de cadena constante, como los secretos, a través de toda la configuración y las directivas de API, por lo que no tiene que almacenarlos directamente en las instrucciones de directiva. Para obtener más información, consulte [How to use properties in Azure API Management policies](api-management-howto-properties.md) (Uso de propiedades en directivas de Administración de API de Azure).
> 
> 

Para obtener información sobre la habilitación o la deshabilitación del acceso de Git mediante la API de REST, consulte [Enable or disable Git access using the REST API](https://msdn.microsoft.com/library/dn781420.aspx#EnableGit) (Habilitación o deshabilitación del acceso de Git mediante la API de REST).

## Guardado de la configuración del servicio en el repositorio de Git
El primer paso antes de la clonación del repositorio es guardar el estado actual de la configuración del servicio en el repositorio. Haga clic en **Guardar configuración en repositorio**.

![Guardar configuración][api-management-save-configuration]

Realice los cambios necesarios en la pantalla de confirmación y haga clic en **Aceptar** para guardar.

![Guardar configuración][api-management-save-configuration-confirm]

Transcurridos unos segundos, la configuración se guarda y se muestra el estado de configuración del repositorio, incluidas la fecha y la hora del último cambio de configuración y la última sincronización entre la configuración del servicio y el repositorio.

![Estado de configuración][api-management-configuration-status]

Una vez que la configuración se guarda en el repositorio, se puede clonar.

Para obtener información acerca de cómo realizar esta operación mediante la API de REST, consulte [Commit configuration snapshot using the REST API](https://msdn.microsoft.com/library/dn781420.aspx#CommitSnapshot) (Confirmación de la instantánea de configuración mediante la API de REST).

## Para clonar el repositorio en el equipo local
Para clonar un repositorio, necesitará la dirección URL del repositorio, un nombre de usuario y una contraseña. El nombre de usuario y la dirección URL se muestran cerca de la parte superior de la pestaña **Repositorio de configuración**.

![Clonación de git][api-management-configuration-git-clone]

La contraseña se genera en la parte inferior de la pestaña **Repositorio de configuración**.

![Generar contraseña][api-management-generate-password]

Para generar una contraseña, primero asegúrese de que el campo **Expiración** refleja la fecha y la hora de caducidad deseada y luego haga clic en **Generar token**.

![Password][api-management-password]

> [!IMPORTANT]
> Anote esta contraseña. Una vez que salga de esta página, la contraseña no se volverá a mostrar.
> 
> 

En los ejemplos siguientes se usa la herramienta Git Bash desde [Git para Windows](http://www.git-scm.com/downloads), pero puede usar cualquier herramienta de Git con la que esté familiarizado.

Abra su herramienta Git en la carpeta deseada y ejecute el siguiente comando para clonar el repositorio de git en el equipo local, usando para ello el comando incluido en el portal para editores.

    git clone https://bugbashdev4.scm.azure-api.net/ 

Proporcione el nombre de usuario y la contraseña cuando se le solicite.

Si recibe algún error, pruebe a modificar su comando `git clone` para incluir el nombre de usuario y la contraseña, como se muestra en el ejemplo siguiente.

    git clone https://username:password@bugbashdev4.scm.azure-api.net/

Si de este modo aparece un error, pruebe a codificar como dirección URL la parte de la contraseña del comando. Una manera rápida de hacer esto es abrir Visual Studio y emitir el siguiente comando en la **Ventana Inmediato**. Para abrir la **Ventana Inmediato**, abra cualquier solución o proyecto en Visual Studio (o cree una nueva aplicación de consola vacía) y elija **Ventanas**, **Inmediato** desde el menú **Depurar**.

    ?System.NetWebUtility.UrlEncode("password from publisher portal")

Utilice la contraseña codificada junto con su nombre de usuario y ubicación de repositorio para construir el comando git.

    git clone https://username:url encoded password@bugbashdev4.scm.azure-api.net/

Una vez que se clone el repositorio, podrá ver y trabajar con él en el sistema de archivos local. Para más información, consulte [Referencia de estructura de archivo y carpeta del repositorio local de Git](#file-and-folder-structure-reference-of-local-git-repository).

## Para actualizar su repositorio local con la configuración de instancia de servicio más reciente
Si realiza cambios en la instancia del servicio de Administración de API en el portal para editores o mediante la API de REST, debe guardar estos cambios en el repositorio para poder actualizar el repositorio local con los cambios más recientes. Para ello, haga clic en **Guardar configuración en repositorio** en la pestaña **Repositorio de configuración** del portal para editores y, a continuación, emita el siguiente comando en el repositorio local.

    git pull

Antes de ejecutar `git pull`, asegúrese de que se encuentra en la carpeta del repositorio local. Si acaba de completar el comando `git clone`, debe cambiar el directorio a su repositorio ejecutando un comando similar al siguiente.

    cd bugbashdev4.scm.azure-api.net/

## Para insertar los cambios desde su repositorio local al repositorio del servidor
Para insertar los cambios desde el repositorio local al repositorio del servidor, debe confirmar los cambios y, a continuación, insertarlos en el repositorio del servidor. Para confirmar los cambios, abra la herramienta de comandos de Git, cambie al directorio del repositorio local y emita los siguientes comandos.

    git add --all
    git commit -m "Description of your changes"

Para insertar todas las confirmaciones en el servidor, ejecute el siguiente comando.

    git push

## Para implementar los cambios de la configuración del servicio en la instancia del servicio de Administración de API
Una vez confirmados los cambios locales e insertados en el repositorio del servidor, puede implementarlos en la instancia del servicio Administración de API.

![Implementación][api-management-configuration-deploy]

Para obtener información acerca de cómo realizar esta operación mediante la API de REST, consulte [Deploy Git changes to configuration database using the REST API](https://msdn.microsoft.com/library/dn781420.aspx#DeployChanges) (Implementación de cambios de Git en la base de datos de configuración mediante la API de REST).

## Referencia de estructura de archivo y carpeta del repositorio local de Git
Los archivos y carpetas del repositorio local de Git contienen la información de configuración acerca de la instancia de servicio.

| Elemento | Description |
| --- | --- |
| carpeta raíz de la administración de API |Contiene la configuración de nivel superior para la instancia de servicio |
| carpeta de API |Contiene la configuración para las API de la instancia de servicio |
| carpeta de grupos |Contiene la configuración para los grupos de la instancia de servicio |
| carpeta de directivas |Contiene las directivas de la instancia de servicio |
| carpeta portalStyles |Contiene la configuración para las personalizaciones del portal para desarrolladores de la instancia de servicio |
| carpeta de productos |Contiene la configuración para los productos de la instancia de servicio |
| carpeta de plantillas |Contiene la configuración para las plantillas de correo electrónico de la instancia de servicio |

Cada carpeta puede contener uno o varios archivos y, en algunos casos, una o varias carpetas; por ejemplo, una carpeta para cada API, producto o grupo. Los archivos dentro de cada carpeta son específicos del tipo de entidad descrito por el nombre de la carpeta.

| Tipo de archivo | Propósito |
| --- | --- |
| json |Información de configuración acerca de la entidad correspondiente |
| html |Descripción de la entidad, a menudo mostrada en el portal para desarrolladores |
| xml |Policy statements |
| css |Hojas de estilo para la personalización del portal para desarrolladores |

Estos archivos pueden crear, eliminar, editar y administrar en el sistema de archivos local, y los cambios se pueden implementar de nuevo en la instancia de servicio de Administración de API.

> [!NOTE]
> Las siguientes entidades no están en el repositorio de Git y no se pueden configurar mediante Git.
> 
> * Usuarios
> * Suscripciones
> * Propiedades
> * Entidades del portal de desarrolladores distintas de los estilos
> 
> 

### Carpeta raíz de la administración de API
La carpeta raíz `api-management` contiene un archivo `configuration.json` con información de nivel superior sobre la instancia de servicio en el formato siguiente.

    {
      "settings": {
        "RegistrationEnabled": "True",
        "UserRegistrationTerms": null,
        "UserRegistrationTermsEnabled": "False",
        "UserRegistrationTermsConsentRequired": "False",
        "DelegationEnabled": "False",
        "DelegationUrl": "",
        "DelegatedSubscriptionEnabled": "False",
        "DelegationValidationKey": ""
      },
      "$ref-policy": "api-management/policies/global.xml"
    }

Los primeros cuatro valores (`RegistrationEnabled`, `UserRegistrationTerms`, `UserRegistrationTermsEnabled` y `UserRegistrationTermsConsentRequired`) se asignan a la siguiente configuración en la pestaña **Identidades** de la sección **Seguridad**.

| Configuración de identidad | Se asigna a |
| --- | --- |
| RegistrationEnabled |Casilla **Redirigir a los usuarios anónimos a la página de inicio de sesión** |
| UserRegistrationTerms |Cuadro de texto **Condiciones de uso del registro de usuario** |
| UserRegistrationTermsEnabled |Casilla **Mostrar condiciones de uso en la página de registro** |
| UserRegistrationTermsConsentRequired |Casilla **Requerir consentimiento** |

![Configuración de identidad][api-management-identity-settings]

La cuatro valores siguientes (`DelegationEnabled`, `DelegationUrl`, `DelegatedSubscriptionEnabled` y `DelegationValidationKey`) se asignan a la siguiente configuración en la pestaña **Delegación** de la sección **Seguridad**.

| Configuración de delegación | Se asigna a |
| --- | --- |
| DelegationEnabled |Casilla **Delegar inicio de sesión y registro** |
| DelegationUrl |Cuadro de texto **Dirección URL del punto de conexión de delegación** |
| DelegatedSubscriptionEnabled |Casilla **Delegar suscripción de producto** |
| DelegationValidationKey |Cuadro de texto **Delegar clave de validación** |

![Configuración de delegación][api-management-delegation-settings]

El valor final, `$ref-policy`, se asigna al archivo de instrucciones de directiva global para la instancia de servicio.

### carpeta de API
La carpeta `apis` contiene una carpeta para cada API de la instancia de servicio que contiene los elementos siguientes.

* `apis<api name>\configuration.json`: es la configuración de la API y contiene información acerca de la dirección URL del servicio back-end y las operaciones. Se trata de la misma información que se devolvería si se llamase a [Obtener una API específica](https://msdn.microsoft.com/library/azure/dn781423.aspx#GetAPI) con `export=true` en formato `application/json`.
* `apis<api name>\api.description.html`: es la descripción de la API y corresponde a la propiedad `description` de la [entidad de API](https://msdn.microsoft.com/library/azure/dn781423.aspx#EntityProperties).
* `apis<api name>\operations`: esta carpeta contiene archivos `<operation name>.description.html` que se asignan a las operaciones de la API. Cada archivo contiene la descripción de una única operación en la API que se asigna a la propiedad `description` de la [entidad de operación](https://msdn.microsoft.com/library/azure/dn781423.aspx#OperationProperties) en la API de REST.

### carpeta de grupos
La carpeta `groups` contiene una carpeta para cada grupo definido en la instancia de servicio.

* `groups<group name>\configuration.json`: es la configuración para el grupo. Se trata de la misma información que se devolvería si se llamase a la operación [Obtener un grupo específico](https://msdn.microsoft.com/library/azure/dn776329.aspx#GetGroup).
* `groups<group name>\description.html`: es la descripción del grupo y se corresponde con la propiedad `description` de la [entidad de servicio](https://msdn.microsoft.com/library/azure/dn776329.aspx#EntityProperties).

### carpeta de directivas
La carpeta `policies` contiene las instrucciones de directiva para la instancia de servicio.

* `policies\global.xml`: contiene las directivas definidas en el ámbito global para la instancia de servicio.
* `policies\apis<api name>`: si tiene directivas definidas en el ámbito de la API, se encuentran en esta carpeta.
* Carpeta `policies\apis<api name><operation name>`: si tiene directivas definidas en el ámbito de la operación, se encuentran en esta carpeta en archivos `<operation name>.xml` que se asignan a las instrucciones de directiva para cada operación.
* `policies\products`: si tiene directivas definidas en el ámbito del producto, se encuentran esta carpeta, que contiene archivos `<product name>.xml` que se asignan a las instrucciones de directiva para cada producto.

### carpeta portalStyles
La carpeta `portalStyles` contiene la configuración y las hojas de estilo para las personalizaciones del portal para desarrolladores de la instancia de servicio.

* `portalStyles\configuration.json`: contiene los nombres de las hojas de estilos usadas por el portal de desarrolladores
* `portalStyles<style name>.css`: cada archivo `<style name>.css` contiene estilos para el portal para desarrolladores (`Preview.css` y `Production.css` de forma predeterminada).

### carpeta de productos
La carpeta `products` contiene una carpeta para cada producto que se define en la instancia de servicio.

* `products<product name>\configuration.json`: es la configuración del producto. Se trata de la misma información que se devolvería si se llamase a la operación [Obtener un producto específico](https://msdn.microsoft.com/library/azure/dn776336.aspx#GetProduct).
* `products<product name>\product.description.html`: es la descripción del producto y se corresponde con la propiedad `description` de la [entidad de producto](https://msdn.microsoft.com/library/azure/dn776336.aspx#Product) de la API de REST.

### plantillas
La carpeta `templates` contiene la configuración para las [plantillas de correo electrónico](api-management-howto-configure-notifications.md) de la instancia de servicio.

* `<template name>\configuration.json`: es la configuración de la plantilla de correo electrónico.
* `<template name>\body.html`: es el cuerpo de la plantilla de correo electrónico.

## Pasos siguientes
Para obtener información sobre otras formas de administrar la instancia de servicio, consulte:

* Administrar la instancia de servicio con los siguientes cmdlets de PowerShell
  * [Azure API Management Deployment Management Cmdlets (Cmdlets de administración de la implementación de Administración de API de Azure)](https://msdn.microsoft.com/library/azure/mt619282.aspx)
  * [Azure API Management Service Management Cmdlets (Cmdlets de administración del servicio Administración de API de Azure)](https://msdn.microsoft.com/library/azure/mt613507.aspx)
* Administrar la instancia de servicio en el portal para editores
  * [Administración de su primera API en Administración de API de Azure](api-management-get-started.md)
* Administrar la instancia de servicio mediante la API de REST
  * [API Management REST API (API de REST de Administración de API)](https://msdn.microsoft.com/library/azure/dn776326.aspx)

## Vea un vídeo de introducción.
> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Configuration-over-Git/player]
> 
> 

[api-management-enable-git]: ./media/api-management-configuration-repository-git/api-management-enable-git.png
[api-management-git-enabled]: ./media/api-management-configuration-repository-git/api-management-git-enabled.png
[api-management-save-configuration]: ./media/api-management-configuration-repository-git/api-management-save-configuration.png
[api-management-save-configuration-confirm]: ./media/api-management-configuration-repository-git/api-management-save-configuration-confirm.png
[api-management-configuration-status]: ./media/api-management-configuration-repository-git/api-management-configuration-status.png
[api-management-configuration-git-clone]: ./media/api-management-configuration-repository-git/api-management-configuration-git-clone.png
[api-management-generate-password]: ./media/api-management-configuration-repository-git/api-management-generate-password.png
[api-management-password]: ./media/api-management-configuration-repository-git/api-management-password.png
[api-management-git-configure]: ./media/api-management-configuration-repository-git/api-management-git-configure.png
[api-management-configuration-deploy]: ./media/api-management-configuration-repository-git/api-management-configuration-deploy.png
[api-management-identity-settings]: ./media/api-management-configuration-repository-git/api-management-identity-settings.png
[api-management-delegation-settings]: ./media/api-management-configuration-repository-git/api-management-delegation-settings.png
[api-management-git-icon-enable]: ./media/api-management-configuration-repository-git/api-management-git-icon-enable.png

<!---HONumber=AcomDC_0810_2016-->