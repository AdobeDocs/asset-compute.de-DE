---
title: Einführung in  [!DNL Asset Compute Service]
description: '[!DNL Asset Compute Service] ist ein Cloud-nativer Asset-Verarbeitungs-Service, der die Komplexität reduziert und die Skalierbarkeit verbessert.'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '278'
ht-degree: 100%

---

# Übersicht über [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] ist ein skalierbarer und erweiterbarer Service von [!DNL Adobe Experience Cloud] zur Verarbeitung digitaler Assets. Sie können Bild-, Video-, Dokument- und andere Dateiformate in verschiedene Ausgabedarstellungen umwandeln, einschließlich Miniaturansichten, extrahiertem Text und Metadaten sowie Archiven.

Entwickelnde können benutzerdefinierte Asset-Anwendungen (auch als benutzerdefinierte Sekundäranwendungen oder Worker bezeichnet) als Plug-in einbinden, um benutzerdefinierte Anwendungsfälle abzudecken. Der Service funktioniert in Adobe [!DNL I/O Runtime]. Er kann durch Headless-[!DNL Adobe Developer App Builder]-Apps in Node.js erweitert werden. Diese können benutzerdefinierte Vorgänge ausführen, z. B. externe APIs aufrufen, um Bildvorgänge durchzuführen oder [!DNL Adobe Sensei]-Unterstützung zu nutzen.

[!DNL Adobe Developer App Builder] ist ein Framework zum Erstellen und Bereitstellen benutzerdefinierter Web-Anwendungen in Adobe [!DNL I/O Runtime], um Adobe Experience Cloud-Lösungen zu erweitern. Um benutzerdefinierte Anwendungen zu kreieren, können die Entwickelnden [!DNL React Spectrum] (das Adobe-Toolkit für Benutzeroberflächen) nutzen, Microservices erstellen, benutzerdefinierte Ereignisse erzeugen und APIs orchestrieren. Siehe [Dokumentation zu Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>Zurzeit kann [!DNL Asset Compute Service] nur über [!DNL Experience Manager] as a [!DNL Cloud Service] verwendet werden. Admins erstellen Verarbeitungsprofile, die den [!DNL Asset Compute Service] aufrufen können, um Assets zur Verarbeitung zu übergeben. Weitere Informationen finden Sie unter [Verwenden von Asset-Microservices und Verarbeitungsprofilen](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

## Unterstützte Anwendungsfälle von [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] unterstützt einige gängige Geschäftsanwendungsfälle wie die grundlegende Bildverarbeitung, Adobe-anwendungsspezifische Konvertierungen und das Erstellen benutzerdefinierter Programme, die komplexe Geschäftsanforderungen koordinieren.

Sie können den Webservice [!DNL Asset Compute] verwenden, um Miniaturansichten für verschiedene Dateitypen und hochwertige Bild-Renderings für die [unterstützten Dateiformate](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/file-format-support) zu generieren. Weitere Informationen finden Sie unter [Anwendungsfälle, die von der benutzerdefinierten Konfiguration unterstützt werden](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>Der Service bietet keine Asset-Speicherung. Diese wird von den Benutzern bereitgestellt. Dasselbe gilt für Verweise auf die Speicherorte von Quell- und Ausgabedarstellungsdateien im Cloud-Speicher.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Übersicht über die Asset-Verarbeitung mit Asset-Microservices in  [!DNL Adobe Experience Manager]  as a  [!DNL Cloud Service]](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview).
>* [Dokumentation zu Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).
>* [Unterstützte Dateiformate für die Verarbeitung](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/file-format-support).

<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
