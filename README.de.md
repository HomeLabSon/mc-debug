# mc-debug

[English](README.md) | **Deutsch**

Rein lesender Debug-Helfer für [mailcow-dockerized](https://github.com/mailcow/mailcow-dockerized).
Er bündelt die Container-, Datenbank- und `doveadm`-Aufrufe, die man braucht, wenn "Mails fehlen" oder ein IMAP-Sync
hängt, damit man sich die langen Einzeiler nicht merken muss. Er ändert nichts am System.

## Voraussetzungen

- Ausführung auf dem mailcow-Host als root oder als Benutzer mit Docker-Zugriff
- Docker Compose v2
- Auswertung der imapsync-Reports getestet mit imapsync 2.178 (in mailcow enthalten); getestete mailcow-Version: 2026-10

## Installation

    install -m 0755 mc-debug /usr/local/bin/mc-debug

Das mailcow-Verzeichnis ist standardmäßig `/opt/mailcow-dockerized`. Abweichend: `MAILCOW_DIR=/pfad mc-debug ...`.

## Benutzung

    mc-debug sync                       Status aller IMAP-Sync-Jobs (auf min_ago und running achten)
    mc-debug syncreport <job-id>        Kurzreport des letzten Laufs eines Jobs
    mc-debug runner                     läuft gerade ein imapsync? Ist das Lock gesetzt?
    mc-debug box <user@domain>          Mailanzahl je Ordner
    mc-debug find <user@domain> <text>  Mails per Betreff-Text in allen Ordnern suchen
    mc-debug trace <text> [zeitraum]    Postfix/Rspamd/Dovecot-Logs durchsuchen (30m, 3h, 2d; Standard 3h)
    mc-debug errors [zeitraum] [all]    Postfix/Dovecot-Probleme nach Meldung gruppiert, Scanner-Rauschen ausgeblendet (Standard 24h)
    mc-debug stats [zeitraum]           Kurzstatistik: Zähler, Rspamd-Aktionen, Top-Domains (Standard 24h)

## Beispiel: Sync-Jobs laufen nicht mehr durch

Symptom: Per IMAP-Sync abgeholte Mails kommen verspätet oder gar nicht an, aber nur manchmal.

1. `mc-debug sync` zeigt einen Job mit `running = 1` und großem `min_ago`, während alle anderen Jobs ebenfalls überfällig sind.
2. `mc-debug runner` zeigt einen einzelnen lange laufenden imapsync-Prozess, der das Lock hält.
3. `mc-debug syncreport <id>` zeigt, was dieser Job gerade tut, z. B. tausende Mails, die einzeln auf der
   Gegenseite entfernt werden.

In unserem Fall arbeitete der Runner die Jobs strikt nacheinander ab, ein langsamer Job verzögerte also alle anderen.
Ursache war ein Gmail-Konto, das gelöschte Mails in "Alle Nachrichten" behielt. Dieselben Mails wurden bei jedem
Lauf verglichen, zurückkopiert und erneut gelöscht.

## Hinweise

- Tabellen- und Spaltennamen (`imapsync`, `returned_text`, ...) stammen aus der mailcow-Datenbank und können sich zwischen Versionen ändern.
- `find` sucht über den dekodierten Betreff per `doveadm`; `trace` findet nur, was die Dienste tatsächlich loggen
  (Queue-IDs, Adressen, Message-IDs), meist keine Betreffs.

## Hinweis zur KI-Unterstützung

Dieses Tool wurde mit Unterstützung von KI-Assistenten erstellt und wird vom Autor auf einer laufenden mailcow-Installation eingesetzt.

## Lizenz

MIT
