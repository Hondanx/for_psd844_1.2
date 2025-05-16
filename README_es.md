
# Configuración de Firewall y NAT en Router Ubuntu (R1)

## 📌 Descripción General

Este proyecto implementa un firewall seguro utilizando `iptables` en un router basado en Linux (R1). Utiliza **filtrado de paquetes**, **inspección con estado** y **NAT (DNAT)** para controlar el tráfico ICMP y TCP en una red simulada con múltiples subredes.

## 🧱 Resumen de la Topología

- **R1** – Actúa como router principal y firewall  
- **R2 y R3** – Routers internos conectados a los servidores FTP (172.31.0.1) y Telnet (172.16.0.1)  
- **Cliente (LAN)** – Conectado a través de `192.168.56.1`, enrutado mediante `192.168.56.254` (R1)

📁 Instrucciones completas para la configuración del laboratorio disponibles en:  
➡️ https://github.com/Hondanx/for_psd844

## 🔧 Aspectos Destacados de la Configuración del Firewall

### ✅ Configuración Base
```bash
# Tráfico de loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Permitir sesiones establecidas
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Establecer políticas predeterminadas
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

### 🚫 Filtrado ICMP

| Acción | Origen | Destino | Resultado |
|--------|--------|---------|-----------|
| Ping al firewall | Cualquiera | R1 | **Bloqueado** |
| LAN → Servidor FTP | 192.168.56.0/24 | 172.31.0.1 | **Rechazado** |
| LAN → Servidor Telnet | 192.168.56.0/24 | 172.16.0.1 | **Permitido** |

```bash
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
iptables -A FORWARD -s 192.168.56.0/24 -d 172.31.0.1 -p icmp --icmp-type echo-request -j REJECT
iptables -A FORWARD -d 172.16.0.1 -p icmp --icmp-type echo-request -j ACCEPT
```

### 📡 Reglas de Servicio

```bash
# FTP
iptables -A FORWARD -p tcp -d 172.31.0.1 --dport 21 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

# Telnet
iptables -A FORWARD -p tcp -d 172.16.0.1 --dport 23 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```

### 🔁 Redirección NAT

```bash
# Control y datos FTP
iptables -t nat -A PREROUTING -s 192.168.56.0/24 -p tcp --dport 21 -j DNAT --to-destination 172.31.0.1:21
iptables -t nat -A PREROUTING -s 192.168.56.0/24 -p tcp --dport 20 -j DNAT --to-destination 172.31.0.1:20

# Telnet
iptables -t nat -A PREROUTING -p tcp --dport 23 -j DNAT --to-destination 172.16.0.1:23
```

## 🧪 Pruebas y Verificación

### ✅ Cliente de Prueba: `192.168.56.1`

1. **Ping antes de las reglas** → éxito  
2. **Ping después de las reglas** → servidor FTP inaccesible (RECHAZADO)  
3. **Telnet a R1** (23) → redirigido correctamente al servidor interno  
4. **iptables -L -v -n** confirma contadores de paquetes permitidos/rechazados

📸 Capturas de pantalla y video incluidos en `/test_results`.

## 📂 Estructura de Archivos

```
.
├── firewall_rules.sh      # comandos iptables
├── test_results/
│   ├── ping_before.png
│   ├── ping_after.png
│   └── telnet_test.mp4
├── Firewall Implementation and TCP_IP Network Monitoring.pptx
├── README.md
```

## 👨‍💻 Autor

Creado por [HONDANX]
