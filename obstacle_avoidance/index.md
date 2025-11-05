---
layout: default
title: Obstacle Avoidance
nav_exclude: false
---

# Obstacle Avoidance

El objetivo de este ejercicio es implementar un algoritmo de navegación que permita a un robot móvil **alcanzar una serie de puntos objetivo** en un entorno desconocido o dinámico, utilizando el **LIDAR** para la **evitación reactiva de obstáculos**.

⚠️ **Handicap**:
  - Para llevar a cabo este ejercicio, solo se permite el uso de **LIDAR** para la detección de obstáculos y la odometría del robot para la localización. 🚫
  - La navegación debe ser continua y robusta, evitando atascos en laberintos o entornos con múltiples obstáculos. ⏱️

    Esto implica tres consideraciones principales:
      1. El algoritmo debe basarse en la lectura de la nube de puntos del LIDAR para percibir el entorno.
      2. El método debe ser lo suficientemente **reactivo** para responder inmediatamente a obstáculos cercanos.
      3. La solución debe incorporar una estrategia para evitar que el robot se quede atrapado en un **mínimo local**.

<p align="center">
  
</p>

>**PRO HINT**
>
>El sensor LIDAR proporciona una rica fuente de información de distancia. Un enfoque elegante y efectivo para combinar el objetivo y la evitación de obstáculos es el método de **Campos de Potencial Artificial**.

___

## Enfoque estándar

### Conceptos Teóricos

Estos son los fundamentos teóricos y las decisiones de diseño tomadas para la solución implementada, basada en el método de **Campos de Potencial Artificial** (Artificial Potential Fields - APF).

#### Campos de Potencial Artificial (APF)

El algoritmo principal modela el movimiento del robot como si estuviera influenciado por fuerzas virtuales:
1.  Una **fuerza atractiva** (Attractive Vector) que siempre apunta al objetivo.
2.  Una **fuerza repulsiva** (Obstacle Vector) que emana de los obstáculos cercanos detectados por el LIDAR.
3.  El **vector de navegación resultante** (Navigation Vector) es la suma vectorial de las fuerzas atractivas y repulsivas, determinando la dirección final del movimiento (`V` y `W`).

#### Vector Atractivo (`target_vector`)

Se calcula la dirección y la magnitud necesaria para mover el robot desde su `robot_pose` actual hacia el `target_pose`. Se implementa en la función `get_target_vector()`, que calcula un vector en el marco de referencia del robot.

#### Vector Repulsivo (`obstacle_vector`)

Este vector agrega las contribuciones de repulsión de todos los puntos del LIDAR que estén dentro del rango de influencia. La magnitud de la fuerza repulsiva de un punto es:
* Inversamente proporcional a la **distancia** del obstáculo.
* Dirigida **en sentido contrario** al obstáculo.

La función `get_obstacle_vector()` procesa las lecturas del LIDAR (`get_laser_measurements()`) y genera un único vector de repulsión total.

#### Escalado Dinámico de Fuerzas

Para lograr una navegación natural y segura, el algoritmo implementa un **escalado dinámico** mediante `calculate_scales(x_component)`:
* Cuando la componente frontal del vector de navegación (`x_component`) es pequeña (lo que implica un giro cerrado o una detención), la escala del vector repulsivo aumenta para priorizar la evitación.
* En movimiento lineal despejado, la escala del vector atractivo domina para mantener el rumbo hacia el objetivo.
* La velocidad lineal (`V`) se reduce en función de la magnitud de la componente lateral (`y_component`) del vector de navegación, permitiendo al robot **decelerar automáticamente durante las maniobras de giro cerradas** para mantener la estabilidad.

___

## Implementación

El código (`obstacle_avoidance.py`) utiliza clases para la gestión de vectores y funciones clave para el cálculo del campo de fuerzas.

### Clases de Utilidad y Conversión

| Clase/Función | Propósito |
| :--- | :--- |
| `Pose2D`, `Vector2D` | Clases para encapsular la posición (x, y, yaw) y vectores 2D. Contienen métodos para operaciones vectoriales como suma, resta y escalado. |
| `_sigmoid(x, k)` | Función sigmoide para un escalado suave y no lineal, utilizada en el ajuste de la velocidad lineal. |
| `polar_to_cartesian()` | Conversión de coordenadas polares (distancia, ángulo del LIDAR) a cartesianas para la suma vectorial. |
| `set_v_safe()`, `set_w_safe()` | Funciones para aplicar velocidades lineales (`V`) y angulares (`W`) de manera segura, respetando los límites absolutos definidos (`ABS_MAX_..._VEL`).

### Funciones Principales

| Función | Descripción |
| :--- | :--- |
| `get_target_vector(target_pose, robot_pose)` | Calcula el vector que atrae al robot hacia el objetivo. |
| `get_obstacle_vector(laser_data)` | Procesa los datos del LIDAR y calcula el vector de fuerza repulsiva total. |
| `calculate_scales(x_component)` | Determina los factores de escala (ponderación) para los vectores de objetivo y obstáculo, ajustando el equilibrio de fuerzas. |

### Flujo de Control Principal

El bucle de ejecución principal sigue esta secuencia:

1.  **Obtener Datos:** Lectura de la pose actual del robot y del objetivo.
2.  **Verificar Objetivo:** Comprueba si la distancia al objetivo está dentro del `TARGET_RADIUS` (`is_target_reached`). Si se alcanzó, solicita el siguiente objetivo a la `WebGUI`.
3.  **Calcular Fuerzas:** Se calculan `target_vector` y `obstacle_vector`.
4.  **Escalar y Sumar:** Se obtienen las escalas dinámicas y se suman los vectores ponderados para obtener el `nav_vector`: `nav_vector = target_vector * scale_target + obstacle_vector * scale_obstacle`.
5.  **Generar Movimiento:** La componente **frontal** (`x_component`) del `nav_vector` se utiliza para la velocidad lineal (`V`), y la componente **lateral** (`y_component`) para la velocidad angular (`W`). Se aplica la lógica de interpolación para la velocidad lineal.
6.  **Aplicar Acciones:** Se establecen las velocidades finales utilizando `set_v_safe()` y `set_w_safe()`.

___

## Parámetros Configurables

Los siguientes parámetros controlan el comportamiento cinemático y de navegación del robot y pueden ser ajustados para modificar la respuesta del sistema:

| Parámetro | Descripción | Valor Predeterminado |
| :--- | :--- | :--- |
| `ABS_MAX_LINEAR_VEL` | Velocidad lineal máxima absoluta del robot. | 2.0 |
| `ABS_MAX_ANGULAR_VEL` | Velocidad angular máxima absoluta del robot. | 1.5 |
| `TARGET_RADIUS` | Radio alrededor del objetivo que se considera "alcanzado". | 1.5 |

___

## Ejemplo de Uso

A continuación, se muestra el principio de funcionamiento del algoritmo y un ejemplo de su aplicación en un entorno simulado.





## Notas de Implementación

**Mínimos Locales:** El problema más conocido del método APF son los **mínimos locales**, donde el robot queda atrapado porque las fuerzas atractivas y repulsivas se anulan. La implementación aborda parcialmente este problema mediante el **escalado dinámico de fuerzas**, dando prioridad a la repulsión cerca de obstáculos para "empujar" al robot fuera de los puntos de equilibrio inerciales, aunque no implementa una solución heurística completa (como agregar ruido aleatorio o seguir el contorno del obstáculo) que sería necesaria para garantizar la superación de todos los mínimos locales.

---
[^1]: Referencia al código fuente: `obstacle_avoidance.py`
