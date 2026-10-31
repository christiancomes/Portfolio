# Mi primer clúster (de verdad): 3 NUCs, mucho foro y una mayúscula que casi me mata — Nucster Día 1

> Soy de ASIR, tengo más ganas que idea, y este es mi homelab: tres NUCs para aprender routing con pfSense, VLANs y HA. Lo documento todo porque si no lo escribo, no lo aprendo. Hoy: inventario, quitar el cartel de pago, fichas y crear el clúster `Nucster`. Spoiler: lo conseguí, pero una mayúscula me dio la lección del día.

## 0. De dónde parto (y qué quiero)

En clase vemos teoría, pero yo quería tocar chicha: un clúster Proxmox de 3 nodos donde romper cosas sin miedo. La idea es clara: pfSense en una VM para aprender routing de verdad, VLANs en el switch para separar mgmt/LAN/LAB/WAN, y más adelante HA y failover para fardar de "si se cae un nodo, ni te enteras". Todo lo que haga irá al blog, con capturas, comandos y cagadas incluidas. Porque vaya si hubo cagadas.

Tres NUC `NUC6CAYH` con Celeron J3455 que tenía por casa, Proxmox VE 9.2.2 recién instalado a lo loco (siguiente, siguiente, sin leer mucho, lo reconozco), cada uno con su IP `192.168.1.10`, `.20` y `.30`, y un Zyxel GS1200-8 en la `.3` que me deja mi primo. Al abrir la GUI lo primero que me saluda, aparte del "No es seguro" del certificado, es el cartelito amarillo: *No valid subscription*. Y otro: *Enterprise repository needs valid subscription*. Genial, empezamos pidiendo perras.

![Switch Zyxel GS1200-8 con 5 puertos a 1000 Full](../../nucster_primeras_capturas/switch.png)

*El switch, con 5 bocas currando a 1G y 3 libres que ya les tengo echado el ojo para la WAN y el trunk.*

## 1. Inventario: los gemelos y el raro

Me pongo a mirar qué tengo, como nos enseñan: primero inventariar, luego tocar. Y aparece la primera sorpresa.

| Nodo | IP | RAM | Disco | Desgaste |
|------|----|-----|-------|----------|
| nucster10 | 192.168.1.10 | 7.17 GiB | Crucial BX500 1 TB | 0% |
| nucster20 | 192.168.1.20 | 7.60 GiB | Crucial BX500 1 TB | 0% |
| nucster30 | 192.168.1.30 | 7.60 GiB | Samsung 860 EVO 250 GB | 18% |

O sea: dos gemelos de 1 TB nuevecitos y un 30 con un Samsung de 250 GB ya con un 18% de desgaste y la raíz de solo 66 GB frente a 93 GB de los otros. Me rayé un poco, pero en los foros me lo dejaron claro: con discos así de distintos, ni se te ocurra Ceph productivo, que te capa al disco pequeño por OSD y en red de 1G con estos Celeron va a pedales. Para practicar HA me vale réplica entre los gemelos o un NFS cutre. Punto. El 30, además, es el único con un adaptador USB-Ethernet aparte de la placa. Ese va a ser el hogar de pfSense, y el USB irá de WAN dedicada. De trunk ni hablar: esos cacharros se llaman `enx...`, se renombran solos y con VLANs dan sustos.

![Resumen nucster10](../../nucster_primeras_capturas/nucster10/nucster10_sumary.png)
![Resumen nucster20](../../nucster_primeras_capturas/nucster20/nucster20_summary.png)
![Resumen nucster30](../../nucster_primeras_capturas/nucster30/nucster30_summary.png)

*Los tres con el mismo J3455, EFI y el aviso de enterprise. Las capturas de discos las quito del post: llevaban los seriales a la vista y eso no se publica, ya aprendí lo de tachar.*

<!-- Sensibles retiradas: nucster10_disks.png (serial 2522E9C103AE), nucster20_disks.png (serial 2523E9C27963), nucster30_disks.png (serial S3YJNX0K870009W) -->

## 2. Fuera el cartel de pago (y susto con los repos nuevos)

Yo venía con apuntes viejos que hablaban de `.list` y me choqué con que PVE 9 usa `.sources` con formato DEB822. El `apt update` me escupía dos `401 Unauthorized` a `enterprise.proxmox.com` (pve y ceph-squid) y me quedé pillado. Documentación oficial + un post del foro y listo:

