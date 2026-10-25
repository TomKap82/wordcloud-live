# WordCloud Live – Anleitung für Windows 11

**Funktionen:** Frage „Was macht guten Unterricht aus?“, pro anonymem Teilnehmer maximal fünf verschiedene Begriffe, zunächst verdeckte Sammlung, Freigabe durch Moderator, QR-Code, Beamer-Vollbild, PNG-Export, mehrere Sammlungen, Ausblendung einzelner Begriffe.

## 1. Firebase-Projekt anlegen

1. Öffne https://console.firebase.google.com/ und klicke auf **Projekt hinzufügen**. Name zum Beispiel `wordcloud-live-kmk`. Die Google-Analytics-Einrichtung ist für diese App nicht nötig.
2. Unter **Erstellen > Firestore Database** eine Datenbank anlegen. Verwende zunächst **Produktionsmodus**; die unten stehenden Regeln werden gleich veröffentlicht. Region möglichst in der EU auswählen. **Veröffentliche die App erst nach den Regeln.**
3. Unter **Erstellen > Authentication > Anmeldemethode** die Anbieter **Anonym** und **E-Mail/Passwort** aktivieren. KEINE Selbstregistrierung für Moderatoren vorsehen.
4. Unter **Authentication > Nutzer** per **Nutzer hinzufügen** dein eigenes Moderatorkonto mit E-Mail und einem starken individuellen Passwort erstellen. Die neue Nutzer-ID (`UID`) kopieren.
5. Unter **Firestore Database > Daten > Sammlung starten** die Sammlung `admins` erstellen. Dokument-ID = exakt deine UID. Feld hinzufügen: `active` (Boolean) = `true`. Diesen Eintrag über die Firebase Console vornehmen, nicht durch die Webseite.
6. Unter **Projekteinstellungen (Zahnrad) > Allgemein > Deine Apps** eine **Web-App** registrieren (z. B. `WordCloud Live`). Firebase Hosting **nicht** auswählen. Die `firebaseConfig`-Angaben ablesen.
7. Datei `firebase-config.js` in VS Code bearbeiten. Die vier Angaben `apiKey`, `authDomain`, `projectId` und `appId` durch die Werte deiner Web-App ersetzen (Anführungszeichen behalten). Firebase-Konfigurationswerte sind öffentliche Kennungen; das Moderatorpasswort gehört niemals in den Code.
8. Unter **Firestore Database > Regeln** den **gesamten Inhalt** aus `firestore.rules` einsetzen und auf **Veröffentlichen** klicken. Nicht die zunächst offenen Testmodus-Regeln behalten.

**Hinweis zur Admin-Einrichtung:** Für eine Moderationssitzung wird sowohl ein E-Mail/Passwort-Konto als auch der passende `admins/{UID}`-Datensatz benötigt. Nur du darfst diesen Datensatz in der Firebase Console erstellen. Weitere Moderatorenkonten werden auf dieselbe Weise freigeschaltet.

## 2. Auf Windows 11 lokal testen

1. ZIP entpacken, etwa nach `C:\Projekte\wordcloud-live`.
2. Ordner in **Visual Studio Code** öffnen.
3. Terminal öffnen (**Terminal > Neues Terminal**).
4. Falls Python installiert ist: `py -m http.server 5500`. Alternativ Node nutzen: `npx serve . -l 5500` (bei einer Nachfrage einmalig bestätigen). **index.html nicht einfach per Doppelklick öffnen**, weil ES-Module einen Webserver erfordern.
5. Im Browser `http://localhost:5500` öffnen. Über **Moderation öffnen** mit dem angelegten Konto anmelden. Eine Sammlung erstellen.
6. In einem **anderen Browser/Inkognito-Fenster** (wichtig: getrennte Firebase-Anmeldung) den auf der Moderationsseite angezeigten Teilnehmerlink öffnen. 1–5 Wörter eingeben und speichern.
7. Moderatorfenster: verdeckte Ergebnisse prüfen; **Wortwolke freigeben** anklicken. Im Teilnehmerfenster erscheint die Wortwolke erst jetzt.
8. Teste auch die Rückkehr zu „Neue Eingaben zulassen“, das Ausblenden eines Worts, Vollbild und PNG.

**Falls „auth/unauthorized-domain“ erscheint:** Firebase Console > Authentication > Einstellungen > Autorisierte Domains > `localhost` hinzufügen.

## 3. GitHub Pages veröffentlichen

1. Melde dich auf https://github.com/ an und lege ein **neues öffentliches Repository** mit Namen `wordcloud-live` an. Kein zusätzliches README erzeugen, weil bereits eines vorhanden ist.
2. Lade die Projektdateien hoch: im Repository **Add file > Upload files** und alle Dateien aus dem entpackten Ordner auswählen, inklusive `index.html`, `app.js`, `style.css`, `firebase-config.js`, `firestore.rules`, `firebase.json`, `README.md`.
3. Mit **Commit changes** speichern.
4. Unter **Settings > Pages** als Quelle **Deploy from a branch** wählen, Branch `main`, Verzeichnis `/(root)` und **Save**.
5. GitHub zeigt nach der Bereitstellung eine URL wie `https://DEINNAME.github.io/wordcloud-live/`.
6. Falls nötig, die GitHub-Pages-Domain `DEINNAME.github.io` unter **Firebase Authentication > Einstellungen > Autorisierte Domains** hinzufügen. Nur den Hostnamen, ohne `https://` oder Pfad, eintragen.
7. Live-Seite öffnen, anmelden, Sammlung erstellen und mit dem QR-Code auf einem zweiten Gerät testen. QR-Code enthält automatisch die echte GitHub-Pages-Adresse.

