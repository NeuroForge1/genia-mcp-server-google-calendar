# Servidor MCP para Google Calendar integrado con GENIA

Este repositorio contiene la implementación del servidor MCP (Model Context Protocol) para Google Calendar, diseñado para integrarse con el sistema GENIA como un orquestador central de herramientas externas.

## Descripción

El servidor MCP de Google Calendar permite a GENIA interactuar con la API de Google Calendar, permitiendo a los usuarios:

- Crear, modificar y eliminar eventos
- Consultar calendarios y eventos
- Gestionar invitaciones y recordatorios
- Sincronizar eventos entre diferentes calendarios

Esta implementación sigue el protocolo MCP, permitiendo una integración fluida con el orquestador central de GENIA.

## Requisitos

- Python 3.13 o superior
- UV Package Manager
- Credenciales OAuth de Google Cloud
- Supabase para almacenamiento seguro de tokens

Para más detalles sobre los requisitos y limitaciones, consulte el archivo [requisitos_google_calendar_mcp.md](requisitos_google_calendar_mcp.md).

## Estructura del Repositorio

- `mcp_orchestrator_google_calendar.py`: Orquestador para gestionar el ciclo de vida del servidor MCP
- `mcp_client_google_calendar.py`: Cliente para interactuar con el servidor MCP
- `validate_google_calendar_mcp.py`: Script para validar la instalación y funcionamiento
- `supabase_service.py`: Servicio para gestión segura de tokens en Supabase
- `Google_Calendar_MCP/`: Código fuente del servidor MCP

## Instalación

1. Asegúrese de tener Python 3.13+ instalado
2. Instale UV Package Manager
3. Clone este repositorio
4. Configure las variables de entorno necesarias
5. Ejecute el script de validación para verificar la instalación

```bash
# Instalar UV Package Manager
curl -sSf https://install.uv.dev | python3 -

# Clonar repositorio
git clone https://github.com/NeuroForge1/genia-mcp-server-google-calendar.git
cd genia-mcp-server-google-calendar

# Instalar dependencias
uv pip install -r requirements.txt

# Ejecutar validación
python validate_google_calendar_mcp.py
```

## Integración con GENIA

Para integrar este servidor MCP con GENIA:

1. Configure las variables de entorno en el archivo `.env` de GENIA
2. Añada la configuración del servidor MCP en el archivo de configuración de GENIA
3. Reinicie el servicio de GENIA

## Uso

El servidor MCP de Google Calendar se puede utilizar a través del orquestador central de GENIA:

```python
from app.mcp_client.mcp_client_google_calendar import get_google_calendar_client

# Obtener cliente
client = get_google_calendar_client(user_id="user123")

# Crear evento
event = {
    "summary": "Reunión de equipo",
    "location": "Sala de conferencias",
    "description": "Discusión de proyectos",
    "start": {
        "dateTime": "2025-05-20T10:00:00",
        "timeZone": "America/Los_Angeles",
    },
    "end": {
        "dateTime": "2025-05-20T11:00:00",
        "timeZone": "America/Los_Angeles",
    },
}
result = client.create_event(calendar_id="primary", event=event)
```

## Despliegue en Render

Para desplegar este servidor MCP en Render:

1. Cree un nuevo servicio web en Render
2. Conecte este repositorio
3. Configure las variables de entorno necesarias
4. Seleccione Python 3.13 como entorno de ejecución
5. Configure el comando de inicio: `uv run calendar_mcp.py`

## Licencia

Este proyecto está licenciado bajo los términos de la licencia MIT.

## Contribuciones

Las contribuciones son bienvenidas. Por favor, abra un issue o pull request para sugerencias o mejoras.
