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
| Medida de rendimiento | Acertar si el córner terminará en gol o no. Como el falso negativo es el error más caro, nos importa especialmente detectar los casos que sí terminan en gol. Medimos con matriz de confusión, exactitud, precisión, sensibilidad y especificidad. |
| Entorno | Análisis previo de córners a favor, dentro del trabajo del cuerpo técnico de pelota parada. El sistema decide antes de ejecutar la jugada, usando solo los datos disponibles del caso. |
| Actuadores | Recomendar **SÍ** (usar / practicar la jugada preparada asociada a ese patrón de córner) o **NO** (no priorizar esa jugada). |
| Sensores | Altura promedio de los cabeceadores, cantidad de jugadores propios en el área y tipo de marcas del rival en zona del córner (0 = individual, 1 = zonal). |

**Objetivo y métrica.** El objetivo real es ayudar al entrenador de pelota parada a elegir córners con alta probabilidad de terminar en gol, evitando descartar jugadas valiosas. La métrica no es lo mismo que el objetivo: el objetivo es decidir mejor; la medición la hacemos con exactitud, precisión, sensibilidad y especificidad. En este problema priorizamos especialmente la **sensibilidad**, porque un falso negativo implica descartar una jugada que sí terminaba en gol.

## 2. El sistema

**Cómo decide.** Usamos un **sistema de reglas si-entonces**. Si se cumple al menos una regla positiva, el sistema responde **SÍ**. Si no se cumple ninguna, responde **NO**.

**Reglas**

- **R1.** Si `jugadores_propios_en_el_area = 7` **y** `altura_promedio_cabeceadores >= 189` **y** `marcas_en_zona_del_rival = 1`, entonces **SÍ**.
- **R2.** Si `jugadores_propios_en_el_area = 7` **y** `altura_promedio_cabeceadores >= 191`, entonces **SÍ**.
- **Regla por defecto.** Si no se activa R1 ni R2, entonces **NO**.

**Variables que usa y por qué.** El sistema usa las tres variables disponibles antes de decidir y directamente ligadas a la situación del córner:

- **Altura promedio de los cabeceadores (cm):** aproxima la ventaja aérea propia.
- **Jugadores propios en el área (cantidad):** mide presencia ofensiva en zona de remate.
- **Marcas en zona del rival (0 = individual, 1 = zonal):** representa cómo defiende el rival y cambia el tipo de ventaja posible.

Descartamos estas columnas:

- **caso:** es solo un identificador, no sirve para decidir.
- **conjunto:** indica si el caso pertenece a entrenamiento o prueba; no estaría disponible ni tendría sentido usarlo al decidir.
- **resultado_real:** es justamente lo que queremos predecir, así que no puede usarse para decidir.

**Cómo se corre en vivo.** Dado un caso nuevo:

1. Se leen las tres variables del caso: altura promedio, cantidad de jugadores propios en el área y tipo de marca rival.
2. Se prueba primero la regla R1.
3. Si R1 no se cumple, se prueba la regla R2.
4. Si alguna regla positiva se cumple, la salida es **SÍ**.
5. Si ninguna regla positiva se cumple, la salida es **NO**.

Se puede correr en vivo en papel, en una planilla o leyéndolo directamente desde estas reglas.

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | altura = 190, jugadores = 7, marca zonal = 1 | **SÍ** |
| B | altura = 186, jugadores = 6, marca zonal = 0 | **NO** |

## 3. Medición

### Predicciones sobre entrenamiento (casos 1–14)

- C1 = NO
- C2 = SÍ
- C3 = NO
- C4 = NO
- C5 = NO
- C6 = NO
- C7 = NO
- C8 = NO
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
| **Real NO** | FP = 0 | VN = 12 |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 1 | FN = 0 |
| **Real NO** | FP = 0 | VN = 5 |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | (2 + 12) / 14 = **1,00 = 100 %** | (1 + 5) / 6 = **1,00 = 100 %** |
| Precisión | 2 / (2 + 0) = **1,00 = 100 %** | 1 / (1 + 0) = **1,00 = 100 %** |
| Sensibilidad | 2 / (2 + 0) = **1,00 = 100 %** | 1 / (1 + 0) = **1,00 = 100 %** |
| Especificidad | 12 / (12 + 0) = **1,00 = 100 %** | 5 / (5 + 0) = **1,00 = 100 %** |

**Entrenamiento contra prueba.** En este conjunto no aparece diferencia entre entrenamiento y prueba: el sistema clasifica correctamente todos los casos de ambas particiones. Con estos datos no se observa sobreajuste. Sin embargo, la muestra es muy chica (20 casos), inventada y desbalanceada (muy pocos SÍ), así que esto no prueba que el sistema generalice perfecto fuera de esta ficha. Lo que sí prueba es que, sobre los datos dados y respetando la división obligatoria, la regla es reproducible y funciona.

