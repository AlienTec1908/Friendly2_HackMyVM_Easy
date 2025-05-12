# Friendly2 - HackMyVM (Easy) - Writeup

![Friendly2 Icon](Friendly2.png)

Dieses Repository enthält einen zusammengefassten Bericht über die Kompromittierung der HackMyVM-Maschine "Friendly2" (Schwierigkeitsgrad: Easy).

## Metadaten

*   **Maschine:** Friendly2 (HackMyVM - Easy)
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Friendly2](https://hackmyvm.eu/machines/machine.php?vm=Friendly2)
*   **Autor des Writeups:** DarkSpirit
*   **Datum:** 2023-05-10
*   **Original Writeup:** [https://alientec1908.github.io/Friendly2_HackMyVM_Easy/](https://alientec1908.github.io/Friendly2_HackMyVM_Easy/)

## Zusammenfassung des Angriffspfads

Nach der initialen Zielidentifizierung mittels `arp-scan` und Zuordnung des Hostnamens `friendly.hmv` in `/etc/hosts` wurde eine Web-Enumeration mit `gobuster` durchgeführt. Diese offenbarte unter anderem ein `/tools`-Verzeichnis.

Der genaue Fundort wird im Original-Writeup nicht spezifiziert, aber es wurde ein privater SSH-Schlüssel (`id_friend`) für den Benutzer `gh0st` gefunden. Dieser Schlüssel war mit einer Passphrase geschützt, die mittels `ssh2john` und `john` (`rockyou.txt`) als `celtic` geknackt wurde. Dies ermöglichte den initialen SSH-Zugriff als Benutzer `gh0st`.

Im Home-Verzeichnis von `gh0st` wurde die User-Flag gefunden. Die weitere Enumeration führte zum Verzeichnis `/var/www/html/tools/`, wo ein Binary namens `joker` lag. Das Ausführen von `./joker -p` setzte die effektive Benutzer-ID auf `root` (`euid=0`), obwohl die reale ID `gh0st` blieb.

Innerhalb dieser privilegierten Shell wurde `/root/root.txt` untersucht, welche sich als "Fake"-Flag herausstellte und auf eine weitere Suche verwies. Die systemweite Suche nach `.txt`-Dateien (`find`) förderte die Datei `/.../ebbg.txt` zutage. Diese enthielt eine kodierte Nachricht (`98199n723q0s44s6rs39r33685q8pnoq`) mit dem Hinweis, dass die Zahlen unkodiert seien.

Der Schritt, wie diese kodierte Nachricht entschlüsselt und für den finalen Root-Zugriff genutzt wurde, ist im Original-Writeup nicht dokumentiert. Nach der (impliziten) erfolgreichen Anwendung dieses Hinweises wurde der Root-Zugriff erlangt und die tatsächliche Root-Flag gelesen.

## Verwendete Tools (Auswahl)

*   `arp-scan`
*   `gobuster`
*   `vi` / Editor
*   `chmod`
*   `ssh`
*   `ssh2john`
*   `john`
*   `ls`
*   `cat`
*   `cd`
*   `joker` (Binary)
*   `id`
*   `su` (erfolglos in joker-Shell)
*   `find`
*   `ss`

## Angriffsschritte (Übersicht)

1.  **Reconnaissance:** Ziel-IP (`arp-scan`), Hostname (`friendly.hmv` in `/etc/hosts`). *(Nmap-Scan fehlt im Writeup, aber Webserver und SSH werden später genutzt).*
2.  **Web Enumeration:** `gobuster` -> findet u.a. `/tools` Verzeichnis.
3.  **Key Discovery & Prep:** SSH-Schlüssel `id_friend` für `gh0st` finden (Fundort unklar). Berechtigungen setzen (`chmod 600`).
4.  **Passphrase Crack:** Schlüssel für John aufbereiten (`ssh2john id_friend > hash`). Passphrase knacken (`john --wordlist=rockyou.txt hash`) -> `celtic`.
5.  **Initial Access:** Login via `ssh gh0st@friendly.hmv -i id_friend` mit Passphrase `celtic`.
6.  **User Flag:** `cat /home/gh0st/user.txt`.
7.  **Privilege Escalation (SUID):** Binary `/var/www/html/tools/joker` finden. Ausführen mit `./joker -p` -> erlangt `euid=0`.
8.  **Root Flag Hint:** `cat /root/root.txt` (Fake Flag). Mittels `find / -name *.txt` die Datei `/.../ebbg.txt` finden. Kodierte Nachricht analysieren.
9.  **Root Access:** *Anwenden der Lösung des `ebbg.txt`-Rätsels (Schritt im Writeup nicht dokumentiert)* -> Erlangen einer vollen Root-Shell.
10. **Root Flag:** `cat /root/root.txt`.

## Flags

*   **User Flag (`/home/gh0st/user.txt`):** `ab0366431e2d8ff563cf34272e3d14bd`
*   **Root Flag (`/root/root.txt`):** `5C42D6BB0EE9CE4CB7E7349652C45C4A`

---

## Disclaimer

Die hier dargestellten Informationen und Techniken dienen ausschließlich Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Methoden dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Genehmigung vorliegt (z.B. in CTF-Umgebungen wie HackMyVM, Penetrationstests mit schriftlicher Erlaubnis). Das unbefugte Eindringen in fremde Computersysteme ist illegal und strafbar. Die Autoren übernehmen keine Haftung für missbräuchliche Verwendung der bereitgestellten Informationen. Handeln Sie stets legal und ethisch verantwortlich.

---
