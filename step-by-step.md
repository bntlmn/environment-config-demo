Step by step procedure for production deployment

1. kubectl create namespace production

2. kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --namespace=production

3. kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=prodpassword123 \
  --namespace=production

4. kubectl get secrets -n production

5. kubectl describe secret app-secret -n production

6. kubectl get secret app-secret -n production -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode

7. vim app-deployment-template.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: nginx-app
    namespace: PLACEHOLDER_NAMESPACE
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: nginx-app
    template:
        metadata:
        labels:
            app: nginx-app
        spec:
        containers:
        - name: nginx
            image: nginx
            env:
            - name: APP_ENV
            valueFrom:
                configMapKeyRef:
                name: app-config
                key: APP_ENV
            - name: DB_PASSWORD
            valueFrom:
                secretKeyRef:
                name: app-secret
                key: DB_PASSWORD

8. sed -e 's/PLACEHOLDER_NAMESPACE/production/' \
    -e 's/replicas: .*$/replicas: 3/' app-deployment-template.yaml > production-deployment.yaml

    kubectl apply -f production-deployment.yaml

9. kubectl expose deployment nginx-app \
  --type=NodePort \
  --name=nginx-service \
  --port=80 \
  --target-port=80 \
  --namespace=production

10. kubectl get pods -n production
    kubectl get service -n production


    kubectl exec -it <pod-name> -n <namespace> -- /bin/bash
    echo $APP_ENV
    echo $DB_PASSWORD


