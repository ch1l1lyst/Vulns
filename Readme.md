## Flujo típico de recon (en orden)

```
1. ping           → host vivo
2. nmap -p-       → puertos abiertos
3. nmap -sCV      → versiones + scripts
4. whatweb        → stack web (solo si hay HTTP)
5. curl -I        → headers de cada HTTP
6. ffuf / gobuster → dirs ocultos en cada HTTP
7. redis-cli INFO / mongo / etc → services no-web
8. searchsploit   → CVE de las versiones exactas
9. gobuster vhost → virtual hosts si hay dominio
```

---

## Recon.md

````markdown
---
tags: [recon]
---

# Recon

## 1. Host vivo

```bash
ping -c 1 TARGET
```
````

![[Ping.PNG]]

TTL 64 = Linux, 128 = Windows.

## 2. Puertos abiertos (full TCP)

```bash
nmap -p- --min-rate 5000 -n -oG ScanALL TARGET
extractPorts ScanALL
```

![[NmapScanAll.PNG]]
![[ExtractPorts.PNG]]
Rápido, sin DNS, guarda grepable. `extractPorts` copia la lista al clipboard.

## 3. Versiones + scripts default

```bash
nmap -p<PUERTOS> -sCV -oN ScanPorts TARGET
```

![[NmapScanPorts.PNG]]
`-sC` scripts + `-sV` versión. Acá salen CVEs.

## 4. Fingerprint web (por cada puerto HTTP)

```bash
whatweb -a 3 http://TARGET:PORT
curl -skI http://TARGET:PORT/
```

Lee `Server`, `X-Powered-By`, cookies, meta generator.

## 5. Directorios ocultos

```bash
ffuf -u http://TARGET:PORT/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -mc 200,301,302,401,403 -t 50
```

Busca `/admin`, `/api`, `/backup`, `.git`, etc.

## 6. Servicios no-web

```bash
redis-cli -h TARGET INFO          # Redis unauth
mongosh TARGET:27017              # Mongo unauth
mysql -h TARGET -u root           # MySQL sin pass
smbclient -L //TARGET -N          # SMB null
```

Cada uno tiene su propio PoC — abrir nota aparte si pega.

## 7. CVEs de las versiones

```bash
searchsploit "Werkzeug 3.1.9"
searchsploit "aiohttp 3.14"
```

También: Google `"<software> <version> CVE"` + revisar GitHub PoCs.

## 8. Virtual hosts (si hay dominio)

```bash
ffuf -u http://TARGET -H "Host: FUZZ.dominio.com" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fs <tamaño_respuesta_default>
```
