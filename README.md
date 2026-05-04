# PRAXA CONTROL — Backend Cloud
## Fase 1: FastAPI + WebSocket + JWT

### Estructura
```
backend/
├── main.py           ← servidor principal
├── requirements.txt  ← dependencias
├── render.yaml       ← config de deploy
└── README.md
```

### Desarrollo local
```bash
# 1. Instalar dependencias
pip install -r requirements.txt

# 2. Variables de entorno (crear archivo .env o exportar)
export SECRET_KEY=praxa_dev_secret
export SCADA_KEY=praxa_dev_key
export EMPRESA_ID=empresa_demo

# 3. Correr servidor
uvicorn main:app --reload --port 8000

# Docs disponibles en: http://localhost:8000/docs
# Health check en:     http://localhost:8000/health
```

### Deploy en Render (paso a paso)

1. Subir esta carpeta `backend/` a un repositorio GitHub
2. Ir a https://render.com → New → Web Service
3. Conectar el repositorio
4. Configurar:
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `uvicorn main:app --host 0.0.0.0 --port $PORT`
   - **Plan:** Starter ($7/mes) — IMPORTANTE: el plan Free duerme el servidor
5. Agregar variables de entorno en el dashboard de Render:
   - `SECRET_KEY` → generar uno largo y aleatorio
   - `SCADA_KEY`  → mismo valor que pondrás en scada_cloud.py
   - `EMPRESA_ID` → ej: `manufactura_tj`
   - `ENV`        → `production`
6. Deploy → copiar la URL pública (ej: https://praxa-tj.onrender.com)

### Conectar el SCADA Python

En `scada_cloud.py`, actualizar las variables:
```python
CLOUD_URL = "wss://praxa-tj.onrender.com"   # tu URL de Render (wss, no ws)
SCADA_KEY = "tu_scada_key_de_render"
```

O mejor: usar variables de entorno en la máquina de planta:
```bash
export SCADA_CLOUD_URL=wss://praxa-tj.onrender.com
export SCADA_KEY=tu_scada_key_de_render
```

### Endpoints disponibles

| Método | Endpoint         | Auth  | Descripción                    |
|--------|-----------------|-------|-------------------------------|
| GET    | /health          | No    | Estado del servidor            |
| POST   | /api/login       | No    | Login → retorna JWT token      |
| GET    | /api/estado      | JWT   | Estado actual del SCADA        |
| GET    | /api/log         | JWT   | Log de eventos                 |
| GET    | /api/inventario  | JWT   | Inventario actual              |
| GET    | /api/metricas    | JWT   | Histórico de KPIs              |
| POST   | /api/emergencia  | JWT   | Activar emergencia en planta   |
| POST   | /api/reanudar    | JWT   | Reanudar operación             |
| WS     | /ws/scada        | SCADA_KEY | Conexión del SCADA Python  |
| WS     | /ws/client       | JWT   | Conexión del frontend web      |

### Flujo de datos
```
simulador.py → scada_cloud.py → WS /ws/scada → main.py → WS /ws/client → frontend React
```
