# Preguntas principales para el redactor

Estas preguntas no pueden resolverse con lógica pura o conocimiento general; requieren acceso a los datos, al código o a decisiones de diseño que no están documentadas.

> **Nota:** La pregunta sobre trading days (191/268) se verificó con cálculo de calendario. 268 = días calendario, 191 = weekdays (lun-vie). Pero hay una discrepancia: el paper dice excluir feriados (lo he puesto yo con lógica) y 191 solo excluye findes. Ver detalles al final.

---

## Estado de resolución (redactor)

Todas las preguntas se verificaron contra el código y los datos reales. Resumen de dónde quedó cada respuesta en `paper.tex`:

| # | Tema | Estado | Dónde |
|---|------|--------|-------|
| 1 | Class balance 524/529/535 | Resuelto | §Target construction: HOLD fijado por el percentil 33; desviación por ventana 70% vs 65%, τ por activo y drift alcista. |
| 2 | Lista de las 19 features | Resuelto | Footnote en §Feature engineering (las 19 por categoría). |
| 3 | Tensorización 2 446 → 1 588/360/496 | Resuelto | §Temporal splits and tensorization: ventana trailing de 30 d con padding; 124/28/39 por acción, 174/40/53 por cripto. |
| 4 | 40/53 fechas únicas vs samples | Resuelto | Mismo párrafo: fechas las marca la grilla diaria de cripto; weekdays son subconjunto → ≈9 activos/fecha. |
| 5 | Feriados 191 vs 184 | Corregido | §Weekend consolidation: se quitan findes y precios faltantes; los feriados NO se eliminan aparte → 191 weekdays. |
| 6 | Figura del dead-zone | Sugerencia (opcional) | Pendiente; hay datos para generarla si se desea. |
| 7 | Regression head auxiliar | Resuelto | §Regression loss: multi-task, comparte z, solo en training, α=0.8 fijo (barrido 0.55–0.95). |
| 8 | Concatenación de splits | Resuelto | §Temporal splits: split por activo, samples independientes, sin fuga cross-asset. |
| 9 | Detalles por branch | Resuelto | §Branch details: capas, dims, decay de filings, bias de atención en log-space. |

---

## 1. Class balance 524/529/535 — ¿por qué no son exactamente 1/3?

**Contexto:** El paper usa percentil 33 por activo para generar labels (≈1/3 por clase). Los números 524/529/535 son aproximadamente 1/3 de 1,588, pero no exactos.

**Pregunta para el redactor:** ¿Estos números son la suma directa de los labels de los 12 activos en el set de entrenamiento? Si es así, ¿la desviación del 1/3 exacto se debe a que cada activo tiene un τ_a distinto y al concatenar las proporciones no se preservan? ¿O hay algún otro motivo (empates en el umbral, días sin datos, etc.)? Es decir, se necesita explicar cómo se hace el cálculo y con qué motivo.

---

## 2. 19 características escalares — ¿cuáles son exactamente?

**Contexto:** El paper menciona que de ~60 candidatos se seleccionaron 19 mediante Spearman, MI, LightGBM y VIF. Lista categorías (returns, volatilities, RSI, MACD, Bollinger, trend slopes, calendar dummies, news counts, filing-decay, cross-asset proxies) pero nunca enumera las 19 que sobrevivieron.

**Pregunta para el redactor:** ¿Puede proporcionar la lista explícita de las 19 features seleccionadas? Se sugiere incluirla como tabla en el apéndice o como footnote en la sección de Feature Engineering.

---

## 3. Tensorized samples — ¿cómo se pasa de 2,446 raw records a 1,588/360/496?

**Contexto:** La tabla dice Sequence length L = 30, lo que sugiere ventanas deslizantes. Pero nunca se explicita que cada sample es una ventana de 30 días ni cómo el solapamiento genera las cuentas exactas.

**Pregunta para el redactor:** ¿El proceso de tensorización es: para cada activo, se toma una ventana deslizante de 30 días (paso 1) sobre los días de trading disponibles, generando (n_días_activos - 29) samples por activo, y luego se concatenan los 12 activos? Si es así, ¿cuántos samples genera cada activo individualmente? ¿Hay padding cuando un activo no tiene suficientes días al inicio? Habría que explicar un poco más detalladamente la formación del dataset, quizá colocar un diagrama.

