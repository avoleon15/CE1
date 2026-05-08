# Simulación del Sistema de Puntos de McDonald's

Comparación entre el sistema **batch** (actual) y el sistema en **tiempo real** (propuesto) para la acreditación de puntos de MyMcDonald's Rewards.

Todos los clientes ganan puntos a la misma tasa fija: **10 puntos por cada $1 gastado**, sin categorías ni multiplicadores, tal como funciona el programa real.

---

## Archivos

| Archivo | Descripción |
|---|---|
| `sistema_batch.ipynb` | Sistema actual — proceso nocturno cada 24 h |
| `sistema_tiempo_real.ipynb` | Solución propuesta — acreditación por evento en segundos |
| `mcdonalds_puntos.ipynb` | Versión original con ambos sistemas y comparación completa |

---

## Requisitos

```bash
pip install pandas numpy matplotlib
```

---

## Diferencia principal entre los dos sistemas

El dato que se procesa es el mismo. Lo único que cambia es **cuándo** los puntos quedan disponibles para el cliente.

### Sistema Batch (`sistema_batch.ipynb`)

```
Cliente compra → transacción guardada → ... espera hasta medianoche ... → puntos acreditados
```

- Un proceso centralizado recorre **todas** las transacciones del día y las procesa de una sola vez.
- El cliente puede esperar **hasta 24 horas** para ver sus puntos, con una media de ~12 horas.
- La clave del sistema es la función `ciclo_batch()`, que asigna cada transacción a una ventana de 24 h:

```python
transacciones['ciclo'] = transacciones['timestamp'].apply(
    lambda ts: int((ts - INICIO).total_seconds() / 3600 // BATCH_HORAS)
)
transacciones['disponible'] = transacciones['ciclo'].apply(
    lambda c: INICIO + timedelta(hours=(c + 1) * BATCH_HORAS)
)
```

### Sistema en Tiempo Real (`sistema_tiempo_real.ipynb`)

```
Cliente compra → evento generado → procesador recibe evento → puntos acreditados (1–10 s)
```

- Cada transacción dispara un evento de forma individual e independiente.
- El procesador calcula y acredita los puntos en **1 a 10 segundos**.
- No hay función de ciclos; la latencia se asigna directamente a cada fila:

```python
latencia_seg = np.random.randint(1, 11, size=len(transacciones))
transacciones['disponible'] = [
    ts + timedelta(seconds=int(lag))
    for ts, lag in zip(transacciones['timestamp'], latencia_seg)
]
```

---

## Beneficios del sistema en tiempo real

| Beneficio | Detalle |
|---|---|
| **Latencia ~4,000× menor** | De ~12 horas de media a ~5 segundos |
| **Puntos canjeables de inmediato** | El cliente termina de pagar y ya puede usar sus puntos en la misma visita |
| **Carga distribuida** | El procesamiento se reparte durante el día; el batch genera picos de cómputo intensos a medianoche |
| **Consistencia** | El saldo visible siempre refleja el estado real; el batch muestra un saldo desactualizado durante horas |
| **Misma lógica de negocio** | Los puntos totales emitidos al final del mes son idénticos en ambos sistemas — solo cambia el momento de disponibilidad |

---

## Cómo ejecutar

Abrir cualquiera de los notebooks en Jupyter o VS Code y ejecutar todas las celdas en orden (`Kernel > Restart & Run All`). Cada notebook genera su propia imagen de resultados al correr.
# CE1
