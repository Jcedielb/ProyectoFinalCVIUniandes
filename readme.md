## Entrega final – Tutorial 22 · Hybrid Rendering → “Gelatinous Cube”

> **Objetivo y motivación**
> Partiendo del *Tutorial 22 – Hybrid Rendering* (raster + ray-tracing) se persigue crear una demo interactiva donde un cubo “gelatinoso” reaccione físicamente al movimiento, los saltos y los impactos con el suelo, conserve reflejos y sombras coherentes y, opcionalmente, pueda “desintegrarse” pieza a pieza.
> La práctica ilustra cómo extender un ejemplo puramente gráfico hacia un *game-play loop* sencillo, añadir deformaciones procedurales en HLSL/GLSL y exponer los parámetros en tiempo real mediante ImGui.

---

### 1. Código de base

| componente original                                                 | LOC aprox. | comentario                                   |
| ------------------------------------------------------------------- | ---------: | -------------------------------------------- |
| `Tutorial22_HybridRendering.cpp/.hpp`                               |      2 940 | motor de escena, BLAS/TLAS, PSO’s originales |
| Shaders `Rasterization.vsh/.psh`, `RayTracing.csh`, `PostProcess.*` |        420 | G-buffer, reflexión y sombra                 |


---

### 2. Extensión sobre el código de base

| **área**                            | **archivo(s)**                   | **LOC añadidas** | **qué se hizo**                                                                                                                                                  |
| ----------------------------------- | -------------------------------- | ---------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Física básica y control             | `Tutorial22_HybridRendering.cpp` |             +510 | • Integración de gravedad, salto, rebote con amortiguación <br>• Movimiento WASD + rotación sobre *Yaw*                                                          |
| Deformación “gel” vertical          | `Rasterization.vsh`              |              +24 | Escala Y dependiente de `g_Constants.BounceAmp` y altura del vértice                                                                                             |
| **Shear** según dirección de marcha | `Rasterization.vsh`              |              +18 | Traslación XZ proporcional a `MoveDir` y `FlowAmp` *(non-uniform index safe)*                                                                                    |
| Rebote procedural multi-instancia   | idem + `Update()`                |              +95 | 1 objeto dinámico  ➜ 10 sub-instancias con *slow-motion* escalonado                                                                                              |
| Squash & Stretch al aterrizar       | `Update()`                       |              +60 | Compresión Y + expansión XZ, recuperación exponencial                                                                                                            |
| Rebotes múltiples tipo pelota       | `Update()`                       |              +25 | Coef. `m_BounceDamping` con umbral de corte                                                                                                                      |
| **UI – Controles runtime**          | `UpdateUI()`                     |              +40 | Sliders: *Gel Strength*, *Gravity*, *Bounce Damping*, *Flow* <br>Botón **“Desintegrar cubo”**                                                                    |
| Desintegración progresiva           | `Update()`, `UpdateUI()`         |             +110 | • Flag `m_Desintegrating` <br>• Cronómetro `m_DesintegrationTime` <br>• Vector `m_PartDesintegrationProgress` <br>• Distribución radial de restos hacia el suelo |
| Sombreado/Reflexión coherente       | `RayTracing.csh`                 |              +27 | Desplazamiento vertical de sombra y reflexión usando  la misma deformación que en el VS                                                                          |
| Const-buffer ampliado               | `Structures.fxh`                 |               +6 | `BounceAmp`, `FlowAmp`, `MoveDir (float4)`                                                                                                                       |
| Nuevas variables hpp                | `Tutorial22_HybridRendering.hpp` |              +22 | Movimiento suavizado, squash, rebote, flags de desintegración & UI                                                                                               |

**Incremento total:** **≈ 930 líneas** (+31 % sobre el tutorial original).

---

### 3. Arquitectura extendida

```
┌─ Renderer (render loop)
│   ├─ Update()
│   │   ├─ Física & Input
│   │   ├─ Lógica “Gel”  (bounce, shear, squash)
│   │   └─ Desintegración   ← NUEVO
│   ├─ UpdateTLAS()
│   ├─ Raster pass
│   ├─ Ray-Tracing pass
│   └─ Post-Process
├─ Constant Buffers
│   └─ GlobalConstants{ BounceAmp, FlowAmp, MoveDir, … } ← ampliado
└─ ImGui UI
    ├─ sliders de parámetros
    └─ botón “Desintegrar”
```

*No se añadieron nuevas clases; la lógica reside en el `Tutorial22_HybridRendering` original para mantener la simplicidad.*

---

### 4. Shaders modificados / nuevos

| shader                              | cambios clave                                                                                                         |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **`Rasterization.vsh`**             | • desplazamiento Y por rebote <br>• shear XZ = `-MoveDir * FlowAmp * h` (h = altura relativa)                         |
| **`RayTracing.csh`**                | • uso de la misma fórmula de rebote para ajustar origen de rayos de sombra/reflexión <br>• atenuación según `FlowAmp` |
| `PostProcess.psh/vsh`               | sin cambios funcionales                                                                                               |

---

### 5. Resultados

<img width="1064" alt="image" src="https://github.com/user-attachments/assets/d2b5fefb-0d5d-43f1-aaca-b4724153d3d8" />

Un cubo que:

1. Se desplaza con WASD, deformándose más arriba que abajo.
2. Al saltar (barra spacio) rebota varias veces, con *squash & stretch* visible.
3. Reflejos y sombras siguen la deformación.
4. Deslizador **Flow Amp** controla cuánta “gelatinosidad” horizontal se aplica.
5. Botón **Desintegrar Cubo** provoca que el cubo se fragmente.

---

> **Créditos**
> Basado en el código de ejemplo de Diligent Graphics LLC (Apache-2.0).
> Extensiones implementadas por *Juan Jose Cediel / CVI*, 2025.
