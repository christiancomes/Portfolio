# Preparación Nodos Nucster — Proxmox VE 9.2.2 sin nag + ficha técnica + previo a clúster

> Objetivo: nodos independientes nucster10 (.10) / nucster20 (.20) / nucster30 (.30) listos para crear el clúster `Nucster`, sin cartel de suscripción, con ficha técnica para blog (pfSense + VLANs después).
> Aplica en cada nodo salvo que se indique. Acceso: Web `https://192.168.1.XX:8006` + Shell root (nodo > Shell o SSH).
> Estado 2026-10-13 13:18: ✅ CLUSTER Nucster 3/3 quorate v3 FINAL (nodes dir solo minúsculas, Shell 3x3 OK). Día cerrado: sin nag, fichas, red, clúster. Siguiente: pfSense en 30 + VLANs + réplica/HA.

---

## Paso 1 — Quitar cartel “No valid subscription”

Son 2 cosas distintas: (A) repos enterprise que dan error en `apt update`, (B) popup JS del nag.

### 1.1 Ver estado actual

```bash
pveversion -v
ls -l /etc/apt/sources.list.d/
cat /etc/apt/sources.list.d/*.sources
apt update
```

Nota PVE9: no existe `/etc/apt/sources.list` ni `*.list`, solo `*.sources` (DEB822). Al inicio verás `401 Unauthorized` en enterprise pve + ceph-squid.

Esperado (tus capturas): `pve-manager/9.2.2`, kernel `7.0.2-6-pve`, repos enterprise activos.

### 1.2 Cambiar a repositorio no-subscription (PVE9 usa formato DEB822 `.sources`, no `.list`)

```bash
ls -l /etc/apt/sources.list.d/
cat /etc/apt/sources.list.d/*.sources

# 1. PVE enterprise -> no-subscription (conserva backup):
cp /etc/apt/sources.list.d/pve-enterprise.sources /root/pve-enterprise.sources.bak
sed -i 's|https://enterprise.proxmox.com|http://download.proxmox.com|; s/Components: pve-enterprise/Components: pve-no-subscription/' /etc/apt/sources.list.d/pve-enterprise.sources

# 2. Ceph: como no usarás Ceph (discos heterogéneos), desactívalo:
# Opción A (recomendada): desactivar:
echo "Enabled: false" >> /etc/apt/sources.list.d/ceph.sources
# Opción B: si algún día lo quieres sin suscripción, en vez de lo anterior:
# sed -i 's|https://enterprise.proxmox.com|http://download.proxmox.com|; s/Components: enterprise/Components: no-subscription/' /etc/apt/sources.list.d/ceph.sources

cat /etc/apt/sources.list.d/*.sources
apt update
```

Si `apt update` ya no da error de 401, OK. Estado verificado en nucster10 (2026-10-13) — ✅ 1.2 COMPLETADO:
- Inicial: `ceph.sources` + `pve-enterprise.sources` en `https://enterprise.proxmox.com` -> `401`.
- Final `pve-enterprise.sources`: `URIs: http://download.proxmox.com/debian/pve` + `Components: pve-no-subscription`.
- Final `ceph.sources`: mismo contenido + `Enabled: false` al final.
- `apt update` final: 4x `Hit` (debian, trixie-updates, security, download pve), `182 packages can be upgraded`, cero `Err`.
- Backups: `/root/pve-enterprise.sources.bak` (repetir `sed` dos veces fue innecesario, es idempotente).
- No ejecutes el `sed` dos veces (es idempotente, pero innecesario). Haz backup una sola vez.

### 1.3 Actualizar sistema — ✅ nucster10 COMPLETADO (2026-10-13)

```bash
apt list --upgradable | head -30
apt full-upgrade -y
pveversion -v
# Si hubo kernel nuevo -> reiniciar desde GUI (Reboot) o:
reboot
```

Estado nucster10: `7.0.2-6-pve` -> `7.0.14-23-pve #1 SMP PREEMPT_DYNAMIC (2026-10-07)` + reboot OK. Post-upgrade: `pve-manager 9.2.21/4f6e0ac86f9e8c7f`, `widget-toolkit 5.2.10`, `pve-ha-manager 5.2.5`. Repite en nucster20/30. Deja los 3 en la misma `pveversion`.

### 1.4 Parchear el popup JS (truco community) — ⏳ EN CURSO nucster10

> Nota: Proxmox solo lo quita oficialmente con suscripción. Este parche se pierde en cada actualización de `proxmox-widget-toolkit` y hay que re-aplicarlo (por eso reapareció tras el 1.3).

```bash
pveversion -v | grep -E "widget|manager"
grep -n "No valid sub" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js /root/proxmoxlib.js.bak-$(date +%F)

sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
grep -n "void({ //Ext.Msg.show" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js  # debe dar 1 línea = parche OK (el title sigue existiendo pero como código muerto)

systemctl restart pveproxy.service
```

