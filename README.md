# PaperArk

**Long-term digital storage on paper.**

**Projektversion: 0.2 · Lizenz: [MIT](./LICENSE) · Archivformat: PaperArk v1**

PaperArk archiviert kleine, wichtige Dateien als standardisierte QR-Codes **oder** als Base32-Textseiten auf DIN-A4. Jede Seite enthält eine englische Wiederherstellungsanleitung und eine vollständige technische Beschreibung des Binärformats.

Das Ziel: Ein zukünftiger Entwickler soll einen Decoder allein anhand des Ausdrucks implementieren können. Die ursprüngliche PaperArk-Software soll zur Wiederherstellung nicht erforderlich sein.

> Prefer boring, standardized, well-documented technology over clever technology.

## Starten

1. [paperark.html](./paperark.html) über GitHub herunterladen: Datei öffnen und **Download raw file** wählen. Alternativ das Repository klonen.
2. Die heruntergeladene Datei per Doppelklick im Browser öffnen.
3. Unter **CREATE ARCHIVE** eine Datei auswählen, hineinziehen oder Text eingeben.

Die Anwendung besteht vollständig aus **einer HTML-Datei**. Diese README dient nur der GitHub-Dokumentation und wird zur Ausführung nicht benötigt. Kein Build, keine Installation, kein Server und keine Internetverbindung erforderlich. JavaScript muss aktiviert sein.

Alle Daten werden lokal verarbeitet. Keine Uploads, Telemetrie, externen Fonts, CDNs oder APIs. Bibliotheken und ihre Lizenztexte sind in die HTML-Datei eingebettet; eine Content Security Policy blockiert Netzwerkverbindungen.

## Transportmedium: QR-Code oder Base32-Seite

Unter **CREATE ARCHIVE → Transport medium** wird das Speichermedium bewusst gewählt:

- **QR Code** — binäres ISO/IEC-18004-Symbol, maximale Datendichte pro Seite, bewährte Reed-Solomon-Fehlerkorrektur des QR-Codes (Stufen L/M/Q/H).
- **Base32 Sheet** — druckbarer Texttransport mit eigener Fehlerkorrektur-Schicht (unten beschrieben), maschinen- und menschenlesbar, robust gegen beschädigte und gelöschte Zeilen.

Beide Medien tragen exakt dieselben PaperArk-v1-Blöcke. Ein Archiv kann vollständig auf einem der beiden Medien gespeichert werden; beim Restore lassen sich beide Importwege mischen (identische Blöcke werden als Duplikate erkannt).

## Archiv erstellen und drucken

1. Datei oder UTF-8-Text auswählen und Dateinamen prüfen.
2. Transportmedium wählen (QR oder Base32).
3. Optional gzip-Kompression verwenden; bereits komprimierte Dateien werden dadurch möglicherweise größer.
4. Optional AES-256-GCM aktivieren und ein Passwort zweimal eingeben.
5. Bei QR: Fehlerkorrektur **L**, **M**, **Q** oder **H** wählen; Standard ist **M**.
6. **Create Paper Archive** anklicken. Die Anwendung erzeugt die Blöcke und verifiziert jedes Medium selbst (QR: unabhängiger Bilddecoder; Base32: Decode-Roundtrip).
7. **Print Archive** anklicken und drucken oder über den Browser als PDF speichern.

**QR-Seiten:** Ein 130 mm großer QR-Code mit Quiet Zone von vier Modulen pro A4-Seite. Vorschau zeigt QR-Version, Kapazität, verwendete Bytes, Nutzungsgrad und Blocknummer.

**Base32-Seiten:** Der Datenblock ist **links bündig** mit der Anleitung gesetzt; der linke Rand von 25 mm bleibt frei für Lochung. Seite 1 trägt die vollständige Wiederherstellungsanleitung und 40 Datenzeilen; die Folgeseiten haben einen kompakten Kopf und tragen bis zu 56 Zeilen. Die Zeilen auf den Folgeseiten werden gleichmäßig verteilt, damit keine halbleere Seite entsteht. Jede Zeile: 64 Datenzeichen + 8 CRC-Zeichen.

