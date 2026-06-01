#  Gestor Automatizado de Cobranzas con IA (v3.0)

Script profesional y resiliente diseñado para estudios contables en Argentina.
Automatiza la clasificación de deudores y la generación de mensajes personalizados de WhatsApp utilizando **Gemini 2.0 Flash**, bajo una arquitectura defensiva y de privacidad estricta.

##  Características Principales (Production-Ready)

- **Privacidad Absoluta (Compliance Ley 25.326):** Los nombres reales de los deudores nunca se envían a las APIs de Google.
- Se aplica una anonimización real mediante enmascaramiento local con placeholders (`{nombre}`) que se reemplazan post-procesamiento.
  
- **Validación Anti-Alucinación:** El sistema analiza mediante expresiones regulares los montos monetarios generados por la IA.
- Si el mensaje incluye un monto que difiere en más de un 50% de la deuda real del Excel, el mensaje se descarta automáticamente por seguridad financiera.
  
- **Locale Explícito:** Soporte nativo y estricto para formato de moneda argentino (`1.500,50`) vs anglosajón (`1,500.50`), evitando la corrupción silenciosa de datos financieros.
  
- **Log de Auditoría Inmutable:** Implementación de un `AuditLog` independiente en formato JSON Lines que registra de forma inmutable el operador del sistema, host, timestamps y hashes SHA256 de los archivos procesados. Utiliza persistencia física por hardware (`fsync`).
  
- **Resiliencia ante Fallos de Red:** Manejo de cuotas de API mediante reintentos automáticos configurados con *Backoff Exponencial con Jitter*.
  
- **Cifrado en Reposo:** Integración opcional con el algoritmo Fernet (AES-128) para encriptar reportes locales y proteger datos sensibles de clientes.

##  Instalación y Uso

1. Cloná el repositorio:
   ```bash
   git clone [https://github.com/davidstocco2024-cell/gestor_cobranzas.git](https://github.com/davidstocco2024-cell/gestor_cobranzas.git)
   cd gestor_cobranzas