```bash
cp /etc/apt/sources.list.d/pve-enterprise.sources /root/pve-enterprise.sources.bak
sed -i 's|https://enterprise.proxmox.com|http://download.proxmox.com|; s/Components: pve-enterprise/Components: pve-no-subscription/' /etc/apt/sources.list.d/pve-enterprise.sources
echo "Enabled: false" >> /etc/apt/sources.list.d/ceph.sources
apt update  # 4x Hit, cero Err, qué gustazo
apt full-upgrade -y  # 182 paquetes, kernel 7.0.2-6 -> 7.0.14-23, reboot con miedo
```

Lo de Ceph desactivado es solo el repo, reversible cuando quiera. Para lo que quiero hoy (pfSense, VLANs) no me hace falta, y con mi hardware sería un castigo. Después del reboot: `pve-manager 9.2.21`, `widget-toolkit 5.2.10`. Igualitos los tres, que eso me lo grabo a fuego: un clúster quiere versiones clavadas.

Luego el popup. El truco de la comunidad de toda la vida, que se borra en cada update del toolkit y hay que re-aplicarlo:

```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
grep -n "void({ //Ext.Msg.show" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
systemctl restart pveproxy.service
```

Aquí me tiré un buen rato tonto: hacía `grep` al `title` y seguía saliendo, y yo "¡que no funciona!". Pues claro que sale: queda como código muerto dentro del `void({...})`. Hay que buscar el `void`, no la ausencia. Y luego `Ctrl+Shift+R` en el navegador, que si no sigues viendo el JS viejo en caché y te vuelves loco. Cuando desapareció el cartelito casi grito.

## 3. Fichas con fastfetch (mi neofetch ha muerto, DEP)

Quería unas fichas bonitas para el blog y pedí instalar `neofetch`. Pues resulta que en Debian 13 está abandonado. Otro foro, otro aprendizaje: ahora se lleva `fastfetch`. Lo instalé en los tres, captura por nodo y a comparar:

![fastfetch nucster10](../../nucster_primeras_capturas/nucster10/nucster10_fastfetch.png)
![fastfetch nucster20](../../nucster_primeras_capturas/nucster20/nucster20_fastfetch.png)
![fastfetch nucster30](../../nucster_primeras_capturas/nucster30/nucster30_fastfetch.png)

*PVE 9.2.21 en los tres, J3455 a 2.30 GHz en burst, la RAM que cuadra con el inventario y la IP de `vmbr0`. Solo sale IP privada y modelo de placa, así que se puede publicar. Los seriales y MACs, tachados o fuera.*

## 4. La red: minúsculas, ping y el bridge que ya venía bien

Aquí me puse serio porque el clúster es un tiquismiquis con los nombres. Proxmox quiere minúsculas siempre, y yo los había instalado como `Nucster10` con mayúscula. Así que:

```bash
hostnamectl hostname nucster10  # y 20, 30
# /etc/hosts canónico en los 3:
# 192.168.1.10 nucster10
# 192.168.1.20 nucster20
# 192.168.1.30 nucster30
ping -c2 nucster10; ping -c2 nucster20; ping -c2 nucster30  # 0% loss, <0.3 ms, subidón
```

DNS Movistar `80.58.61.250`, zona `Europe/Madrid`, `NTP active` y `synchronized: yes` en todos (corosync se pone nervioso si la hora baila), internet a `1.1.1.1` en ~8 ms. Y sorpresa agradable: el `vmbr0` ya venía `VLAN aware: Yes` sobre `nic0` con gateway `192.168.1.1`. No tuve que tocar nada, solo verificarlo para el futuro trunk del pfSense.

![Red nucster10 con vmbr0 VLAN-aware](../../nucster_primeras_capturas/nucster10/nucster10_vlan_aware.png)
![Red nucster20 con vmbr0 VLAN-aware](../../nucster_primeras_capturas/nucster20/nucster20_vlan_aware.png)
![Red nucster30 con vmbr0 VLAN-aware y USB enx inactivo](../../nucster_primeras_capturas/nucster30/nucster30_vlan_aware.png)

