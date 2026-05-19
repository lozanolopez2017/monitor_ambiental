# ESTACIÓN METEOROLÓGICA ESP32 + MONITOR DE LUZ (v36)

## DESCRIPCIÓN GENERAL
Este proyecto convierte una **ESP32** en un monitor ambiental para invernadero con:

- Medición de luz (LDR en porcentaje 0–100%)
- Medición de temperatura y humedad (AHT20)
- Visualización local en pantalla OLED SSD1306
- Configuración y control remoto por Telegram
- Configuración local por Bluetooth
- Sistema de alarmas (luz, temperatura, humedad)
- Alerta de posible incendio (prioritaria)

El objetivo es supervisar rápidamente el estado del entorno y recibir avisos útiles sin necesidad de estar físicamente junto al dispositivo.

---

## QUÉ MIDE ESTE DISPOSITIVO
- **Luz:** porcentaje calculado a partir de LDR (0% oscuridad – 100% iluminación máxima)
- **Temperatura:** grados °C (sensor AHT20)
- **Humedad:** porcentaje relativo (sensor AHT20)

---

## PANTALLAS OLED DE 128×64 PX
El equipo tiene **4 modos de visualización**:

| Modo | Descripción |
|------|-------------|
| 0 | Indicador de luz |
| 1 | Indicador de temperatura |
| 2 | Indicador de humedad |
| 3 | Dashboard con luz/temperatura/humedad |

Los indicadores usan animación y dibujo optimizado (arco, marcas y aguja) para una lectura visual rápida.

---

## CÓMO FUNCIONA A NIVEL GENERAL

### 1) Al arrancar el dispositivo
- Inicia **SPIFFS**
- Carga y valida `config.txt`
- Inicializa pantalla y sensor AHT20
- Muestra imagen de inicio
- Intenta conectar al WiFi (3 intentos)

### 2) Si no logra conectar WiFi
- Muestra imagen de *“no wifi”*
- Activa modo **Bluetooth** para configuración

### 3) En funcionamiento normal
- Lee sensores periódicamente
- Actualiza pantalla según modo seleccionado
- Revisa alarmas y envía avisos por Telegram
- Atiende comandos del bot

---

## CONTROL POR TELEGRAM
El bot incluye un menú con botones para:

- Ver datos actuales  
- Configurar alarmas de luz, temperatura y humedad  
- Activar/desactivar notificaciones  
- Encender/apagar pantalla OLED  
- Elegir pantalla visible en el dispositivo  

El dispositivo solo acepta el **chat ID configurado** en `config.txt` para evitar accesos no autorizados.

---

## ALARMAS DISPONIBLES

### 1) Luz mínima
- Se dispara cuando la luz cae por debajo del umbral configurado.
- El mensaje distingue entre:
  - “Todas las lámparas apagadas” si luz ≤ 10%
  - “¿Una o varias lámparas fundidas?” si luz > 10%
- Incluye espera de estabilización para evitar falsos avisos.

### 2) Temperatura mínima y máxima
- Aviso por debajo del mínimo o por encima del máximo.
- Recuperación con **histeresis** para evitar repetición de mensajes.

### 3) Humedad mínima y máxima
- Aviso por debajo del mínimo o por encima del máximo.
- Recuperación con **histeresis**.

---

## ALERTA DE POSIBLE INCENDIO (PRIORITARIA)
Esta alerta es **independiente** de las notificaciones normales.

- **Disparo:** temperatura ≥ 60 °C  
- **Mensaje:** “🔥 ALERTA POSIBLE INCENDIO 🔥”  
- **Recuperación:**
  - Si `tempMaxAlarm` está configurada → al bajar por debajo de ese valor  
  - Si `tempMaxAlarm = 0` → usa fallback de **45 °C**

> Esta alerta sigue activa incluso si las notificaciones generales están desactivadas.

---

## CONFIGURACIÓN POR BLUETOOTH
En modo BT (por fallo WiFi o pulsación larga de BOOT) se puede:

- Leer y modificar parámetros de `config.txt`
- Calibrar LDR
- Ajustar red WiFi y Telegram
- Definir pantalla inicial
- Borrar y regenerar `config.txt`

---

## ARCHIVO `config.txt`
Claves principales:

- wifi_ssid
- wifi_password
- telegram_token
- telegram_chat_id
- default_screen
- max_luz
- umbral_luz_min
- temp_min_alarm
- temp_max_alarm
- hum_min_alarm
- hum_max_alarm
- notificaciones


---

## CONEXIONES DE HARDWARE

### I2C (OLED + AHT20)
- SDA → GPIO 21  
- SCL → GPIO 22  

### OLED SSD1306
- Dirección I2C: **0x3C**

### LDR
- Entrada analógica en **GPIO 33**

### LED integrado
- GPIO 2

---

## RESUMEN RÁPIDO
Este dispositivo permite monitorizar **luz, temperatura y humedad** en un invernadero, visualizar los datos en una pantalla OLED y recibir alertas por Telegram. Incluye configuración por Bluetooth y una alerta prioritaria de posible incendio para reaccionar rápidamente ante situaciones críticas.
