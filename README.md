# PaperArk

**Long-term digital storage on paper.**

**Projektversion: 0.1 · Lizenz: [MIT](./LICENSE) · Archivformat: PaperArk v1**

PaperArk archiviert kleine, wichtige Dateien als standardisierte QR-Codes auf DIN-A4-Seiten. Jede Seite enthält eine englische Wiederherstellungsanleitung und eine vollständige technische Beschreibung des Binärformats.

Das Ziel: Ein zukünftiger Entwickler soll einen Decoder allein anhand des Ausdrucks implementieren können. Die ursprüngliche PaperArk-Software soll zur Wiederherstellung nicht erforderlich sein.

> Prefer boring, standardized, well-documented technology over clever technology.

## Starten

1. [paperark.html](./paperark.html) über GitHub herunterladen: Datei öffnen und **Download raw file** wählen. Alternativ das Repository klonen.
2. Die heruntergeladene Datei per Doppelklick im Browser öffnen.
3. Unter **CREATE ARCHIVE** eine Datei auswählen, hineinziehen oder Text eingeben.

Die Anwendung besteht vollständig aus **einer HTML-Datei**. Diese README dient nur der GitHub-Dokumentation und wird zur Ausführung nicht benötigt. Kein Build, keine Installation, kein Server und keine Internetverbindung erforderlich. JavaScript muss aktiviert sein.

Alle Daten werden lokal verarbeitet. Keine Uploads, Telemetrie, externen Fonts, CDNs oder APIs. Bibliotheken und ihre Lizenztexte sind in die HTML-Datei eingebettet; eine Content Security Policy blockiert Netzwerkverbindungen.

## Archiv erstellen und drucken

1. Datei oder UTF-8-Text auswählen und Dateinamen prüfen.
2. Optional gzip-Kompression verwenden; bereits komprimierte Dateien werden dadurch möglicherweise größer.
3. Optional AES-256-GCM aktivieren und ein Passwort zweimal eingeben.
4. QR-Fehlerkorrektur **L**, **M**, **Q** oder **H** wählen; Standard ist **M**.
5. **Create Paper Archive** anklicken. Die Anwendung teilt die Daten bei Bedarf auf mehrere QR-Codes auf und überprüft jeden erzeugten Code mit einem unabhängigen Bilddecoder.
6. **Print Archive** anklicken und drucken oder über den Browser als PDF speichern.

Pro A4-Seite wird ein 130 mm großer QR-Code mit einer Quiet Zone von vier Modulen ausgegeben. Die Vorschau zeigt QR-Version, Kapazität, verwendete Bytes, Nutzungsgrad und Blocknummer. Metadaten, Original-SHA-256 und Formatspezifikation stehen auf jeder Seite.

**Druckeinstellungen:** A4, tatsächliche Größe / 100 %, Schwarz auf Weiß, Browser-Kopf- und Fußzeilen deaktivieren. Alle Seiten aufbewahren. Einen echten Ausdruck vor der Archivierung zurückscannen und überprüfen; für dichte Codes sind scharfe Scans mit 600 dpi empfehlenswert.

## Optionale Base32 Recovery Copy

