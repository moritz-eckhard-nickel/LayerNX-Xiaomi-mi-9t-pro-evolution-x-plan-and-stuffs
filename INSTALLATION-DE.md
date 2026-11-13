# Evolution X auf Xiaomi Mi 9T Pro / Redmi K20 Pro (raphael)

Zielpaket: `EvolutionX-17.0-20261112-raphael-12.4-Official.zip`.
Diese Anleitung beschreibt eine Neuinstallation mit Datenlöschung. Die Reihenfolge und die benötigten Images wurden am 13.11.2026 mit der offiziellen Geräteseite und deren Installationsanleitung abgeglichen: https://evolution-x.org/device/raphael

## 1. Vorher prüfen und sichern

- Nur für das tatsächlich unterstützte Gerät mit Codename **raphael**. Mi 9T ohne Pro ist ein anderes Gerät. Bei einer anderen Variante, etwa raphaelin, zuerst deren ausdrücklich dokumentierte Unterstützung prüfen.
- Der Bootloader muss bereits entsperrt sein. Das Aktivieren von „OEM-Entsperrung“ allein entsperrt ihn nicht. Falls er gesperrt ist: zuerst Xiaomis offiziellen, modellgerechten Mi-Unlock-Prozess abschließen. Keine generischen Unlock-Befehle ausprobieren. Auch das Entsperren kann alle Daten löschen.
- **Format Data löscht Apps und persönliche Daten einschließlich internem Speicher.** Fotos, Downloads, Messenger-Backups und 2FA-Wiederherstellungscodes extern sichern und die Sicherung prüfen. Zugangsdaten für Google-/Xiaomi-Konten bereithalten.
- Akku möglichst über 60 %, zuverlässiges USB-Datenkabel, direkter USB-Port am Computer; keinen instabilen Hub verwenden.
- Die offizielle Geräteseite verlinkt für diesen Build die Firmware **V12.5.2.0.RFKMIXM**: https://xmfirmwareupdater.com/firmware/raphael/stable/V12.5.2.0.RFKMIXM/ . Vor dem Flashen prüfen, welche Firmware dein Gerät aktuell hat und ob für deine Variante zusätzliche Vorgaben gelten. Die dort erzeugte Installationsanleitung enthält keinen separaten Firmware-Flashschritt; deshalb hier keine zusätzlichen Flashbefehle auf Verdacht ausführen.
- Alle drei Dateien müssen aus der offiziellen Quelle und zum vorgesehenen raphael-Release gehören. Keine willkürlichen Recovery-Images oder Addons mischen.
- Bootloader nach Installation einer Custom-ROM nicht wieder sperren.

## 2. Dateien auf den Computer bringen

Du brauchst die Dateien auf dem **lokalen Computer, an dem dein Handy per USB hängt**. Eine Datei in dieser Cloud ist noch nicht auf deinem Computer. Die Cloud kann dein lokal angeschlossenes Handy nicht über USB flashen.

Lege einen Ordner an, beispielsweise `C:\Android\raphael` unter Windows oder `~/Android/raphael` unter Linux/macOS. Darin liegen später:

```text
boot.img
dtbo.img
EvolutionX-17.0-20261112-raphael-12.4-Official.zip
```

Die ROM-ZIP **nicht entpacken**. Prüfe unter Windows mit eingeblendeten Dateiendungen, dass kein `.zip.zip` oder `.img.img` entstanden ist. Keine unvollständigen `.part`-Dateien verwenden.

Offizielle Download-Adressen:

- https://cdn.evolution-x.org/raphael/17/boot/boot.img
- https://cdn.evolution-x.org/raphael/17/dtbo/dtbo.img
- https://cdn.evolution-x.org/raphael/17/EvolutionX-17.0-20261112-raphael-12.4-Official.zip

Die zusätzlich bereitgestellten signierten Bunny-CDN-Links laufen nach einer Stunde ab. Bei Ablauf einen neuen Link über die offizielle Download-Seite beziehen. Ein HTTP-Fehler ist keine gültige ROM-Datei.

Falls der Herausgeber Prüfsummen bereitstellt, vor dem Flashen vergleichen. Eigene SHA-256-Werte erkennen spätere Übertragungsfehler, bestätigen ohne offiziellen Vergleichswert aber nicht die Herkunft.

