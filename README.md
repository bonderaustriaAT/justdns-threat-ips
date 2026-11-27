# JustDNS Threat-IPs (v4)

Eine **IPv4-Reputationsliste** von Quell-IPs, die wiederholt unautorisierte
SSH-Login-Versuche gegen JustDNS-Infrastruktur gefahren haben. Automatisch
gepflegt, live aktualisiert.

> **Das ist KEINE DNS-Blockliste.** Hier stehen IP-Adressen, keine Domains.
> Nicht in einen DNS-Resolver (JustDNS/Pi-hole/AdGuard) laden. Diese Liste
> gehört in eine **Firewall / ein ipset / eine RPZ-IP-Regel**.

## Format

Eine IPv4-Adresse pro Zeile. `#` leitet einen Kommentar ein. Keine CIDR-Blöcke,
keine Ports, keine Domains.

```
203.0.113.7
198.51.100.42
```

## Qualität

Die Liste ist automatisiert, aber gefiltert: Einzel-Ausreißer werden ignoriert,
nur global-routbare IPv4 aufgenommen, bekannte Gute (Resolver, eigene Infra)
bleiben per Allowlist draußen. Ziel sind wenige Fehltreffer.

## Fehleintrag gemeldet?

Issue öffnen mit der IP. Sie wird entfernt und ausgenommen. Dynamische/geteilte
IPs (CGNAT, Mobilfunk, Cloud-Exits) gehören nicht dauerhaft auf eine öffentliche
Liste; im Zweifel fliegt eine IP raus.

## Nutzung (Firewall, nicht DNS)

### Richtige URL: raw, nicht blob

Immer die **raw-URL** einbinden, nie die GitHub-Weboberfläche:

```
https://raw.githubusercontent.com/bonderaustriaAT/justdns-threat-ips/main/threat-ips-v4.txt
```

Die blob-URL (`https://github.com/.../blob/main/threat-ips-v4.txt`) liefert eine
HTML-Seite, keinen Klartext. Ein Fetcher, der sie lädt, findet darin nur ein
paar zufällige IP-artige Strings statt der ganzen Liste (typisches Symptom:
"erkennt nur 3 Einträge"). In der GitHub-Ansicht auf **Raw** klicken und die dann
angezeigte URL verwenden, sie beginnt mit `raw.githubusercontent.com`.

### ipset + iptables (Linux-Gateway)

```sh
ipset create justdns-threat hash:ip -exist
curl -fsSL https://raw.githubusercontent.com/bonderaustriaAT/justdns-threat-ips/main/threat-ips-v4.txt \
  | grep -vE '^\s*#|^\s*$' \
  | while read ip; do ipset add justdns-threat "$ip" -exist; done
iptables -I INPUT -m set --match-set justdns-threat src -j DROP
```

### UniFi (UDM / UDM Pro / Cloud Gateway)

UniFi kann die Liste als externe IP-Quelle ziehen. Überall, wo nach einer
URL/Link zur Liste gefragt wird, die **raw-URL** von oben eintragen, nicht die
blob-URL. Die Liste ist bereits UniFi-tauglich: eine IPv4 pro Zeile, `#`-Zeilen
sind Kommentare und werden ignoriert, keine CIDR-Blöcke.

Je nach UniFi-Version zwei Wege:

- **Firewall-Gruppe (Port/IP Group) per URL:** Settings > Security (bzw.
  Firewall & Security) > eine IP-Gruppe anlegen und als Quelle die raw-URL
  eintragen. UniFi holt und parst die Zeilen selbst. Die Gruppe dann in einer
  Firewall-/Traffic-Regel als Quelle auf DROP/Block setzen (Richtung WAN IN).
- **CyberSecure / Custom Threat Feed:** falls lizenziert, unter den
  Threat-Management-Feeds die raw-URL als eigenen Feed hinterlegen.

## Lizenz

CC0 / Public Domain.
