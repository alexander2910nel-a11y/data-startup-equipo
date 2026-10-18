# E0 · Propuesta del Proyecto
**Proyecto Integrador DATA STARTUP**  
**Materias:** Cómputo en la Nube (IAD-2408) · Analítica de Datos (IAD-2403)  
**Giro del negocio:** Refaccionaria automotriz en Guadalajara, Jalisco  
**Proveedor de Nube:** Amazon Web Services (AWS) - Región `us-east-1`  

---

## 1. Problema y Cliente

### Descripción del problema
Las refaccionarias independientes en Guadalajara operan con catálogos amplios de autopartes de alta y media rotación (aceites, filtros, balatas, bujías, baterías, anticongelantes, etc.), pero toman sus decisiones de compra semanal de manera empírica ("a ojo" revisando anaqueles vacíos). Esto provoca dos problemas financieros directos:
1. **Ventas perdidas por desabasto (Stockout):** Quedarse sin refacciones de alta rotación en días pico (viernes, sábados y quincenas) o al inicio del temporal de lluvias en Guadalajara, haciendo que el mecánico o cliente compre en otra refaccionaria.
2. **Capital inmovilizado:** Comprar exceso de mercancía de baja rotación sin respaldo en datos históricos reales.

---

## 2. Lean Canvas (9 Bloques)

| Bloque | Descripción |
| :--- | :--- |
| **1. Problema** | • Decisiones de reabastecimiento tomadas a ojo sin analizar ciclos semanales ni quincenales.<br>• Pérdida de ventas en mostrador y con talleres mecánicos por falta de inventario en piezas clave.<br>• Registros de ventas en archivos CSV con errores de captura (duplicados, negativos, faltantes) que no se aprovechan. |
| **2. Segmento de Clientes** | • Refaccionarias independientes y multimarca ubicadas en Guadalajara, Jalisco, que venden al mostrador y a talleres mecánicos locales. |
| **3. Propuesta de Valor** | • Plataforma en la nube que limpia automáticamente el historial de ventas de la refaccionaria, muestra en 30 segundos el estado del negocio y pronostica cuántas piezas se venderán la siguiente semana para saber qué pedir al proveedor. |
| **4. Solución** | • Almacenamiento en la nube (AWS S3) dividido en zona cruda (`raw/`) y zona limpia (`clean/`).<br>• Dashboard empresarial en Streamlit con KPIs, 6 visualizaciones e identificación de ciclos mediante Transformada de Fourier (FFT).<br>• Servicio de pronóstico de demanda y clasificación en producción. |
| **5. Canales** | • Contacto directo con dueños y encargados de mostrador en zonas comerciales de autopartes en Guadalajara.<br>• Demostración directa del tablero web desde celular o computadora. |
| **6. Fuentes de Ingreso** | • **Suscripción mensual (SaaS):** Cuota fija de $600 MXN al mes por refaccionaria. |
| **7. Estructura de Costos** | • **Etapa actual (Prototipo/MVP):** $0 MXN operando sobre la capa gratuita y créditos de estudiante en AWS con alerta de presupuesto activa en $0.<br>• **Operación estimada al escalar:** ~$25 a $35 MXN mensuales por cliente en servicios administrados de AWS. |
| **8. Métricas Clave** | • Error del pronóstico semanal de ventas (MAE / RMSE).<br>• Reducción de quiebres de inventario en los productos estrella.<br>• Tiempo de actualización de datos de `raw/` a `clean/`. |
| **9. Ventaja Injusta** | • Herramienta diseñada específicamente para el flujo de trabajo de una refaccionaria local (lectura directa de sus reportes CSV/API sin obligarlos a cambiar su sistema de caja actual) considerando los ciclos de quincena y clima de Guadalajara. |

---

## 3. Entrevista con el Dueño / Encargado del Giro (15+ minutos)

* **Perfil entrevistado:** Encargado de compras y mostrador de una refaccionaria en Guadalajara.
* **Duración:** 18 minutos.

### Las 5 preguntas y sus respuestas
1. **¿Cómo deciden actualmente qué piezas pedir al proveedor cada semana y en qué cantidad?**
   > *"Los lunes damos una vuelta por los estantes para ver de qué quedó poquito y revisamos por encima lo que se vendió el viernes y sábado. Pedimos más o menos lo de siempre o si el distribuidor trae oferta en cajas de aceite o bujías."*
