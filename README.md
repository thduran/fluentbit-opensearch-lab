[PT-BR, clique aqui](#pt-br)

# FluentBit, OpenSearch & Kubernetes

This project shows the implementation of an observability and log centralization stack.
The goal is to architect and implement a pipeline capable of collecting logs of distributed applications, process them and index them to real-time analysis.

---

## Architecture overview

**app (microservice) ➔ stdout/stderr ➔ node filesystem ➔ FluentBit (daemonSet) ➔ OpenSearch**

### Technologies
* **Kubernetes:** EKS (AWS) and Minikube.
* **Log collector:** Fluent Bit.
* **Log storage and analysis:** OpenSearch and OpenSearch Dashboards.
* **IaC:** Helm and eksctl.
* **App:** e-commerce simulation.

---

## Part 1: local lab (on-premise simulation)
*Focus: understanding of self-hosted components*

Here, I simulated an on-premise using **minikube** in WSL2.
* **Challenge:** Manual configuration of OpenSearch via Helm, dealing with resource limitation (memory/CPU) and kernel configurations (`vm.max_map_count`) to run Elasticsearch/OpenSearch.
* **Configuration:** Fluent Bit configured to communicate via internal cluster DNS (`svc.cluster.local`).

---

## Part 2: AWS
*Focus: scalability, managed services and debugging*

Migration of the workload to AWS, using managed services.

### Components:
1.  **Amazon OpenSearch Service:** managed cluster to ensure high availability and security (HTTPS/TLS).
2.  **AWS EKS:** Kubernetes cluster created via `eksctl` with managed EC2.
3.  **Security:** Integration via HTTPS with Basic Auth (simulating restrict access control).

---

## Challenges and troubleshooting

### 1. APIs deprecation (warning vs error)
* **Problem:** Logs warning `v1 Endpoints is deprecated`.
* **Diagnosis:** Identified that it was an alert for version 1.33 of K8s, not an impeditive error.
* **Learning:** To know differentiate "noise" from critical operational logs.

### 2. Application connectivity
* **Problem:** The application was running but it pointed `connection refused` while trying to access via load balancer and port-forward.
* **Diagnosis:** The container exposed port `5000` (.NET/Flask standard), but the Kubernetes service was configured to port `80`.
* **Solution:** Changed `targetPort` to 5000 and redeploy.

### 3. Democratizing troubleshooting
* **Scenario:** The application was implemented without a real DB.
* **Outcome:** The application failed (`psycopg.OperationalError`), generating error logs.
* **Impact:** The observability pipeline captured it in OpenSearch. It allows that devs can identify root causes with no access to server needed.

---

## Value to business

1.  **Time for troubleshooting:** With centralized and searchable logs, time to diagnose an error decreases drastically.
2.  **Security and compliance:** Logs are not "spread" in servers that can be deleted; they are persisted in a safe environment.
3.  **Operational efficiency:** **Amazon OpenSearch** eliminates necessity of hardware maintenance, OS updates, data cluster security patches.

---

## ⚙️ Steps

### Prerequisites
* AWS CLI configured.
* `kubectl`, `helm` and `eksctl` installed.

### 1. Provision infrastructure
```bash
# Create EKS cluster
eksctl create cluster --name lab-logs --region us-east-1 --nodes 2 --node-type t3.medium

# Create OpenSearch in AWS console
# Select "dev/test", instance "t3.small.search" and public access (lab).
```

### 2. Deploy the Fluent Bit (the collector)

```bash
[OUTPUT]
    Name opensearch
    Host search-your-domaind-aws.us-east-1.es.amazonaws.com
    Port 443
    HTTP_User admin
    HTTP_Passwd yourPassword
    tls On
```

Installation via Helm

```bash
helm repo add fluent [https://fluent.github.io/helm-charts](https://fluent.github.io/helm-charts)
helm install fluent-bit fluent/fluent-bit -f values.yaml
```

### 3. Deploy the application

`kubectl apply -f fake-shop.yaml`

### 4. View

Access the OpenSearch Dashboards URL given by AWS, create the Index Pattern logstash-* and visualize the logs in real-time in the Discover tab.

---

<div id="pt-br"></div>

# FluentBit, OpenSearch & Kubernetes

Este projeto demonstra a implementação de uma stack de observabilidade e centralização de logs.
O objetivo foi arquitetar e implantar um pipeline capaz de coletar logs de aplicações distribuídas em Kubernetes, processá-los e indexá-los para análise em tempo real.

---

## Visão geral da arquitetura

**app (microservice) ➔ stdout/stderr ➔ node filesystem ➔ FluentBit (daemonSet) ➔ OpenSearch**

### Tecnologias utilizadas
* **Kubernetes:** EKS (AWS) e Minikube.
* **Log collector:** Fluent Bit.
* **Log storage e análise:** OpenSearch e OpenSearch Dashboards.
* **IaC:** Helm e eksctl.
* **App:** Simulação de e-commerce.

---

## Parte 1: lab local (simulação on-premise)
*Foco: entendimento dos componentes self-hosted*

Nesta etapa, simulei um ambiente on-premise utilizando **minikube** no WSL2.
* **Desafio:** Configuração manual do OpenSearch via Helm, lidando com limitações de recursos (memória/CPU) e configurações de kernel (`vm.max_map_count`) para o Elasticsearch/OpenSearch rodar.
* **Configuração:** Fluent Bit configurado para comunicar via DNS interno do cluster (`svc.cluster.local`).

---

## Parte 2: AWS
*Foco: escalabilidade, serviços Gerenciados e debugging*

Migração do workload para a AWS, utilizando serviços gerenciados.

### Componentes:
1.  **Amazon OpenSearch Service:** Cluster gerenciado para garantir alta disponibilidade e segurança (HTTPS/TLS).
2.  **AWS EKS:** Cluster Kubernetes criado via `eksctl` com nós EC2 gerenciados.
3.  **Segurança:** Integração via HTTPS com autenticação Basic Auth (simulando controle de acesso restrito).

---

## Desafios e troubleshooting

### 1. Depreciação de APIs (warning vs error)
* **Problema:** Logs acusando `v1 Endpoints is deprecated`.
* **Diagnóstico:** Identifiquei que se tratava de um alerta para a versão 1.33 do K8s, e não um erro impeditivo.
* **Aprendizado:** Saber diferenciar "ruído" de logs operacionais críticos.

### 2. Conectividade da Aplicação
* **Problema:** A aplicação subiu, mas retornava `connection refused` ao tentar acessar via load balancer e port-forward.
* **Diagnóstico:** O container expunha a porta `5000` (padrão do .NET/Flask), mas o service Kubernetes estava configurado para a porta `80`.
* **Solução:** Ajuste no manifesto YAML (`targetPort: 5000`) e redeploy.

### 3. Democratização de troubleshooting
* **Cenário:** A aplicação foi implantada sem um banco de dados real.
* **Resultado:** A aplicação falhou (`psycopg.OperationalError`), gerando logs de erro massivos.
* **Impacto:** O pipeline de observabilidade capturou isso no OpenSearch. Isso permite que desenvolvedores identifiquem a causa de problemas sem precisar acessar o servidor.

---

## Valor para o negócio

1.  **Tempo para troubleshooting:** Com logs centralizados e pesquisáveis, o tempo para diagnosticar um erro cai drasticamente.
2.  **Segurança e compliance:** Logs não ficam "espalhados" em servidores que podem ser deletados; eles são persistidos em ambiente seguro.
3.  **Eficiência operacional:** O uso do **Amazon OpenSearch** elimina a necessidade de manutenção de hardware, updates de SO, patches de segurança do cluster de dados.

---

## ⚙️ Passo a passo

### Pré-requisitos
* AWS CLI configurado.
* `kubectl`, `helm` e `eksctl` instalados.

### 1. Provisionar infraestrutura
```bash
# Criar cluster EKS
eksctl create cluster --name lab-logs --region us-east-1 --nodes 2 --node-type t3.medium

# Criar OpenSearch no console AWS
# Selecionar "dev/test", instância "t3.small.search" e acesso público (para lab).
```

### 2. Deploy do Fluent Bit (o coletor)

```bash
[OUTPUT]
    Name opensearch
    Host search-seu-dominio-aws.us-east-1.es.amazonaws.com
    Port 443
    HTTP_User admin
    HTTP_Passwd SuaSenha
    tls On
```

Instalação via Helm

```bash
helm repo add fluent [https://fluent.github.io/helm-charts](https://fluent.github.io/helm-charts)
helm install fluent-bit fluent/fluent-bit -f values.yaml
```

### 3. Deploy da aplicação

`kubectl apply -f fake-shop.yaml`

### 4. Visualizar

Acesse a URL do OpenSearch Dashboards disponibilizada pela AWS, crie o Index Pattern logstash-* e visualize os logs em tempo real na aba Discover.