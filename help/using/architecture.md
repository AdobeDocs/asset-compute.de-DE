---
title: Architektur von  [!DNL Asset Compute Service]
description: So arbeiten [!DNL Asset Compute Service] -API, Programme und SDK zusammen, um einen Cloud-nativen Asset-Verarbeitungs-Service bereitzustellen.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: ht
source-wordcount: '478'
ht-degree: 100%

---

# Architektur von [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] basiert auf der Server-losen Adobe [!DNL `I/O Runtime`]-Plattform. Der Service bietet Unterstützung mit Adobe Sensei-Inhalts-Services für Assets. Der aufrufende Client (es wird nur [!DNL Experience Manager] as a [!DNL Cloud Service] unterstützt) erhält die von Adobe Sensei generierten Informationen, nach denen er für das Asset gesucht hat. Die zurückgegebenen Informationen liegen im JSON-Format vor.

[!DNL Asset Compute Service] kann erweitert werden, indem benutzerdefinierte Programme basierend auf [!DNL Adobe Developer App Builder] erstellt werden. Diese benutzerdefinierten Programme sind Headless-[!DNL Project Adobe Developer App Builder]-Apps und führen Aufgaben wie das Hinzufügen benutzerdefinierter Konvertierungs-Tools oder den Aufruf externer APIs durch, um Bildvorgänge auszuführen.

[!DNL Project Adobe Developer App Builder] ist ein Framework zum Erstellen und Bereitstellen benutzerdefinierter Web-Anwendungen auf Adobe [!DNL `I/O Runtime`]. Um benutzerdefinierte Anwendungen zu kreieren, können die Entwickelnden [!DNL React Spectrum] (das Adobe-Toolkit für Benutzeroberflächen) nutzen, Microservices erstellen, benutzerdefinierte Ereignisse erzeugen und APIs orchestrieren. Siehe [Dokumentation zu Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).

Die Architektur basiert auf dem Folgenden:

* Die Modularität der Anwendungen – sie enthalten nur das, was für eine bestimmte Aufgabe benötigt wird – ermöglicht es, die Anwendungen voneinander zu entkoppeln und diese einfach zu halten.

* Das Server-lose Konzept von [!DNL Adobe I/O] Runtime bietet zahlreiche Vorteile: asynchrone, hochskalierbare, isolierte, vorgangsbasierte Verarbeitung, die sich ideal für die Asset-Verarbeitung eignet.

* Der binäre Cloud-Speicher bietet die erforderlichen Funktionen zum individuellen Speichern von und Zugreifen auf Asset-Dateien und -Ausgabedarstellungen, ohne dass volle Zugriffsberechtigungen auf den Speicher erforderlich sind, und zwar unter Verwendung von vorab signierten URL-Referenzen. Übertragungsbeschleunigung, CDN-Caching und die gemeinsame Speicherung von Compute-Programme im Cloud-Speicher ermöglichen einen optimalen Zugriff auf Inhalte mit geringer Latenz. Es werden sowohl AWS- als auch Azure-Clouds unterstützt.

![Architektur von Asset Compute Service](assets/architecture-diagram.png)

*Abbildung: Architektur von [!DNL Asset Compute Service] und wie der Service mit [!DNL Experience Manager], Datenspeicherung und Verarbeitungsprogrammen zusammenarbeitet.*

Die Architektur besteht aus folgenden Teilen:

* **Eine API- und Orchestrierungsebene** empfängt Anforderungen (im JSON-Format), die den Service anweisen, ein Quellelement in mehrere Ausgabedarstellungen umzuwandeln. Die Anfragen sind asynchron und werden mit einer Aktivierungs-ID, also einer Auftrags-ID, zurückgegeben. Die Anweisungen sind rein deklarativ. Bei allen standardmäßigen Verarbeitungsvorgängen (z. B. Erstellung von Miniaturansichten, Textextraktion) geben die Benutzenden nur das gewünschte Ergebnis an, nicht jedoch die Anwendungen, die bestimmte Ausgabedarstellungen verarbeiten. Generische API-Funktionen wie Authentifizierung, Analyse, Ratenbegrenzung werden über das Adobe API Gateway vor dem Service abgewickelt und verwalten alle Anfragen, die an [!DNL Adobe I/O] Runtime gesendet werden. Das Programm-Routing wird dynamisch von der Orchestrierungsebene ausgeführt. Clients legen benutzerdefinierte Anwendungen für bestimmte Ausgabedarstellungen fest, die mit ihrem eigenen Satz eindeutiger Parameter bereitgestellt werden. Die Anwendungsausführung kann vollständig parallelisiert werden, da es sich um separate Server-lose Funktionen in Adobe [!DNL `I/O Runtime`] handelt.

* **Anwendungen zur Verarbeitung von Assets**, die auf bestimmte Dateiformate oder Zielausgabedarstellungen spezialisiert sind. Eine Anwendung ähnelt dem UNIX®-Pipe-Konzept: Eine Eingabedatei wird in eine oder mehrere Ausgabedateien umgewandelt.

* **Eine [gemeinsame Anwendungsbibliothek](https://github.com/adobe/asset-compute-sdk)** verarbeitet allgemeine Aufgaben. Beispiele hierfür sind das Herunterladen der Quelldatei, das Hochladen der Ausgabedarstellungen, die Fehlerberichterstattung, das Senden von Ereignissen und die Überwachung. Dieses Design stellt sicher, dass die Anwendungsentwicklung unkompliziert bleibt und dem Server-losen Konzept entspricht, wobei die Interaktionen auf das lokale Dateisystem beschränkt sind.

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
