# Deploy do Airflow em um Cluster Local de Kubernetes (KinD) com Helm

## Visão Geral
Este documento fornece um guia para configurar e fazer o deploy do Apache Airflow em um cluster Kubernetes local, utilizando o KinD (Kubernetes in Docker) e Helm. O objetivo é criar um deploy reprodutível e entender os conceitos básicos do Kubernetes.

## Ferramentas Necessárias
- **[KinD](https://kind.sigs.k8s.io/)**: Ferramenta para executar clusters locais de Kubernetes usando contêineres Docker como “nós”.
- **[Docker](https://www.docker.com/)**: Nosso cluster de K8s será executado no Docker, com cada contêiner atuando como um nó de Kubernetes.
- **[Helm](https://helm.sh/)**: Gerenciador de pacotes para aplicações Kubernetes, utilizado para definir, fazer o deploy e gerenciar os componentes necessários.
- **[Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)**: Ferramenta de linha de comando que permite executar comandos em clusters Kubernetes.

## Passos de Configuração
1. **Clone do Repositório**: Clone o repositório GitHub que contém todos os arquivos necessários para o deploy.

2. **Instalação das Ferramentas**: Instale as ferramentas listadas usando seu gerenciador de pacotes preferido. (Ex.: Homebrew para Mac).

3. **Configuração do Cluster KinD**: Crie e configure um cluster KinD local utilizando o arquivo `kind-cluster.yaml`. Utilize `extraMounts` para persistir dados e logs do Airflow em seu sistema de arquivos local.

    Exemplo de configuração:
    ```yaml
    extraMounts:
      - hostPath: . /dags
        containerPath: /dags
      - hostPath: . /logs
        containerPath: /logs
    ```

4. **Criar o Cluster**: Execute `kind create cluster --config kind-cluster.yaml` para criar o cluster.

5. **Limpeza do Cluster**: Para remover o cluster após o uso, execute `kind delete cluster --name airflow-cluster`.

## Pré-requisitos para o Deploy
1. **Namespace Airflow**: Crie um namespace `airflow` para organizar os recursos.

    Comando:
    ```bash
    kubectl create namespace airflow --dry-run=client -o yaml > airflow.yaml
    kubectl apply -f airflow.yaml
    ```

2. **Volumes Persistentes**: Configure volumes persistentes para armazenar dados e logs do Airflow com independência do ciclo de vida do pod.

    Comandos:
    ```bash
    kubectl apply -f kubernetes-persistent-volumes.yaml -n airflow
    kubectl apply -f persistent-volume-claims.yaml
    ```

3. **Configuração do Ingress Controller**: Configure um Ingress Controller (NGINX) para rotear o tráfego externo para os pods corretos, permitindo o acesso à UI do Airflow.

    Comandos:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
    kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s
    ```

## Deploy do Airflow com Helm
1. **Adicionar Repositório Helm**: Adicione o repositório do Airflow e atualize os charts.

    Comandos:
    ```bash
    helm repo add apache-airflow https://airflow.apache.org
    helm repo update
    ```

2. **Deploy do Helm Chart**: Faça o deploy do Airflow no cluster KinD utilizando os valores configurados.

    Comando:
    ```bash
    helm upgrade --install airflow apache-airflow/airflow -n airflow -f values.yaml --debug
    ```
3. **Verificação do Deploy**: Verifique se os pods do Airflow estão em execução e se o Ingress Controller está configurado corretamente.

    Comandos:
    ```bash
    kubectl get deployment -n airflow -o wide
    ```
   
4. **Acessar a UI do Airflow**: Acesse a UI do Airflow em [http://localhost/](http://localhost/) utilizando as credenciais padrão (airflow / airflow).

