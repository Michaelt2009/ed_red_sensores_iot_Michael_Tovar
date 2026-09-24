# Bitacora individual - Semana 03

> Bitácora individual de la Semana 3: Búsqueda lineal, búsqueda binaria y eficiencia algorítmica.

## 1. Datos de la actividad

- **Estudiante:** Michael David Tovar Parada
- **Equipo:** Equipo Sensores IoT
- **Semana:** 3
- **Fecha del laboratorio:** 2026-09-23
- **Fecha del taller:** 2026-09-23
- **Tema principal:** Búsqueda lineal, búsqueda binaria, análisis de complejidad $O(n)$ vs $O(\log n)$, precondiciones y manejo de ramas en Git.
- **Pregunta de la semana:** ¿Cómo encontramos una lectura específica cuando el repositorio pasa de cientos a cientos de miles o millones de registros sin degradar el rendimiento del sistema?

## 2. Prediccion antes de ejecutar

1. **Que creo que va a ocurrir?**
   Que la búsqueda lineal tardará significativamente más tiempo y realizará exactamente $n$ comparaciones en el peor caso, mientras que la búsqueda binaria encontrará el elemento en unas pocas decenas de comparaciones debido a la reducción logarítmica.

2. **Que parte del programa o del algoritmo puede fallar?**
   La búsqueda binaria sobre arreglos no ordenados (experimento de PM2.5), ya que al violar su precondición descartará mitades donde sí está el dato, devolviendo `-1`. También la actualización de punteros (`inicio = medio + 1`) si no se maneja bien, produciendo un bucle infinito.

3. **Como comprobare mi prediccion?**
   Comparando el contador de comparaciones de `BuscadorLecturas` en el banco de pruebas y contrastando los aciertos de lineal frente a binaria en el Experimento 4.

## 3. Evidencia del laboratorio

### Resultado observado

Al ejecutar `IngestaSensores`, se obtuvieron las siguientes métricas experimentales:

- **Experimento 1 y 2 (Peor caso - Último elemento):**
    - $1.000$ lecturas: Lineal = $1.000$ comparaciones | Binaria = $10$ comparaciones. Relación: $\approx 100\times$.
    - $100.000$ lecturas: Lineal = $100.000$ comparaciones | Binaria = $17$ comparaciones. Relación: $\approx 5.882\times$.
    - $1.000.000$ lecturas: Lineal = $1.000.000$ comparaciones | Binaria = $20$ comparaciones. Relación: $\approx 50.000\times$.

- **Experimento 3 (Dato inexistente):**
    - Lineal: $100.000$ comparaciones (recorre todo el arreglo para comprobar que no está).
    - Binaria: $17$ comparaciones (concluye que no existe cuando `inicio > fin`).

- **Experimento 4 (Fallo de precondición en PM2.5):**
    - Valores buscados existentes: 20
    - Encontrados por búsqueda lineal: 20
    - Encontrados por búsqueda binaria: 0 (o significativamente menor por falta de ordenamiento).

### Diferencia entre la prediccion y el resultado

La predicción coincidió exactamente con la teoría: la búsqueda binaria reduce drásticamente las comparaciones, pero demostró que un algoritmo lógicamente correcto produce resultados erróneos si los datos no cumplen la precondición de orden.

### Error o comportamiento inesperado

- **Que ocurrio?** Riesgo de ciclo infinito en búsqueda binaria si se asigna `inicio = medio`.
- **Por que ocurrio?** Al calcular divisiones enteras, `(inicio + fin) / 2` puede retornar el mismo valor de `inicio`, estancando el ciclo si no se descarta el elemento evaluado.
- **Como lo corregimos o que falta corregir?** Se ajustaron los límites estrictamente a `inicio = medio + 1` y `fin = medio - 1`.

## 4. Explicacion en lenguaje llano

Buscar de forma lineal es como buscar una palabra en un diccionario hojeando página por página desde la A hasta la Z. La búsqueda binaria es abrir el diccionario por la mitad exacta: si la palabra que buscas va después, arrancas y botas toda la primera mitad; si va antes, descartas la segunda. En cada intento botas la mitad de lo que te queda, encontrando la palabra en segundos.

### Ejemplo o analogia

Adivinar un número del 1 al 100 donde te dicen "más alto" o "más bajo". La peor forma es preguntar 1, 2, 3... (lineal). La forma óptima es decir 50, luego 75 o 25, descartando mitades completas (binaria). La analogía deja de ser exacta si los números estuvieran revueltos en papeles dentro de una bolsa, pues no podrías saber hacia qué lado descartar.

## 5. El vacio que encontre

- **Mi duda concreta es:** ¿Por qué la búsqueda binaria no puede aplicarse directamente a variables continuas como PM2.5 en una red IoT en tiempo real?
- **Lo que ya puedo explicar es:** Que la búsqueda binaria exige precondición de orden estricto, y las lecturas de los sensores llegan ordenadas por tiempo (cronológicamente), no por concentración de contaminante.
- **Para resolver la duda consulte:** El experimento 4 de la guía de la Semana 3 y análisis de precondiciones.
- **Ahora lo entiendo asi:** Mantener ordenado un arreglo por múltiples atributos al tiempo es costoso; si quiero buscar rápido por PM2.5 primero debo pagar el costo de ordenar o indexar.

## 6. Trazado de la solucion

Traza de búsqueda binaria sobre arreglo ordenado `[0, 1, 2, 3]` buscando el valor `3`:

