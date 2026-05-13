---
title: Bereitstellen eines benutzerdefinierten  [!DNL Asset Compute Service] -Programms
description: Bereitstellen eines benutzerdefinierten  [!DNL Asset Compute Service] -Programms.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
TQID: https://experienceleague.adobe.com/JN29pTaNB93DKALUqIbXhwswlzHiYQSowFZAbgHA5TA
product_v2:
  - id: d09181b5-a36a-43de-ba01-36641440bc43
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
role_v2:
  - id: ff6a42d2-313e-452e-93a6-792e4fad9ff8
source-git-commit: 2510f77fed8d0f0708e09f32d0b13a437d2ede4f
workflow-type: tm+mt
source-wordcount: 210
ht-degree: 100%

---

# Bereitstellen eines benutzerdefinierten Programms {#deploy-custom-application}

Verwenden Sie zum Bereitstellen des Programms den Befehl [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy). Der Befehl zeigt im Terminal eine URL für den Zugriff auf das benutzerdefinierte Programm an. Die URL hat das Format `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Verwenden Sie den Befehl [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action), um die URL ohne erneute Bereitstellung des Programms abzurufen.

Verwenden Sie die URL in einem [Verarbeitungsprofil in  [!DNL Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use), um Ihr Programm mit [!DNL Experience Manager] as a [!DNL Cloud Service] zu integrieren.

Stellen Sie sicher, dass Ihr App Builder-Projekt und der Arbeitsbereich der Umgebung von [!DNL Experience Manager] as a [!DNL Cloud Service] entsprechen, in der Sie Ihre Aktion durchführen möchten. Es gibt verschiedene Umgebungen für Entwicklung, Staging und Produktion. Sie können die Umgebung überprüfen, indem Sie die `AIO_runtime_*`-Anmeldeinformationen überprüfen, die in Ihrer ENV-Datei im Stamm Ihres Adobe Developer App Builder-Programms definiert sind. Um beispielsweise eine Bereitstellung in einem `Stage`-Arbeitsbereich vorzunehmen, hat `AIO_runtime_namespace` das Format `xxxxxx_xxxxxxxxx_stage`. Verwenden Sie zur Integration in eine Produktionsumgebung von [!DNL Experience Manager] as a [!DNL Cloud Service] die Programm-URLs aus Ihrem Adobe Developer App Builder-`Production`-Arbeitsbereich.

>[!CAUTION]
>
>Verwenden Sie keine persönlichen Arbeitsbereiche in wichtigen [!DNL Experience Manager]-Umgebungen.

>[!MORELIKETHIS]
>
>* [Verstehen und Verwalten von Umgebungen in  [!DNL Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
