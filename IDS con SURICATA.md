# Reporte Técnico: Implementación y Monitoreo de Sistema de Detección de Intrusos (IDS) con Suricata
<img width="649" height="472" alt="image" src="https://github.com/user-attachments/assets/5989b9bf-a63b-4778-a6d1-1d0da76d578b" />

## 1. Resumen Ejecutivo
En este laboratorio se implementó y configuró un Sistema de Detección de Intrusos basado en red (NIDS/HIDS) utilizando Suricata en un entorno de Kali Linux. A diferencia de un firewall transaccional que solo permite o bloquea conexiones, este proyecto se centró en la Inspección Profunda de Paquetes (DPI) para analizar el comportamiento del tráfico. El objetivo principal fue crear firmas personalizadas capaces de detectar e identificar tácticas de reconocimiento de red silenciosas.

Parámetros de Prueba:

* **Sistema Operativo: Kali Linux**

* **Motor IDS: Suricata (v8.0.5)**

* **Herramientas Ofensivas: Nmap, utilidad Ping**

* **Tácticas MITRE ATT&CK: Reconnaissance - Active Scanning (T1595) / Network Service Discovery (T1046)**
---

## 2. Implementación Técnica
La configuración se realizó desde la terminal, priorizando la creación de reglas manuales y el monitoreo en tiempo real.

### Paso 1: Preparación del entorno de red
Para garantizar que el motor IDS pudiera analizar el tráfico sin interferencias, se purgaron las reglas previas del firewall local, permitiendo el flujo libre de paquetes hacia la interfaz de análisis.

```Bash
sudo iptables -F
```


### Paso 2: Creación de Firmas Personalizadas (Ruleset)
Se configuró el archivo local.rules para dictar el comportamiento analítico de Suricata,se entro al archivo usando  sudo nano e indicando la direccion del archivo
```Bash
sudo nano /etc/suricata/rules/local.rules
```

<img width="560" height="64" alt="escribir las reglas de suri" src="https://github.com/user-attachments/assets/c5c5c03e-c79f-4b55-8614-e0602336a08f" />


* Se programaron dos alertas específicas:


Detección de mapeo básico de red mediante el protocolo ICMP.

Detección de escaneos de puertos sigilosos. Se configuró la regla para buscar explícitamente paquetes TCP que contuvieran la bandera SYN (Synchronize) activada, técnica comúnmente utilizada por herramientas como Nmap para evadir logs de conexión estándar.

``` Plaintext
alert icmp any any -> any any (msg:"ALERTA - Trafico ICMP (Ping) Detectado"; sid:1000001; rev:1;)
alert tcp any any -> any any (flags: S; msg:"ALERTA - Posible Escaneo TCP SYN (Nmap)"; sid:1000002; rev:1;)
```
<img width="1258" height="181" alt="reglas de suricata" src="https://github.com/user-attachments/assets/65358468-663a-4897-9037-231e647def4f" />


### Paso 3: Inicialización del Motor de Detección en entorno aislado
Se detuvo el servicio en segundo plano de Suricata para evitar colisiones en los archivos de registro y se forzó su ejecución manual sobre la interfaz de Loopback (lo), indicando la ruta explícita del archivo de firmas personalizado mediante la bandera -S.

``` Bash
sudo systemctl stop suricata
sudo suricata -c /etc/suricata/suricata.yaml -S /etc/suricata/rules/local.rules -i lo
```
<img width="857" height="181" alt="Corremos el suricata con nuestras reglas" src="https://github.com/user-attachments/assets/7ecb70ea-0645-4b8b-b3f3-ad3d81df2723" />


(Nota técnica: Durante el despliegue inicial, el archivo suricata.yaml buscaba las reglas por defecto. Se aplicó el parámetro -S como método de remediación para inyectar directamente las reglas locales creadas en el Paso 2).


### 3. Pruebas de Concepto (PoC) y Centro de Monitoreo
Para validar la eficacia del IDS, se estableció un flujo de trabajo de múltiples terminales, simulando un Centro de Operaciones de Seguridad (SOC) en tiempo real.

Se estableció una terminal de monitoreo leyendo el archivo de salida del IDS:

``` Bash
sudo tail -f /var/log/suricata/fast.log
```
<img width="967" height="602" alt="Lector de archivos de suricata" src="https://github.com/user-attachments/assets/3b95ebb0-e8e5-490d-8c09-9a39ec0fc567" />

Simultáneamente, desde una terminal ofensiva, se lanzaron dos vectores de ataque hacia la propia máquina (127.0.0.1):