Windows PowerShell:

```powershell
Get-FileHash .\boot.img -Algorithm SHA256
Get-FileHash .\dtbo.img -Algorithm SHA256
Get-FileHash .\EvolutionX-17.0-20261112-raphael-12.4-Official.zip -Algorithm SHA256
```

Linux: `sha256sum boot.img dtbo.img EvolutionX-17.0-20261112-raphael-12.4-Official.zip`

macOS: `shasum -a 256 boot.img dtbo.img EvolutionX-17.0-20261112-raphael-12.4-Official.zip`

## 3. ADB und Fastboot vorbereiten

Aktuelle Android SDK Platform-Tools direkt von Google herunterladen und entpacken:
https://developer.android.com/tools/releases/platform-tools

Am einfachsten kopierst du die drei Dateien in den entpackten `platform-tools`-Ordner. Öffne genau dort ein Terminal. Unter Windows: Ordner im Explorer öffnen, in die Adresszeile `powershell` schreiben und Enter drücken.

Die folgenden Befehle sind für Windows PowerShell geschrieben. Unter Linux/macOS `./adb` und `./fastboot` anstelle von `.\adb.exe` und `.\fastboot.exe` verwenden. Sind die Tools bereits im PATH, reichen `adb` und `fastboot`.

```powershell
.\adb.exe version
.\fastboot.exe --version
```

Beide müssen eine Versionsausgabe liefern. Unter Windows können passende Android-/Xiaomi-USB-Treiber nötig sein. ADB und Fastboot können unterschiedliche Treiber benötigen.

## 4. In den Bootloader wechseln

Variante A: Handy ausschalten. **Leiser + Power** gedrückt halten, bis der Fastboot-Bildschirm erscheint. Danach per USB verbinden.

Variante B: Im laufenden Android USB-Debugging aktivieren, Computer am Handy autorisieren und ausführen:

```powershell
.\adb.exe devices
.\adb.exe reboot bootloader
```

Bei `unauthorized` zuerst die RSA-Abfrage am entsperrten Handy bestätigen. Für Variante A ist USB-Debugging nicht erforderlich.

Im Fastboot-Modus prüfen:

```powershell
.\fastboot.exe devices
.\fastboot.exe getvar product
.\fastboot.exe oem device-info
```

Erwartet: genau dein Gerät mit Seriennummer, Produkt `raphael` und eine eindeutige Bestätigung des entsperrten Bootloaders, soweit der Bootloader diese Abfrage unterstützt. Fastboot schreibt auch normale Ausgaben oft nach stderr; das ist allein kein Fehler. Wenn Produkt oder Unlock-Status unklar sind, vor dem Flashen klären. Wenn `oem device-info` nicht unterstützt wird, nicht auf Verdacht entsperren oder flashen.

Hinweis zum Quellenabgleich: Die aktuelle Geräte-Webseite und ihre erzeugten Flashbefehle nennen für Android 17 **boot und dtbo**. In der separaten OTA-JSON steht zusätzlich `recovery` in der Liste der Installationsimages. Das ist eine Abweichung zwischen offiziellen Metadaten. Diese Anleitung folgt der vom Nutzer gelieferten und auf der Geräteseite bestätigten Befehlsfolge; daraus keinen zusätzlichen Recovery-Flashbefehl ableiten. Bei abweichenden aktuellen Maintainer-Anweisungen vor dem Flashen klären.

## 5. Recovery-Images flashen

**Jede Zeile einzeln ausführen und den Abschluss prüfen.** Im kopierten Ursprungstext fehlte zwischen den beiden Befehlen ein Zeilenumbruch.

```powershell
.\fastboot.exe flash boot .\boot.img
```

Erst wenn Übertragung und Schreiben mit `OKAY` abgeschlossen sind:

```powershell
.\fastboot.exe flash dtbo .\dtbo.img
```

Erst wenn auch das mit `OKAY` abgeschlossen ist:

```powershell
.\fastboot.exe reboot recovery
```

Keine zusätzlichen Partitionen löschen, keinen Slot wechseln und keine Befehle wie `erase`, `flashall` oder `flashing lock` ergänzen. Bei einem `FAILED` zuerst die genaue Meldung klären. Nach dem Flashen direkt in die vorgesehene Recovery starten.

