# Reto_11_Azul_Oscuro

Proyecto integral de investigacion que combina **Optimizacion Multi-Objetivo (MOO)**, **Aprendizaje por Refuerzo (RL)**, **Simulacion de Eventos Discretos** y **Diagnostico de Fallos en Rodamientos** para el diseno y mantenimiento de motores electricos.

## Descripcion General

Este proyecto aborda el desafio completo del ciclo de vida de motores electricos, desde el diseno optimo hasta el mantenimiento predictivo, utilizando tecnicas avanzadas de inteligencia artificial y simulacion.

## Estructura del Proyecto

### Fase 1: Optimizacion Multi-Objetivo (MOO)
Analisis y optimizacion del diseno de motores electricos utilizando algoritmos evolutivos.

#### [01_01_analisis_moo.ipynb](01_01_analisis_moo.ipynb)
**Analisis exploratorio de datos de optimizacion multi-objetivo**
- Carga dataset de 4 objetivos con 6 variables de diseno (hm, alfa_m, er, dsi, bst, bss)
- Mapea objetivos tecnicos: Costo Total (TC), Par de Cogging, Eficiencia (-η_ed), Ripple de Torque
- Calcula matrices de correlacion entre variables y objetivos
- Visualiza paisajes objetivos y distribuciones
- Genera mapas de calor codificados por colores (esquema teal/oscuro)

#### [01_02_MOO.ipynb](01_02_MOO.ipynb)
**Framework principal de optimizacion multi-objetivo usando jMetalPy**
- Algoritmos implementados: NSGA-II, NSGA-III, SPEA2
- Entrena modelos surrogate (RandomForest + HistGradientBoosting) para aproximar funciones objetivo
- Aplica busqueda de cuadrícula para ajuste de hiperparametros
- Evalua soluciones usando indicadores de calidad (Hipervolumen, GD, IGD, Epsilon)
- Genera graficos comparativos de frentes de Pareto y rendimiento de algoritmos

#### [01_03_MOO_75.ipynb](01_03_MOO_75.ipynb), [01_04_MOO_50.ipynb](01_04_MOO_50.ipynb), [01_05_MOO_25.ipynb](01_05_MOO_25.ipynb)
**Optimizacion MOO con subconjuntos reducidos del dataset (75%, 50%, 25%)**
- Estudia como varia el rendimiento de algoritmos con la completitud del dataset
- Analisis comparativo de metricas para evaluacion de escalabilidad y robustez

### Fase 2: Simulacion de Eventos Discretos

#### [02_Simpy.ipynb](02_Simpy.ipynb)
**Simulacion de fabrica de motores electricos usando SimPy**
- Simula operaciones de fabrica durante 5 dias (7,200 minutos)
- **Componentes simulados:**
  - **Clase Factory**: Controlador central de operaciones
    - Contenedores de stock: Cobre (Cu) y Aluminio (Al) con capacidad de 500 unidades
    - Recursos de produccion: 2 operadores de rotor, 2 operadores de estator, 3 lineas de ensamblaje
    - Almacenamiento de salida: Bodega de motores con capacidad de 20 unidades
- **Flujos de produccion:**
  1. **Fabricacion de rotor**: ~60 min (distribucion normal) por 2 robots
  2. **Fabricacion de estator**: ~120 min (distribucion normal) por 2 robots
  3. **Inspeccion de calidad**: Deteccion probabilistica de defectos basada en tiempo de fabricacion
  4. **Ensamblaje**: Combina rotor + estator en ~120 min
  5. **Gestion de stock**: Reabastecimiento automatico cuando el inventario baja del umbral
- **Metricas clave:**
  - Tiempos de espera por stock insuficiente
  - Tasas de defectos en rotores y estatores
  - Niveles de produccion de motores y bodega
  - Dinamicas de oferta/demanda de stock

**Aplicacion real:** Optimizacion de puntos criticos de stock y cuellos de botella en produccion

### Fase 3: Optimizacion con Aprendizaje por Refuerzo

#### [env_R11.py](env_R11.py) & [env_R12.py](env_R12.py)
**Entornos personalizados compatibles con Gymnasium para RL**
- **Clase principal:** `motorEnv` y `motorEnv2`
- **Espacio de estados:** Coordenadas (var1, var2) del paisaje de optimizacion 2D
- **Espacio de acciones:** 4 acciones discretas (mover ±1 en direcciones var1 o var2)
- **Sistema de recompensas:**
  - **+1000**: Alcanzar minimo global de funcion objetivo w
  - **-1**: Movimiento a estado valido nuevo
  - **-100**: Accion invalida (fuera de limites)
  - **-1 a -100**: Penalizaciones por movimiento por defecto

**Datos de entorno:**
- `env_R11.py`: Usa dataset `Datos_v1.csv`
- `env_R12.py`: Usa dataset `Datos_v2.csv`

