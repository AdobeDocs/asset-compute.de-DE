---
title: Lernen Sie, wie Sie  [!DNL Asset Compute Service] erweitern
description: Wann und wie die Funktionalität von [!DNL Asset Compute Service] für die Verarbeitung benutzerdefinierter Assets erweitert wird.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
TQID: https://experienceleague.adobe.com/T-Q9ssFC8lirvK3Wl7goCbvso0k--ubsO64cF5MmNn0
product_v2: id: d09181b5-a36a-43de-ba01-36641440bc43id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
role_v2: id: ff6a42d2-313e-452e-93a6-792e4fad9ff8
source-git-commit: 2510f77fed8d0f0708e09f32d0b13a437d2ede4f
workflow-type: tm+mt
source-wordcount: 304
ht-degree: 91%

---

# Einführung in die Erweiterbarkeit {#introduction-to-extensibilty}

Viele Ausgabedarstellungsanforderungen, wie die Konvertierung in Formate und die Größenanpassung von Bildern, erfolgen über [Verarbeitungsprofile in  [!DNL Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview). Komplexere Geschäftsanforderungen erfordern möglicherweise eine individuell erstellte Lösung, die den Anforderungen eines Unternehmens entspricht. [!DNL Asset Compute Service] kann erweitert werden, indem Sie benutzerdefinierte Programme erstellen, die von den Verarbeitungsprofilen in [!DNL Experience Manager] aufgerufen werden. Diese benutzerdefinierten Programme decken die [unterstützten Anwendungsfälle](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) ab.

>[!NOTE]
>
>[!DNL Asset Compute Service] ist nur zur Verwendung mit [!DNL Experience Manager] as a [!DNL Cloud Service] verfügbar.

Bei den benutzerdefinierten Programmen handelt es sich um Headless-Apps von [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder). Die Erweiterung von [!DNL Asset Compute Service] mit benutzerdefinierten Programmen wird durch das [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) und die Entwicklungs-Tools von Adobe Developer App Builder vereinfacht. Mit diesen Tools können sich Entwickelnde auf Geschäftslogik konzentrieren. Das Erstellen benutzerdefinierter Anwendungen ist so einfach wie das Erstellen einer Server-losen Adobe [!DNL I/O Runtime]-Aktion. Es handelt sich dabei um eine einzige JavaScript-Funktion von Node.js. Dies wird durch das [grundlegende Beispiel eines benutzerdefinierten Programms](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) veranschaulicht.

## Voraussetzungen und Bereitstellungsanforderungen {#prerequisites-and-provisioning}

Stellen Sie sicher, dass die folgenden Voraussetzungen erfüllt sind:

* Die Adobe Developer App Builder-Tools sind auf Ihrem Gerät installiert.
* Eine [!DNL Experience Cloud]-Organisation. Weitere Informationen finden Sie unter [Starten Ihrer App Builder-Journey](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#acquire-access-and-credentials).
* [!DNL Experience Manager] as a [!DNL Cloud Service] muss für die Experience Cloud-Organisation aktiviert sein.
* Die [!DNL Adobe Experience Cloud]-Organisation ist Teil des [!DNL Adobe Developer App Builder]-Sneak-Peek-Programms für Entwickelnde. Erfahren Sie, [wie Sie den Zugriff beantragen](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/set-up#access-and-credentials).
* Stellen Sie für die Entwickelnden eine Entwicklerrolle oder Administratorberechtigungen in der Organisation sicher.
* Stellen Sie sicher, dass Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) lokal installiert ist.

<!-- 
TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