**Druckeinstellungen:** A4, tatsächliche Größe / 100 %, Schwarz auf Weiß, Browser-Kopf- und Fußzeilen deaktivieren. Alle Seiten aufbewahren. Einen echten Ausdruck vor der Archivierung zurückscannen und überprüfen; für dichte Codes sind scharfe Scans mit 600 dpi empfehlenswert.

## Base32-Transport v2

Der Base32-Kanal ist ein gleichwertiges Medium, kein Notnagel. Er kodiert die exakte Aneinanderreihung der PaperArk-Blöcke als druckbaren Text und schützt sie mit drei ineinandergreifenden Mechanismen:

1. **Reed-Solomon RS(255,215)** über GF(256) (primitives Polynom 0x11d, Generator 2): Jeder 215-Byte-Datenblock erhält 40 Paritätsbytes. Korrigierbar sind 40 Erasures **oder** 20 Byte-Fehler pro Codewort.
2. **Symbol-Interleaving:** Die Codewörter werden zeichenweise über die Seite verschachtelt. Eine vollständig unlesbare Zeile (40 Bytes) verteilt sich auf viele Codewörter und überlastet kein einzelnes.
3. **Zeilen-CRC mit Zeilennummer:** Das 8-Zeichen-CRC-Feld jeder Zeile trägt eine 13-Bit-Zeilensnummer plus 27-Bit-CRC32 über Nummer und Datenfeld. Eine fehlerhafte Zeile wird zur Erasure (Position bekannt → doppelte Korrekturkapazität). Gelöschte Zeilen fallen durch die fehlende Nummer auf und werden ebenfalls als Erasure behandelt. Vertauschte Seiten werden über die Nummern sortiert.

Zusätzlich absorbiert das RFC-4648-Alphabet (ohne `0`, `1`, `8`, `9`) die häufigsten OCR-Verwechslungen: Eine Verwechslung von `O`↔`0` oder `I`↔`1` erzeugt ein ungültiges Zeichen und ist damit erkennbar, statt still einen Wert zu verändern.

**Stream-Aufbau:** 4 Byte Big-Endian-Länge L, dann L Bytes PaperArk-Blöcke, dann Null-Padding. Die Codewortanzahl ist ein Vielfaches von 8, dadurch ist die Zeichenzahl immer ein exaktes Vielfaches von 64 — jede gedruckte Zeile ist voll gefüllt, es existiert kein `=`-Padding, das bei Beschädigung der letzten Zeile die Stromlänge verschieben könnte.

**Dekodierstrategien:** (1) zeilennummernverankerte Rekonstruktion mit Erasures für fehlende/CRC-fehlerhafte Zeilen; (2) naives vertrauendes Chunking für verstreute Einzelfehler (OCR auf sauberen Druck).

**Wiederherstellung:** Alle Zeilen aller Seiten (beliebige Reihenfolge, Zeilenumbrüche optional, Kleinbuchstaben erlaubt) in **RESTORE ARCHIVE → Paste Base32 transport data** einfügen und importieren. Nicht selbst reparieren — was dubios aussieht, wird durch Reed-Solomon korrigiert.

Pro A4-Seite passen etwa 1.5 kB Nutzdaten (vor Header-Overhead). Die Blöcke selbst sind identisch zum QR-Medium; es gibt kein zweites Archivformat.

> Base32 is a transport encoding with its own error correction, not a new archive format.

## Archiv wiederherstellen

1. **RESTORE ARCHIVE** öffnen.
2. QR-Bilder auswählen (vorzugsweise PNG/JPEG, ein QR-Code pro Bild, beliebige Reihenfolge) **und/oder** Base32-Transportdaten einfügen.
3. Fehlende Blöcke ergänzen. Identische Duplikate werden ignoriert; beschädigte oder widersprüchliche Blöcke werden abgewiesen.
4. Bei verschlüsselten Archiven das Passwort eingeben.
5. **Restore & verify** anklicken und die wiederhergestellte Datei herunterladen.