Verificación: `Ctrl+Shift+R` (limpiar caché JS), cierra sesión, entra a `https://192.168.1.10:8006`, navega 2-3 secciones. No debe salir el popup. Repite en .20 y .30. Evidencia nucster10: `.js` 852881 B vs `.bak` 852872 B (+9 B).

> Nota toolkit 5.2.10 (manager 9.2.21): el `title` sigue apareciendo en `grep` tras el parche porque queda como código muerto dentro de `void({ //...`. Verifica con `grep -n "void({ //Ext.Msg.show"`.

#### Depuración 5.2.10

```bash
sed -n '615,645p' /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
ls -l /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js* /root/proxmoxlib.js.bak*
sed -n '615,645p' /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js > /root/nag-block.txt
cat /root/nag-block.txt
```

Reversión si algo falla:

```bash
cp /root/proxmoxlib.js.bak-* /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
systemctl restart pveproxy.service
```

---

## Paso 2 — Ficha técnica por nodo (neofetch/fastfetch + captura)

`neofetch` está abandonado en Debian 13. Para blog usa `fastfetch` (igual de visual, mantenido). Si quieres estética `neofetch` clásica, instala ese.

### 2.1 Instalar

```bash
apt update
apt install -y fastfetch
# opcional nostálgico:
# apt install -y neofetch
```

### 2.2 Captura texto (para el post) + captura pantalla (para imagen)

```bash
hostname; echo "---"
fastfetch --logo none > /root/ficha-$(hostname).txt
cat /root/ficha-$(hostname).txt

echo "===== pve =====" >> /root/ficha-$(hostname).txt
pveversion -v >> /root/ficha-$(hostname).txt
echo "===== cpu =====" >> /root/ficha-$(hostname).txt
lscpu | grep -E "Model name|Socket|Core|Thread|MHz" >> /root/ficha-$(hostname).txt
echo "===== ram/disco/red =====" >> /root/ficha-$(hostname).txt
free -h >> /root/ficha-$(hostname).txt
lsblk -d -o NAME,SIZE,MODEL,SERIAL >> /root/ficha-$(hostname).txt
smartctl -H /dev/sda 2>/dev/null | head -5 >> /root/ficha-$(hostname).txt
ip -br a >> /root/ficha-$(hostname).txt
cat /root/ficha-$(hostname).txt
```

Luego en Shell interactivo ejecuta `fastfetch` a secas y haz captura como las `nucsterXX_summary.png` que ya tienes.

### 2.3 Qué anotar (ya detectado en tus capturas, confirma con fastfetch)

| Nodo | IP | CPU | RAM | Root HD | Disco SDA | Wear |
|------|----|-----|-----|---------|-----------|------|
| nucster10 | 192.168.1.10 | 4x Celeron J3455 @1.5GHz | 7.17 GiB | 93.93 GiB (4.14 usado) | CT1000BX500SSD1 1TB s/n 2522E9C103AE | 0% |
| nucster20 | 192.168.1.20 | idem | 7.60 GiB | 93.93 GiB (4.18 usado) | CT1000BX500SSD1 1TB s/n 2523E9C27963 | 0% |
| nucster30 | 192.168.1.30 | idem | 7.60 GiB | 66.45 GiB (4.14 usado) | Samsung 860 EVO 250GB s/n S3YJNX0K870009W | 18% |

Guarda: `/root/ficha-nucster10.txt`, etc. + PNGs en `nucster_primeras_capturas/nucsterXX/`. Servirán para entrada blog `00-ficha-tecnica`.

---

## Paso 3 — Dejar todo listo para `pvecm create Nucster` (sin crear clúster aún)

No ejecutes `pvecm` todavía. Esto es previo obligatorio.

### 3.1 Red base coherente

```bash
# En cada nodo:
hostnamectl hostname nucster10  # /20 /30 según nodo
cat /etc/hosts
# Debe contener los 3:
# 192.168.1.10 nucster10
# 192.168.1.20 nucster20
# 192.168.1.30 nucster30
```

Edita con `nano /etc/hosts` si falta alguna línea. Luego:

```bash
ping -c2 nucster10; ping -c2 nucster20; ping -c2 nucster30
cat /etc/resolv.conf  # DNS: pon tu gateway o 1.1.1.1 si no hay DNS local aún
timedatectl; chronyc sources 2>/dev/null || systemctl status systemd-timesyncd
```

Switch Zyxel GS1200-8 (192.168.1.3): cambia Gateway de `0.0.0.0` a `192.168.1.1` (o tu router), reserva puertos 6-8 libres para WAN/LAN pfSense.

> Diseño con nucster30 = pfSense (único con USB-Ethernet + onboard): el USB (normalmente Realtek r8152/8153) es inestable para trunk/HA (cortes, renombres `enx...`, sin VLAN offload). Úsalo solo como WAN dedicada o mgmt, y lleva todo el tráfico LAN/LAB por la onboard en trunk al switch (router-on-a-stick). pfSense en un solo nodo es SPOF: para HA real futuro necesitarías 2 pfSense en CARP en 2 nodos. Verifica en nucster30:
> ```bash
> lsusb | grep -i eth; ip -br link; ls /sys/class/net
> ```

