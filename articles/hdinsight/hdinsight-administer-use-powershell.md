---
title: Administración de clústeres de Hadoop en HDInsight con Azure PowerShell | Microsoft Docs
description: Vea cómo realizar tareas administrativas para clústeres de Hadoop en HDInsight usando Azure PowerShell.
services: hdinsight
editor: cgronlun
manager: jhubbard
tags: azure-portal
author: mumian
documentationcenter: ''

ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/10/2016
ms.author: jgao

---
# Administración de clústeres de Hadoop en HDInsight con PowerShell de Azure
[!INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

Azure PowerShell es un potente entorno de scripting que puede usar para controlar y automatizar la implementación y la administración de sus cargas de trabajo en Azure. En este artículo, aprenderá a administrar clústeres de Hadoop en HDInsight de Azure con una consola local de PowerShell de Azure mediante el uso de Windows PowerShell. Para obtener más información acerca de los cmdlets de PowerShell de HDInsight, consulte [Documentación de referencia de los cmdlets de HDInsight][hdinsight-powershell-reference].

**Requisitos previos**

Antes de empezar este artículo, debe tener lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

## Azure PowerShell
[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

Si ha instalado la versión 0.9 x de Azure PowerShell, debe desinstalarla antes de instalar una versión más reciente.

Para comprobar la versión del PowerShell instalado:

    Get-Module *azure*

Para desinstalar la versión anterior, ejecute Programas y características en el panel de control.

## Creación de clústeres
Consulte [Crear clústeres basados en Linux en HDInsight con Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md).

## Enumeración de clústeres
Use el comando siguiente para enumerar todos los clústeres de la suscripción actual:

    Get-AzureRmHDInsightCluster

## Presentación de clústeres
Use el comando siguiente para mostrar los detalles de un clúster específico de la suscripción actual:

    Get-AzureRmHDInsightCluster -ClusterName <Cluster Name>

## Eliminación de clústeres
Use el comando siguiente para eliminar un clúster:

    Remove-AzureRmHDInsightCluster -ClusterName <Cluster Name>

También puede eliminar un clúster quitando el grupo de recursos que contiene el clúster. Tenga en cuenta que esto eliminará todos los recursos del grupo, incluida la cuenta de almacenamiento predeterminada.

    Remove-AzureRmResourceGroup -Name <Resource Group Name>

## Escalado de clústeres
La característica de escalado de clústeres permite cambiar la cantidad de nodos de trabajo que usa un clúster que se ejecuta en HDInsight de Azure sin necesidad de volver a crear el clúster.

> [!NOTE]
> Solo son compatibles los clústeres con la versión 3.1.3 de HDInsight, o superior. Si no está seguro de la versión del clúster, puede comprobar la página de propiedades. Vea [Enumeración y visualización de clústeres](hdinsight-administer-use-portal-linux.md#list-and-show-clusters).
> 
> 

A continuación se muestra el efecto que tiene cambiar la cantidad de nodos de datos de cada tipo de clúster compatible con HDInsight:

* Hadoop
  
    Puede aumentar sin ningún problema la cantidad de nodos de trabajo en un clúster de Hadoop que se encuentre en ejecución, sin que afecte a ningún trabajo pendiente o en ejecución. También se pueden enviar trabajos nuevos mientras la operación está en curso. Los errores que puedan surgir en una operación de escalado se enfrentan oportunamente, por lo que el clúster siempre queda en estado funcional.
  
    Cuando se realiza la reducción vertical de un clúster de Hadoop al disminuir la cantidad de nodos de datos, se reinician algunos de los servicios del clúster. Esto provoca que todos los trabajos pendientes y en ejecución fallen al completarse la operación de escalado. Sin embargo, puede volver a enviar los trabajos una vez finalizada la operación.
* HBase
  
    Puede agregar nodos sin problemas al clúster de HBase mientras se encuentra en ejecución, así como eliminarlos. Los servidores regionales se equilibran automáticamente en unos pocos minutos tras completar la operación de escalado. Sin embargo, puede equilibrar manualmente los servidores regionales iniciando sesión en el nodo principal del clúster y ejecutando los comandos siguientes desde una ventana del símbolo del sistema:
  
        >pushd %HBASE_HOME%\bin
        >hbase shell
        >balancer
* Storm
  
    Puede agregar o quitar sin problemas nodos de datos de su clúster de Storm mientras se encuentra en ejecución. Sin embargo, después de finalizar correctamente la operación de escalado, deberá volver a equilibrar la topología.
  
    Esto se puede realizar de dos formas:
  
  * La interfaz de usuario web de Storm
  * La herramienta de la interfaz de línea de comandos (CLI)
    
    Consulte la [documentación de Apache Storm](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html) para obtener más detalles.
    
    La interfaz de usuario web de Storm se encuentra disponible en el clúster de HDInsight:
    
    ![reequilibrio de escalado de storm de HDInsight](./media/hdinsight-administer-use-management-portal/hdinsight.portal.scale.cluster.storm.rebalance.png)
    
    El siguiente es un ejemplo de cómo usar el comando CLI para volver a equilibrar la topología de Storm:
    
    ## Reconfigure the topology "mytopology" to use 5 worker processes,
    ## the spout "blue-spout" to use 3 executors, and
    ## the bolt "yellow-bolt" to use 10 executors
      $ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

Para cambiar el tamaño del clúster de Hadoop con Azure PowerShell, ejecute el siguiente comando desde un equipo cliente:

    Set-AzureRmHDInsightClusterSize -ClusterName <Cluster Name> -TargetInstanceCount <NewSize>


## Concesión o revocación del acceso
Los clústeres de HDInsight tienen los siguientes servicios web HTTP (todos estos servicios tienen extremos RESTful):

* ODBC
* JDBC
* Ambari
* Oozie
* Templeton

De manera predeterminada, estos servicios se conceden para el acceso. Puede revocar/conceder el acceso. Para revocar:

    Revoke-AzureRmHDInsightHttpServicesAccess -ClusterName <Cluster Name>

Para conceder:

    $clusterName = "<HDInsight Cluster Name>"

    # Credential option 1
    $hadoopUserName = "admin"
    $hadoopUserPassword = "<Enter the Password>"
    $hadoopUserPW = ConvertTo-SecureString -String $hadoopUserPassword -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential($hadoopUserName,$hadoopUserPW)

    # Credential option 2
    #$credential = Get-Credential -Message "Enter the HTTP username and password:" -UserName "admin"

    Grant-AzureRmHDInsightHttpServicesAccess -ClusterName $clusterName -HttpCredential $credential

> [!NOTE]
> Al conceder/revocar el acceso, restablecerá el nombre de usuario y la contraseña del clúster.
> 
> 

Esto también se puede hacer a través del Portal. Consulte [Administración de HDInsight mediante el Portal de Azure][hdinsight-admin-portal]

## Actualización de las credenciales de usuario HTTP
Es el mismo procedimiento que [Concesión o revocación del acceso HTTP](#grant/revoke-access). Si al clúster se le ha concedido el acceso HTTP, primero debe revocarlo. Y después conceda el acceso con nuevas credenciales de usuario HTTP.

## Búsqueda de la cuenta de almacenamiento predeterminada
El siguiente script de Powershell muestra cómo obtener el nombre y la clave de la cuenta de almacenamiento predeterminada para un clúster.

    $clusterName = "<HDInsight Cluster Name>"

    $cluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroupName = $cluster.ResourceGroup
    $defaultStorageAccountName = ($cluster.DefaultStorageAccount).Replace(".blob.core.windows.net", "")
    $defaultBlobContainerName = $cluster.DefaultStorageContainer
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey 

## Búsqueda del grupo de recursos
En el modo de Resource Manager, cada clúster de HDInsight pertenece a un grupo de recursos de Azure. Para buscar el grupo de recursos:

    $clusterName = "<HDInsight Cluster Name>"

    $cluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroupName = $cluster.ResourceGroup


## Envío de trabajos
**Para enviar trabajos de MapReduce**

Vea [Ejecución de ejemplos de Hadoop MapReduce en HDInsight basado en Windows](hdinsight-run-samples.md).

**Para enviar trabajos de Hive**

Vea [Ejecución de consultas de Hive con PowerShell](hdinsight-hadoop-use-hive-powershell.md).

**Para enviar trabajos de Pigs**

Vea [Ejecución de trabajos de Pig mediante PowerShell](hdinsight-hadoop-use-pig-powershell.md).

**Para enviar trabajos de Sqoop**

Consulte [Uso de Sqoop con HDInsight](hdinsight-use-sqoop.md).

**Para enviar trabajos de Oozie**

Vea [Uso de Oozie con Hadoop para definir y ejecutar un flujo de trabajo en HDInsight](hdinsight-use-oozie.md).

## Carga de archivos de datos al almacenamiento de blobs de Azure
Consulte [Carga de datos en HDInsight][hdinsight-upload-data].

## Otras referencias
* [Documentación de referencia de los cmdlets de HDInsight][hdinsight-powershell-reference]
* [Administración de HDInsight mediante el Portal de Azure][hdinsight-admin-portal]
* [Administración de HDInsight con la interfaz de línea de comandos][hdinsight-admin-cli]
* [Creación de clústeres de HDInsight][hdinsight-provision]
* [Carga de datos en HDInsight][hdinsight-upload-data]
* [Envío de trabajos de Hadoop mediante programación][hdinsight-submit-jobs]
* [Introducción a HDInsight de Azure][hdinsight-get-started]

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[hdinsight-get-started]: hdinsight-hadoop-linux-tutorial-get-started.md
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-provision-custom-options]: hdinsight-provision-clusters.md#configuration
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md

[hdinsight-admin-cli]: hdinsight-administer-use-command-line.md
[hdinsight-admin-portal]: hdinsight-administer-use-management-portal.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-mapreduce]: hdinsight-use-mapreduce.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-flight]: hdinsight-analyze-flight-delay-data.md

[hdinsight-powershell-reference]: https://msdn.microsoft.com/library/dn858087.aspx

[powershell-install-configure]: powershell-install-configure.md

[image-hdi-ps-provision]: ./media/hdinsight-administer-use-powershell/HDI.PS.Provision.png

<!---HONumber=AcomDC_0914_2016-->