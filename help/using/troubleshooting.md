---
title: Fehlerbehebung [!DNL Asset Compute Service]
description: Fehlerbehebung und Debugging von benutzerdefinierten Programmen mit  [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '273'
ht-degree: 76%

---

# Fehlerbehebung {#troubleshoot}

Einige allgemeine Tipps, die Ihnen bei der Fehlerbehebung mit Asset Compute Service helfen können:

* Stellen Sie sicher, dass das JavaScript-Programm beim Start nicht abstürzt. Solche Abstürze hängen normalerweise mit einer fehlenden Bibliothek oder Abhängigkeit zusammen.
* Stellen Sie sicher, dass alle zu installierenden Abhängigkeiten in der `package.json`-Datei des Programms referenziert sind.
* Stellen Sie sicher, dass alle Fehler, die von der Bereinigung eines Fehlers herrühren können, keine eigenen Fehler erzeugen, die das ursprüngliche Problem verdecken.

* Beim erstmaligen Starten des Entwickler-Tools mit einer neuen [!DNL Asset Compute Service]-Integration kann es vorkommen, dass die erste Verarbeitungsanfrage fehlschlägt, wenn das Asset Compute-Ereignisjournal noch nicht vollständig eingerichtet wurde. Warten Sie einige Zeit, bis das Journal eingerichtet ist, bevor Sie eine weitere Anfrage senden.
* Sicherstellen aller erforderlichen APIs - Asset compute, Adobe [!DNL I/O Events], Ereignisverwaltung und Laufzeit sind in Ihrer Adobe enthalten. [!DNL `I/O Project`] und Workspace vermeiden `/register` oder `/process` Anforderungsfehler.

## Anmeldungsprobleme über Adobe [!DNL aio-cli] {#login-via-aio-cli}

Wenn Sie Probleme haben, sich bei der [!DNL Adobe Developer Console] [durch die Adobe [!DNL aio-cli]](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)und fügen Sie dann manuell die Anmeldeinformationen hinzu, die zum Entwickeln, Testen und Bereitstellen Ihres benutzerdefinierten Programms erforderlich sind:

1. Navigieren Sie zu Ihrem Adobe Developer App-Builder-Projekt und -Arbeitsbereich in der [Adobe-Entwicklerkonsole](https://developer.adobe.com/console/user/servicesandapis) und klicken Sie oben rechts auf **[!UICONTROL Herunterladen]**. Öffnen Sie diese Datei und speichern Sie diese JSON an einem sicheren Ort auf Ihrem Computer.

1. Navigieren Sie zur ENV-Datei in Ihrem Adobe Developer App Builder-Programm.

1. Adobe hinzufügen [!DNL I/O Runtime] Anmeldedaten. Adobe abrufen [!DNL I/O Runtime] Anmeldedaten von der heruntergeladenen JSON. Die Anmeldeinformationen finden Sie unter `project.workspace.services.runtime`. Fügen Sie die [!DNL Adobe I/O] Runtime-Anmeldeinformationen in den `AIO_runtime_XXX`-Variablen hinzu:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Fügen Sie den absoluten Pfad zur in Schritt 1 heruntergeladenen JSON hinzu:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Richten Sie den Rest der [erforderlichen Anmeldeinformationen](develop-custom-application.md) für das Entwickler-Tool ein.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