---

## 4. Unique trading dates 40/53 vs samples 360/496

**Contexto:** Si hay 360 samples de validación y solo 40 fechas únicas, implica ~9 activos por fecha en promedio (no 12). Similarmente, 496 test samples / 53 fechas ≈ 9.4 activos/fecha.

**Pregunta para el redactor:** ¿Es correcto que no todos los activos tienen datos en todas las fechas (por ejemplo, equities no tienen datos en fines de semana pero crypto sí, y las fechas únicas se cuentan sobre la unión)? ¿Por qué ~9 activos por fecha y no 12? ¿Hay días donde algunos activos no tienen datos por falta de news o por gaps en la serie de precios? Personalmente no comprendo esta parte o para qué se usa.

---

## 5. Trading days 191 / 268 — discrepancia feriados

**Verificado con cálculo exacto:**
- Rango 2025-08-01 a 2026-04-25 = **268 días calendario** ✓ (crypto)
- Weekdays (lun-vie) = **191** ✓ (coincide con el paper)
- Feriados NYSE estándar que caen en weekday = **7** (Labor Day, Thanksgiving, Christmas, New Year's, MLK, Presidents' Day, Good Friday)
- Trading days reales (weekdays - feriados) = **184**

**Discrepancia:** El paper dice "weekend and holiday records are removed" (yo lo he agregado porque no lo menciona específicamente) pero 191 solo excluye findes de semana, no feriados. Si se excluyeran feriados, el número sería 184, no 191. 🤔

**Preguntas para el redactor:**
1. ¿Se excluyeron feriados del dataset o solo findes de semana? Si se excluyeron, ¿por qué el número es 191 y no 184?
2. ¿Es posible que el dataset fuente (FinMMEval) ya no incluya feriados y el paper simplemente no los procesó adicionalmente?
3. ¿El número 191 se refiere a weekdays (lun-vie) sin verificar si hubo feriados, o se verificó contra el calendario de trading real?

---

## 6. Sugerencia: gráfica de los retornos para visualizar el dead-zone

**Contexto:** La explicación del percentil 33 y el dead-zone es matemática pero no intuitiva. Sería mucho más efectivo mostrar una figura con la distribución de los retornos absolutos (o los retornos crudos) para 2–3 activos representativos (e.g., MSFT de baja volatilidad y ETH de alta), con líneas verticales en $-\tau_a$ y $+\tau_a$ marcando el dead-zone. Así se ve de un vistazo:

- Que el threshold se adapta a cada activo
- Que ~1/3 de los días queda en cada región
- La diferencia de escala entre activos (MSFT: ±0.18%, ETH: ±0.86%)

**Pregunta para el redactor:** ¿Tienes acceso a los retornos para generar esta figura? Se podría incluir como panel (a) y (b) en una misma figura, o como figura separada en la sección de Target Construction.

---

## 7. Regression head auxiliar — ¿por qué y cómo?

**Contexto:** Al final de Target Construction se menciona:

> "an auxiliary regression target — the return normalized by a 20-day rolling realized volatility, lagged one day to avoid leakage — supervises a secondary head."

No se explica:
1. **Propósito:** ¿Por qué hay un head de regresión además del de clasificación? ¿Es multi-task learning, regularización, o se usa en inference?
2. **Definición:** ¿El target es $r_{t,a} / \sigma_{t-1,a}$ donde $\sigma_{t-1,a}$ es la volatilidad realizada de los 20 días anteriores? ¿O es forward-looking? ¿"Lagged one day" se refiere a la volatilidad (día $t-1$) o al return (día $t$ vs $t-1$)?
3. **Uso:** ¿Este head solo se entrena o también se usa en test? ¿Cómo se combina con el head de clasificación en el gated fusion? ¿Comparten representación o son completamente separados?
4. **Peso en la pérdida:** La ecuación $\mathcal{L} = \alpha\mathcal{L}_{\text{CE}} + (1-\alpha)\mathcal{L}_{\text{Huber}}$ — ¿ese Huber loss corresponde a este regression head? Si es así, ¿cómo se justifica $\alpha=0.8$? ¿Hubo algún barrido de este hiperparámetro?