Falls `reboot recovery` nicht unterstützt wird, die gerätespezifisch dokumentierte Tastenkombination verwenden; beim raphael üblicherweise **Lauter + Power**, bis Recovery startet. Nicht mit Leiser + Power verwechseln, das führt zu Fastboot.

## 6. Daten formatieren — hier werden deine Daten gelöscht

Prüfe am Handy, dass die erwartete Evolution-X-Recovery gestartet ist. Die Menütexte können je nach Version leicht abweichen.

1. **Factory Reset** auswählen.
2. **Format Data/Factory Reset** auswählen.
3. Löschung ausdrücklich bestätigen.
4. Auf den erfolgreichen Abschluss warten.
5. Zum Hauptmenü zurückkehren.

Bei Fehlern beim Formatieren nicht einfach mit dem Sideload fortfahren. Ein bloßes Cache-Wipe ersetzt diesen Schritt nicht.

## 7. ROM per ADB sideloaden

Am Handy:

1. **Apply Update** öffnen.
2. **Apply from ADB** auswählen.
3. Handy auf diesem Bildschirm lassen und USB angeschlossen lassen.

Am Computer im Ordner mit den Platform-Tools und der ZIP:

```powershell
.\adb.exe devices
```

Die Seriennummer sollte mit dem Status **sideload** erscheinen. Dieser Modus ist anders als normales Android; eine ADB-Shell muss hier nicht funktionieren.

Dann genau diesen Befehl starten:

```powershell
.\adb.exe sideload .\EvolutionX-17.0-20261112-raphael-12.4-Official.zip
```

Währenddessen Kabel nicht trennen, Computer nicht schlafen lassen und Handy nicht neu starten. ADB überträgt die ZIP; die Recovery prüft und installiert sie. Die ZIP muss dafür nicht vorher im Telefonspeicher liegen.

**Entscheidend ist die Erfolgsmeldung am Handy**, etwa `Install completed` oder eine vergleichbare eindeutige Bestätigung. ADB kann bei ungefähr 47 % stehenbleiben oder am Ende eine ungewöhnliche Lesemeldung zeigen, obwohl die Recovery die Installation erfolgreich abgeschlossen hat. 47 % oder `failed to read command` allein beweisen weder Erfolg noch Misserfolg. Immer Recovery-Ergebnis und eventuelle Fehler lesen.

Bei `signature verification failed`, falschem Gerätebezug, beschädigter ZIP oder Installationsabbruch stoppen und Ursache klären. Signaturprüfung nicht pauschal abschalten und Fehler nicht durch wiederholtes Formatieren übergehen.

## 8. Optionale Addons

Dieser konkrete Build ist laut offizieller Geräteseite ein **gapps-Build**: Google-Apps sind enthalten, zusätzliche GApps daher nicht installieren. Der offizielle Changelog nennt außerdem einen bereits mit **KSU Next** ausgestatteten Kernel; für diese Basisinstallation kein weiteres Root-Paket hinzufügen.

Für die erste Installation kannst du Addons weglassen. GApps, Root und andere Pakete nur installieren, wenn die Dokumentation dieser konkreten ROM sie vorsieht und ihre Version passt. Keine zusätzlichen GApps installieren, wenn die ROM sie bereits enthält.

Laut bereitgestellter Anleitung vor Addons vollständig zurück in die Recovery neu starten, anschließend erneut **Apply Update → Apply from ADB** öffnen. Erst dann ein ausdrücklich kompatibles Paket sideloaden:

```powershell
.\adb.exe sideload .\addon_filename.zip
```

`addon_filename.zip` ist ein Platzhalter. Jedes Paket separat übertragen und das Ergebnis am Handy prüfen. Ob ein Addon vor dem ersten Systemstart installiert werden muss, bestimmt dessen offizielle Anleitung.

## 9. System starten

Nach bestätigter erfolgreicher Installation **Reboot system now** auswählen. Der erste Start kann mehrere Minuten dauern. Nicht nach wenigen Minuten ungeduldig unterbrechen. Bei über etwa 15–20 Minuten unverändertem Bootlogo oder wiederholten Neustarts zur Diagnose in die Recovery zurückgehen; nicht automatisch erneut formatieren.

