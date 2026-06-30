# 🔐 VPN Cisco IPSec IKEv1 Site-to-Site — Basada en Políticas

<div align="center">

![Cisco](https://img.shields.io/badge/Cisco-IOS-blue?style=for-the-badge&logo=cisco)
![IPSec](https://img.shields.io/badge/IPSec-IKEv1-green?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-PNETLab-orange?style=for-the-badge)
![License](https://img.shields.io/badge/Uso-Educativo-red?style=for-the-badge)

**Sael Germán García** | Matrícula: `2025-0725`  
Asignatura: Seguridad de Redes | Profesor: Jonathan Rondón  
Instituto Tecnológico de las Américas — ITLA | 2026

</div>

---

## 📋 Descripción

Implementación de una **VPN Site-to-Site** entre dos routers Cisco utilizando **IPSec con IKEv1** en modo basado en políticas. La VPN permite que la **LAN-A** y la **LAN-B** se comuniquen de forma segura a través de un router ISP intermedio, simulando una conexión cifrada entre dos sedes remotas.

---

## 🗺️ Topología de Red

### 📊 Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|:-----------:|:--------:|:------------:|:-------:|-------------|
| R1 | Ethernet0/0 | 10.7.25.1 | /30 | WAN hacia ISP |
| R1 | Ethernet0/1 | 10.7.25.65 | /27 | Gateway LAN-A |
| ISP | Ethernet0/0 | 10.7.25.2 | /30 | Enlace hacia R1 |
| ISP | Ethernet0/1 | 10.7.25.6 | /30 | Enlace hacia R2 |
| R2 | Ethernet0/0 | 10.7.25.5 | /30 | WAN hacia ISP |
| R2 | Ethernet0/1 | 10.7.25.97 | /27 | Gateway LAN-B |
| VPC1 | eth0 | 10.7.25.66 | /27 | Cliente LAN-A |
| VPC2 | eth0 | 10.7.25.98 | /27 | Cliente LAN-B |

---

## ⚙️ Parámetros de la VPN

| Parámetro | Valor |
|:---------:|-------|
| Tipo de VPN | Site-to-Site punto a punto |
| Modo | Basada en políticas (crypto map + ACL) |
| Versión IKE | IKEv1 |
| Autenticación | Pre-Shared Key |
| Clave compartida | VPN12345 |
| Cifrado Fase 1 | AES 256 |
| Hash Fase 1 | SHA |
| Grupo Diffie-Hellman | Grupo 5 |
| Transform-set IPSec | esp-aes 256 esp-sha-hmac |
| Modo IPSec | Tunnel |
| PFS | Grupo 5 |
| ACL tráfico R1 | 10.7.25.64/27 → 10.7.25.96/27 |
| ACL tráfico R2 | 10.7.25.96/27 → 10.7.25.64/27 |

---

## 🚀 Scripts de Configuración

### ISP
```cisco
hostname ISP
interface Ethernet0/0
 description ENLACE_A_R1
 ip address 10.7.25.2 255.255.255.252
 no shutdown
interface Ethernet0/1
 description ENLACE_A_R2
 ip address 10.7.25.6 255.255.255.252
 no shutdown
```

### R1 — Configuración base
```cisco
hostname R1
interface Ethernet0/0
 description WAN_HACIA_ISP
 ip address 10.7.25.1 255.255.255.252
 no shutdown
interface Ethernet0/1
 description LAN_A
 ip address 10.7.25.65 255.255.255.224
 no shutdown
ip route 0.0.0.0 0.0.0.0 10.7.25.2
```

### R1 — IPSec IKEv1
```cisco
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key VPN12345 address 10.7.25.5

ip access-list extended VPN-TRAFFIC
 permit ip 10.7.25.64 0.0.0.31 10.7.25.96 0.0.0.31

crypto ipsec transform-set TS-IKEV1 esp-aes 256 esp-sha-hmac
 mode tunnel

crypto map VPN-MAP 10 ipsec-isakmp
 description VPN_IKEV1_HACIA_R2
 set peer 10.7.25.5
 set transform-set TS-IKEV1
 set pfs group5
 match address VPN-TRAFFIC

interface Ethernet0/0
 crypto map VPN-MAP
```

### R2 — Configuración base
```cisco
hostname R2
interface Ethernet0/0
 description WAN_HACIA_ISP
 ip address 10.7.25.5 255.255.255.252
 no shutdown
interface Ethernet0/1
 description LAN_B
 ip address 10.7.25.97 255.255.255.224
 no shutdown
ip route 0.0.0.0 0.0.0.0 10.7.25.6
```

### R2 — IPSec IKEv1
```cisco
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key VPN12345 address 10.7.25.1

ip access-list extended VPN-TRAFFIC
 permit ip 10.7.25.96 0.0.0.31 10.7.25.64 0.0.0.31

crypto ipsec transform-set TS-IKEV1 esp-aes 256 esp-sha-hmac
 mode tunnel

crypto map VPN-MAP 10 ipsec-isakmp
 description VPN_IKEV1_HACIA_R1
 set peer 10.7.25.1
 set transform-set TS-IKEV1
 set pfs group5
 match address VPN-TRAFFIC

interface Ethernet0/0
 crypto map VPN-MAP
```

---

## ✅ Verificación del Túnel

```cisco
show crypto isakmp sa
show crypto ipsec sa
show crypto session
```

| Comando | Estado esperado |
|:-------:|----------------|
| `show crypto isakmp sa` | QM_IDLE / ACTIVE |
| `show crypto ipsec sa` | pkts encaps y decaps activos |
| `show crypto session` | UP-ACTIVE |

---

## 📊 Resultados

| Prueba | Resultado |
|:------:|:---------:|
| Ping VPC1 → VPC2 | ✅ Exitoso |
| Ping VPC2 → VPC1 | ✅ Exitoso |
| IKEv1 SA | ✅ QM_IDLE / ACTIVE |
| IPSec SA | ✅ ACTIVE con encaps/decaps |
| Crypto session | ✅ UP-ACTIVE |

---

## 🖼️ Capturas de Pantalla

- 📸 [Figura 1 — Topología en PNETLab](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%201.%20Topolog%C3%ADa%20implementada%20en%20PNETLab%20para%20la%20VPN%20Cisco%20IPSec%20IKEv1%20Site-to-Site%20basada%20en%20pol%C3%ADtic....png)
- 📸 [Figura 2 — Verificación de interfaces en R1](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%202.%20Verificaci%C3%B3n%20de%20interfaces%20en%20R1%20mediante%20el%20comando%20show%20ip%20interface%20brief.png)
- 📸 [Figura 3 — Verificación de interfaces en R2](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%203.%20Verificaci%C3%B3n%20de%20interfaces%20en%20R2%20mediante%20el%20comando%20show%20ip%20interface%20brief.png)
- 📸 [Figura 4 — Dirección IP en VPC1](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%204.%20Direcci%C3%B3n%20IP%20configurada%20en%20VPC1.png)
- 📸 [Figura 5 — Dirección IP en VPC2](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%205.%20Direcci%C3%B3n%20IP%20configurada%20en%20VPC2.png)
- 📸 [Figura 6 — Parámetros criptográficos en R1](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%206.%20Par%C3%A1metros%20criptogr%C3%A1ficos%20configurados%20en%20R1%20para%20IKEv1%20e%20IPSec.png)
- 📸 [Figura 7 — ACL VPN-TRAFFIC en R1](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%207.%20Lista%20de%20acceso%20VPN-TRAFFIC%20en%20R1%20con%20coincidencias%20de%20tr%C3%A1fico%20interesante.png)
- 📸 [Figura 8 — Parámetros criptográficos en R2](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%208.%20Par%C3%A1metros%20criptogr%C3%A1ficos%20configurados%20en%20R2%20para%20IKEv1%20e%20IPSec.png)
- 📸 [Figura 9 — ACL VPN-TRAFFIC en R2](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%209.%20Lista%20de%20acceso%20VPN-TRAFFIC%20en%20R2%20con%20coincidencias%20de%20tr%C3%A1fico%20interesante.png)
- 📸 [Figura 10 — Ping VPC1 → VPC2 y traceroute](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%2010.%20Prueba%20de%20conectividad%20desde%20VPC1%20hacia%20VPC2%20y%20traceroute.png)
- 📸 [Figura 11 — Ping VPC2 → VPC1 y traceroute](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%2011.%20Prueba%20de%20conectividad%20desde%20VPC2%20hacia%20VPC1%20y%20traceroute....png)
- 📸 [Figura 12 — IKEv1 en R1 QM_IDLE](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%2012.%20Estado%20de%20IKEv1%20en%20R1%20mostrando%20QM_IDLE%20y%20sesi%C3%B3n%20activa.png)
- 📸 [Figura 13 — IPSec SA en R1 encaps/decaps](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%2013.%20Verificaci%C3%B3n%20de%20IPSec%20SA%20en%20R1%20con%20contadores%20de%20encapsulaci....png)
- 📸 [Figura 14 — IPSec SA R1 inbound/outbound](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%2014.%20Continuaci%C3%B3n%20de%20la%20verificaci%C3%B3n%20IPSec%20SA%20en%20R1%20con%20SAs%20inbou....png)
- 📸 [Figura 17 — IPSec SA en R2 encaps/decaps](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%2017.%20Verificaci%C3%B3n%20de%20IPSec%20SA%20en%20R2%20con%20contadores%20de%20encapsulaci%C3%B3n%20y%20decapsulaci%C3%B3n.png)
- 📸 [Figura 18 — IPSec SA R2 inbound/outbound](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%2018.%20Continuaci%C3%B3n%20de%20la%20verificaci%C3%B3n%20IPSec%20SA%20en%20R2%20con%20SAs%20inbou....png)
- 📸 [Figura 19 — Sesión criptográfica R2 UP-ACTIVE](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas/Figura%2019.%20Sesi%C3%B3n%20criptogr%C3%A1fica%20en%20R2%20en%20estado%20UP-ACTIVE.png)

---

## 📁 Archivos del Repositorio

| Archivo | Descripción |
|:-------:|-------------|
| [`Script de configuracion-VPN site to site punto...`](VPN-Cisco-IPSec-IKEv1-Site-to-Site-basada-en-politicas-capturas) | Scripts de configuración Cisco IOS |
| [`SaelGerman_2025-0725_VPN-IPSec IKEv1_Site-to-Site _Basada en Políticas_P2.pdf`](SaelGerman_2025-0725_VPN-IPSec%20IKEv1_Site-to-Site%20_Basada%20en%20Pol%C3%ADticas_P2.pdf) | Documentación técnica completa |

## 📚 Referencias

1. Cisco Systems. *Cisco IOS Security Configuration Guide: Securing the Data Plane*. Documentación oficial Cisco IOS.
2. Reconocimiento especial: Troubleshooting, base del script y documentación apoyado en Inteligencia Artificial.

---

<div align="center">

*Este laboratorio fue desarrollado exclusivamente con fines académicos y educativos.*

</div>
