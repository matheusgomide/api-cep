# Guia de Deploy da API-CEP no Kubernetes

## 📋 Pré-requisitos

1. **Docker** instalado para build da imagem
2. **kubectl** configurado e conectado ao seu cluster
3. **Acesso a um Container Registry** (Docker Hub, GCR, ACR, etc.)

## 🚀 Passo a Passo para Deploy

### 1. Build da Imagem Docker

```bash
# Build da imagem
docker build -t api-cep:latest .

# Tag para o seu registry (exemplo com Docker Hub)
docker tag api-cep:latest iesodias/api-cep:latest

# Push para o registry
docker push iesodias/api-cep:latest
```

### 2. Atualizar o Manifesto

Edite o arquivo `k8s-deployment.yaml` e substitua a imagem:

```yaml
image: iesodias/api-cep:latest  # <- Trocar por: seu-usuario/api-cep:latest
```

### 3. Deploy no Kubernetes

```bash
# Aplicar os manifestos
kubectl apply -f k8s-deployment.yaml

# Verificar o status do deployment
kubectl get deployments

# Verificar os pods
kubectl get pods -l app=api-cep

# Verificar o service
kubectl get service api-cep-service
```

### 4. Testar a Aplicação

```bash
# Port-forward para testar localmente
kubectl port-forward service/api-cep-service 8080:80

# Em outro terminal, testar a API
curl http://localhost:8080/cep/01310100
```

### 5. Expor Externamente (Opcional)

Para expor a aplicação externamente, você pode:

**Opção A: Usando LoadBalancer**
```bash
# Editar o service para tipo LoadBalancer
kubectl patch service api-cep-service -p '{"spec":{"type":"LoadBalancer"}}'
```

**Opção B: Usando Ingress**

Crie um arquivo `k8s-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-cep-ingress
  labels:
    backstage.io/kubernetes-id: api-cep
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: api-cep.exemplo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-cep-service
            port:
              number: 80
```

Aplique o Ingress:
```bash
kubectl apply -f k8s-ingress.yaml
```

## 🔍 Comandos Úteis

```bash
# Ver logs dos pods
kubectl logs -l app=api-cep --tail=100 -f

# Escalar replicas
kubectl scale deployment api-cep-deployment --replicas=3

# Ver detalhes do deployment
kubectl describe deployment api-cep-deployment

# Ver eventos
kubectl get events --sort-by='.lastTimestamp'

# Deletar recursos
kubectl delete -f k8s-deployment.yaml
```

## 🏷️ Integração com Backstage

O `catalog-info.yaml` já está configurado com a annotation:
```yaml
backstage.io/kubernetes-id: api-cep
```

E os manifestos K8s incluem as labels necessárias:
```yaml
labels:
  backstage.io/kubernetes-id: api-cep
```

Isso permite que o Backstage visualize automaticamente os recursos do Kubernetes relacionados à sua aplicação.

## 🔧 Customizações

### Ajustar Recursos

Edite os limites de CPU e memória em `k8s-deployment.yaml`:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

### Adicionar Variáveis de Ambiente

```yaml
env:
- name: LOG_LEVEL
  value: "info"
- name: VIACEP_TIMEOUT
  value: "10"
```

### Usar ConfigMap ou Secrets

```bash
# Criar um ConfigMap
kubectl create configmap api-cep-config --from-literal=log_level=info

# Criar um Secret
kubectl create secret generic api-cep-secret --from-literal=api_key=sua-chave
```

Referencie no Deployment:
```yaml
envFrom:
- configMapRef:
    name: api-cep-config
- secretRef:
    name: api-cep-secret
```

## 📊 Monitoramento

### Health Checks

A aplicação já está configurada com:
- **Liveness Probe**: Verifica se o container está funcionando
- **Readiness Probe**: Verifica se o container está pronto para receber tráfego

### Logs

```bash
# Seguir logs em tempo real
kubectl logs -l app=api-cep -f

# Ver logs de um pod específico
kubectl logs <nome-do-pod>
```

## 🐛 Troubleshooting

### Pod não inicia

```bash
kubectl describe pod <nome-do-pod>
kubectl logs <nome-do-pod>
```

### Problema com imagem

```bash
# Verificar se a imagem existe
docker pull seu-usuario/api-cep:latest

# Criar um ImagePullSecret se necessário
kubectl create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<usuario> \
  --docker-password=<senha>
```

### Service não responde

```bash
# Testar conectividade dentro do cluster
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- \
  curl http://api-cep-service/cep/01310100
```