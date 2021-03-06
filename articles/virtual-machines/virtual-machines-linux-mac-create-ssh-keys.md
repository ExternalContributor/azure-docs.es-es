---
title: "Creación de claves SSH en Linux y Mac | Microsoft Docs"
description: "Generar y usar claves SSH en Linux y Mac para los modelos de implementación clásica en Azure y el Administrador de recursos."
services: virtual-machines-linux
documentationcenter: 
author: vlivech
manager: timlt
editor: 
tags: 
ms.assetid: 34ae9482-da3e-4b2d-9d0d-9d672aa42498
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/25/2016
ms.author: v-livech
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 689445bc656b5cdebc7689f7fec6e2ea2683576e


---
# <a name="create-ssh-keys-on-linux-and-mac-for-linux-vms-in-azure"></a>Creación de claves SSH en Linux y Mac para máquinas virtuales de Linux en Azure
Con un par de claves SSH, puede crear máquinas virtuales en Azure que usen de forma predeterminada claves SSH para la autenticación, lo que elimina la necesidad de usar contraseñas para iniciar sesión.  Las contraseñas se pueden adivinar y dejan las máquinas virtuales expuestas a intentos constantes de adivinarlas por fuerza bruta. Las máquinas virtuales creadas con plantillas de Azure o `azure-cli` pueden incluir la clave pública SSH como parte de la implementación, por lo que no es necesario configurarla después de esta.  Si va a conectarse a una máquina virtual Linux desde Windows, consulte [este documento.](virtual-machines-linux-ssh-from-windows.md)

Este artículo requiere:

