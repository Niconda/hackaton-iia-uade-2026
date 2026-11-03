# T12 · ¿Este córner termina en gol?

> **Hackatón IIA · UADE · 3.4.257.**

**Integrantes:** Nicolas Ondarra · Emiliano Picardo  
**IA declarada:** ChatGPT  
**Pista:** Reglas  
**Publicación en LinkedIn:** Pendiente de publicación del grupo

---

## 1. Formulación y PEAS

**La pregunta.** Para cada caso, ¿este córner termina en gol? **SÍ** significa: el córner termina en gol.

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | Medir qué tan bien predice si el córner termina en gol. Reportamos matriz de confusión, exactitud, precisión, sensibilidad y especificidad. Como el falso negativo es el error más caro, priorizamos especialmente la **sensibilidad**. |
| Entorno | Córners a favor analizados por el cuerpo técnico de pelota parada. El sistema recibe únicamente información disponible antes de conocer el resultado del córner. |
| Actuadores | Emitir al entrenador una salida **SÍ** (“predice que el córner termina en gol”) o **NO** (“predice que no termina en gol”). La decisión posterior de practicar o usar una jugada pertenece al cuerpo técnico, no es la predicción misma. |
| Sensores | Altura promedio de los cabeceadores y cantidad de jugadores propios en el área, datos disponibles antes de ejecutar el córner. El tipo de marca rival también está disponible en la ficha, pero la regla final decide no usarlo. |

**Objetivo y métrica.** El objetivo real del sistema es **predecir correctamente si un córner termina en gol**, dando especial importancia a no dejar escapar córners que sí terminan en gol. La métrica no es el objetivo: usamos exactitud, precisión, sensibilidad y especificidad para medir el desempeño. Priorizamos la **sensibilidad** porque un falso negativo significa decir NO cuando en realidad el córner terminaba en gol.

## 2. El sistema

**Cómo decide.** Usamos un **sistema de reglas si-entonces construido únicamente con los casos de entrenamiento 1–14**. La regla quedó definida antes de mirar el rendimiento de los casos 15–20 y no se modifica después de evaluarlos.

**Regla final**

- **R1.** Si `jugadores_propios_en_el_area = 7` **y** `altura_promedio_cabeceadores >= 189`, entonces **SÍ**.
- **Regla por defecto.** Si no se cumple R1, entonces **NO**.

**Cómo se definió la regla usando solo entrenamiento.** En los casos 1–14 hay solo **dos positivos reales**, C2 y C9. Ambos tienen **7 jugadores propios en el área** y alturas de **189 cm y 190 cm**. Por eso tomamos 7 jugadores como condición común y fijamos **189 cm** como umbral: es la menor altura observada entre los positivos de entrenamiento y permite conservar a ambos positivos. Con esa regla, el caso C8 (189 cm, 7 jugadores) queda como falso positivo. Aceptamos deliberadamente ese error porque, según la ficha, el **falso negativo es más caro** y preferimos mantener sensibilidad 100 % en entrenamiento antes que hacer una regla demasiado estricta.

**Variables que usa y por qué.** El sistema usa:

- **Altura promedio de los cabeceadores (cm):** aproxima la capacidad aérea del grupo que ataca el córner.
- **Jugadores propios en el área (cantidad):** representa la presencia ofensiva disponible para finalizar la jugada.

Variables disponibles pero descartadas:

- **Marcas en zona del rival:** no la usamos en la regla final. En entrenamiento, exigir marca zonal permitiría ajustar perfectamente los dos únicos positivos, pero con una muestra positiva tan pequeña sería una regla demasiado específica. Preferimos una regla más simple y sensible.
- **caso:** es solo un identificador, no sirve para decidir.
- **conjunto:** indica si el registro es de entrenamiento o prueba; usarlo sería incorrecto para decidir.
- **resultado_real:** es lo que queremos predecir y solo se conoce después, por lo que nunca se usa como entrada.

**Cómo se corre en vivo.** Para resolver un caso nuevo:

1. Se mira la cantidad de jugadores propios en el área.
2. Se mira la altura promedio de los cabeceadores.
3. Si `jugadores = 7` y `altura >= 189`, se responde **SÍ**.
4. En cualquier otro caso, se responde **NO**.