Nach dem Start Einrichtung abschließen und WLAN, Mobilfunk, Kamera und Fingerabdruck prüfen. Erst danach Backups zurückspielen.

## 10. Häufige Probleme

| Beobachtung | Nächster sinnvoller Schritt |
| --- | --- |
| `adb`/`fastboot` nicht gefunden | Terminal im richtigen Platform-Tools-Ordner öffnen; unter PowerShell `.\` voranstellen. |
| `waiting for any device` / leere Fastboot-Liste | Fastboot-Bildschirm, Datenkabel, direkten USB-Port und Fastboot-Treiber prüfen. |
| ADB zeigt `unauthorized` im laufenden Android | Handy entsperren und Debugging-Abfrage bestätigen. |
| `adb devices` leer in Recovery | **Apply from ADB** erneut wählen; Kabel und Recovery-ADB-Treiber prüfen. |
| Mehrere Geräte / Emulatoren | Andere Geräte/Emulatoren trennen oder mit `-s SERIENNUMMER` gezielt auswählen. |
| `cannot read` / Datei nicht gefunden | Dateiname, Endung, Arbeitsordner und vollständigen Download prüfen. Bei Leerzeichen Pfad in Anführungszeichen setzen. |
| Flashen wegen gesperrtem Bootloader abgelehnt | Stoppen; offiziellen Unlock-Prozess abschließen. |
| ZIP-Verifikation oder Installation fehlgeschlagen | Exakten Fehler sichern, Gerätebezug und offizielle Prüfsumme prüfen, gegebenenfalls neu herunterladen. |
| Sideload scheinbar bei 47 % fertig | Recovery-Ergebnis prüfen; Prozentanzeige allein ist kein Erfolgskriterium. |
| Bootschleife | Recovery aufrufen, Fehlermeldungen/Logs sichern und Geräte-/Firmware-/Release-Voraussetzungen prüfen. Keine zufälligen Images flashen. |

## Download- und Prüfstatus dieser Cloud

Alle drei Dateien wurden am 13.11.2026 erfolgreich über die offiziellen CDN-Adressen mit aktivierter TLS-Prüfung heruntergeladen. Speicherort: `/workspace/shared/downloads/raphael/`.

| Datei | Größe in Bytes | Ergebnis |
| --- | ---: | --- |
| `boot.img` | 134217728 | Android-Bootimage-Header erkannt |
| `dtbo.img` | 33554432 | DTBO-Header erkannt |
| `EvolutionX-17.0-20261112-raphael-12.4-Official.zip` | 3812053104 | Größe und SHA-256 stimmen mit offizieller Geräteseite und OTA-JSON überein; alle 28 ZIP-Einträge bestehen den CRC-Test |

Offizielle SHA-256 der ROM-ZIP:

```text
d624d942ec8c4bde1fa46fc689cd78e0735c8604f084bc0acd773d5c4b839178
```

Die lokalen SHA-256 aller drei Dateien stehen in [`release-metadata/SHA256SUMS.txt`](release-metadata/SHA256SUMS.txt) sowie im Downloadordner. Die beiden separaten Images unterscheiden sich von den gleichnamigen Images innerhalb der ZIP. Separate Images werden hier gemäß offizieller Anleitung zum Starten der Recovery verwendet; die ZIP installiert anschließend ihre eigenen Images. Der Unterschied allein belegt keinen Defekt. Für die separaten Images wurde keine unabhängig veröffentlichte Prüfsumme bestätigt; deren lokale Hashes sind deshalb nur Übertragungsvergleichswerte. Keine Images eigenmächtig austauschen.

Die ZIP-Metadaten und der Installationsskript nennen `raphael` und `raphaelin` als zugelassene Geräte. Daraus folgt keine unabhängig geprüfte Eignung der separat heruntergeladenen Recovery-Images für jede Variante. Diese Anleitung bleibt auf das ausgewählte raphael-Gerät ausgerichtet.

Offizielle Release-Metadaten, OTA-Metadaten und Changelog sind im Downloadordner mitgespeichert. Der Changelog nennt Android 17 QPR0, November-2026-Sicherheitspatches und KSU Next. Die Anleitung wurde nicht an einem angeschlossenen Gerät getestet; es wurde kein Handy geflasht und keine Umgebung veröffentlicht.
