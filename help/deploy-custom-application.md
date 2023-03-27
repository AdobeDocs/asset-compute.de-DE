---
title: Bereitstellen eines benutzerdefinierten  [!DNL Asset Compute Service] -Programms
description: Bereitstellen eines benutzerdefinierten  [!DNL Asset Compute Service] -Programms.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 50f69e16772cee7f79a812f2b86f0ef0221db369
workflow-type: ht
source-wordcount: '190'
ht-degree: 100%

---

# Bereitstellen eines benutzerdefinierten Programms {#deploy-custom-application}

Verwenden Sie zum Bereitstellen des Programms den Befehl [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy). Der Befehl zeigt im Terminal eine URL für den Zugriff auf das benutzerdefinierte Programm an. Die URL hat das Format `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Verwenden Sie den Befehl [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action), um die URL ohne erneute Bereitstellung des Programms abzurufen.

Verwenden Sie die URL in einem [Verarbeitungsprofil in  [!DNL Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=de), um Ihr Programm mit [!DNL Experience Manager] as a [!DNL Cloud Service] zu integrieren.

Stellen Sie sicher, dass Ihr App Builder-Projekt und der Arbeitsbereich der Umgebung von [!DNL Experience Manager] as a [!DNL Cloud Service] entsprechen, in der Sie Ihre Aktion durchführen möchten. Es gibt verschiedene Umgebungen für Entwicklung, Staging und Produktion. Sie können die Umgebung überprüfen, indem Sie die `AIO_runtime_*`-Anmeldeinformationen überprüfen, die in Ihrer ENV-Datei im Stamm Ihres Adobe Developer App Builder-Programms definiert sind. Um beispielsweise eine Bereitstellung in einem `Stage`-Arbeitsbereich vorzunehmen, hat `AIO_runtime_namespace` das Format `xxxxxx_xxxxxxxxx_stage`. Verwenden Sie zur Integration in eine Produktionsumgebung von [!DNL Experience Manager] as a [!DNL Cloud Service] die Programm-URLs aus Ihrem Adobe Developer App Builder-`Production`-Arbeitsbereich.

>[!CAUTION]
>
>Verwenden Sie keinen persönlichen Arbeitsbereich für wichtige [!DNL Experience Manager]-Umgebungen.

>[!MORELIKETHIS]
>
>* [Verstehen und Verwalten von Umgebungen in  [!DNL Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html?lang=de).