No se necesita conocer la etiqueta real ni saber si el caso pertenece a entrenamiento o prueba.

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | altura = 190, jugadores = 7, marca zonal = 1 | **SÍ** |
| B | altura = 186, jugadores = 6, marca zonal = 0 | **NO** |

## 3. Medición

La regla anterior se definió usando solamente los casos **1–14**. Recién después de congelarla se aplicó sin cambios a los casos **15–20**.

### Predicciones sobre entrenamiento (casos 1–14)

- C1 = NO
- C2 = SÍ
- C3 = NO
- C4 = NO
- C5 = NO
- C6 = NO
- C7 = NO
- C8 = SÍ
- C9 = SÍ
- C10 = NO
- C11 = NO
- C12 = NO
- C13 = NO
- C14 = NO

### Predicciones sobre prueba (casos 15–20)

- C15 = NO
- C16 = NO
- C17 = NO
- C18 = NO
- C19 = SÍ
- C20 = NO

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 2 | FN = 0 |
| **Real NO** | FP = 1 | VN = 11 |

Control: 2 + 0 + 1 + 11 = **14**.

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 1 | FN = 0 |
| **Real NO** | FP = 0 | VN = 5 |

Control: 1 + 0 + 0 + 5 = **6**.

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | (2 + 11) / 14 = **13/14 = 92,9 %** | (1 + 5) / 6 = **6/6 = 100 %** |
| Precisión | 2 / (2 + 1) = **2/3 = 66,7 %** | 1 / (1 + 0) = **1/1 = 100 %** |
| Sensibilidad | 2 / (2 + 0) = **2/2 = 100 %** | 1 / (1 + 0) = **1/1 = 100 %** |
| Especificidad | 11 / (11 + 1) = **11/12 = 91,7 %** | 5 / (5 + 0) = **5/5 = 100 %** |

**Entrenamiento contra prueba.** La regla obtiene 92,9 % de exactitud en entrenamiento y 100 % en prueba. Esto **no demuestra una generalización perfecta**, porque la prueba contiene solo 6 casos y apenas **un positivo real**. En total hay solo **3 positivos en los 20 casos**: 2 en entrenamiento y 1 en prueba. Por eso, aunque la sensibilidad de prueba sea 100 %, en realidad significa **1/1**, una estimación muy poco estable. Lo importante metodológicamente es que los casos de prueba no se usaron para diseñar ni modificar la regla: se aplicaron recién después de congelarla.

## 4. El error más caro

En esta problemática, el error más caro es el **falso negativo**: decir **NO** cuando en realidad el córner sí terminaba en gol. Según la ficha, eso implica **descartar una jugada que hacía goles**. El falso positivo también tiene costo —practicar una jugada que no rinde—, pero es menos grave.

Para reducir falsos negativos, elegimos con los datos de entrenamiento una regla deliberadamente sensible: `jugadores = 7` y `altura >= 189`. El umbral **189** no salió del caso de prueba: se definió porque es la menor altura de los dos positivos del entrenamiento (C2 = 189 y C9 = 190). Esta elección mantiene **0 falsos negativos en entrenamiento**, aunque acepta un falso positivo (C8). Es un intercambio consciente: sacrificamos algo de precisión para proteger la sensibilidad, que es la métrica más vinculada al error más caro.

## 5. ¿Comparado contra qué?

**Línea base trivial.** La clase mayoritaria es **NO**, por lo tanto la línea base trivial responde siempre NO.

- **Entrenamiento:** hay 12 NO de 14 → **12/14 = 85,7 %** de exactitud.
- **Prueba:** hay 5 NO de 6 → **5/6 = 83,3 %** de exactitud.

Nuestro sistema obtiene **92,9 %** en entrenamiento y **100 %** en prueba. Supera a la línea base en **7,2 puntos porcentuales** en entrenamiento y en **16,7 puntos porcentuales** en prueba.

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | Altura promedio de los cabeceadores y cantidad de jugadores propios en el área. La ficha también contiene tipo de marca rival, pero la regla final no lo usa. |
| ¿Qué métrica? | Exactitud, precisión, sensibilidad y especificidad. Priorizamos sensibilidad porque el falso negativo es el error más caro. |
| ¿Comparado contra qué? | Contra la línea base trivial que siempre responde **NO**. |
| ¿Quién lo midió? | El grupo, sobre la ficha T12 y respetando la división fija: entrenamiento 1–14 para construir la regla y prueba 15–20 únicamente para evaluarla. |