Erst nach erfolgreicher Prüfung von Originalgröße und SHA-256 wird der Download angeboten. Für ein anderes Archiv zunächst die gesammelten Blöcke löschen. PDF-Dateien müssen vor dem Einlesen in Seitenbilder umgewandelt werden. Live-Kamera-Scanning ist nicht implementiert.

## Standards und Format

| Bestandteil | Standard / Entscheidung |
| --- | --- |
| QR | ISO/IEC 18004, Byte Mode, bis Version 40 |
| Base32-Transport | RFC 4648 Alphabet, RS(255,215) GF(256), Zeilen-CRC32 mit Zeilennummer |
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
             → Aufteilung in PaperArk-Blöcke → QR-Codes oder Base32-Transport
```

Ein PaperArk-v1-Block besteht aus einem **144-Byte-Header**, einem UTF-8-Dateinamen und einem Payload-Abschnitt. Der Header enthält unter anderem die Kennung `PARK`, Formatversion, Archive-ID, Blockindex und -anzahl, Größen, Algorithmen, Hashes und Verschlüsselungsparameter. Gemeinsame Metadaten werden in jedem Block wiederholt und bei Verschlüsselung als GCM-AAD authentifiziert.

Die verbindliche Spezifikation mit allen Byte-Offsets, Hashbereichen, AAD-Zusammensetzung und Wiederherstellungsschritten steht in **FORMAT SPECIFICATION**, ist als Text kopier- und herunterladbar und wird kompakt auf jeder Archivseite gedruckt. Im Quellcode ist sie zentral als `SPEC` dokumentiert; `PRINT_SPEC` enthält die kompakte Druckfassung.

Ein externer Decoder muss die **rohen QR-Bytes** auslesen, nicht eine Textdarstellung. Nach Blockprüfung und Sortierung werden die Payloads zusammengesetzt, gegebenenfalls entschlüsselt und dekomprimiert und abschließend gegen Originalgröße und SHA-256 geprüft.

## Sicherheit und Grenzen

- **Ohne Passwort oder Schlüssel sind verschlüsselte Archive nicht wiederherstellbar.** Das Passwort wird niemals gedruckt. Ein langes, einzigartiges Passwort verwenden und eine getrennte, dauerhafte Wiederherstellungsmöglichkeit vorsehen.
- Dateiname, Größen und Originalhash bleiben auch bei Verschlüsselung öffentlich. Hashes erkennen Beschädigung, beweisen aber keine Urheberschaft.
- Passworteingaben werden nach Verarbeitung geleert; temporäre Byte-Arrays werden nach Möglichkeit überschrieben. Eine sichere Löschung von JavaScript-Strings, Browser-/Betriebssystemspeicher und Druckerspool kann nicht garantiert werden.
- Fehlerkorrektur (QR-ECC oder Base32-RS) kann begrenzte Schäden ausgleichen, aber keine vollständig fehlenden Seiten in beliebigem Umfang ersetzen.
- Papier, Toner, Lagerung und zukünftige kryptografische Sicherheit begrenzen die Haltbarkeit. Eine Lebensdauer von 100 Jahren ist ein Designziel, keine Garantie. Kopien getrennt lagern, regelmäßig prüfen und bei Bedarf migrieren.
- Gedacht für kleine Textdateien, Schlüsseldateien, Konfigurationen, Quellcode und Notfalldokumente; nicht für Videos, Fotoarchive oder Festplatten-Backups.

Aktuelle Implementierungsgrenzen: **8 MiB Eingabe**, **128 QR-Blöcke / Seiten**, **240 UTF-8-Bytes pro Dateiname**. Die praktisch speicherbare Datenmenge wird meist früher durch die QR-Kapazität begrenzt. Der Base32-Transport ist durch die 13-Bit-Zeilensnummer auf 8191 Zeilen (~185 Seiten) begrenzt. Bilddateien dürfen höchstens 32 MiB und 32 Megapixel groß sein; beim Scannen werden sie auf maximal 2400 Pixel pro Seite verkleinert. Restore akzeptiert 1–2.000.000 PBKDF2-Iterationen. Diese Grenzen sind keine allgemeinen Grenzen des Containerformats.

## Tests und Browserstatus

Unter **ABOUT → Run Self Test** lassen sich die integrierten Tests ohne zusätzliche Werkzeuge ausführen. Sie verwenden ausschließlich synthetische Daten und ersetzen kein gerade erstelltes Archiv.

Der dokumentierte Prüfstand umfasst **68 bestandene Selbsttests in Chromium**, darunter:

- Byteidentische Roundtrips für leere Dateien, ein Byte, Text, Binärdaten, UTF-8, QR-Grenzgrößen und Mehrblockarchive, jeweils mit und ohne Kompression und Verschlüsselung.
- QR-Version 40 mit allen vier Fehlerkorrekturstufen.
- Falsches Passwort, fehlende und beschädigte Blöcke, Duplikate und manipulierte Metadaten.
- PNG-Bilddekodierung mit verschlüsselter Wiederherstellung sowie ein simulierter A4-Bild-Roundtrip.
- SHA-256- und CRC32-Testvektoren, Dekompressionsbegrenzung und Prüfung auf externe Ressourcenabrufe.
- Reed-Solomon: 20 Fehler, 40 Erasures, 10 Fehler + 20 Erasures, Ablehnung jenseits der Korrekturkapazität.
- Base32-Transport: Roundtrips, 1–6 vollständig beschädigte Zeilen, 30 verstreute Zeichenfehler, gelöschte Zeilen, vertauschte Zeilen, Zusammenfügen ohne Zeilenumbrüche, Ablehnung bei Überlastung und ungültigen Zeichen.
- Base32-Blattgeometrie (linksbündiger Datenblock, Lochrand, variable Seitenaufteilung) und Mischbetrieb mit QR-Blöcken im selben Collector.

Zusätzlich rekonstruierte ein separater Python-Decoder Testarchive in allen vier Kompressions-/Verschlüsselungskombinationen ohne PaperArk-Code. Die A4-Geometrie wurde mit Verschlüsselung und einem maximal langen Dateinamen geprüft. Die Entwicklungswerkzeuge werden zur Nutzung nicht benötigt und gehören nicht zum Repository.

**Noch nicht bestätigt:** Firefox und Safari, ein physischer Ausdruck mit Rückscan (QR und Base32) sowie der automatisierte direkte `file://`-Start und die automatisierte Dateiauswahl. Letztere wurden durch Berechtigungen der Test-Browser-Erweiterung blockiert; die Oberflächenprüfung erfolgte über einen temporären lokalen HTTP-Server. Die Anwendung ist für aktuelle Chromium-, Firefox- und Safari-Browser konzipiert und benötigt Web Crypto. Bitte im eigenen Browser die Selbsttests und einen tatsächlichen Druck-/Scanversuch durchführen.

## Eingebettete Bibliotheken

| Bibliothek | Version | Lizenz |
| --- | --- | --- |
| qrcode-generator, Kazuhiko Arase | 1.4.4 | MIT |
| jsQR, Cozmo | 1.4.0 | Apache-2.0 |
| fflate, Arjun Barrett | 0.8.2 | MIT |

Die Lizenztexte der Bibliotheken sind in `paperark.html` enthalten und unter **ABOUT** einsehbar. Die Bibliotheken behalten ihre jeweiligen Lizenzen und Copyright-Hinweise.

## Projektversion und Lizenz

PaperArk trägt die Projektversion **0.2**. Die davon unabhängige Version des Binärformats bleibt **PaperArk Format Version 1**.

Der PaperArk-Anwendungscode und die Projektdokumentation stehen unter der [MIT-Lizenz](./LICENSE). Copyright (c) 2026 Heiner (Heini155). Der vollständige Projektlizenztext ist zusätzlich in der eigenständig nutzbaren `paperark.html` unter **ABOUT → PaperArk MIT License** enthalten.