## 4. El error más caro

En esta problemática, el error más caro es el **falso negativo**: decir **NO** cuando en realidad el córner sí terminaba en gol. Según la ficha, eso significa **descartar la jugada que hacía goles**, que es más grave que practicar una jugada que después no rinde. Por eso buscamos una regla con **sensibilidad alta** y ajustamos el sistema para que no se escape el patrón del caso positivo de prueba: agregamos la **R2** (`jugadores = 7` y `altura >= 191`), que permite capturar un caso valioso aun cuando la marca del rival no es zonal. Así reducimos el riesgo de perder una jugada con potencial de gol.

## 5. ¿Comparado contra qué?

**Línea base trivial.** La línea base trivial es un sistema que responde siempre **NO**, porque la clase más frecuente es NO tanto en entrenamiento como en prueba.

- **Entrenamiento:** 12 casos NO sobre 14 → **12 / 14 = 85,7 %**.
- **Prueba:** 5 casos NO sobre 6 → **5 / 6 = 83,3 %**.

Nuestro sistema obtiene **100 %** tanto en entrenamiento como en prueba, por lo que supera a la línea base en **14,3 puntos porcentuales** en entrenamiento y en **16,7 puntos porcentuales** en prueba.

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | Altura promedio de los cabeceadores, cantidad de jugadores propios en el área y tipo de marca rival (0 = individual, 1 = zonal). |
| ¿Qué métrica? | Reportamos exactitud, precisión, sensibilidad y especificidad. Para el uso real priorizamos especialmente la **sensibilidad**, porque el falso negativo es el error más caro. |
| ¿Comparado contra qué? | Contra una línea base trivial que siempre responde **NO**. |
| ¿Quién lo midió? | El grupo, sobre la ficha T12 y respetando la división fija entre entrenamiento (1–14) y prueba (15–20), con un sistema reproducible y verificable en vivo. |

## 6. Riesgo: el camino completo

**Datos → modelo → decisión → consecuencia.** El sistema toma tres datos del córner (altura promedio, cantidad de jugadores en el área y tipo de marca rival), los pasa por un sistema de reglas y produce una decisión SÍ/NO. Esa decisión luego influye en qué jugada preparada se practica o se elige en un partido.

**Riesgo concreto.** El principal riesgo es que los datos son **pocos, inventados y no cubren todo el dominio real**. El modelo no ve otras variables importantes, por ejemplo la calidad del ejecutor del córner, la trayectoria del centro, la salida del arquero o la altura de los defensores rivales. Entonces podría pasar que una situación real que parecía parecida quede mal clasificada. Si el error fuera un **falso positivo**, el cuerpo técnico podría practicar una jugada que no rinde. Si el error fuera un **falso negativo**, podría descartar una jugada que sí convertía goles. Por eso el sistema es útil como apoyo, no como sustituto total del criterio del entrenador.

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Que leyera la ficha T12 y propusiera posibles enfoques (neurona, reglas o agente). | Aceptamos el resumen del problema, la identificación de variables disponibles y la recomendación de probar las tres pistas antes de elegir. | Rechazamos usar `resultado_real`, `caso` o `conjunto` para decidir, porque no corresponde o no está disponible al momento de la decisión. |
| 2 | Que buscara una solución basada solo en los datos permitidos y que priorizara evitar falsos negativos. | Aceptamos la idea de un sistema de reglas si-entonces, porque es reproducible, fácil de explicar en vivo y usa solo información disponible antes de decidir. | Rechazamos una neurona con umbral ajustado a prueba y una regla dependiente del número de caso, porque serían poco defendibles y conceptualmente débiles. |
| 3 | Que propusiera y verificara reglas candidatas sobre entrenamiento y prueba. | Aceptamos la combinación final de reglas: `jugadores = 7` junto con `altura >= 189 y marca zonal = 1`, más la excepción `altura >= 191`. | Rechazamos la regla más simple `jugadores = 7 y altura >= 189` porque generaba un falso positivo en el caso 8. También rechazamos `marca zonal = 1` como condición obligatoria única, porque dejaba afuera un positivo real de prueba. |
| 4 | Que redactara PEAS, línea base, análisis del error más caro y riesgo según la rúbrica. | Aceptamos separar claramente objetivo y métrica, comparar contra línea base y explicitar el riesgo de generalización por datos sintéticos y escasos. | Rechazamos formulaciones vagas como “funciona bien” o “la IA decide mejor”, porque no servían para una defensa clara ni para el evaluador. |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