Reconocimiento ICMP: ping -c 3 127.0.0.1
<img width="743" height="282" alt="Primer prueba de fuego haciendo un ping normal" src="https://github.com/user-attachments/assets/45f7b0dd-6d58-4ba3-bf7f-39cb7e670d8e" />

Resultados:
<img width="1350" height="187" alt="resultados del ataque" src="https://github.com/user-attachments/assets/283fc30b-0609-426e-98fa-4defc5e82590" />



Escaneo de puertos TCP SYN (Stealth): sudo nmap -sS -p- 127.0.0.1
1.-Simulación de ataque: Escaneo sigiloso TCP SYN (Stealth)
<img width="755" height="197" alt="ataque con nmap" src="https://github.com/user-attachments/assets/b17faaeb-28b3-4e1e-8096-003918d2809d" />

2. Interceptación en tiempo real (Centro de Monitoreo)
<img width="1741" height="606" alt="atqeu y escaneo" src="https://github.com/user-attachments/assets/3ebeabb8-0894-4bef-9eef-046bd8048a62" />

3. Evidencia forense y telemetría de la amenaza
<img width="1469" height="407" alt="resultados se suricata para nmap" src="https://github.com/user-attachments/assets/075745b8-b616-411e-92d2-f93551613e59" />






### 4. Análisis de Telemetría y Resultados
El sistema IDS interceptó y clasificó el ataque con éxito. El análisis detallado de los logs generados (ejemplo a continuación) confirma la capacidad de Suricata para extraer metadatos profundos del tráfico de red:

07/09/2026-20:26:15.549089 [] [1:1000002:1] ALERTA-Posible escaneo TCP SYN (Nmap) [] [Classification: (null)] [Priority: 3] {TCP} 127.0.0.1:36121 -> 127.0.0.1:16827

#### Desglose Forense de la Alerta:

* **07/09/2026-20:26:15.549089 (Marca de Tiempo y Microsegundos)**: Registra la fecha y hora exacta del evento. Los dígitos finales (.549089) son microsegundos, una precisión vital para correlacionar y ordenar cronológicamente ataques automatizados que envían miles de paquetes en un solo segundo.

* **[] (Separadores de Visibilidad)**: Símbolos visuales implementados por el motor para ayudar al analista a identificar rápidamente el inicio y fin del nombre de la alerta dentro de archivos de registros masivos.

* **[1:1000002:1] (Identificador / SID)**: Es la "placa" única de la alerta. Se compone del ID del motor generador (1), el Signature ID o número de firma que se configuró manualmente en las reglas (1000002), y el número de revisión de dicha regla (1).

* **ALERTA-Posible escaneo TCP SYN (Nmap) (Contexto de la Amenaza)**: El mensaje programado que se dispara al encontrar una coincidencia. Indica que el motor inspeccionó el paquete y detectó la bandera SYN (Synchronize) activada, confirmando que el atacante intentó iniciar un saludo de tres vías (Three-way handshake) incompleto para mapear la red de forma sigilosa sin dejar rastros en los logs convencionales.

* **[Classification: (null)] [Priority: 3] (Clasificación y Nivel de Riesgo)**: Muestra la categorización de la amenaza. Al ser una regla personalizada de laboratorio, la clasificación es nula. El motor le asigna por defecto una prioridad nivel 3 (Tráfico anómalo/Información), recordando que en IDS el nivel 1 representa la criticidad máxima.

* **{TCP} (Protocolo de Transporte)**: Confirma el tipo de protocolo que el atacante utilizó para encapsular el paquete y enviarlo por la red.

* **127.0.0.1:36121 -> 127.0.0.1:16827 (Vector de Ataque: Origen y Destino)**: Es la evidencia direccional del crimen. Demuestra que la IP atacante (127.0.0.1) abrió un puerto aleatorio efímero (:36121) para enviar el paquete malicioso (->) hacia la máquina víctima (127.0.0.1), impactando específicamente en el puerto de destino (:16827). Durante el análisis completo, se observó la iteración frenética de este último puerto, característico del barrido total de la herramienta Nmap.

Conclusión:
La implementación demostró exitosamente la diferencia crítica entre un Firewall y un IDS. Mientras que un firewall tradicional habría descartado silenciosamente o permitido estos paquetes sin generar un contexto sobre el ataque, Suricata logró perfilar el comportamiento del adversario, proporcionando inteligencia procesable para el analista de seguridad.