**Pregunta para el redactor:** Agradeceríamos una explicación de la motivación y formulación exacta de este regression head, incluyendo su función de pérdida y cómo se integra con el clasificador.

---

## 8. Concatenación de splits por activo — ¿cómo se hace exactamente?

**Contexto:** En Temporal splits and tensorization se dice:

> "The per-asset splits are then concatenated, preserving chronological order and preventing look-ahead leakage."

No se explica:
1. **Mecanismo:** ¿Se concatenan los splits de cada activo uno tras otro (todo asset A train, luego asset B train, etc.) o se intercalan por fecha (todos los assets del día 1, luego día 2, etc.)?
2. **Orden cronológico:** Si cada activo tiene su propia línea temporal con splits 65/15/20, ¿cómo se preserva el orden cronológico entre activos diferentes? Un sample de AAPL del 2026-03-01 y uno de MSFT del 2025-09-01 estarían en el mismo split pero con fechas muy distintas.
3. **Look-ahead leakage:** ¿Cómo evita leakage exactamente? Si los splits son por activo, un batch podría contener samples de distintos activos en la misma fecha sin que eso constituya leakage (porque cada activo es independiente), pero ¿se asegura eso con la concatenación o hay un paso adicional?
4. **Estructura del dataloader:** ¿El modelo ve un batch con 12 assets distintos de la misma fecha, o un batch con secuencias de un solo asset? Esto afecta directamente el diseño de la gated fusion y las normalizaciones por lote.

**Pregunta para el redactor:** ¿Podría detallar el proceso de tensorización y concatenación? Un diagrama o pseudocódigo breve ayudaría mucho.

---

## 9. Detalles arquitectónicos de cada branch — faltan en Branches (§4.2)

**Contexto:** Se agregó una tabla en el párrafo Branches con input, operación y output de cada rama, pero varios detalles arquitectónicos no están documentados y no pueden inferirse.

**Preguntas para el redactor (sección Branches, §4.2):**

1. **Scalar MLP:** ¿Cuántas capas? ¿Dimensiones ocultas? ¿Función de activación? ¿Batch norm / dropout?
2. **PatchTST internal dimension:** ¿$d_{\text{model}}$? La tabla dice d=32 para la salida fusionada, pero ¿el transformer interno usa la misma dimensión o una más grande? ¿Cuántos heads de atención? ¿FFN interno?
3. **Attention pooling bias:** Se dice que los logits de atención reciben un bias aditivo del peso de decaimiento temporal ($w$). ¿Es $ \text{logits}_{i} = q \cdot k_i + w_i$ o $ \text{logits}_{i} = q \cdot k_i + \log(w_i)$? ¿O el peso se aplica como multiplicación directa sobre los scores? ¿Cómo se proyectan las keys (FinBERT emb) y el query (symbol emb ⊕ type flag) a una dimensión común para el producto punto?
4. **News attention output:** Después del pooling ponderado sobre las $k_t$ headlines, ¿se proyecta linealmente a $d=32$? ¿O FinBERT ya produce embeddings de dimensión 32?
5. **Filing projection:** Además del decay, ¿hay una capa lineal que proyecta el running embedding a $d=32$? ¿Cuál es la dimensión del embedding original de los filings?
6. **Running filing embedding:** ¿Cómo se inicializa? ¿Se actualiza con cada nuevo filing (10-K/10-Q) —sumando, promediando o reemplazando— o se recalcula desde cero? ¿Es un vector aprendido durante training o precomputado con otro modelo?
7. **45-day half-life decay:** ¿Decae el embedding completo multiplicativamente ($\mathcal{E}_t = \mathcal{E}_{t-1} \times 0.5^{\Delta/45}$)? ¿O se aplica componente a componente? ¿El decay se reinicia cuando llega un filing nuevo? 
