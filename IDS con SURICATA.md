Reporte Técnico: Implementación y Monitoreo de Sistema de Detección de Intrusos (IDS) con Suricata
1. Resumen Ejecutivo
En este laboratorio se implementó y configuró un Sistema de Detección de Intrusos basado en red (NIDS/HIDS) utilizando Suricata en un entorno de Kali Linux. A diferencia de un firewall transaccional que solo permite o bloquea conexiones, este proyecto se centró en la Inspección Profunda de Paquetes (DPI) para analizar el comportamiento del tráfico. El objetivo principal fue crear firmas personalizadas capaces de detectar e identificar tácticas de reconocimiento de red silenciosas.

Parámetros de Prueba:

Sistema Operativo: Kali Linux

Motor IDS: Suricata (v8.0.5)

Herramientas Ofensivas: Nmap, utilidad Ping

Tácticas MITRE ATT&CK: Reconnaissance - Active Scanning (T1595) / Network Service Discovery (T1046)

2. Implementación Técnica
La configuración se realizó desde la terminal, priorizando la creación de reglas manuales y el monitoreo en tiempo real.

Paso 1: Preparación del entorno de red
Para garantizar que el motor IDS pudiera analizar el tráfico sin interferencias, se purgaron las reglas previas del firewall local, permitiendo el flujo libre de paquetes hacia la interfaz de análisis.

Bash
sudo iptables -F
[INSERTA TU IMAGEN AQUI - Captura limpiando iptables e instalando Suricata]

Paso 2: Creación de Firmas Personalizadas (Ruleset)
Se configuró el archivo local.rules para dictar el comportamiento analítico de Suricata. Se programaron dos alertas específicas:

Detección de mapeo básico de red mediante el protocolo ICMP.

Detección de escaneos de puertos sigilosos. Se configuró la regla para buscar explícitamente paquetes TCP que contuvieran la bandera SYN (Synchronize) activada, técnica comúnmente utilizada por herramientas como Nmap para evadir logs de conexión estándar.

Plaintext
alert icmp any any -> any any (msg:"ALERTA - Trafico ICMP (Ping) Detectado"; sid:1000001; rev:1;)
alert tcp any any -> any any (flags: S; msg:"ALERTA - Posible Escaneo TCP SYN (Nmap)"; sid:1000002; rev:1;)
[INSERTA TU IMAGEN AQUI - Captura del editor nano con las reglas]

Paso 3: Inicialización del Motor de Detección en entorno aislado
Se detuvo el servicio en segundo plano de Suricata para evitar colisiones en los archivos de registro y se forzó su ejecución manual sobre la interfaz de Loopback (lo), indicando la ruta explícita del archivo de firmas personalizado mediante la bandera -S.

Bash
sudo systemctl stop suricata
sudo suricata -c /etc/suricata/suricata.yaml -S /etc/suricata/rules/local.rules -i lo
[INSERTA TU IMAGEN AQUI - Captura de la Terminal 1 con el mensaje "Engine started"]

(Nota técnica: Durante el despliegue inicial, el archivo suricata.yaml buscaba las reglas por defecto. Se aplicó el parámetro -S como método de remediación para inyectar directamente las reglas locales creadas en el Paso 2).

3. Pruebas de Concepto (PoC) y Centro de Monitoreo
Para validar la eficacia del IDS, se estableció un flujo de trabajo de múltiples terminales, simulando un Centro de Operaciones de Seguridad (SOC) en tiempo real.

Se estableció una terminal de monitoreo leyendo el archivo de salida del IDS:

Bash
sudo tail -f /var/log/suricata/fast.log
Simultáneamente, desde una terminal ofensiva, se lanzaron dos vectores de ataque hacia la propia máquina (127.0.0.1):

Reconocimiento ICMP: ping -c 3 127.0.0.1

Escaneo de puertos TCP SYN (Stealth): sudo nmap -sS -p- 127.0.0.1

[INSERTA TU IMAGEN AQUI - Captura mostrando Nmap corriendo en una ventana, y las alertas rojas en la otra ventana]

4. Análisis de Telemetría y Resultados
El sistema IDS interceptó y clasificó el ataque con éxito. El análisis detallado de los logs generados (ejemplo a continuación) confirma la capacidad de Suricata para extraer metadatos profundos del tráfico de red:

07/09/2026-20:26:15.549089 [] [1:1000002:1] ALERTA-Posible escaneo TCP SYN (Nmap) [] [Classification: (null)] [Priority: 3] {TCP} 127.0.0.1:36121 -> 127.0.0.1:16827

Desglose Forense de la Alerta:

Precisión Temporal: El evento fue registrado con una granularidad de microsegundos (.549089), vital para correlacionar ataques automatizados que envían miles de paquetes por segundo.

Identificación (SID): La firma 1000002 fue disparada al detectar el patrón específico del paquete.

Inspección de Banderas TCP: El sistema no solo detectó el intento de conexión, sino que identificó que se trataba de un saludo inicial asíncrono (SYN), confirmando la naturaleza de "escaneo silencioso" en lugar de tráfico web normal.

Rastreo de Origen y Destino: El log documenta que el atacante (IP de origen) utilizó un puerto aleatorio efímero (:36121) para golpear repetidamente puertos de destino secuenciales (ej. :16827, :11950) en la máquina víctima, característico del barrido total de Nmap (-p-).

Conclusión:
La implementación demostró exitosamente la diferencia crítica entre un Firewall y un IDS. Mientras que un firewall tradicional habría descartado silenciosamente o permitido estos paquetes sin generar un contexto sobre el ataque, Suricata logró perfilar el comportamiento del adversario, proporcionando inteligencia procesable para el analista de seguridad.

¡Listo! Léelo con calma. Tiene el mismo nivel técnico que el de iptables, pero la conclusión hace que ambos proyectos se conecten perfectamente en tu perfil. Avísame si quieres cambiarle algo o si ya lo subes directo a GitHub.
