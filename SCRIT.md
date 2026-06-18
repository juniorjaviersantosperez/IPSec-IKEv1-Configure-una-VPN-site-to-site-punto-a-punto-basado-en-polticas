# IPSec Site-to-Site con IKEv1 (Policy-Based VPN)

## Topología

### R1 (ISP)
- e0/0: `200.1.15.1/24`
- e0/1: `200.1.99.1/24`

### R2
- WAN: `200.1.15.2/24`
- LAN: `10.15.99.1/24`

### R3
- WAN: `200.1.99.2/24`
- LAN: `192.168.99.1/24`

### Parámetros VPN
- Pre-Shared Key: `1599vpn`
- Cifrado: `AES-256`
- Integridad: `SHA256`
- Grupo DH: `14`
- Modo: `Policy-Based VPN`

---

# Configuración de R1 (ISP)

```bash
enable
configure terminal

hostname R-ISP

interface ethernet 0/0
 ip address 200.1.15.1 255.255.255.0
 no shutdown
exit

interface ethernet 0/1
 ip address 200.1.99.1 255.255.255.0
 no shutdown
exit

ip route 10.15.99.0 255.255.255.0 200.1.15.2
ip route 192.168.99.0 255.255.255.0 200.1.99.2

end
write memory
```

---

# Configuración Base de R2

```bash
enable
configure terminal

hostname R2

interface ethernet 0/1
 ip address 10.15.99.1 255.255.255.0
 no shutdown
exit

interface ethernet 0/0
 ip address 200.1.15.2 255.255.255.0
 no shutdown
exit

ip route 192.168.99.0 255.255.255.0 200.1.15.1
ip route 200.1.99.0 255.255.255.0 200.1.15.1

end
write memory
```

---

# Configuración Base de R3

```bash
enable
configure terminal

hostname R3

interface ethernet 0/1
 ip address 192.168.99.1 255.255.255.0
 no shutdown
exit

interface ethernet 0/0
 ip address 200.1.99.2 255.255.255.0
 no shutdown
exit

ip route 10.15.99.0 255.255.255.0 200.1.99.1
ip route 200.1.15.0 255.255.255.0 200.1.99.1

end
write memory
```

---

# Configuración VPN en R2

```bash
enable
configure terminal

crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400
exit

crypto isakmp key 1599vpn address 200.1.99.2

crypto ipsec transform-set TS-1599 esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

ip access-list extended ACL-VPN-1599
 permit ip 10.15.99.0 0.0.0.255 192.168.99.0 0.0.0.255
exit

crypto map CMAP-1599 10 ipsec-isakmp
 set peer 200.1.99.2
 set transform-set TS-1599
 match address ACL-VPN-1599
exit

interface ethernet 0/0
 crypto map CMAP-1599
exit

end
write memory
```

---

# Configuración VPN en R3

```bash
enable
configure terminal

crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400
exit

crypto isakmp key 1599vpn address 200.1.15.2

crypto ipsec transform-set TS-1599 esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

ip access-list extended ACL-VPN-1599
 permit ip 192.168.99.0 0.0.0.255 10.15.99.0 0.0.0.255
exit

crypto map CMAP-1599 10 ipsec-isakmp
 set peer 200.1.15.2
 set transform-set TS-1599
 match address ACL-VPN-1599
exit

interface ethernet 0/0
 crypto map CMAP-1599
exit

end
write memory
```

---

# Verificación

## Fase 1 (IKEv1)

```bash
show crypto isakmp sa
```

Estado esperado:

```text
QM_IDLE
```

## Fase 2 (IPSec)

```bash
show crypto ipsec sa
```

## Estado de la sesión

```bash
show crypto session
```

## Verificación de rutas

```bash
show ip route
```

## Prueba real de conectividad

```bash
ping 10.15.99.1 source 192.168.99.1
```

---

# Resultado Esperado

```text
ISAKMP SA: QM_IDLE
IPSec SA: ACTIVE
Contadores de paquetes encapsulados y decapsulados incrementando
Comunicación exitosa entre 10.15.99.0/24 y 192.168.99.0/24
```
