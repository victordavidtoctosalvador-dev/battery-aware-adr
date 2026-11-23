# Battery-Aware ADR para LoRaWAN

Código, configuración y datos de simulación del trabajo **"Optimización del ciclo de vida de baterías en nodos LoRaWAN para agroindustria mediante el control adaptativo del Spreading Factor basado en el estado de carga (Battery-Aware ADR)"**.

Autores: Víctor David Tocto Salvador, Gabriel Steven Rojas Cerrón — Universidad Tecnológica del Perú (UTP).
Contacto: [u22300524@utp.edu.pe] · [u22209423@utp.edu.pe]

Este repositorio acompaña al artículo y permite auditar y reproducir los resultados reportados en las Tablas IV y V y en los Anexos A y B.

---

## 1. Qué hay aquí

El estudio compara tres algoritmos de gestión del Spreading Factor (SF) en ns-3, bajo las mismas condiciones de red, canal y energía:

- **GC** — ADR estándar de LoRaWAN v1.0.4 (grupo de control).
- **GE1** — reimplementación de BLO (Battery Life Optimization) de De Andrade et al., asignación centralizada por MILP.
- **GE2** — Battery-Aware ADR, el algoritmo propuesto: cada nodo consulta su propio estado de carga (SoC) antes de transmitir y ajusta el SF según tres regímenes.

Regímenes de GE2 (techo de SF según SoC):

| Estado de carga (SoC) | Régimen  | Techo de SF |
|-----------------------|----------|-------------|
| SoC > 60 %            | Normal   | Sin ajuste adicional por batería |
| 30 % < SoC ≤ 60 %     | Ahorro   | SF máximo objetivo 10 |
| 0 % < SoC ≤ 30 %      | Crítico  | SF máximo objetivo 8  |

---

## 2. Requisitos

- **ns-3 v3.48**
- Módulo **LoRaWAN de SignetLabDEI** (Magrin et al.)
- **Ubuntu 24.04 LTS**
- Python 3 (para el procesamiento de resultados: pandas, numpy, scipy, matplotlib)

---

## 3. Estructura del repositorio

```
battery-aware-adr/
├── README.md
├── src/
│   ├── battery-aware-adr/      # Algoritmo propuesto GE2 (C++)
│   ├── blo-reimplementation/   # Reimplementación de BLO (GE1)
│   └── energy-model/           # Modelo de energía modificado
├── config/
│   └── au915_params.*          # Parámetros de la Tabla II
├── scripts/
│   ├── run_campaign.sh         # Ejecuta las 60 corridas (RngRun 101–120)
│   └── process_results.py      # Consolida resumen.csv y calcula indicadores
├── data/
│   ├── resumen.csv
│   ├── supervivencia_por_nodo.csv
│   ├── metricas_entrega.csv
│   └── energia_final.csv
└── results/
    └── RESULTADO.json          # Salida cruda por ejecución
```

> Ajusta los nombres de carpetas y scripts a los reales de tu proyecto.

---

## 4. Parámetros de la simulación (campaña final, AU915)

| Parámetro | Valor |
|-----------|-------|
| Banda de frecuencia | AU915 |
| Acceso al canal | uplinkDwellTime = 0 |
| Bandwidth / Coding rate | 125 kHz / 4/5 |
| Número de nodos | 500 |
| Gateway | 1 con 8 vías de recepción |
| Potencia de Tx | 20 dBm en los nodos y 30 dBm en el gateway |
| Payload | 51 bytes |
| Intervalo de generación | 60 s por nodo |
| Duración de la simulación | 24 h por corrida |
| Capacidad de batería | 50 mAh, carga inicial al 100 % |
| Propagación (d0, γ, Lpl) | 40 m; 2,08; 127,41 dB |
| Shadowing (σ) | 0 dB |
| Rango de SF | 7–12 |
| Radio de despliegue | 1000 m (distribución circular) |
| Réplicas por grupo | 20 |

---

## 5. Cómo reproducir

1. Compila ns-3 v3.48 con el módulo LoRaWAN y los archivos de `src/` integrados.
2. Ejecuta la campaña final:
   ```bash
   bash scripts/run_campaign.sh
   ```
   La campaña usa **RngSeed = 1** y **RngRun del 101 al 120**, formando 20 bloques por grupo (60 corridas en total). Cada bloque corre GC, GE1 y GE2 con la misma topología, tráfico, propagación y batería.
3. Procesa los resultados:
   ```bash
   python3 scripts/process_results.py
   ```
   Esto consolida `resumen.csv` (una fila por corrida) a partir del `RESULTADO.json` de cada ejecución y calcula los indicadores.

---

## 6. Trazabilidad de métricas (Anexo B)

| Indicador | Columna en resumen.csv | Archivo fuente por corrida |
|-----------|------------------------|----------------------------|
| Supervivencia a 24 h | supervivencia_final_pct | supervivencia_por_nodo.csv |
| Entrega de demanda 0-fin | PDR_demanda_0_fin_pct | metricas_entrega.csv |
| PDR de aplicación | PDR_TX_app_pct | metricas_entrega.csv |
| SoC residual medio / mínimo | SoC_final_medio_todos_pct / SoC_final_minimo_pct | energia_final.csv |
| Consumo de radio por nodo | total_radio_mAh_por_nodo | energia_final.csv |