2. **¿Cuáles son los días o fechas donde más se les complica surtir o donde más se les junta la gente?**
   > *"Los viernes y sábados siempre hay más movimiento porque los talleres quieren entregar carros antes del domingo y la gente aprovecha su descanso. También en las quincenas (días 15 y 30) se nota luego luego cómo sube la venta, y cuando empiezan las lluvias fuertes aquí en Guadalajara todos buscan limpiaparabrisas, baterías y frenos."*
3. **¿Qué ocurre cuando un cliente o mecánico llega por una pieza común y se les terminó?**
   > *"Se va a la refaccionaria de al lado. El problema es que si venía por todo el kit de afinación y no tengo el filtro o las bujías, ya no me compra tampoco el aceite ni el anticongelante."*
4. **¿Cómo llevan el registro de sus ventas diarias y qué problemas tienen con esos datos?**
   > *"El sistema de la caja nos deja sacar tablas en Excel o CSV, pero salen con muchos detalles: a veces registran devoluciones como números negativos, capturan mal las cantidades, dejan celdas vacías o escriben el mismo producto con minúsculas y mayúsculas, así que nadie revisa esos archivos."*
5. **Si tuviera un tablero sencillo que limpiara esos archivos solo y le dijera cada lunes cuántas unidades se van a vender en la semana, ¿lo usaría?**
   > *"Sí, sobre todo si es rápido de leer y me dice claro cuál producto es el que más dinero me deja y cuánto debo pedir antes del viernes o de la quincena."*

### Aprendizajes clave de la entrevista
* La decisión crítica ocurre al inicio de la semana (lunes) y antes de cada quincena para cubrir los picos de viernes y sábado.
* Los archivos exportados por la caja tienen exactamente los 5 problemas de calidad de datos que resolveremos en el preprocesamiento (duplicados, faltantes, negativos, atípicos y formato).

---

## 4. Cadena de Preguntas

* **Pregunta de Negocio:**  
  ¿Cuántas unidades de cada refacción debo pedir al distribuidor al inicio de la semana para cubrir la demanda sin quedarme sin stock en fin de semana ni en quincena?
* **Pregunta Analítica:**  
  ¿Cuántas unidades se venderán por día durante los próximos 7 a 15 días para cada producto del catálogo?
* **Variable a Predecir:**  
  `unidades` diarias vendidas por `producto` (numérica discreta, medida en piezas/día, con horizonte de 7 a 30 días).

---

## 5. Inventario de Datos

* **Fuente:** Conjunto de datos de ventas diarias de refaccionaria (adaptado con el generador sintético del proyecto para simular el historial del punto de venta y compatible con la carga de archivos CSV reales del negocio).
* **Periodo:** Del `2025-01-01` al `2026-09-30` (21 meses continuos — 638 días).
* **Volumen:** **7,686 registros** (cumple el requisito de mínimo 12 meses de datos diarios y más de 5,000 registros).

### Columnas del conjunto de datos
* `fecha` (`date`): Fecha de registro de la venta (`YYYY-MM-DD`).
* `producto` (`string`): Nombre de la refacción o autoparte vendida.
* `unidades` (`int`): Cantidad de unidades vendidas en el día *(variable objetivo)*.
* `precio_unitario` (`float`): Precio unitario de venta en pesos mexicanos (MXN).
* `total` (`float`): Monto total vendido en el día por ese producto (`unidades * precio_unitario`).

---

## 6. Evidencias de Arranque Técnico y Uso de IA
* **Cuenta de nube y presupuesto $0:** Cuenta en AWS configurada con alerta de presupuesto de gasto cero (`Zero spend budget`), MFA activo en cuenta raíz y usuarios IAM individuales para cada integrante del equipo (`docs/img/alerta_presupuesto.png`).
* **Repositorio en GitHub:** Repositorio activo con ambos integrantes colaborando (`docs/img/github_equipo.png`).
* **Uso de IA Generativa:** Se utilizó **Gemini** como apoyo para estructurar la documentación en Markdown y la plantilla base de Streamlit con Plotly.
