# Hello World DevOps

Aplicação simples para o desafio DevOps Cloud Jr.  
Exibe **Hello World** na rota `/` e implementa **health check** na rota `/health`.

---

## Estrutura do Projeto
```
.
├── app.py                # Código da aplicação Flask
├── requirements.txt      # Dependências Python
├── Dockerfile            # Build da imagem Docker
└── k8s/
   ├── deployment.yaml    # Deployment Kubernetes (com probes)
   ├── service.yaml       # Service LoadBalancer
   └── hpa.yaml           # Horizontal Pod Autoscaler
```

---

## Como rodar localmente
```bash
git clone https://github.com/<SEU-USUARIO>/Technical-Challenge-Jr.-Cloud-DevOps-Engineer.git
cd Technical-Challenge-Jr.-Cloud-DevOps-Engineer
pip install -r requirements.txt
python app.py
```
Acesse:  
```
http://localhost:5000/
http://localhost:5000/health
```

---

## Rodar com Docker
```bash
docker build -t hello-world-app .
docker run -d -p 5000:5000 hello-world-app
```

---

## Deploy no Azure (App Service + Docker)

### 1. Criar Resource Group
```bash
az group create --name <RESOURCE_GROUP_NAME> --location eastus2
```

### 2. Criar Container Registry (ACR)
```bash
az acr create --resource-group <RESOURCE_GROUP_NAME> --name <ACR_NAME> --sku Basic
```

### 3. Criar Cluster AKS
```bash
az aks create --resource-group <RESOURCE_GROUP_NAME> --name <AKS_CLUSTER_NAME> --node-count 2 --enable-addons monitoring --generate-ssh-keys --attach-acr <ACR_NAME>
```

### 4. Conectar ao Cluster
```bash
az aks get-credentials --resource-group <RESOURCE_GROUP_NAME> --name <AKS_CLUSTER_NAME>
```

### 5. Fazer Build e Push da Imagem para ACR
```bash
az acr build --registry <ACR_NAME> --image hello-world-app:v1 .
```

### 6. Aplicar os Manifestos Kubernetes
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/hpa.yaml
```

### 7. Verificar o Status
```bash
kubectl get pods
kubectl get svc hello-world-service
```

Acesse:
```
http://<EXTERNAL-IP>/
http://<EXTERNAL-IP>/health
```

---

### Endpoints:
- `/` → Hello World
- `/health` → {"status": "UP"}
---

## Infraestrutura como Código (IaC)

Arquivos Terraform para:

- Criar Resource Group
- Criar Azure Container Registry (ACR)
- Criar Azure Service Plan para Linux
- Criar Azure Linux Web App com imagem Docker do ACR
- Configurar integração entre ACR e Web App
---

## Comandos Terraform
```bash
terraform init
terraform plan
terraform apply
```

---

## Referências
- [Terraform – Provedor AzureRM](https://registry.terraform.io/providers/hashicorp/azurerm/latest)
- [Azure CLI AKS](https://learn.microsoft.com/en-us/azure/aks/kubernetes-walkthrough)