**Achtung:** Die öffentliche `firebase-config.js` darf niemals Passwörter, Service-Account-Schlüssel oder andere Geheimnisse enthalten. Die Sicherheit liegt in Firebase Authentication und Firestore Rules.

## 4. Fortbildung durchführen

1. Moderation auf dem Notebook starten, **Neue Sammlung** anklicken. Standardfrage: „Was macht guten Unterricht aus?“
2. Den automatisch erzeugten QR-Code groß anzeigen (ggf. Fenster zoomen); Teilnehmende scannen und geben höchstens fünf Begriffe ein.
3. Die Ergebnisse bleiben für Teilnehmende verborgen. Die Moderatorin/der Moderator sieht intern eine Vorschau, die per Schalter ausgeschaltet werden kann.
4. **Wortwolke freigeben** beendet die Eingabe und zeigt den Teilnehmenden ebenfalls die Ergebnisse. Auf dem Beamer **Vollbild** aktivieren.
5. Begriffe durch Anklicken in der Moderationsliste ausblenden (erneutes Anklicken blendet sie wieder ein).
6. Über **Als PNG speichern** die sichtbare Wolke für Dokumentation/Präsentation sichern.
7. Neue Sammlung: **Übersicht** > **Sammlung starten**. Alten Sitzungscode bei Bedarf notieren.

## 5. Datenmodell und Sicherheitsgrenzen

- `admins/{uid}`: nur manuell eingerichtet; Berechtigungsprüfung für Moderator-Login.
- `sessions/{CODE}`: Frage, Status `collecting` bzw. `revealed`, Liste ausgeblendeter Begriffe.
- `sessions/{CODE}/entries/{anonymousUid}`: genau ein Dokument mit einer Liste von höchstens fünf Wörtern je anonymer Firebase-Identität.
- Vor Freigabe kann jeder Teilnehmende **nur den eigenen** Beitrag lesen; die Moderation kann alle lesen. Nach Freigabe können authentifizierte Teilnehmende alle Einsendungen der Sitzung lesen. Das ist keine Verschlüsselung, sondern rollenbasierte Zugriffskontrolle.
- Die E-Mail/Passwort-Anmeldung kann nicht per öffentlichem Webformular neu angelegt werden; Konten/UIDs werden über Firebase Console verwaltet. Die Admin-Regeln prüfen zusätzlich den Passwort-Anmeldeanbieter.
- Eine neue Anmeldung im Inkognito-Browser, das Löschen von Browserdaten oder ein anderes Gerät erzeugt ggf. eine neue anonyme UID; das 5-Wort-Limit ist **pro anonymer Identität**, nicht fälschungssicher pro Mensch. Gegen gezielte Spam-Angriffe genügt diese reine Clientlösung nicht. Für größere öffentliche Veranstaltungen benötigt man serverseitige Rate-Limits / App Check / zusätzliche Anti-Missbrauchsmaßnahmen.
- Teilnehmende mit Sitzungscode können die Fragestellung und den Freigabestatus lesen. Ein Sitzungscode dient als schwer erratbarer Link, **nicht** als Admin-Passwort.
- Die Word-Cloud-Bibliotheken und das QR-Code-Skript werden über CDN geladen; zum Betrieb ist eine Internetverbindung erforderlich. Für stärker eingeschränkte Schulnetze können diese Bibliotheken lokal eingebunden werden.
- Die Anwendung zeigt in der Moderation standardmäßig bereits während der Sammlung eine interne Vorschau; für die Teilnehmenden erscheint **erst nach Freigabe** eine Wolke.

## 6. Typische Fehler

- **„Firebase konfigurieren“**: Platzhalter in `firebase-config.js` nicht ersetzt.
- **„permission-denied“ als Moderator**: Konto-UID stimmt nicht mit der Dokument-ID in `admins` überein, Regeln nicht veröffentlicht oder E-Mail/Passwort nicht aktiviert.
- **„permission-denied“ als Teilnehmer**: Anonymous Auth nicht aktiviert, Firestore-Regeln fehlen, falsches Firebase-Projekt oder Sammlung nicht in `collecting`.
- **QR-Code leer**: QRCode-CDN blockiert; der Teilnehmer-Link darunter lässt sich auch kopieren.
- **Wolke leer**: noch keine Einsendungen, Vorschau deaktiviert oder d3-CDN blockiert.
- **Zweiter Test mit demselben Browser-Profil**: Moderator und Teilnehmer teilen dort dieselbe Auth-Sitzung. Für den Test ein Inkognito-Fenster oder anderes Gerät nutzen.

## Erweiterbarkeit

Die App ist ein statisches Projekt ohne Node-Build; für Styling direkt `style.css` ändern. Die eigentliche Logik steht in `app.js`. Später sind CSV-Export, Zeitlimit, Passwortschutz für Teilnehmende und ein Dashboard mit Sitzungsübersicht möglich.
