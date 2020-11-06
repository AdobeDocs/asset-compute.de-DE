---
title: Lernen Sie, wie Sie  [!DNL Asset Compute Service] erweitern.
description: Wann und wie die Funktionalität von  [!DNL Asset Compute Service]  für die Verarbeitung benutzerdefinierter Assets erweitert wird.
translation-type: ht
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: ht
source-wordcount: '275'
ht-degree: 100%

---


# Einführung in die Erweiterbarkeit {#introduction-to-extensibilty}

Viele Ausgabedarstellungsanforderungen, wie die Konvertierung in Formate und die Größenanpassung von Bildern, erfolgen über [Verarbeitungsprofile in  [!DNL Experience Manager]  as a Cloud Service](https://docs.adobe.com/content/help/de-DE/experience-manager-cloud-service/assets/asset-microservices-overview.html). Komplexere Geschäftsanforderungen erfordern möglicherweise eine individuell erstellte Lösung, die den Anforderungen eines Unternehmens entspricht. [!DNL Asset Compute Service] kann erweitert werden, indem Sie benutzerdefinierte Anwendungen erstellen, die von den Verarbeitungsprofilen in [!DNL Experience Manager] aufgerufen werden. Diese benutzerdefinierten Anwendungen decken die [unterstützten Anwendungsfälle](https://docs.adobe.com/content/help/de-DE/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) ab.

>[!NOTE]
>
>[!DNL Asset Compute Service] ist nur zur Verwendung mit [!DNL Experience Manager] as a Cloud Service verfügbar.

Bei den benutzerdefinierten Anwendungen handelt es sich um Headless-[Project Firefly](https://github.com/AdobeDocs/project-firefly)-Apps. Die Erweiterung von [!DNL Asset Compute Service] mit benutzerdefinierten Anwendungen wird durch das [Asset Compute-SDK](https://github.com/adobe/asset-compute-sdk) und die Entwickler-Tools von Project Firefly vereinfacht. Dadurch können sich Entwickler auf Geschäftslogik konzentrieren. Das Erstellen benutzerdefinierter Anwendungen ist so einfach wie das Erstellen einer Server-losen Adobe I/O Runtime-Aktion. Es handelt sich dabei um eine einzige JavaScript-Funktion von Node.js. Dies wird durch das [grundlegende Beispiel einer benutzerdefinierten Anwendung](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) veranschaulicht.

## Voraussetzungen und Bereitstellungsanforderungen {#prerequisites-and-provisioning}

Stellen Sie sicher, dass die folgenden Voraussetzungen erfüllt sind:

* Auf Ihrem Computer sind die Project Firefly-Tools installiert.
* Eine [!DNL Experience Cloud]-Organisation. Weitere Informationen finden Sie [hier](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* [!DNL Experience Manager] as a Cloud Service muss für die Experience-Organisation aktiviert sein.
* Die [!DNL Adobe Experience Cloud]-Organisation ist Teil des Developer Preview-Programms von [!DNL Project Firefly]. Lernen Sie, [wie Sie den Zugriff beantragen](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md).
* Stellen Sie für den Entwickler eine Entwicklerrolle oder Administratorberechtigungen in der Organisation sicher.
* Stellen Sie sicher, dass [Adobe I/O CLI](https://github.com/adobe/aio-cli) lokal installiert ist.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->