## 6. Riesgo: el camino completo

**Datos → modelo → decisión → consecuencia.** El sistema toma la altura promedio de los cabeceadores y la cantidad de jugadores propios en el área, aplica la regla y produce una predicción **SÍ/NO** sobre si el córner terminará en gol. El entrenador puede usar esa predicción como una señal más para decidir qué jugadas preparar o priorizar.

**Riesgo concreto.** La muestra es muy pequeña, inventada y desbalanceada: hay solamente **3 positivos reales en 20 casos**. Además, el sistema no contempla otras variables futbolísticas relevantes, como calidad del ejecutor, trayectoria del centro, altura de los defensores, salida del arquero o movimiento previo. Por eso un 100 % en los 6 casos de prueba no debe interpretarse como “100 % de efectividad real”. Un falso positivo puede llevar a practicar una jugada que no rinde; un falso negativo puede hacer descartar una jugada que sí convertía. El sistema debe usarse como apoyo y validarse con más datos antes de tomar decisiones reales.

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Que leyera la ficha T12 y propusiera posibles enfoques (neurona, reglas o agente). | Aceptamos el resumen del problema, la identificación de variables disponibles y la idea de usar reglas por ser reproducibles y fáciles de defender en vivo. | Rechazamos usar `resultado_real`, `caso` o `conjunto` para decidir, porque no corresponden a información válida disponible al momento de la decisión. |
| 2 | Que buscara una solución basada solo en los datos permitidos y que priorizara evitar falsos negativos. | Aceptamos trabajar con un sistema de reglas si-entonces y priorizar sensibilidad por el costo del falso negativo. | Rechazamos reglas que dependieran de la etiqueta real, del número de caso o de la pertenencia a entrenamiento/prueba. |
| 3 | Que propusiera una regla inicial. | En la primera versión aceptamos una combinación con marca zonal y una segunda regla de altura. | Después de la devolución del evaluador rechazamos esa versión porque la segunda regla había sido incorporada mirando C19, que pertenece a prueba. Eso invalidaba la independencia de la evaluación. |
| 4 | Que redactara PEAS, línea base, error más caro y riesgo según la rúbrica. | Aceptamos separar objetivo y métricas, comparar contra una línea base y explicitar los riesgos de una muestra pequeña. | Rechazamos formulaciones vagas como “funciona bien” porque no permiten una defensa verificable. |
| 5 | Le pasamos completa la devolución de la **corrida 1 del evaluador (7,5/10)** y pedimos corregir únicamente los agujeros señalados sin usar prueba para diseñar el sistema. | Aceptamos alinear el actuador con la salida real del clasificador; eliminar la R2 construida mirando C19; rediseñar la regla usando exclusivamente C1–C14; justificar el umbral 189 a partir de los dos positivos de entrenamiento; recalcular matrices y métricas; y aclarar que la sensibilidad 100 % de prueba equivale a solo 1/1 y que existen 3 positivos en total. | Rechazamos conservar cualquier ajuste basado en C19 o presentar el 100 % de prueba como evidencia de generalización perfecta, porque eso repetiría exactamente los problemas detectados por el evaluador. |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | **7.5/10** | **“¿Cuándo y con qué criterio definiste el umbral de altura de R2? Si la evaluás usando solo los casos 1–14, ¿seguís teniendo una regla que se sostenga?”** Además señaló que el actuador no coincidía con la pregunta, que R2 había sido diseñada mirando el caso 19 de prueba, que había solo un positivo en prueba y que faltaba registrar la corrida. | Alineamos el actuador con la predicción SÍ/NO; eliminamos R2; redefinimos la regla usando únicamente C1–C14; fijamos el umbral 189 a partir del mínimo de los positivos de entrenamiento; aceptamos un FP para priorizar sensibilidad; recalculamos matrices y métricas; aclaramos que prueba tiene 1 positivo y que hay 3 positivos en total; y registramos esta corrida. |
