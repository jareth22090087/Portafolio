# Reporte Técnico: [Nombre del Laboratorio o Máquina]

**Fecha:** [DD/MM/AAAA]
**Autor:** [T]
**Plataforma:** [ej. TryHackMe / HackTheBox / Local]
**Dificultad:** [ej. Fácil / Media / Difícil]

---

## 1. Resumen Ejecutivo

*Esta sección es para gerentes o personas no técnicas. Debe ser breve (1-2 párrafos) y enfocarse en el impacto comercial/sistémico, no en los comandos técnicos.*

[Escribe aquí un resumen de alto nivel. ¿Cuál era el objetivo? ¿Se logró comprometer el sistema? ¿Qué datos estaban en riesgo? ¿Cuál es la recomendación principal para solucionar el problema?]

**Puntos Clave:**
*   Se obtuvo acceso de usuario/administrador.
*   [X] vulnerabilidades críticas encontradas.
*   La remediación principal requiere [actualización de software / cambio de configuración].

---

## 2. Metodología

*Demuestra que tienes un proceso ordenado. Menciona los marcos de referencia que utilizaste para guiar tus pruebas.*

Para este laboratorio, se siguió una metodología estructurada basada en:
*   [ ] **OWASP Web Security Testing Guide (WSTG):** Para la identificación de vulnerabilidades en aplicaciones web.
*   [ ] **MITRE ATT&CK Framework:** Para la identificación de tácticas y técnicas de adversarios.
*   [ ] **PTES (Penetration Testing Execution Standard):** Para las fases generales de la prueba.

### Fases de la Prueba:
1.  **Reconocimiento y Enumeración:** Escaneo de puertos y servicios.
2.  **Análisis de Vulnerabilidades:** Identificación de fallos potenciales.
3.  **Explotación:** Ganar acceso inicial.
4.  **Post-Explotación:** Escalada de privilegios y persistencia.

---

## 3. Enumeración y Reconocimiento

*Muestra cómo descubriste el objetivo. Incluye los comandos principales y capturas de pantalla de los resultados.*

### 3.1 Escaneo de Puertos (Nmap)

**Comando ejecutado:**
```bash
nmap -sC -sV -oN nmap_initial.txt [IP_DEL_OBJETIVO]
