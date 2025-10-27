# Simulação de Falha e Resiliência de Job/Endpoint no Kubernetes (KIND)

Este repositório contém os arquivos e a documentação de uma simulação de falha de serviço no Kubernetes, executada localmente com KIND no WSL Ubuntu.

## Resumo do Projeto

O objetivo deste projeto é simular um cenário de falha crítica (como a queda de uma região da AWS) onde um `Job` (processo em lote, ex: importação diária) perde acesso ao seu endpoint primário e precisa executar uma estratégia de resiliência para garantir a conclusão da tarefa.

Implementamos uma solução que utiliza:
1.  **Retry com Backoff:** O Job tenta se conectar ao serviço primário 3 vezes, com uma espera crescente (5s, 10s), dando tempo para o serviço se recuperar de falhas temporárias.
2.  **Fallback:** Após as tentativas falharem, o Job aciona automaticamente um "Plano B", conectando-se a um endpoint de contingência (fallback).
3.  **ConfigMap:** O endpoint de fallback serve uma página HTML customizada para confirmar visualmente que a rota de contingência foi usada.

## Cenário do Desafio

* **Aplicação:** Um `Job` crítico de importação de dados.
* **Dependências:** O `Job` precisa de um `Secret` (com chaves de API) e de um `Service` (endpoint interno) para funcionar.
* **A Falha:** O endpoint primário (`internal-api-primary`) fica inacessível.
* **A Solução:** O `Job` deve detectar a falha, tentar se reconectar (backoff) e, por fim, usar um endpoint de fallback (`internal-api-fallback`) para concluir a tarefa com sucesso.

## Tecnologias Utilizadas

* **Docker Desktop** (com integração WSL)
* **WSL 2** (Ubuntu)
* **KIND** (Kubernetes in Docker)
* **Kubectl**
* **VSCode** (para edição dos YAMLs)

---

## Como Executar (Passo a Passo)

Siga estes passos para replicar a simulação no seu ambiente WSL.

### Pré-requisitos

1.  **Docker Desktop com WSL 2:**
    É fundamental ter o Docker Desktop instalado no Windows e integrado com sua distribuição WSL (Ubuntu).
    * Vá em `Settings > Resources > WSL Integration`.
    * Habilite a integração com sua distro Ubuntu.
    * Clique em `Apply & Restart`.

    ![Integração Docker WSL](https://i.imgur.com/gYf0aWp.png)

2.  **KIND (Kubernetes in Docker):**
    Instale o KIND (Kubernetes in Docker) dentro do seu terminal WSL Ubuntu.

    ```bash
    # 1. Baixar o binário
    curl -Lo ./kind [https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64](https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64)
    
    # 2. Tornar executável
    chmod +x ./kind
    
    # 3. Mover para o PATH
    sudo mv ./kind /usr/local/bin/kind
    
    # 4. Verificar a instalação
    kind version
    ```

### Passo 0: Criar o Cluster KIND

Crie um cluster de Kubernetes local chamado `failure-k8s`.

```bash
kind create cluster --name failure-k8s