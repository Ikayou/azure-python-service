
## 🛠 Erweiterte CI/CD Details (Multi-Environment)

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

##  Voraussetzungen für den Workflow
Damit die Pipeline erfolgreich läuft, müssen folgende GitHub Secrets konfiguriert sein:
- `AZURE_CREDENTIALS`: Zugangsdaten für den Azure Service Principal.
- `MANIFEST_REPO_PAT`: Ein Personal Access Token mit Schreibzugriff auf das `gitops-manifests`-Repository.