Unter **CREATE ARCHIVE → Add Base32 recovery copy** lässt sich zusätzlich zu den QR-Seiten eine menschen- und OCR-lesbare Sicherung ausgeben. Die Option ist standardmäßig ausgeschaltet. Sie verwendet [Base32 gemäß RFC 4648](https://www.rfc-editor.org/rfc/rfc4648.html#section-6) mit dem Alphabet `ABCDEFGHIJKLMNOPQRSTUVWXYZ234567` und `=`-Padding.

Die Ausgabe kodiert die **exakten, bereits erzeugten QR-Blöcke einschließlich ihrer Header**, in Blockreihenfolge aneinandergehängt, als einen einzigen Base32-Datenstrom. Es gibt keine zweite Verarbeitung der Originaldatei, keine neuen Header und kein neues Archivformat. Padding steht nur am Ende des gesamten Datenstroms.

Die zusätzlichen A4-Seiten folgen auf die QR-Seiten. Jede enthält Archive-ID, Seitennummer und englische Wiederherstellungshinweise. Der Datenbereich verwendet schwarze 11-pt-Monospace-Schrift ohne Ligaturen, 64 Zeichen pro Zeile und maximal 32 Datenzeilen pro Seite; nur die letzte Zeile darf kürzer sein. Die Seitenzahlen der Vorschau berücksichtigen die zusätzlichen Base32-Seiten.

Zur Wiederherstellung unter **RESTORE ARCHIVE → Paste Base32 recovery data** ausschließlich die Datenzeilen aller Base32-Seiten in Seitenreihenfolge einfügen und **Import Base32 recovery data** anklicken. Leerzeichen, Tabs und Zeilen-/Seitenumbrüche werden ignoriert; ASCII-Kleinbuchstaben werden akzeptiert. Überschriften und Anleitung nicht mitkopieren. Ungültige Zeichen, falsches Padding und nicht-nullgesetzte Padding-Bits werden zurückgewiesen.

Anschließend wie gewohnt **Restore & verify** verwenden. Base32- und QR-Import nutzen dieselbe Blockprüfung und Restore-Pipeline, einschließlich AES-GCM, Dekompression und Original-SHA-256. Bereits eingelesene identische QR-Blöcke werden als Duplikate erkannt. Ein fehlerhafter Base32-Import verändert die zuvor gesammelten Blöcke nicht.

Für eine externe Implementierung: Nach RFC-4648-Decoding die vollständigen PaperArk-Blöcke nacheinander auslesen. Jeder Block ist `144 + N + P` Bytes lang; `N` und `P` stehen als unsigned Big-Endian-16-Bit-Felder an den blockrelativen Offsets 108 und 110. Danach gilt unverändert die auf den QR-Seiten gedruckte PaperArk-v1-Spezifikation. Die QR-Seiten mit dieser Spezifikation daher zusammen mit der Base32-Kopie aufbewahren.

Die integrierten Selbsttests enthalten zusätzlich 20 Base32-Tests: RFC-Testvektoren, Binär-/Whitespace-/Padding-Prüfungen, Mehrseiten-Rekonstruktion und vollständige Base32-Archiv-Roundtrips mit und ohne Kompression und Verschlüsselung. Alle 65 Tests (45 bisherige und 20 zusätzliche) wurden in Chromium erfolgreich ausgeführt. Ein bedienter Roundtrip über vier Base32-Seiten mit gzip und Verschlüsselung lieferte ebenfalls eine byteidentische Download-Datei; die A4-Seitengeometrie wurde geprüft. Ein physischer Druck-/OCR-Rückscan ist damit nicht bestätigt.

> Base32 is a redundant transport representation, not a new archive format.

## Archiv wiederherstellen

1. **RESTORE ARCHIVE** öffnen.
2. QR-Bilder auswählen, vorzugsweise PNG oder JPEG und jeweils ein QR-Code pro Bild. Mehrere Dateien dürfen gemeinsam und in beliebiger Reihenfolge eingelesen werden.
3. Fehlende Blöcke ergänzen. Identische Duplikate werden ignoriert; beschädigte oder widersprüchliche Blöcke werden abgewiesen.
4. Bei verschlüsselten Archiven das Passwort eingeben.
5. **Restore & verify** anklicken und anschließend die wiederhergestellte Datei herunterladen.

Erst nach erfolgreicher Prüfung von Originalgröße und SHA-256 wird der Download angeboten. Für ein anderes Archiv zunächst die gesammelten Blöcke löschen. PDF-Dateien müssen vor dem Einlesen in Seitenbilder umgewandelt werden. Live-Kamera-Scanning ist nicht implementiert.

## Standards und Format

| Bestandteil | Standard / Entscheidung |
| --- | --- |
| QR | ISO/IEC 18004, Byte Mode, bis Version 40 |
| Text und Dateinamen | UTF-8 |
| Integerfelder | Unsigned, Big Endian |
| Integrität | SHA-256 über Originaldaten und jeden Block |
| Kompression | Optional gzip / DEFLATE, RFC 1952 / RFC 1951 |
| Verschlüsselung | Optional AES-256-GCM, 96-Bit-Nonce, 128-Bit-Tag |
| Passwortableitung | PBKDF2-HMAC-SHA-256, 600.000 Iterationen, zufälliges 128-Bit-Salt |
| Zufallswerte und Kryptografie | Web Crypto API |

Die Verarbeitung erfolgt in dieser Reihenfolge:

```text
Originaldatei → optional gzip → optional AES-256-GCM
             → Aufteilung in PaperArk-Blöcke → QR-Codes
```

Ein PaperArk-v1-Block besteht aus einem **144-Byte-Header**, einem UTF-8-Dateinamen und einem Payload-Abschnitt. Der Header enthält unter anderem die Kennung `PARK`, Formatversion, Archive-ID, Blockindex und -anzahl, Größen, Algorithmen, Hashes und Verschlüsselungsparameter. Gemeinsame Metadaten werden in jedem Block wiederholt und bei Verschlüsselung als GCM-AAD authentifiziert.

Die verbindliche Spezifikation mit allen Byte-Offsets, Hashbereichen, AAD-Zusammensetzung und Wiederherstellungsschritten steht in **FORMAT SPECIFICATION**, ist als Text kopier- und herunterladbar und wird kompakt auf jeder Archivseite gedruckt. Im Quellcode ist sie zentral als `SPEC` dokumentiert; `PRINT_SPEC` enthält die kompakte Druckfassung.

Ein externer Decoder muss die **rohen QR-Bytes** auslesen, nicht eine Textdarstellung. Nach Blockprüfung und Sortierung werden die Payloads zusammengesetzt, gegebenenfalls entschlüsselt und dekomprimiert und abschließend gegen Originalgröße und SHA-256 geprüft.

## Sicherheit und Grenzen

- **Ohne Passwort oder Schlüssel sind verschlüsselte Archive nicht wiederherstellbar.** Das Passwort wird niemals gedruckt. Ein langes, einzigartiges Passwort verwenden und eine getrennte, dauerhafte Wiederherstellungsmöglichkeit vorsehen.
- Dateiname, Größen und Originalhash bleiben auch bei Verschlüsselung öffentlich. Hashes erkennen Beschädigung, beweisen aber keine Urheberschaft.
- Passworteingaben werden nach Verarbeitung geleert; temporäre Byte-Arrays werden nach Möglichkeit überschrieben. Eine sichere Löschung von JavaScript-Strings, Browser-/Betriebssystemspeicher und Druckerspool kann nicht garantiert werden.
- QR-Fehlerkorrektur kann begrenzte Schäden innerhalb eines Codes ausgleichen, aber keine vollständig fehlende Seite ersetzen.
- Papier, Toner, Lagerung und zukünftige kryptografische Sicherheit begrenzen die Haltbarkeit. Eine Lebensdauer von 100 Jahren ist ein Designziel, keine Garantie. Kopien getrennt lagern, regelmäßig prüfen und bei Bedarf migrieren.
- Gedacht für kleine Textdateien, Schlüsseldateien, Konfigurationen, Quellcode und Notfalldokumente; nicht für Videos, Fotoarchive oder Festplatten-Backups.

Aktuelle Implementierungsgrenzen: **8 MiB Eingabe**, **128 QR-Blöcke / Seiten**, **240 UTF-8-Bytes pro Dateiname**. Die praktisch speicherbare Datenmenge wird meist früher durch die QR-Kapazität begrenzt. Bilddateien dürfen höchstens 32 MiB und 32 Megapixel groß sein; beim Scannen werden sie auf maximal 2400 Pixel pro Seite verkleinert. Restore akzeptiert 1–2.000.000 PBKDF2-Iterationen. Diese Grenzen sind keine allgemeinen Grenzen des Containerformats.

## Tests und Browserstatus

Unter **ABOUT → Run Self Test** lassen sich die integrierten Tests ohne zusätzliche Werkzeuge ausführen. Sie verwenden ausschließlich synthetische Daten und ersetzen kein gerade erstelltes Archiv.

Der dokumentierte Prüfstand umfasst **45 bestandene Selbsttests in Chromium**, darunter:

- Byteidentische Roundtrips für leere Dateien, ein Byte, Text, Binärdaten, UTF-8, QR-Grenzgrößen und Mehrblockarchive, jeweils mit und ohne Kompression und Verschlüsselung.
- QR-Version 40 mit allen vier Fehlerkorrekturstufen.
- Falsches Passwort, fehlende und beschädigte Blöcke, Duplikate und manipulierte Metadaten.
- PNG-Bilddekodierung mit verschlüsselter Wiederherstellung sowie ein simulierter A4-Bild-Roundtrip.
- SHA-256-Testvektoren, Dekompressionsbegrenzung und Prüfung auf externe Ressourcenabrufe.

Zusätzlich rekonstruierte ein separater Python-Decoder Testarchive in allen vier Kompressions-/Verschlüsselungskombinationen ohne PaperArk-Code. Die A4-Geometrie wurde mit Verschlüsselung und einem maximal langen Dateinamen geprüft. Die Entwicklungswerkzeuge werden zur Nutzung nicht benötigt und gehören nicht zum Repository.

**Noch nicht bestätigt:** Firefox und Safari, ein physischer Ausdruck mit Rückscan sowie der automatisierte direkte `file://`-Start und die automatisierte Dateiauswahl. Letztere wurden durch Berechtigungen der Test-Browser-Erweiterung blockiert; die Oberflächenprüfung erfolgte über einen temporären lokalen HTTP-Server. Die Anwendung ist für aktuelle Chromium-, Firefox- und Safari-Browser konzipiert und benötigt Web Crypto. Bitte im eigenen Browser die Selbsttests und einen tatsächlichen Druck-/Scanversuch durchführen.

## Eingebettete Bibliotheken

| Bibliothek | Version | Lizenz |
| --- | --- | --- |
| qrcode-generator, Kazuhiko Arase | 1.4.4 | MIT |
| jsQR, Cozmo | 1.4.0 | Apache-2.0 |
| fflate, Arjun Barrett | 0.8.2 | MIT |

Die Lizenztexte der Bibliotheken sind in `paperark.html` enthalten und unter **ABOUT** einsehbar. Die Bibliotheken behalten ihre jeweiligen Lizenzen und Copyright-Hinweise.

## Projektversion und Lizenz

PaperArk trägt die Projektversion **0.1**. Die davon unabhängige Version des Binärformats bleibt **PaperArk Format Version 1**.

Der PaperArk-Anwendungscode und die Projektdokumentation stehen unter der [MIT-Lizenz](./LICENSE). Copyright (c) 2026 Heiner (Heini155). Der vollständige Projektlizenztext ist zusätzlich in der eigenständig nutzbaren `paperark.html` unter **ABOUT → PaperArk MIT License** enthalten.
