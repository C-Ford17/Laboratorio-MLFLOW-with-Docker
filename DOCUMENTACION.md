# Documentación: App de Traducción con Docker

## Arquitectura
┌─────────────────────────────────────────────────────────┐
│ Docker Network │
├─────────────────────────────────────────────────────────┤
│ │
│ ┌──────────────────┐ ┌──────────────────────┐ │
│ │ traductor-app │ │ mlflow-server │ │
│ ├──────────────────┤ ├──────────────────────┤ │
│ │ Puerto: 7860 │───────→│ Puerto: 5000 │ │
│ │ Gradio UI │ │ MLflow Tracking UI │ │
│ └──────────────────┘ └──────────────────────┘ │
│ ↓ │
│ Volumen: /mlflow/mlruns │
│ │
└─────────────────────────────────────────────────────────┘


## Cómo Pasar la API Key

Al ejecutar el contenedor
docker run -d
-e GOOGLE_API_KEY=sk-xxxxxx
ford17/traductor-genai:1.0.0

**Nunca** incluir la clave en el Dockerfile o imagen.

## Comandos Principales

### Build

docker build -f Dockerfile.mlflow -t mlflow-server:latest .
docker build -t traductor-app:1.0.0 .

### Run (Sin docker-compose)

docker network create traductor-network
docker run -d --name mlflow-server --network traductor-network -p 5000:5000 -v mlflow_data:/mlflow/mlruns mlflow-server:latest
docker run -d --name traductor-app --network traductor-network -p 7860:7860 -e GOOGLE_API_KEY=xxx -e MLFLOW_TRACKING_URI=http://mlflow-server:5000 traductor-app:1.0.0

### Tag & Push

docker tag traductor-app:1.0.0 ford17/traductor-genai:1.0.0
docker push ford17/traductor-genai:1.0.0

### Pull & Run (Remoto)

docker pull ford17/traductor-genai:1.0.0
docker run -d -p 7860:7860 -e GOOGLE_API_KEY=xxx -e MLFLOW_TRACKING_URI=http://host-mlflow:5000 ford17/traductor-genai:1.0.0

## Observaciones

- **Latencia**: Promedio 1.3s por traducción (incluye llamada API Gemini)
- **Calidad**: Gemini 2.5-flash genera traducciones precisas
- **Artifacts**: Se guardan automáticamente en MLflow para auditoría

## Requisitos

- Docker 20.10+
- 2GB RAM mínimo
- Clave API Google Gemini