#### [03_optimizacion_RL1.ipynb](03_optimizacion_RL1.ipynb)
**Optimizacion de hiperparametros RL para el primer entorno**
- **Fase 1 - Busqueda de cuadrícula** (250 episodios por combinacion):
  - Prueba combinaciones de hiperparametros: Alpha [0.4, 0.2], Gamma [0.99, 0.999], Decay [0.99, 0.995], Multiplier [200, 300]
  - Usa formula de guia personalizada: `reward_guided = reward + improvement_factor × (w_old - w_new) × multiplier`
  - Selecciona mejor combinacion por recompensa promedio de ultimos 30 episodios
  - Guarda mejores parametros en `Datos/Transformados/mejores_params_R11.npy`
- **Fase 2 - Entrenamiento con parada temprana:**
  - Entrena agente Q-learning con mejores parametros de Fase 1
  - Implementa parada temprana basada en deteccion de meseta de aprendizaje

#### [03_optimizacion_RL2.ipynb](03_optimizacion_RL2.ipynb)
**Optimizacion RL extendida para el segundo entorno con tecnicas avanzadas**
- Mejoras: Estrategias de entrenamiento mas sofisticadas y secuencias de episodios mas largas

### Fase 4: Diagnostico de Fallos en Rodamientos y Procesamiento de Senales

#### [04_01_analisis_normal.ipynb](04_01_analisis_normal.ipynb)
**Extraccion de caracteristicas y analisis de operacion normal de rodamientos**
- **Fuente de datos:** `Datos/03_Validacion/datos_bearing/normal/` (61 archivos CSV a diferentes frecuencias de rotacion)
- **Pipeline de analisis:**
  1. Carga datos de sensores multi-canal (8 canales: tacometro, 6 acelerometros, microfono)
  2. **Frecuencia de muestreo:** 51,200 Hz
  3. **Extraccion de caracteristicas** del dominio del tiempo:
     - Estadisticas: Media, Desviacion Estandar, Curtosis, Asimetria
     - Valores pico, RMS, Factor de Cresta
  4. **Analisis tiempo-frecuencia:** FFT, espectrogramas
  5. Genera dashboards de visualizacion comparando comportamientos baseline
  6. Entrena clasificador baseline para deteccion de desbalance/desalineacion

#### [04_02_analisis_rodamiento.ipynb](04_02_analisis_rodamiento.ipynb)
**Analisis comprehensivo de fallos en rodamientos (posiciones overhang, underhang)**
- **Tipos de fallos analizados:**
  - **Fallos de Carrera Exterior:** Diferentes posiciones de montaje del rodamiento
  - **Fallos de Bolas:** Dano localizado en elementos rodantes
  - **Fallos de Jaula:** Fallos en anillo retenedor
- **Jerarquia de datos:**
  ```
  bearing_fault_detection/
  ├── overhang/outer_race/, ball_fault/, cage_fault/
  └── underhang/outer_race/, ball_fault/, cage_fault/
  ```
- **Analisis:**
  - Compara firmas normales vs. fallos
  - Extrae caracteristicas avanzadas (analisis de envolvente, indices de modulacion)
  - Entrena clasificadores RandomForest para discriminacion de tipos de fallo
  - Genera matrices de confusion y curvas ROC

#### [04_03_analisis_imbalance.ipynb](04_03_analisis_imbalance.ipynb)
**Analisis diagnostico de desbalance rotacional**
- **Niveles de severidad:** 6g, 10g, 15g, 20g, 25g, 30g, 35g masas de desbalance
- **Procesamiento de senales:**
  - Detecta firma caracteristica de desbalance (pico de frecuencia rotacional 1×)
  - Analiza progresion de severidad con desbalance incrementado
  - Entrena clasificadores para predecir nivel de masa de desbalance (regresion)

#### [04_04_nalisis_desalineacion.ipynb](04_04_nalisis_desalineacion.ipynb)
**Analisis de fallos de desalineacion en rodamientos**
- **Tipos de desalineacion:**
  - **Desalineacion horizontal:** 0.5mm, 1.0mm, 1.5mm, 2.0mm
  - **Desalineacion vertical:** 0.51mm, 0.63mm, 1.27mm, 1.40mm, 1.78mm, 1.90mm
- **Caracteristicas:**
  - Detecta armonicos 2× y 3× de frecuencia rotacional
  - Analiza relaciones de fase entre canales de aceleracion
  - Cuantifica severidad de desalineacion desde caracteristicas de senal

#### [04_05_Validacion_DSP.ipynb](04_05_Validacion_DSP.ipynb)
**Framework comprehensivo de diagnostico y procesamiento de senales (DSP)**
- **Alcance:** Integra todas las condiciones de rodamientos (normal, desbalance, desalineacion, fallos)
- **Flujo de trabajo:**
  1. **Carga de datos y extraccion de metadatos:**
     - Parsea estructura jerarquica de directorios
     - Asigna niveles de severidad basados en tipo de condicion
     - Crea dataset unificado
  2. **Extraccion avanzada de caracteristicas:**
     - Dominio del tiempo: Estadisticas, analisis de envolvente
     - Dominio de frecuencia: FFT, densidad espectral de potencia
     - Analisis wavelet: Transformadas continuas/discretas
     - Envolvente de senal: Transformada de Hilbert en frecuencias de fallo de rodamientos
  3. **Seleccion de caracteristicas:** Ranking F-statistic con scikit-learn
  4. **Entrenamiento de modelo y validacion cruzada:**
     - Clasificador RandomForest para diagnostico multi-clase
     - Validacion cruzada estratificada K-fold
     - RandomizedSearchCV para ajuste de hiperparametros
  5. **Evaluacion de rendimiento:**
     - Reportes de clasificacion, matrices de confusion
     - Metricas de precision por clase
     - Visualizacion de limites de decision
  6. **Graficos de validacion final:** Charts diagnosticos comprehensivos

