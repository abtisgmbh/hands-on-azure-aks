# Challenge 05 - Monitoring und Logging

Monitoring und Logging sind wichtige Bestandteile einer produktiven Applikationsumgebung. Es ist das Debugging für Prod.

Azure Monitor bietet die Möglichkeit alle Logs einer Applikation, die in einem AKS Cluster läuft, einzusehen. Das Feature Container Insights verbindet den AKS Cluster mit einem Log Analaytics Workspace.

https://docs.microsoft.com/de-de/azure/azure-monitor/containers/container-insights-overview


Aufgabe:

- Schau dir die Logs der *LottoWeb* Pods an
- Schau dir die Umgebungsvariablen der *LottoWeb* Pods an
- Prüfe die Auslastung der Nodes
- Erstelle Resource Quotas für die *LottoWeb* Pods
  - Requests:
      - CPU: 100m
      - Memory: 64Mi
  - Limits
      - CPU: 250m
      - Memory: 256Mi
- Stelle das Kubernetes Dashboard bereit und greife darauf zu
- Verbinde den Kubernetes Cluster mit einem Log Analytics Workspace

[[Hint: Monitoring für AKS Lösungen](hints/configure-monitoring.md)]
