# Lösung Challenge 06 - Horizontal Pod Autoscaling

In diesem Lab behelfen wir uns ein bisschen mit der [Demo von Kubernetes](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## 1. Apache Deployment erstellen

- Datei `deployment-apache.yaml` erstellen und den folgenden Inhalt einfügen:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: php-apache
    spec:
      selector:
        matchLabels:
          run: php-apache
      replicas: 1
      template:
        metadata:
          labels:
            run: php-apache
        spec:
          containers:
          - name: php-apache
            image: k8s.gcr.io/hpa-example
            ports:
            - containerPort: 80
            resources:
              limits:
                cpu: 500m
              requests:
                cpu: 200m
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: php-apache
      labels:
        run: php-apache
    spec:
      ports:
      - port: 80
      selector:
        run: php-apache
    ```

- Datei in Cluster bereitstellen

    Bash/PowerShell:
    ```bash
    kubectl apply -f deployment-apache.yaml
    ```

## 2. Horizontal Pod Autoscaler erstellen

- Horizontal Pod Autoscaler erstellen

    Bash/PowerShell:
    ```bash
    kubectl autoscale deployment php-apache --cpu-percent=20 --min=1 --max=10
    ```

- Prüfen

    Bash/PowerShell:
    ```bash
    kubectl get hpa
    ```

## 3. Last hinzufügen

- Öffne ein neues Terminal
- Erstelle einen Pod der das Deployment befeuert

    Bash/PowerShell:
    ```bash
    kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
    ```

- Sieh dir an was passiert

    Bash/PowerShell:
    ```bash
    watch kubectl get hpa,deployment
    ```

## 4. Last wegnehmen

- Beende den Pod `load-generator` und schau was passiert