## Flujo de Datos y Trabajo

```
FASE 1 (MOO)
├─→ Opt_dataset_4Objectives_CC.csv
├─→ Analisis exploratorio (01_01)
├─→ Comparacion de algoritmos (01_02)
└─→ Tests de escalabilidad (01_03-01_05) → Frentes de Pareto

FASE 2 (SIMULACION)
├─→ 02_Simpy.ipynb
├─→ Simulacion de dinamicas de fabrica
└─→ Analisis de cuellos de botella en produccion

FASE 3 (RL)
├─→ Datos_v1.csv & Datos_v2.csv
├─→ env_R11.py, env_R12.py (Entornos)
├─→ 03_optimizacion_RL1.ipynb (Busqueda de cuadrícula + Q-learning)
├─→ 03_optimizacion_RL2.ipynb (RL avanzado)
└─→ mejores_params_R11.npy (Mejores parametros)

FASE 4 (DIAGNOSTICO DE RODAMIENTOS)
├─→ datos_bearing/ (Datos de sensores 8-canales)
├─→ 04_01: Analisis baseline normal
├─→ 04_02: Clasificacion de fallos en rodamientos
├─→ 04_03: Deteccion de severidad de desbalance
├─→ 04_04: Cuantificacion de desalineacion
└─→ 04_05: Framework unificado de validacion DSP
```

## Caracteristicas Tecnicas Clave

| Componente | Tecnologia | Proposito |
|------------|------------|-----------|
| **MOO** | jMetalPy (NSGA-II, NSGA-III, SPEA2) | Optimizacion multi-objetivo de motores |
| **Surrogate** | RandomForest + HistGradientBoosting | Aproximacion de objetivos costosos |
| **Simulacion** | SimPy | Modelado de fabrica de eventos discretos |
| **RL** | Q-Learning (Gymnasium) | Navegacion de paisaje multi-paso |
| **Procesamiento de Senales** | SciPy (wavelets, Hilbert) | Diagnosticos de fallos en rodamientos |
| **Clasificacion ML** | RandomForest | Clasificacion basada en condicion |
| **Visualizacion** | Matplotlib + Seaborn | Esquema de color teal personalizado (#00929a) |

## Dependencias

Para instalar todas las dependencias necesarias:

```bash
pip install -r requirements.txt
```

**Librerias principales:**
- **Ciencia de Datos:** numpy, pandas, matplotlib, seaborn
- **Machine Learning:** scikit-learn
- **Procesamiento de Senales:** scipy
- **Optimizacion Multi-Objetivo:** jmetal
- **Simulacion:** simpy
- **Wavelets:** PyWavelets
- **Reinforcement Learning:** gymnasium
- **Utilidades:** joblib, tqdm

## Estructura de Datos

```
Datos/
├── 01_Optimizacion/
│   └── Opt_dataset_4Objectives_CC.csv
├── 02_Reinforcement_learning/
│   ├── Datos_v1.csv
│   └── Datos_v2.csv
└── 03_Validacion/
    ├── bearing_fault_full.url
    ├── bearing_fault_reduced.url
    └── datos_bearing/
        ├── horizontal-misalignment/
        ├── imbalance/
        ├── normal/
        ├── overhang/
        └── vertical-misalignment/
```

## Como Ejecutar

1. **Instalar dependencias:** `pip install -r requirements.txt`
2. **Ejecutar en orden las fases:**
   - Fase 1: Notebooks 01_01 → 01_02 → 01_03-01_05
   - Fase 2: Notebook 02_Simpy
   - Fase 3: env_R11.py/env_R12.py → 03_optimizacion_RL1 → 03_optimizacion_RL2
   - Fase 4: Notebooks 04_01 → 04_02 → 04_03 → 04_04 → 04_05

## Resultados y Artefactos

- **Frentes de Pareto** de algoritmos MOO
- **Parametros optimos RL** guardados en `mejores_params_R11.npy`
- **Modelos entrenados** para diagnostico de rodamientos
- **Metricas de rendimiento** y visualizaciones comparativas
- **Analisis de `Analisis del Agente.json`** (metricas de evaluacion de agente)

## Objetivos del Proyecto

- **Diseno optimo** de motores electricos usando MOO
- **Optimizacion de procesos** de fabricacion mediante simulacion
- **Aprendizaje autonomo** de navegacion en espacios de optimizacion
- **Mantenimiento predictivo** basado en analisis de vibraciones
- **Integracion completa** de tecnicas de IA para ciclo de vida de motores

---

*Proyecto desarrollado como parte del Reto 11 - Azul Oscuro*