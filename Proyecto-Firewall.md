# Implementación de Firewall Granular en Linux (iptables)

## Resumen Ejecutivo
En este laboratorio se configuró un firewall a nivel de red (Host-based IDS/IPS) en un entorno Kali Linux utilizando `iptables`. El objetivo fue aplicar el principio de menor privilegio, bloqueando el tráfico no autorizado y registrando los intentos de conexión sospechosos.

## Parámetros de Prueba
* **Sistema Operativo:** Kali Linux
* **Herramientas:** iptables, Netcat (nc), journalctl
* **Táctica MITRE ATT&CK:** Defense Evasion / Discovery

## Implementación Técnica

### 1. Bloqueo de puertos inseguros
Se estableció una regla para descartar de manera silenciosa (DROP) las peticiones al puerto 80 (HTTP) para evitar escaneos de vulnerabilidades:
```bash
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```
<img width="1796" height="964" alt="paso1 terminal" src="https://github.com/user-attachments/assets/6eee94dc-3123-4dc8-a2c1-91ee63f24119" />
