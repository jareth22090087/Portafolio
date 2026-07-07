# Reporte Técnico: Implementación de Firewall Granular en Linux (iptables)

## 1. Resumen Ejecutivo
En este laboratorio se configuró un firewall a nivel de red (Host-based IDS/IPS) en un entorno Kali Linux utilizando `iptables`. El objetivo fue aplicar el principio de menor privilegio (Zero Trust), bloqueando el tráfico no autorizado, creando listas blancas de acceso y registrando los intentos de conexión sospechosos para su posterior análisis.

**Parámetros de Prueba:**
* **Sistema Operativo:** Kali Linux
* **Herramientas:** iptables, Netcat (nc), journalctl
* **Tácticas MITRE ATT&CK:** Defense Evasion (T1562) / Discovery (T1046)

---

## 2. Implementación Técnica

La configuración se realizó a través de la terminal de Kali Linux, aplicando reglas secuenciales a la cadena `INPUT`.
<img width="1796" height="964" alt="paso1 terminal" src="https://github.com/user-attachments/assets/289041d3-8563-49f3-91b7-b9d86bc02030" />


### Paso 1: Bloqueo de puertos inseguros (Denegación por defecto)
Se estableció una regla para descartar de manera silenciosa (`DROP`) las peticiones entrantes al puerto 80 (HTTP). Esto evita que atacantes externos realicen escaneos de vulnerabilidades web o identifiquen el servicio.

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```
<img width="1489" height="240" alt="Paso 2 Bloquear puetos especificos" src="https://github.com/user-attachments/assets/d0987434-93b0-47b8-b436-3beb1f71d877" />

---
### Paso 2: Creación de Lista Blanca (Whitelist) para administración remota
Se implementó un control de acceso estricto para el puerto 22 (SSH). En lugar de dejar el puerto abierto al público, se autorizó explícitamente (ACCEPT) únicamente a una IP de confianza (192.168.1.50).
```bash
sudo iptables -A INPUT -p tcp -s 192.168.1.50 --dport 22 -j ACCEPT
```
<img width="1805" height="629" alt="Paso3 Permitir el SSH desde una ip concreta" src="https://github.com/user-attachments/assets/20291349-3088-4de8-a829-ba53eee4696b" />

---
### Paso 3: Cierre total del servicio SSH para externos
Siguiendo la lógica secuencial de iptables, inmediatamente después de la lista blanca, se bloqueó el acceso al puerto 22 para cualquier otra IP. Al estar debajo de la regla del Paso 2, la IP autorizada entra, pero el resto de la red es rechazada.
```bash

sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```
<img width="1620" height="596" alt="Paso 4 bloquer al resto del mundo" src="https://github.com/user-attachments/assets/5f1bfe00-6961-49af-95d5-04a9cc57cf05" />

---

### Paso 4: Implementación de Auditoría y Logging (Puerto 21)
Para monitorear posibles intentos de fuerza bruta, se configuró una regla de auditoría en el puerto 21 (FTP). Se utilizó el objetivo no terminal -j LOG con un prefijo personalizado para alertar al sistema sin interrumpir el flujo del paquete.
```bash

sudo iptables -A INPUT -p tcp --dport 21 -j LOG --log-prefix "Alerta- intento FTP: "
```
<img width="1504" height="459" alt="paso 5Crear un sistema de logs" src="https://github.com/user-attachments/assets/a1665c7a-ffa9-45b0-b90b-b58c3462a2ac" />


(Nota técnica: Al ser una regla LOG, el tráfico se registra en la bitácora pero continúa su evaluación en la cadena de reglas).
### Paso 5: Verificación de Reglas
Se validó la tabla de enrutamiento para asegurar el orden correcto de las políticas de filtrado.
```bash

sudo iptables -L -v -n
```
<img width="1412" height="663" alt="paso 6 verificar reglas" src="https://github.com/user-attachments/assets/7374183d-2029-4288-aac0-21e747fe6aaf" />

*Resultado esperado:* El firewall muestra la aceptación exclusiva de la IP permitida en el puerto 22, la denegación del puerto 80, y el monitoreo activo del puerto 21.
---
## 3. Pruebas de Concepto (PoC) y Resolución de Conflictos
Durante la validación, se realizó un ataque simulado desde el propio equipo (localhost) usando Netcat:
```

nc -vz 127.0.0.1 21
```

Problema identificado: Inicialmente, el sistema no registró el ataque en los logs debido a la política de confianza predeterminada de Linux sobre la interfaz de Loopback (lo), la cual se procesa antes que las reglas añadidas con -A (Append).
Resolución (Remediación): Se forzó la prioridad de la regla insertándola explícitamente en la posición número 1 de la libreta del firewall utilizando la bandera -I (Insert).
```

sudo iptables -I INPUT 1 -p tcp --dport 21 -j LOG --log-prefix "Alerta- intento FTP: "
```
<img width="1779" height="605" alt="Resultados" src="https://github.com/user-attachments/assets/827d737f-3780-4a64-8368-dade692888ed" />

Resultado Final: Al elevar la prioridad de la regla, el sistema de bitácoras del kernel (journalctl) comenzó a registrar exitosamente la IP origen, puertos y protocolo del atacante, demostrando la eficacia del monitoreo.
