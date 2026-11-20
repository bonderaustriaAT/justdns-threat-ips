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

Beispiel mit `ipset` + `iptables` auf einem Linux-Gateway:

```sh
ipset create justdns-threat hash:ip -exist
curl -fsSL https://raw.githubusercontent.com/bonderaustriaAT/justdns-threat-ips/main/threat-ips-v4.txt \
  | grep -vE '^\s*#|^\s*$' \
  | while read ip; do ipset add justdns-threat "$ip" -exist; done
iptables -I INPUT -m set --match-set justdns-threat src -j DROP
```

## Lizenz

CC0 / Public Domain.
