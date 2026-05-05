# Azure Python Service (CI)

Dieses Repository verwaltet den Quellcode für eine Flask-Anwendung, die auf Azure Kubernetes Service (AKS) läuft, sowie die GitHub Actions für den automatischen Build und Deployment.

## Struktur
- **backend/**: Hauptanwendung der Flask-App
  - `main.py`: Anwendungscode (Port: 8000)
  - `Dockerfile`: Definition für den Build des Azure Container Registry (ACR) Images
  - `requirements.txt`: Abhängigkeiten (z.B. Flask)
- **.github/workflows/**: CI/CD Pipeline
  - `deploy.yml`: Automatisiertes Docker-Build, Push zum ACR und Aktualisierung des Manifest-Repositorys

## CI/CD Workflow
1. Ein Push in den `main`-Branch löst die GitHub Actions aus.
2. Das Docker-Image wird erstellt und in die Azure Container Registry (ACR) gepusht.
3. Der Image-Tag in der `deployment.yaml` im `gitops-manifests`-Repository wird automatisch auf die neueste Version aktualisiert.