*Los tres iguales, con las MAC tachadas en rojo (ya voy aprendiendo). En el 30 se ve el `enx00e04c...` del USB en `No`: ahí quieto hasta que monte la WAN.*

Mi plano de VLANs, que ya dejo apuntado: 1 para mgmt, 10 para LAN detrás del pfSense, 20 para LAB donde romper, 99 para WAN al router.

## 5. El clúster: pre-vuelo, la `y` que no vale y el fantasma

Con todo verde hice el pre-vuelo como mandan los buenos: misma versión en los 3, `pvecm status` diciendo standalone (sin `corosync.conf`), `qm list` y `pct list` vacíos para no chocar VMIDs, y un backup `tgz` de 14K por nodo con `/etc/pve`, red, hosts y repos. Dije "adelante cluster" con el corazón a mil:

```bash
# en el 10:
pvecm create Nucster --link0 192.168.1.10  # Quorum 1/1, ¡existe Nucster!
# en el 20: OJO, hay que responder "yes" entero, con "y" aborta y te deja a medias
pvecm add 192.168.1.10 --link0 192.168.1.20  # Quorum 2/2
# en el 30:
pvecm add 192.168.1.10 --link0 192.168.1.30  # Quorum 3/3, config v3
```

El 20 me echó por responder `y` en vez de `yes` (el fingerprint de marras), me cerró la sesión y me entró el pánico porque no me dejaba entrar por SSH. Desde el 10 el ping iba perfecto y el SSH pedía password, o sea que el nodo estaba vivo. Al final entré por consola física y respiré: el abort había sido antes del intercambio de claves, no había roto nada. Segundo intento con `yes` completo y dentro.

![Clúster visto desde nucster10: 3/3 quorate](../../nucster_primeras_capturas/nucster10/nucster10_cluster.png)
![Clúster visto desde nucster20](../../nucster_primeras_capturas/nucster20/nucster20_cluuster.png)
![Clúster visto desde nucster30](../../nucster_primeras_capturas/nucster30/nucster30_cluster.png)

*Los tres diciendo `Nodes: 3, Quorum: 2, Quorate: Yes`. Con 3 votos aguantas que se caiga uno, que para practicar failover es justo lo que quiero.*

Y entonces llegó LA lección del día. La GUI del 10 gritaba `unknown node 'Nucster10' (500)`, decía `Standalone node` viendo 3 nodos a la vez, los iconos con `?` gris y la Shell cruzada daba `401 No ticket`. Me quería morir. Mirando con calma: mi `hostnamectl` a minúsculas había dejado un directorio fantasma `/etc/pve/nodes/Nucster10` con la cert del 11 de octubre, mientras el bueno `nucster10` era del día 13. El `corosync.conf` estaba perfecto en minúsculas, solo sobraba el fósil. Sin rebuild ni dramas:

```bash
ls /etc/pve/nodes  # nucster10  Nucster10  nucster20  nucster30 <- sobra uno
mv /etc/pve/nodes/Nucster10 /root/Nucster10-stale-backup
systemctl restart pveproxy.service pvedaemon.service
# logout/login + hard refresh, y a correr
```

Mano de santo: iconos verdes, Shell 3×3 desde cualquier nodo y `pvecm status` 3/3 quorate clavado. Valiosa lección de ASIR apuntada en rojo: **en Proxmox los nombres siempre en minúsculas desde la instalación, y si cambias el hostname luego, revisa `/etc/pve/nodes`**. La documentación lo dice, los foros lo repiten, y yo lo aprendí a base de susto. Así se aprende de verdad.

## Cierre: qué me llevo hoy

Día cerrado y qué día: sin cartel de pago, los tres en `9.2.21` con kernel `7.0.14-23`, fichas listas, red preparada y clúster `Nucster` sano y quorate. He tocado repos DEB822, parche JS con `void`, fastfetch, hosts, NTP, bridges VLAN-aware, quórum, backups y limpieza de un nodo fantasma. Con la doc oficial y cuatro posts lo saqué adelante, que es exactamente lo que quería demostrarme.

Siguiente capítulo: pfSense en el 30 con el USB de WAN, trunks al GS1200-8 con mis VLANs 1/10/20/99 y el primer grupo HA de juguete con réplica entre los gemelos 10↔20. Si se cae uno y la CT sigue viva en otro, me pongo medalla.
