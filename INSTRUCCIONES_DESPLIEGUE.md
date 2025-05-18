# Instrucciones de Despliegue del Servidor MCP de Google Calendar

Este documento proporciona instrucciones detalladas para desplegar el servidor MCP de Google Calendar en un entorno de producción, específicamente en Render, y para integrarlo con el sistema GENIA.

## Requisitos Previos

Antes de comenzar el despliegue, asegúrese de tener:

1. **Cuenta en Render** con acceso para crear servicios web
2. **Proyecto en Google Cloud** con la API de Google Calendar habilitada
3. **Credenciales OAuth** configuradas para la API de Google Calendar
4. **Instancia de Supabase** configurada para almacenar tokens de usuario
5. **Variables de entorno** necesarias para la configuración

## Preparación del Entorno

### 1. Configuración de Supabase

1. Cree una tabla `user_tokens` en su base de datos Supabase con la siguiente estructura:

```sql
CREATE TABLE user_tokens (
    id uuid PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id text NOT NULL,
    service text NOT NULL,
    tokens jsonb NOT NULL,
    created_at timestamp with time zone DEFAULT now(),
    updated_at timestamp with time zone DEFAULT now()
);

CREATE INDEX idx_user_tokens_user_id ON user_tokens(user_id);
CREATE INDEX idx_user_tokens_service ON user_tokens(service);
CREATE UNIQUE INDEX idx_user_tokens_user_service ON user_tokens(user_id, service);
```

2. Obtenga la URL y la clave de API de Supabase desde la configuración de su proyecto.

### 2. Configuración de Google Cloud

1. Cree un proyecto en Google Cloud Console
2. Habilite la API de Google Calendar
3. Configure credenciales OAuth para aplicación de escritorio
4. Descargue el archivo `credentials.json`

## Despliegue en Render

### 1. Crear un Nuevo Servicio Web

1. Inicie sesión en su cuenta de Render
2. Haga clic en "New" y seleccione "Web Service"
3. Conecte su repositorio de GitHub (NeuroForge1/genia-mcp-server-google-calendar)
4. Configure el servicio con los siguientes parámetros:

   - **Name**: `genia-mcp-server-google-calendar`
   - **Environment**: `Python 3`
   - **Build Command**: `uv pip install -r requirements.txt`
   - **Start Command**: `python calendar_mcp.py`

### 2. Configurar Variables de Entorno

Configure las siguientes variables de entorno en Render:

```
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-supabase-key
SUPABASE_JWT_SECRET=your-jwt-secret
GOOGLE_CREDENTIALS_JSON={"web":{"client_id":"...","project_id":"...","auth_uri":"...","token_uri":"...","auth_provider_x509_cert_url":"...","client_secret":"..."}}
MCP_SERVER_PORT=8080
```

Nota: Para `GOOGLE_CREDENTIALS_JSON`, copie todo el contenido del archivo `credentials.json` y escápelo adecuadamente para JSON.

### 3. Configurar Opciones Avanzadas

1. En la sección "Advanced", configure:
   - **Auto-Deploy**: Enabled
   - **Branch**: master
   - **Health Check Path**: /health

2. Habilite la opción "Python 3.13" en la sección de entorno.

### 4. Desplegar el Servicio

1. Haga clic en "Create Web Service"
2. Espere a que el servicio se despliegue y esté en estado "Live"
3. Anote la URL del servicio (ej. `https://genia-mcp-server-google-calendar.onrender.com`)

## Integración con GENIA

### 1. Configurar el Cliente MCP en GENIA

1. Asegúrese de que los archivos `mcp_orchestrator_google_calendar.py` y `mcp_client_google_calendar.py` estén en el directorio `app/mcp_client/` de GENIA.

2. Asegúrese de que el archivo `supabase_service.py` esté en el directorio `app/services/` de GENIA.

3. Configure las variables de entorno en el archivo `.env` de GENIA:

```
GOOGLE_CALENDAR_MCP_URL=https://genia-mcp-server-google-calendar.onrender.com
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-supabase-key
SUPABASE_JWT_SECRET=your-jwt-secret
```

### 2. Actualizar el Orquestador Central

Modifique el archivo `mcp_orchestrator_extended.py` para incluir el servidor MCP de Google Calendar:

```python
# En la función register_servers()
def register_servers(self):
    # Otros servidores MCP
    # ...
    
    # Registrar servidor MCP de Google Calendar
    from app.mcp_client.mcp_orchestrator_google_calendar import GoogleCalendarMCPOrchestrator
    google_calendar_orchestrator = GoogleCalendarMCPOrchestrator()
    self.register_server("google_calendar", google_calendar_orchestrator)
```

### 3. Actualizar las Rutas API

Modifique el archivo `mcp_routes.py` para incluir endpoints para Google Calendar:

```python
# Importar cliente de Google Calendar
from app.mcp_client.mcp_client_google_calendar import get_google_calendar_client

# Añadir rutas para Google Calendar
@router.post("/mcp/google_calendar/connect")
async def connect_google_calendar(request: Request):
    # Implementación de conexión
    # ...

@router.get("/mcp/google_calendar/events")
async def get_calendar_events(request: Request):
    # Implementación para obtener eventos
    # ...

@router.post("/mcp/google_calendar/events")
async def create_calendar_event(request: Request):
    # Implementación para crear eventos
    # ...
```

## Verificación del Despliegue

### 1. Verificar el Servidor MCP

1. Acceda a la URL del servidor MCP en Render
2. Verifique que el endpoint `/health` devuelva un estado 200
3. Verifique los logs en Render para asegurarse de que no hay errores

### 2. Verificar la Integración con GENIA

1. Reinicie el servicio de GENIA
2. Acceda al panel de administración de GENIA
3. Verifique que Google Calendar aparezca como una herramienta disponible
4. Intente conectar una cuenta de Google Calendar
5. Verifique que se puedan crear y consultar eventos

## Solución de Problemas

### Error: "Python 3.13+ required"

Asegúrese de que Render esté configurado para usar Python 3.13 o superior. Si Render no ofrece esta versión, puede usar un Dockerfile personalizado:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

RUN curl -sSf https://install.uv.dev | python3 - && \
    uv --version

COPY . .

RUN uv pip install -r requirements.txt

CMD ["python", "calendar_mcp.py"]
```

### Error: "UV Package Manager not found"

Instale UV Package Manager en el script de construcción:

```bash
curl -sSf https://install.uv.dev | python3 -
uv pip install -r requirements.txt
```

### Error: "Supabase connection failed"

Verifique que las variables de entorno `SUPABASE_URL` y `SUPABASE_KEY` estén correctamente configuradas y que la tabla `user_tokens` exista en su base de datos.

## Mantenimiento

### Actualizaciones

Para actualizar el servidor MCP de Google Calendar:

1. Realice cambios en el repositorio
2. Haga commit y push a la rama master
3. Render desplegará automáticamente los cambios

### Monitoreo

Configure alertas en Render para monitorear:

1. Uso de CPU y memoria
2. Tiempo de respuesta
3. Errores 5xx

## Seguridad

Para mejorar la seguridad:

1. Rote regularmente las claves de API de Supabase
2. Utilice variables de entorno para todas las credenciales
3. Configure CORS adecuadamente en el servidor MCP
4. Implemente rate limiting para prevenir abusos

## Contacto y Soporte

Para soporte técnico o consultas, contacte al equipo de GENIA a través de:

- GitHub: [NeuroForge1/genia-mcp-server-google-calendar](https://github.com/NeuroForge1/genia-mcp-server-google-calendar)
- Email: soporte@genia.systems
