# Azure Python Service (CI)

Dieses Repository verwaltet den Quellcode für eine Flask-Anwendung, die auf Azure Kubernetes Service (AKS) läuft, sowie die GitHub Actions für den automatischen Build und Deployment.

## Struktur
- **backend/**: Hauptanwendung der Flask-App
  - `main.py`: Anwendungscode (Port: 8000)
  - `Dockerfile`: Definition für den Build des Azure Container Registry (ACR) Images
  - `requirements.txt`: Abhängigkeiten (z.B. Flask)
- **.github/workflows/**: CI/CD Pipeline
  - `ci-cd.yml`: Automatisiertes Docker-Build, Push zum ACR und Aktualisierung des Manifest-Repositorys

## Erweiterte CI/CD Details (Multi-Environment)

Die Pipeline wurde so erweitert, dass sie nicht nur das Image baut, sondern auch den gesamten GitOps-Zyklus für mehrere Umgebungen steuert.

### Automatisierungsschritte in GitHub Actions
- **Docker Build & Tagging**: Das Image wird mit dem spezifischen **GitHub Commit SHA** (`${{ github.sha }}`) getaggt. Dies stellt eine eindeutige Zuordnung zwischen Code und Image sicher.
- **Cross-Repository Update**: Die Action klont das separate `gitops-manifests`-Repository und navigiert präzise in die Verzeichnisse der Overlays.
- **Kustomize Integration**: Mittels `kustomize edit set image` werden die Image-Tags in folgenden Umgebungen gleichzeitig aktualisiert:
    - `./apps/backend/overlays/staging`
    - `./apps/backend/overlays/production`
- **Automated Commit**: Nach der Aktualisierung committet die Action die Änderungen im Namen von `github-actions[bot]` und pusht sie zurück in das Manifest-Repository, was den Sync in **Argo CD** auslöst.

### Sicherheit & Integration
- **Azure Login**: Die Pipeline nutzt `azure/login@v1` mit hinterlegten **Service Principal Secrets**, um sicher auf die Azure-Ressourcen zuzugreifen.
- **ACR Authentication**: Vor dem Push erfolgt ein Docker-Login direkt über die Azure CLI (`az acr login`), um die Integrität der Container-Registry zu gewährleisten.

## Voraussetzungen für den Workflow
Damit die Pipeline erfolgreich läuft, müssen folgende GitHub Secrets konfiguriert sein:
- `AZURE_CREDENTIALS`: Zugangsdaten für den Azure Service Principal.
- `MANIFEST_REPO_PAT`: Ein Personal Access Token mit Schreibzugriff auf das `gitops-manifests`-Repository.

## CI/CD Workflow
1. Ein Push in den `main`-Branch löst die GitHub Actions aus.
2. Das Docker-Image wird erstellt und in die Azure Container Registry (ACR) gepusht.
3. Der Image-Tag in der `deployment.yaml` im `gitops-manifests`-Repository wird automatisch auf die neueste Version aktualisiert.