---
title: Muestra de datos en contenedores de blob de Azure, SQL Server y tablas de Hive | Microsoft Docs
description: Exploración de datos almacenados en diversos entornos de Azure
services: machine-learning
documentationcenter: ''
author: bradsev
manager: jhubbard
editor: cgronlun

ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/19/2016
ms.author: fashah;garye;bradsev

---
# <a name="heading"></a>Muestra de datos en contenedores de blob de Azure, SQL Server y tablas de Hive
Este documento incluye vínculos a temas que trata cómo muestrear los datos que se almacenan en una de las tres diferentes ubicaciones de Azure:

* Se realiza la muestra de los **datos del contenedor de blobs de Azure** descargándolos mediante programación y, a continuación, realizando un muestreo de ellos con el código Python de ejemplo.
* Se realiza el muestreo de los **datos de SQL Server** con SQL y el lenguaje de programación Python.
* Se realiza el muestreo de los **datos de las tablas de Hive** mediante consultas de Hive.

El **menú** siguiente vincula a temas que describen cómo se realiza el muestreo de datos desde cada uno de estos entornos de Azure Storage.

[!INCLUDE [cap-sample-data-selector](../../includes/cap-sample-data-selector.md)]

Esta tarea de muestreo es un paso en el [proceso de ciencia de datos en equipos (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/).

## ¿Por qué realizar un muestreo de datos?
Si el conjunto de datos que pretende analizar es grande, suele ser una buena idea reducirlo a un tamaño más pequeño, pero representativo, que sea más manejable. Esto facilita la comprensión y exploración de los datos, y el diseño de características. Su rol en el proceso de análisis de Cortana es permitir la rápida creación de prototipos de las funciones de procesamiento de datos y de los modelos de aprendizaje automático.

<!---HONumber=AcomDC_0921_2016-->