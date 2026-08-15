# 🤖 BOT MENSUAL — Polymarket «Elon Musk # tweets in <mes> <año>?»

Bot **24/7 gratis** que vigila el mercado **mensual** de Polymarket sobre el
número de tweets de @elonmusk en el mes natural (p. ej. «Elon Musk # tweets
in August 2026?», del 1 al 31 de agosto 00:00 ET). Cada tipo de ventana
(48 h, semanal, mensual) tiene **su propia tabla de apuestas y su propia
metodología** en su propio repo. Este es el de la ventana **MENSUAL**.

---

## 🎯 Metodología propia de la ventana mensual

El mercado mensual cotiza en **bins de 20 en 20** (20-39, 40-59, …, 1000+)
con cuotas muy altas en el centro (7–20×). Las probabilidades reales del
modelo en esos bins son **25–50%** — la regla absoluta de 48 h (p ≥ 60%)
**nunca dispararía** aquí. Por eso la mensual usa:

| Parámetro | Valor |
|---|---|
| Stake inicial | **$2.00** |
| Factor de progresión | **×1.35** (V2: ×1.38) |
| Pasos máx. (stop-loss de ciclo) | **5** (pérdida máx. ciclo ≈ $11.72) |
| Cuota mínima | **≥ 3.00** |
| Regla de entrada | **VENTAJA**: p_modelo ≥ precio + **10pp** (y p ≥ 15%) |
| λ del mes | AVG7 × días del mes × ajuste |
| λ restante | AVG7 × ajuste × horas restantes / 24 |
| Bankroll de papel | $500 |

**¿Por qué ventaja y no p ≥ 60%?** Ejemplo real (ago-2026): bin 780-799 →
p_modelo 27% vs precio del mercado 11.7% (cuota 8.55). El modelo ve
+15.7pp de ventaja: si el modelo es bueno, esa es una apuesta EV+ aunque
la probabilidad individual sea baja. La progresión suave (2.00 × 1.35 × 5)
limita la pérdida por ciclo a ~$12, y las ganancias pagan 2–17×.

## 📦 Archivos del bot

| Archivo | Función |
|---|---|
| `bot_mensual.py` | Orquestador: recoge tweets → actualiza mercado → paper/real → Excel |
| `senal.py` | Motor Poisson + **tabla propia mensual** + regla de ventaja |
| `senal_vivo.py` | Señal en vivo con todos los mercados mensuales abiertos |
| `mercado_polymarket.py` | Descarga el mercado mensual (busca TODOS los slugs mes-año) |
| `papel_mensual.py` | Paper trading (sin dinero real) |
| `operar_real_mensual.py` | Operación real (requiere config_real.json + secret) |
| `notificar.py` | Avisos ntfy/Telegram (casi-señal incluida) |
| `probar_orden_mensual.py` | Prueba segura de firma de órdenes (5 shares @ 0.01) |
| `recoger_tweets.py` | Recoge tweets de @elonmusk (jina + xcancel) |
| `chequear_cuenta.py` | Ver saldo real en Polymarket (pUSD) |
| `saldo_ntfy.py` | Incluye saldo real en los avisos |
| `excel_historial.py` | Genera Historial_Operaciones_Mensual.xlsx |
| `.github/workflows/bot.yml` | Cron cada 15 min, 24/7 gratis |

## 🚀 Puesta en marcha

1. Crea el repo **público** `bot-polymarket-elon-mensual` (y
   `bot-polymarket-elon-mensual-v2` para la variante de factor 1.38).
2. Sube TODOS los archivos de esta carpeta (botón **Add file → Upload files**
   o con token temporal).
3. En **Settings → Secrets and variables → Actions**, añade los secrets:
   - `POLY_PRIVATE_KEY` → tu `wallet_private_key`
   - `POLY_WALLET_ADDRESS` → tu `wallet_address`
   - `REAL_CONFIRMADO` → déjalo vacío (0) hasta validar la firma
4. Ejecuta una pasada manual: **Actions → Bot Polymarket Elon (MENSUAL) →
   Run workflow** y comprueba que acaba en verde.
5. Cuando quieras operar real: ejecuta `probar_orden_mensual.py`, y si la
   orden de prueba sale bien, pon `"confirmado": true` en tu `config_real.json`
   y `REAL_CONFIRMADO=1` en el secret.

## 🔁 El ciclo de apuestas

1. El bot calcula AVG7 (7 días completos) y V2 (últimos 2 días).
2. λ_mes = AVG7 × días del mes × ajuste(momentum).
3. Para cada bin: p_modelo = Poisson(λ restante, teniendo en cuenta los
   tweets ya publicados en el mes).
4. Entra si **p_modelo ≥ precio + 10pp** y cuota ≥ 3.00 (YES), o lo
   contrario para NO.
5. Con TODOS los mercados mensuales disponibles elige el de **mayor EV**.
6. Si falla, sube un paso (×1.35). Si gana, reinicia. Máx. 5 pasos.
7. Avisa por ntfy al abrir/cerrar, casi-señales y resumen diario.
