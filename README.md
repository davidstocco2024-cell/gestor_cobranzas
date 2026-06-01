# Gestor de Cobranzas IA

> Script profesional para estudios contables en Argentina. Clasifica deudores, genera mensajes personalizados de WhatsApp con Gemini 2.0 Flash y opera bajo una arquitectura defensiva con cumplimiento de la **Ley 25.326**.

---

## Características

| Feature | Detalle |
|---|---|
| 🔒 Privacidad real | Los nombres nunca viajan a Google — placeholder local `{nombre}` |
| 🇦🇷 Locale nativo | Soporte `ar` (`1.500,50`) y `en` (`1,500.50`) sin corrupción silenciosa |
| 🤖 IA con fallback | Gemini 2.0 Flash + mensajes estáticos por segmento si la API falla |
| 🔁 Retry automático | Backoff exponencial con jitter ante errores de red o cuota |
| ✅ Anti-alucinación | Valida que los montos generados por IA no difieran más del 50% del real |
| 📋 Audit log inmutable | JSON Lines con `fsync`, hashes SHA-256 y trazabilidad de operador |
| 🔐 Cifrado en reposo | Fernet (AES-128) opcional para reportes locales |
| 🖥️ CLI completo | `--locale`, `--pausa`, `--auto-confirm`, `--sin-cifrado`, `--no-audit` |

---

## Requisitos

- Python 3.8+
- Cuenta Google con acceso a [Gemini API](https://aistudio.google.com/app/apikey) (tier gratuito disponible)

---

## Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/gestor_cobranzas.git
cd gestor_cobranzas

# 2. Instalar dependencias
pip install -r requirements.txt
```

**requirements.txt**
```
pandas
openpyxl
python-dotenv
google-genai
cryptography
```

---

## Configuración

Crear un archivo `.env` en la raíz del proyecto:

```env
GEMINI_API_KEY=tu_clave_aqui
```

Obtener la clave en: https://aistudio.google.com/app/apikey

---

## Estructura del proyecto

```
gestor_cobranzas/
├── main.py                  # Script principal
├── deudores.xlsx            # Excel de entrada (ver formato abajo)
├── .env                     # API key (no subir a Git)
├── .encryption_key          # Clave Fernet generada automáticamente (no subir a Git)
├── .gitignore
├── requirements.txt
├── output/                  # Reportes Excel generados
├── mensajes/                # Archivos TXT listos para WhatsApp
├── audit/                   # Audit log JSONL (compliance)
└── cobranzas.log            # Log operacional
```

---

## Formato del Excel de entrada

El archivo `deudores.xlsx` debe contener exactamente estas columnas:

| nombre | monto | fecha_vencimiento |
|---|---|---|
| Juan Pérez | 15000 | 15/03/2025 |
| Ana García | 8500,50 | 01/02/2025 |

- **nombre**: texto libre
- **monto**: numérico o string con formato AR (`1.500,50`) o EN (`1,500.50`)
- **fecha_vencimiento**: formato `DD/MM/YYYY`

---

## Uso

### Ejecución básica

```bash
python main.py
```

### Opciones disponibles

```bash
python main.py --help
```

```
--archivo       Excel de entrada (default: deudores.xlsx)
--locale        Formato numérico: ar = 1.500,50 | en = 1,500.50 (default: ar)
--pausa         Segundos entre llamadas a Gemini (default: 4.0)
--auto-confirm  Saltea confirmación interactiva — útil para cron/CI
--sin-cifrado   Desactiva cifrado de archivos de salida (testing)
--no-audit      Desactiva el audit log
--audit-log     Ruta del audit log JSONL (default: audit/cobranzas.audit.jsonl)
```

### Ejemplos

```bash
# Uso estándar con Excel en formato argentino
python main.py --locale ar

# Excel con formato americano (importado de otro sistema)
python main.py --archivo clientes.xlsx --locale en

# Ejecución automática en servidor (sin confirmación, tier pago con menor pausa)
python main.py --auto-confirm --pausa 1.5

# Testing local sin cifrar ni auditar
python main.py --sin-cifrado --no-audit
```

---

## Segmentación de deudores

| Segmento | Criterio | Tono del mensaje |
|---|---|---|
| **Al día** | 0 días de mora | Agradecimiento |
| **Baja** | 1 – 15 días | Recordatorio amable |
| **Media** | 16 – 45 días | Urgencia moderada |
| **Alta** | Más de 45 días | Comunicación formal |

---

## Privacidad y compliance (Ley 25.326)

Este script fue diseñado para operar con datos sensibles de clientes:

- **Nombres nunca enviados a Google**: el prompt usa `{nombre}` como placeholder literal. Gemini genera el mensaje con el placeholder, que se reemplaza localmente con el nombre real antes de guardar. El nombre completo nunca sale del servidor.
- **Logs anonimizados**: los nombres se registran como hashes SHA-256 truncados (`d_a3f7c291...`). El archivo `cobranzas.log` no contiene PII.
- **Audit log con trazabilidad**: cada ejecución registra operador, host, PID, timestamp, hash SHA-256 del Excel de entrada y del reporte generado. Escritura `fsync` para durabilidad ante crashes.
- **Cifrado en reposo**: si `cryptography` está instalado, los reportes Excel y TXT se cifran con Fernet (AES-128). La clave se guarda en `.encryption_key` con permisos `600`.

> ⚠️ **Importante**: hacé backup del archivo `.encryption_key`. Sin él, los reportes cifrados anteriores son irrecuperables.

---

## Archivos generados

Cada ejecución genera tres archivos con timestamp para mantener historial:

```
output/reporte_cobranzas_20250526_1430.xlsx   # Reporte completo con segmento y mensaje
mensajes/mensajes_20250526_1430.txt            # Mensajes listos para copiar a WhatsApp
audit/cobranzas.audit.jsonl                   # Audit log acumulativo
```

---

## .gitignore recomendado

```gitignore
.env
.encryption_key
output/
mensajes/
audit/
cobranzas.log
__pycache__/
*.pyc
```

---

## Limitaciones conocidas

- Procesamiento secuencial: 30 deudores × 4 s = ~2 min en tier gratuito. Para volúmenes mayores, considerar tier pago con `--pausa 1` o implementar `ThreadPoolExecutor`.
- El tier gratuito de Gemini tiene límite de 15 RPM. Si el proceso falla por cuota, reducir `--pausa` está contraindicado; esperar 1 minuto y reintentar.
- Los archivos cifrados con Fernet solo se pueden leer con la misma `.encryption_key`. No hay recuperación sin backup de esa clave.

---

## Licencia

MIT — libre uso y modificación con atribución.
