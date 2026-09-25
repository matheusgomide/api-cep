
```yaml
# catalog-info.yaml (da sua aplicação existente)
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: minha-aplicacao
  annotations:
    # Annotation existente (GitHub Actions)
    github.com/project-slug: seu-usuario/seu-repo
    
    # NOVA annotation (Kubernetes)
    backstage.io/kubernetes-id: minha-aplicacao
spec:
  type: service
  owner: team-a
  lifecycle: production
```

## Passo 4: Criar/Atualizar Deployment no Kubernetes

Crie um arquivo `k8s-deployment.yaml` com um deployment simples:

```yaml
# k8s-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minha-aplicacao-deployment
  labels:
    # OBRIGATÓRIO: deve bater com backstage.io/kubernetes-id
    backstage.io/kubernetes-id: minha-aplicacao
spec:
  replicas: 2
  selector:
    matchLabels:
      app: minha-aplicacao
  template:
    metadata:
      labels:
        app: minha-aplicacao
        # OBRIGATÓRIO: deve bater com backstage.io/kubernetes-id  
        backstage.io/kubernetes-id: minha-aplicacao
    spec:
      containers:
      - name: app
        image: nginx:alpine  # Imagem simples para demo
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: minha-aplicacao-service
  labels:
    backstage.io/kubernetes-id: minha-aplicacao
spec:
  selector:
    app: minha-aplicacao
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```