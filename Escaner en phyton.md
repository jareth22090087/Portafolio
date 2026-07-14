# Reporte Técnico: Desarrollo e Iteración de Escáner de Puertos y Vulnerabilidades en Python

## 1. Resumen Ejecutivo
En este laboratorio se diseñó y programó progresivamente una herramienta de seguridad ofensiva utilizando Python en un entorno Kali Linux. El desarrollo se realizó de forma iterativa: se comenzó con un script básico para sondear un único puerto, se escaló a un escáner masivo de rangos de puertos, y finalmente se transformó en un **Escáner de Vulnerabilidades** mediante técnicas de **Banner Grabbing** y cruce de datos con firmas locales (CVEs).

El objetivo fue comprender la automatización del reconocimiento de red (Reconnaissance) y el perfilamiento de servicios desactualizados.

**Parámetros del Entorno de Prueba:**
* **Sistema Operativo:** Kali Linux
* **Lenguaje:** Python 3 (v3.13.14)
* **Librerías Core:** `socket`, `sys`
* **Tácticas MITRE ATT&CK:** Active Scanning (T1595) / Gather Victim Host Information (T1592)

---

## 2. Desarrollo e Implementación Técnica
La construcción de la herramienta se dividió en fases lógicas, mejorando el código fuente en cada iteración para añadir capacidades ofensivas más complejas.

### Fase 1: Escáner Básico de un Solo Puerto
Se creó el script principal mediante consola, verificando la versión del intérprete de Python.
<img width="1904" height="891" alt="creamos un documneto de py" src="https://github.com/user-attachments/assets/877d8fc1-d192-41f0-b4c9-6ad23a6749e4" />


En esta primera versión, la lógica se limitó a importar la librería `socket` y apuntar a un único puerto estático (puerto 22). Se utilizó la función `connect_ex` con un condicional simple para determinar si el puerto estaba abierto (0) o cerrado.
<img width="816" height="460" alt="codigo documentado parte 1" src="https://github.com/user-attachments/assets/211521bd-4a2c-46b9-a383-104d923a1a33" />


Al ejecutar esta primera versión contra la máquina local (127.0.0.1), el programa validó correctamente que el puerto 22 no estaba en uso.


<img width="686" height="168" alt="Primer resultado" src="https://github.com/user-attachments/assets/54cb98a9-d03c-451a-927a-fb2bbcaeaab5" />




### Fase 2: Escáner Automatizado (Programa Mejorado)
Una vez validada la conexión unitaria, se refactorizó el código. Se implementó un bucle `for puerto in range(1, 1025):` para automatizar el escaneo de los puertos bien conocidos. Se ajustó el *timeout* a 0.5 segundos dentro del bucle para acelerar el proceso y se añadió el manejo de excepciones (`KeyboardInterrupt` y `socket.error`).
<img width="1896" height="761" alt="Programa mejorado" src="https://github.com/user-attachments/assets/f38ca349-5f77-42cc-a13f-8682cf036a2a" />


Al ejecutar esta versión de barrido, el escáner recorrió silenciosa y rápidamente el rango especificado sin colgarse, finalizando el proceso limpiamente al no encontrar puertos abiertos de interés.
<img width="903" height="246" alt="Resultados puertos cerrados" src="https://github.com/user-attachments/assets/c80ace30-d3a1-4a90-9782-428c4ae56fdc" />
Para comprobar que la detección funcionaba cuando un servicio estaba activo, se utilizó Netcat para levantar un *listener* en el puerto 22, confirmando la viabilidad de la librería `socket` para el escaneo.
<img width="657" height="265" alt="Sabrimso un puerto para probar el script" src="https://github.com/user-attachments/assets/f60ed760-8116-437d-81ab-c4880d00c993" />

resultados
<img width="987" height="301" alt="evidencias" src="https://github.com/user-attachments/assets/5bd41cf4-a592-43f4-b250-7ea682871cc6" />



### Fase 3: Implementación de Banner Grabbing (Inspección Capa 7)
Se evolucionó el script de un simple escáner de red a un inspector de aplicaciones. Se insertó un bloque `try-except` que,
al detectar un puerto abierto, inyecta un payload HTTP genérico (`b"GET / HTTP/1.0\r\n\r\n"`). La respuesta del servicio se captura (hasta 1024 bytes) y se decodifica,
permitiendo extraer la versión exacta del demonio o servidor web subyacente.

<img width="831" height="526" alt="Bannergrabbing" src="https://github.com/user-attachments/assets/2949c827-6b18-475c-9c5b-a8407c419207" />

  Resultados

  <img width="1298" height="267" alt="resultados de bannergrabbing" src="https://github.com/user-attachments/assets/da8fb761-47a9-467f-9dee-b9498ef01eb9" />


### Fase 4: Motor de Detección de Vulnerabilidades (Integración CVE)
Se integró un diccionario local (`vulnerabilidades_conocidas`) que actúa como base de datos de inteligencia de amenazas, relacionando cadenas de texto (firmas) con identificadores CVE (Common Vulnerabilities and Exposures).
<img width="995" height="429" alt="Mejora para revisar si este contiene alguna CVES" src="https://github.com/user-attachments/assets/be5d4491-a4d8-4d0c-918d-b718f2cf355a" />


Se creó la función `buscar_cves(banner)` para cruzar la información obtenida en la Fase 3 con esta base de datos, 
automatizando la emisión de alertas críticas en caso de coincidencia.
<img width="972" height="698" alt="continuacion de la mejora y donde se ejecuta en el script" src="https://github.com/user-attachments/assets/83333dc6-6ea2-4fca-8f89-aa4e98cd9c47" />


---

## 3. Prueba de Concepto (PoC) Final
Para la validación integral del Escáner de Vulnerabilidades terminado, se simuló un objetivo vulnerable en el entorno local levantando un servidor web nativo de Python en el puerto 80.

<img width="568" height="113" alt="Levantamos un servidor para probar la mejora" src="https://github.com/user-attachments/assets/9a82887c-2f11-4b79-9546-281434edf7a3" />


Al lanzar la herramienta contra este objetivo, se documentó la siguiente cadena de ataque automatizada:
1. Identificación del puerto 80 abierto en el barrido.
2. Extracción exitosa del Banner (`Server: SimpleHTTP/0.6 Python/3.13.14`).
3. Correlación algorítmica y disparo de la **ALERTA CRÍTICA** (CVE-2026-9999).
<img width="1455" height="220" alt="resultados de prueba" src="https://github.com/user-attachments/assets/51606bc0-f284-45c9-b2bd-bcc27b5a3460" />


---

## 4. Conclusión Técnica
La evolución progresiva de este script demuestra el ciclo de vida del desarrollo de herramientas ofensivas (Tooling). 
* **Optimización:** Pasar de un escaneo secuencial lento a un bucle con *timeouts* gestionados previno bloqueos por latencia de red.
* **Precisión:** La agregación de Banner Grabbing transformó un simple "descubrimiento de estado" (abierto/cerrado) en una verdadera vectorización de amenazas, permitiendo automatizar el descubrimiento de vectores de ataque iniciales tal como lo operan escáneres comerciales avanzados.
