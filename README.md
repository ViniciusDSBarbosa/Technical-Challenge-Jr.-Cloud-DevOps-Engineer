# Hello World DevOps

Aplicação simples para o desafio DevOps Cloud Jr.  
Exibe **Hello World** na rota `/` e implementa **health check** na rota `/health`.

---

## Como rodar localmente
```bash
git clone https://github.com/SEU-USUARIO/Technical-Challenge-Jr.-Cloud-DevOps-Engineer.git
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
az group create --name hello-rg --location eastus2
```

### 2. Criar Container Registry
```bash
az acr create --resource-group hello-rg --name helloacr --sku Basic
az acr login --name helloacr
```

### 3. Push da imagem
```bash
docker tag hello-world-app helloacr.azurecr.io/hello-world-app:v1
docker push helloacr.azurecr.io/hello-world-app:v1
```

### 4. Criar App Service Plan e Web App
```bash
az appservice plan create --name hello-plan --resource-group hello-rg --sku B1 --is-linux
az webapp create   --resource-group hello-rg   --plan hello-plan   --name hello-world-devops   --deployment-container-image-name helloacr.azurecr.io/hello-world-app:v1
```

### 5. Configurar container no WebApp
```bash
az webapp config container set   --name hello-world-devops   --resource-group hello-rg   --docker-custom-image-name helloacr.azurecr.io/hello-world-app:v1   --docker-registry-server-url https://helloacr.azurecr.io
```
---

## Deploy no AKS
### Opção 1 – Usando Azure Cloud Shell (mais simples pelo portal)
No Portal do Azure, vá para o seu cluster AKS.

Clique em Conectar → escolha Cloud Shell.

Ele vai abrir um terminal no próprio navegador com az e kubectl já configurados.

Rode este comando para pegar as credenciais do cluster:

```bash
az aks get-credentials --resource-group <RESOURCEGROUPNAME> --name hello-aks
```
Isso conecta seu kubectl ao cluster.

## Como enviar os arquivos YAML para o Cloud Shell
No Cloud Shell vá até "Gerenciar arquivos":

Clique nele → selecione os arquivos da pasta k8s/ (deployment.yaml, service.yaml, hpa.yaml).

Eles vão parar no diretório padrão (~/).

Depois, aplique os manifestos:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```
Para confirmar se os pods estão rodando:

```bash
kubectl get pods
```
Para pegar o IP público:
```bash
kubectl get svc hello-world-service
```
Quando o EXTERNAL-IP aparecer, acesse:

http://<EXTERNAL-IP>/
http://<EXTERNAL-IP>/health

## Opção 2 – Localmente
Instale o Azure CLI + kubectl no seu PC.

Rode:

```bash
az login
az aks get-credentials --resource-group <RESOURCEGROUPNAME> --name hello-aks
kubectl apply -f k8s/
```
E siga os mesmos comandos para ver os pods e pegar o IP.

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/hpa.yaml
```

Endpoints:
- `/` → Hello World
- `/health` → {"status": "UP"}

---

## CI/CD com GitHub Actions
O workflow `.github/workflows/ci-cd.yaml` faz:
- Build da imagem
- Push para ACR
- Deploy no Azure App Service

## Referências Terraform

- Provedor AzureRM: [Terraform Registry – AzureRM Provider]({{https://registry.terraform.io/providers/hashicorp/azurerm/latest}})
- Grupo de Recursos: [azurerm_resource_group]({{https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group}})
- Azure Container Registry: [azurerm_container_registry]({{https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/container_registry}})
- App Service Plan: [azurerm_app_service_plan azurerm_app_service]({{https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/app_service_plan}})
- App Service: [azurerm_app_service]({{https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/app_service}})