| Paso | inicio | fin | medio | valor medio | comparacion (medio vs obj) | acción tomada |
|---|---:|---:|---:|---:|---|---|
| 1 | 0 | 3 | 1 | 1 | $1 < 3$ | $inicio = medio + 1 \rightarrow inicio = 2$ |
| 2 | 2 | 3 | 2 | 2 | $2 < 3$ | $inicio = medio + 1 \rightarrow inicio = 3$ |
| 3 | 3 | 3 | 3 | 3 | $3 == 3$ | Elemento encontrado en índice 3. Retorna 3. |

## 7. Decision de diseño

- **Problema que debiamos resolver:** Permitir consultas rápidas de lecturas en colecciones de hasta 1.000.000 de registros.
- **Estructura, algoritmo o estrategia elegida:** Búsqueda binaria sobre arreglos generados cronológicamente por `timestamp`.
- **Alternativa descartada:** Búsqueda lineal continua para todas las consultas.
- **Por que elegimos la primera:** Para $1.000.000$ de lecturas, la búsqueda lineal requiere hasta $1.000.000$ de comparaciones frente a solo $20$ de la binaria, optimizando la CPU.
- **Que evidencia respalda la decision:** La tabla del Experimento 2 muestra una mejora de rendimiento de más de $50.000$ veces en cantidad de operaciones.

## 8. Aporte al proyecto

- **Archivo(s) o modulo(s) trabajado(s):** `src/BuscadorLecturas.java`, `src/GeneradorDatos.java`, `src/BancoDePruebas.java`, `src/IngestaSensores.java`, `docs/decisiones.md`.
- **Cambio realizado:** Implementación de algoritmos de búsqueda ($O(n)$ y $O(\log n)$), generador sintético a escala, integración en el único `main` y documentación de precondiciones.
- **Como se conecta con la capa anterior:** Utiliza la clase `LecturaSensor` y complementa el almacenamiento de `RepositorioLecturas` con capacidades analíticas de búsqueda.
- **Que queda pendiente para la siguiente semana:** Semana 4: Algoritmos de ordenamiento para ordenar datos bajo criterios como PM2.5 o temperatura.

## 9. Commits realizados

| Commit | Mensaje | Que demuestra |
|---|---|---|
| `(hash de rama)` | `feat: implementa busquedas, experimentos semana 3 y documentacion` | Creación de módulos de búsqueda, banco de pruebas y bitácora en la rama de funcionalidad. |
| `(hash de merge)` | `Merge branch 'feature/semana-3-busqueda'` | Integración limpia de la línea de desarrollo a la línea base estable `main`. |

## 10. Reexplicacion final

La eficiencia en el procesamiento de datos masivos no depende únicamente de la velocidad del hardware, sino de la clase de complejidad del algoritmo. Mientras que la búsqueda lineal escala de forma proporcional al volumen de datos ($O(n)$), la búsqueda binaria divide sistemáticamente el espacio de búsqueda ($O(\log n)$), requiriendo como precondición ineludible que la colección se encuentre estrictamente ordenada por la clave de consulta.

## 11. Reflexion individual

1. **Lo que ahora puedo hacer y antes no podia:**
   Trabajar con ramas independientes en Git (`feature/`), aislando cambios experimentales para proteger la estabilidad de la rama `main`.
2. **El error o supuesto que mas me enseno:**
   Comprender que un algoritmo no falla solo por sintaxis, sino por violación de precondiciones (caso de búsqueda binaria aplicada sobre datos de PM2.5 no ordenados).
3. **La pregunta que llevaria a la proxima clase:**
   ¿Cuál es el costo computacional de ordenar un arreglo frente al beneficio de realizar múltiples búsquedas binarias sobre él?
4. **Que parte del trabajo fue realmente mia:**
   La creación de la rama, la integración del banco de pruebas al `main` único de `IngestaSensores`, el seguimiento de la traza de ejecución y la verificación experimental de las métricas.

## 12. Declaracion del uso de Inteligencia Artificial

- **Herramienta utilizada:** Asistente AI (Gemini).
- **Aportes específicos de la IA:**
    1. Configuración de terminal: Solución al error de reconocimiento de Git en PowerShell mediante la integración interactiva con Git Bash.
    2. Gestión de control de versiones: Instrucciones para la creación y alternancia de ramas (`git switch -c feature/semana-3-busqueda`) para cumplir con la regla de no desarrollar directo sobre `main`.
    3. Soporte conceptual y algorítmico: Formulación de la traza analítica para evidenciar el porqué `inicio = medio + 1` evita bucles infinitos y redacción técnica de precondiciones.
- **Validación del estudiante:** Todo el código fue ensamblado, compilado y ejecutado localmente en IntelliJ, contrastando los resultados de consola de los experimentos con los conceptos teóricos vistos en clase.

## Lista de verificacion antes de entregar

- [x] Escribi la prediccion antes de consultar el resultado.
- [x] Inclui evidencia concreta del laboratorio.
- [x] Explique un concepto sin depender de jerga.
- [x] Registre un vacio, una duda o un error real.
- [x] Trace al menos un caso paso a paso.
- [x] Justifique una decision del proyecto y una alternativa descartada.
- [x] Registre mis commits y mi aporte individual.
- [x] Deje claro que queda pendiente.
- [x] Renombre el archivo con el formato `sXX-nombre.md`.