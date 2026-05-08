# AdGuard Home – Desplegament amb Docker Compose

## Descripció
Desplegament d'AdGuard Home com a servidor DNS local amb bloqueig de publicitat
i tracking, usant Docker Compose sobre Kali Linux.

---

## Requisits previs
- Kali Linux
- Docker (`docker.io`)
- Docker Compose v2

### Instal·lació de dependències
```bash
sudo apt install -y docker.io
sudo systemctl start docker && sudo systemctl enable docker
sudo usermod -aG docker $USER

# Docker Compose manual (no disponible com a paquet a Kali)
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-linux-x86_64" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

---

## Estructura del projecte

adguard-home/
├── docker-compose.yml
├── README.md
└── adguard/
├── work/    # dades en temps real (stats, logs)
└── conf/    # configuració persistent

---

## Desplegament

```bash
git clone https://github.com/EL_TEU_USUARI/adguard-home.git
cd adguard-home
mkdir -p adguard/work adguard/conf
docker-compose up -d
```

Accedeix a la Web UI de configuració inicial:

http://localhost:3000

---

## Configuració DNS de la màquina

Editar `/etc/resolv.conf` per apuntar a AdGuard:
```bash
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
sudo chattr +i /etc/resolv.conf  # fer-ho persistent
```

Verificació:
```bash
dig google.com @127.0.0.1
# Resposta: 142.251.140.238 (IP real = funciona)
```

---

## Llistes de bloqueig actives

| Llista | Descripció |
|--------|------------|
| AdGuard DNS filter | Publicitat i tracking (oficial AdGuard) |
| AdGuard DNS Popup Hosts filter | Popups i notificacions |
| Dan Pollock's List | Hosts de publicitat coneguts |
| Steven Black's List | Llista unificada ads + malware |

---

## Demostració de bloqueig

```bash
# Dominis de publicitat → retornen 0.0.0.0 (BLOQUEJATS)
dig doubleclick.net @127.0.0.1      # → 0.0.0.0 ✅
dig googleadservices.com @127.0.0.1 # → 0.0.0.0 ✅
dig ads.yahoo.com @127.0.0.1        # → 0.0.0.0 ✅

# Domini legítim → retorna IP real (NO bloquejat)
dig google.com @127.0.0.1           # → 142.251.140.238 ✅
```

---

## Decisions de disseny

**Volums persistents:** Les carpetes `adguard/work` i `adguard/conf` es munten
com a volums per garantir que la configuració i les estadístiques no es perden
en reiniciar el contenidor.

**Xarxa bridge aïllada:** S'utilitza una xarxa Docker dedicada (`adguard-net`)
per aïllar el servei.

**Port 53 lliure:** A Kali Linux no s'executa `systemd-resolved`, per tant el
port 53 estava disponible sense cap configuració addicional.

**Llistes triades:** Es van seleccionar llistes amb bona reputació i manteniment
actiu, prioritzant la llista oficial d'AdGuard per la seva fiabilitat.

---

## Ports utilitzats

| Port | Protocol | Servei |
|------|----------|--------|
| 53 | TCP/UDP | DNS |
| 80 | TCP | Web UI |
| 3000 | TCP | Setup inicial |
| 443 | TCP | HTTPS (opcional) |

## Captures 

Captura 1 — Dashboard principal
<img width="1366" height="660" alt="image" src="https://github.com/user-attachments/assets/cfa13f35-9569-4470-a734-6ddd1377365e" />

Captura 2 — Llistes de bloqueig actives
<img width="1366" height="662" alt="image" src="https://github.com/user-attachments/assets/75629892-ea22-40f3-8230-0802530bdcfd" />

Captura 3 — Query Log 
<img width="618" height="626" alt="image" src="https://github.com/user-attachments/assets/a36d04bc-d7f6-42a4-9277-3147b758ddf3" />

Captura 4 — Configuració DNS 
<img width="612" height="437" alt="{6BABE1A0-6C2C-465F-BC33-F4BD4228FC76}" src="https://github.com/user-attachments/assets/1dd9249c-e535-48c6-ad5f-a6a35cafa41c" />