### 3.2 Bridge VLAN-aware (preparar pfSense + VLANs, sin romper acceso) — ✅ VERIFICADO 2026-10-13

GUI: `Nodo > System > Network > vmbr0 > Edit > VLAN aware: Yes > Apply`, luego `Apply Configuration`. No cambies IP de `vmbr0` hoy.

```bash
cat /etc/network/interfaces
bridge-vlan-aware yes  # debe aparecer
```

Estado: 10/20/30 ya `Yes` en `nic0`, CIDR `.10/.20/.30/24`, Gateway `192.168.1.1`. 30 muestra `enx00e04c...` USB presente pero inactivo (reservar para WAN). `wlp2s0` y `nic1` inactivos. MACs ocultas en capturas ✅.

Plan VLAN sugerido para blog (define ahora, crea después):

- VLAN 1 native / management 192.168.1.0/24
- VLAN 10 LAN tras pfSense, VLAN 20 LAB, VLAN 99 WAN (al router)

### 3.3 Storage: no usar Ceph productivo, pero HA sí se puede practicar

nucster30 = 250GB vs 1TB en otros + 18% wear = no apto para Ceph productivo ni ZFS replicado a 3. Pero como quieres aprender HA/failover:
- Quórum ya lo tienes: 3 nodos -> tolera 1 caída (2/3 votos). Bien.
- HA sin storage compartido solo reinicia la VM en otro nodo y PIERDE disco si es `local-lvm`. Para practicar failover real necesitas réplica:
  - Opción 1 (recomendada con tu HW): réplica ZFS / `pvesr` entre nucster10 <-> nucster20 (ambos 1TB iguales), nucster30 solo quórum + CTs ligeros.
  - Opción 2: NFS compartido (una VM o NAS externo) como `shared storage` para migrar en vivo. Punto único de fallo, pero vale para lab.
  - Opción 3 (solo lab efímero): Ceph con 1 OSD por nodo limitado a 250GB para ver HA completo, sabiendo que desperdicias 750GB y va lento en 1G + J3455/7GB RAM.

Verifica en GUI: `Datacenter > Storage` igual en los 3. No actives HA hoy, solo deja storage listo.

### 3.4 Base común + checklist pre-clúster

```bash
apt install -y qemu-guest-agent vim curl sudo htop
systemctl status pve-cluster pvedaemon pveproxy --no-pager | head -20
```

Checklist antes de crear clúster (HOY si das luz verde):

- [x] Los 3 en `PVE 9.2.21 / kernel 7.0.14-23` (declarado) — verificar con `pveversion -v | grep -E "manager|kernel"` sin updates: `apt update && apt list --upgradable`
- [x] `ping` entre hostnames OK, NTP sincronizado (declarado)
- [ ] Sin VMs/CTs con mismo VMID en distintos nodos (`qm list`, `pct list` vacíos o sin choques) — FALTA EVIDENCIA
- [ ] Backup de `/etc/pve`, `/etc/network/interfaces`, `/etc/hosts` — FALTA
- [ ] Decidido: `pvecm create Nucster` se ejecuta en nucster10, luego `pvecm add 192.168.1.10` desde 20 y 30

### 3.5 Pre-vuelo + creación `Nucster` — ✅ COMPLETADO 2026-10-13 (3/3, v3, quorate)

```bash
# 1. En los 3 nodos (pre-vuelo, inocuo):
pveversion -v | grep -E "manager|widget"
pvecm status
qm list; pct list
cat /etc/hosts; cat /etc/resolv.conf
timedatectl | grep -E "synchronized|NTP"

# 2. Backup en los 3 (inocuo):
tar -czf /root/nucster-backup-$(hostname)-$(date +%F).tgz /etc/pve /etc/network/interfaces /etc/hosts /etc/apt/sources.list.d

# 3. SOLO en nucster10 (destructivo leve, irreversible sin rebuild):
pvecm create Nucster --link0 192.168.1.10
pvecm status  # debe dar 1 nodo, quorum 1/1, versión 1

# 4. En nucster20 y luego nucster30 (pide password root del 10):
pvecm add 192.168.1.10 --link0 192.168.1.20  # en 20
pvecm add 192.168.1.10 --link0 192.168.1.30  # en 30
pvecm status  # en cualquiera: 3 nodos, quorum 3/3
```

Aviso SDD: `create` es irreversible (hay que destruir y recrear para deshacer), `pveproxy` se reinicia y la GUI parpadea. Sin VMs en marcha es seguro, con VMs hay que migrar/apagar antes. No lo ejecutes hasta que me pases el pre-vuelo (1) y me digas `adelante`.

Con esto cierras el día: sin nag, fichas para blog, red lista para pfSense.