* una cuenta de Azure ([obtenga aquí una prueba gratuita](https://azure.microsoft.com/pricing/free-trial/));
* la [CLI de Azure](../xplat-cli-install.md) en la que se inició sesión con `azure login`;
* la CLI de Azure *debe estar en* el modo Azure Resource Manager `azure config mode arm`.

## <a name="quick-commands"></a>Comandos rápidos
En los siguientes comandos, reemplace los ejemplos por sus propias elecciones.

Las claves SSH se mantienen de forma predeterminada en el directorio `.ssh`.  

```bash
cd ~/.ssh/
```

Si no tiene un directorio `~/.ssh`, el comando `ssh-keygen` lo creará automáticamente con los permisos correctos.

```bash
ssh-keygen -t rsa -b 2048 -C "myusername@myserver"
```

Escriba el nombre del archivo que se guarda en el directorio `~/.ssh/`:

```bash
id_rsa
```

Escriba la frase de contraseña para id_rsa:

```bash
correct horse battery staple
```

Ahora hay un par de claves SSH `id_rsa` e `id_rsa.pub` en el directorio `~/.ssh`.

```bash
ls -al ~/.ssh
```

Agregue la clave recién creada a `ssh-agent` en Linux y Mac (también se agrega a Llaveros en OSX):

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

Copie la clave pública SSH en el servidor Linux:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub myusername@myserver
```

Pruebe el inicio de sesión mediante claves en lugar de contraseña:

```bash
ssh -o PreferredAuthentications=publickey -o PubkeyAuthentication=yes -i ~/.ssh/id_rsa myusername@myserver
Last login: Tue April 12 07:07:09 2016 from 66.215.22.201
$
```

## <a name="detailed-walkthrough"></a>Tutorial detallado
El uso de claves públicas y privadas SSH es la manera más fácil de iniciar sesión en los servidores de Linux. La [criptografía de clave pública](https://en.wikipedia.org/wiki/Public-key_cryptography) ofrece una manera mucho más segura que las contraseñas de iniciar sesión en la máquina virtual Linux o BSD en Azure, que pueden ser forzadas con mucha más facilidad. La clave pública se puede compartir con cualquiera. Sin embargo, la clave privada es propiedad exclusivamente suya (o de su infraestructura de seguridad local).  La clave privada SSH debe tener una [contraseña muy segura ](https://www.xkcd.com/936/) (origen:[xkcd.com](https://xkcd.com)) para protegerla.  Esta contraseña solo sirve para acceder a la clave privada SSH y **no es** la contraseña de la cuenta de usuario.  Cuando agrega una contraseña a la clave SSH, cifra la clave privada para que no se pueda usar sin la contraseña que la desbloquea.  Si un atacante robara la clave privada y esta no tuviera contraseña, podría usarla para iniciar sesión en los servidores que tuvieran instalada la clave pública correspondiente.  Si una clave privada está protegida con una contraseña, el atacante no podrá usarla, ya que la contraseña constituye un nivel adicional de seguridad en la infraestructura de Azure.

En este artículo se crean archivos de clave con el formato *ssh-rsa* , que son los recomendados para las implementaciones en Resource Manager.  *ssh-rsa* se requieren en el [Portal](https://portal.azure.com) tanto para las implementaciones clásicas como para las de Resource Manager.

## <a name="create-the-ssh-keys"></a>Creación de las claves SSH
Azure requiere al menos claves públicas y privadas de 2048 bits con el formato ssh-rsa. Para crear las claves, use `ssh-keygen`, que realiza una serie de preguntas y después escribe una clave privada y una clave pública correspondiente. Cuando se crea una máquina virtual de Azure, la clave pública se copia en `~/.ssh/authorized_keys`.  Las claves SSH en `~/.ssh/authorized_keys` se utilizan para presentar un desafío al cliente, que debe proporcionar la clave privada correspondiente en una conexión de inicio de sesión SSH.

## <a name="using-sshkeygen"></a>Uso de ssh-keygen
Este comando crea un par de claves SSH protegidas con contraseña (cifradas) mediante RSA de 2048 bits e incluye un comentario para que se pueda identificar fácilmente.  

Para empezar, cambie de directorio, para que todas las claves SSH se creen en ese directorio.

```bash
cd ~/.ssh
```

Si no tiene un directorio `~/.ssh`, el comando `ssh-keygen` lo creará automáticamente con los permisos correctos.

```bash
ssh-keygen -t rsa -b 2048 -C "myusername@myserver"
```

*Explicación del comando*

`ssh-keygen` = programa usado para crear las claves;

`-t rsa` = el tipo de clave que se va a crear que está en el [formato RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)

`-b 2048` = bits de la clave;

`-C "myusername@myserver"` = comentario que se anexa al final del archivo de clave pública para identificarlo fácilmente.  Normalmente se usa un correo electrónico como comentario, pero puede utilizar lo que mejor le vaya para su infraestructura.

### <a name="using-pem-keys"></a>Uso de claves PEM
Si usa el modelo de implementación clásica (Portal de Azure clásico o la CLI de administración de servicios de Azure `asm`), es posible que tenga que usar claves SSH con formato PEM para acceder a las máquinas virtuales Linux.  Aquí se muestra cómo crear una clave PEM a partir de una clave SSH pública y un certificado x509 existentes.

Para crear una clave con formato PEM a partir de una clave pública SSH existente:

```bash
ssh-keygen -f ~/.ssh/id_rsa.pub -e > ~/.ssh/id_ssh2.pem
```

## <a name="example-of-sshkeygen"></a>Ejemplo de ssh-keygen
```bash
ssh-keygen -t rsa -b 2048 -C "myusername@myserver"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/myusername/.ssh/id_rsa): id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa.
Your public key has been saved in id_rsa.pub.
The key fingerprint is:
14:a3:cb:3e:78:ad:25:cc:55:e9:0c:08:e5:d1:a9:08 myusername@myserver
The key's randomart image is:
+--[ RSA 2048]----+
|        o o. .   |
|      E. = .o    |
|      ..o...     |
|     . o....     |
|      o S =      |
|     . + O       |
|      + = =      |
|       o +       |
|        .        |
+-----------------+
```

Archivos de clave guardados:

`Enter file in which to save the key (/home/myusername/.ssh/id_rsa): id_rsa`

El nombre del par de claves en este artículo.  Tener un par de claves denominado **id_rsa** es el valor predeterminado y algunas herramientas podrían esperar **id_rsa** como nombre del archivo de la clave privada, así que es buena idea tener uno. El directorio `~/.ssh/` es la ubicación predeterminada para los pares de claves SSH y el archivo de configuración de SSH.

```bash
ls -al ~/.ssh
-rw------- 1 myusername staff  1675 Aug 25 18:04 id_rsa
-rw-r--r-- 1 myusername staff   410 Aug 25 18:04 rsa.pub
```
Una lista del directorio `~/.ssh` . `ssh-keygen` crea el directorio `~/.ssh` si no existe y también establece los modos de propiedad y de archivo correctos.

Contraseña de la clave:

`Enter passphrase (empty for no passphrase):`

`ssh-keygen` se refiere a una contraseña como "frase de contraseña".  Es *muy* recomendable agregar una contraseña a los pares de clave. Sin una contraseña que los proteja, cualquiera que disponga del archivo de clave privada puede usarlo para iniciar sesión en cualquier servidor que tenga la clave pública correspondiente. Agregar una contraseña ofrece más protección en caso de que alguien obtenga acceso al archivo de clave privada, lo que le dará tiempo para cambiar las claves que se usan para autenticarse.

## <a name="using-sshagent-to-store-your-private-key-password"></a>Uso de ssh-agent para almacenar la contraseña de clave privada
Para no tener que escribir la contraseña del archivo de clave privada con cada inicio de sesión SSH, puede usar `ssh-agent` para almacenar en caché la contraseña del archivo de clave privada. Si usa Mac, Llaveros en OSX almacena de forma segura las contraseñas de clave privada cuando invoque `ssh-agent`.

En primer lugar, compruebe que `ssh-agent` se esté ejecutando.

```bash
eval "$(ssh-agent -s)"
```

Ahora, agregue la clave privada a `ssh-agent` mediante el comando `ssh-add`.

```bash
ssh-add ~/.ssh/id_rsa
```

La contraseña de clave privada está almacenada ahora en `ssh-agent`.

## <a name="create-and-configure-an-ssh-config-file"></a>Creación y configuración de un archivo de configuración de SSH
Es procedimiento recomendado crear y configurar un archivo `~/.ssh/config` para acelerar los inicios de sesión y optimizar el comportamiento del cliente SSH.

En el ejemplo siguiente se muestra una configuración estándar.

### <a name="create-the-file"></a>Creación del archivo
```bash
touch ~/.ssh/config
```

### <a name="edit-the-file-to-add-the-new-ssh-configuration"></a>Edite el archivo para agregar la nueva configuración de SSH:
```bash
vim ~/.ssh/config
```

### <a name="example-sshconfig-file"></a>Archivo `~/.ssh/config` de ejemplo:
```bash
# Azure Keys
Host fedora22
  Hostname 102.160.203.241
  User myusername
# ./Azure Keys
# Default Settings
Host *
  PubkeyAuthentication=yes
  IdentitiesOnly=yes
  ServerAliveInterval=60
  ServerAliveCountMax=30
  ControlMaster auto
  ControlPath ~/.ssh/SSHConnections/ssh-%r@%h:%p
  ControlPersist 4h
  IdentityFile ~/.ssh/id_rsa
```

Esta configuración de SSH ofrece secciones para cada servidor para que cada uno tenga su propio par de claves dedicado. La configuración predeterminada (`Host *`) es para los hosts que no coincidan con ninguno de los hosts específicos más arriba en el archivo de configuración.

### <a name="config-file-explained"></a>Explicación del archivo de configuración
`Host` = nombre del host al que se llama en el terminal.  `ssh fedora22` indica a `SSH` que use los valores del bloque de configuración con la etiqueta `Host fedora22`. NOTA: Puede ser cualquier etiqueta lógica para su uso y no representa el nombre de host real de ningún servidor.

`Hostname 102.160.203.241` = dirección IP o nombre DNS del servidor al que se está accediendo.

`User myusername` = cuenta de usuario remoto que se usará al iniciar sesión en el servidor.

`PubKeyAuthentication yes` = indica a SSH que desea usar una clave SSH para iniciar sesión.

`IdentityFile /home/myusername/.ssh/id_id_rsa` = clave privada SSH y clave pública correspondiente que se usan para la autenticación.

## <a name="ssh-into-linux-without-a-password"></a>SSH en Linux sin contraseña
Ahora que tiene un par de claves SSH y un archivo de configuración de SSH configurado, puede iniciar sesión en la máquina virtual Linux de forma rápida y segura. La primera vez que inicie sesión en un servidor mediante una clave SSH, el símbolo del sistema le pedirá la frase de contraseña para ese archivo de clave.

```bash
ssh fedora22
```

### <a name="command-explained"></a>Explicación del comando
Cuando se ejecuta `ssh fedora22`, primero SSH busca y carga la configuración del bloque `Host fedora22` y, después, carga la configuración restante del último bloque, `Host *`.

## <a name="next-steps"></a>Pasos siguientes
El siguiente paso es crear máquinas virtuales de Linux en Azure con la nueva clave pública SSH.  Las máquinas virtuales de Azure que se crean con una clave pública SSH como inicio de sesión están mejor protegidas que las creadas con contraseñas, el método de inicio de sesión predeterminado.  Las máquinas virtuales de Azure creadas con claves SSH están configuradas de forma predeterminada con las contraseñas deshabilitadas, a fin de evitar los intentos de adivinarlas por fuerza bruta.

* [Creación de una máquina virtual Linux protegida mediante una plantilla de Azure](virtual-machines-linux-create-ssh-secured-vm-from-template.md)
* [Creación de una máquina virtual Linux protegida mediante el Portal de Azure](virtual-machines-linux-quick-create-portal.md)
* [Crear una máquina virtual de Linux segura mediante la CLI de Azure](virtual-machines-linux-quick-create-cli.md)




<!--HONumber=Nov16_HO2-->


