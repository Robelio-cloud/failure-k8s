Simulação de Falha e Resiliência de Job/Endpoint no Kubernetes (KIND)
Este repositório contém os arquivos e a documentação de uma simulação de falha de serviço no Kubernetes, executada localmente com KIND no WSL Ubuntu.

Resumo do Projeto
O objetivo deste projeto é simular um cenário de falha crítica (como a queda de uma região da AWS) onde um Job (processo em lote, ex: importação diária) perde acesso ao seu endpoint primário e precisa executar uma estratégia de resiliência para garantir a conclusão da tarefa.

Implementamos uma solução que utiliza:

Retry com Backoff: O Job tenta se conectar ao serviço primário 3 vezes, com uma espera crescente (5s, 10s), dando tempo para o serviço se recuperar de falhas temporárias.

Fallback: Após as tentativas falharem, o Job aciona automaticamente um "Plano B", conectando-se a um endpoint de contingência (fallback).

ConfigMap: O endpoint de fallback serve uma página HTML customizada para confirmar visualmente que a rota de contingência foi usada.

Cenário do Desafio
Aplicação: Um Job crítico de importação de dados.

Dependências: O Job precisa de um Secret (com chaves de API) e de um Service (endpoint interno) para funcionar.

A Falha: O endpoint primário (internal-api-primary) fica inacessível.

A Solução: O Job deve detectar a falha, tentar se reconectar (backoff) e, por fim, usar um endpoint de fallback (internal-api-fallback) para concluir a tarefa com sucesso.

Tecnologias Utilizadas
Docker Desktop (com integração WSL)

WSL 2 (Ubuntu)

KIND (Kubernetes in Docker)

Kubectl

VSCode (para edição dos YAMLs)

Como Executar (Passo a Passo)
Siga estes passos para replicar a simulação no seu ambiente WSL.

Pré-requisitos
Docker Desktop com WSL 2: É fundamental ter o Docker Desktop instalado no Windows e integrado com sua distribuição WSL (Ubuntu). 

Vá em Settings > Resources > WSL Integration.

Habilite a integração com sua distro Ubuntu.


Clique em Apply & Restart.

KIND (Kubernetes in Docker): Instale o KIND (Kubernetes in Docker) dentro do seu terminal WSL Ubuntu. 

Bash

# 1. Baixar o binário [cite: 36, 39]
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64

# 2. Tornar executável [cite: 42]
chmod +x ./kind

# 3. Mover para o PATH [cite: 43, 46]
sudo mv ./kind /usr/local/bin/kind

# 4. Verificar a instalação [cite: 47, 49]
kind version
Passo 0: Criar o Cluster KIND
Crie um cluster de Kubernetes local chamado failure-k8s. 


Bash

kind create cluster --name failure-k8s
O terminal mostrará a criação do cluster, finalizando com "Thanks for using kind! 👋".



Passo 1: Clonar o Repositório
Obtenha todos os arquivos YAML necessários clonando este repositório. 

Bash

# Clona o repositório [cite: 73]
git clone https://github.com/Robelio-cloud/failure-k8s.git
 
# Acessa o diretório do projeto [cite: 81]
cd failure-k8s
Passo 2: Aplicar os Recursos da Simulação
Agora, aplique todos os arquivos YAML (o ConfigMap, o Secret, os Serviços e o Job) no seu cluster de uma só vez. 

Bash

kubectl apply -f .

Saída Esperada: 

configmap/fallback-page-config created
secret/api-credentials configured
deployment.apps/fallback-api-deployment created
service/internal-api-fallback created
service/internal-api-primary created
job.batch/daily-import-job-resiliente created
Verificando a Simulação (As Evidências)
Com os recursos aplicados, o Job iniciará imediatamente. Vamos verificar os resultados.

Evidência 1: Logs do Job (Backoff + Fallback)
Esta é a evidência principal. O log do Job nos mostrará a falha, as tentativas de backoff e o sucesso final via fallback.

Encontre o nome do Pod do Job: O Kubernetes cria um pod para o job com um nome aleatório.

Bash

kubectl get pods
(Procure pelo pod que começa com daily-import-job-resiliente-... e status Running ou Completed ).

Veja os Logs: Use o nome do pod encontrado para ver os logs. O comando -f (follow) permite ver os logs ao vivo.

Bash

# Substitua 'daily-import-job-resiliente-dqmvf' pelo nome do seu pod
kubectl logs -f daily-import-job-resiliente-dqmvf 
Resultado do Log (Sucesso): O log mostrará:


O Job iniciando.

A Tentativa 1 falhando e o script aguardando 5 segundos. 

A Tentativa 2 falhando e o script aguardando 10 segundos. 

A Tentativa 3 falhando, levando à falha geral do primário. 

O ACIONANDO FALLBACK.

O curl imprimindo o HTML da nossa página customizada ("Sistema de Contingência").




A mensagem final de SUCESSO.

[Mon Oct 27 19:31:45 UTC 2025] Job de importação (com backoff) iniciado. [cite: 335]
[Mon Oct 27 19:31:45 UTC 2025] Chave API carregada (log): secret-key... [cite: 335]
[Mon Oct 27 19:31:45 UTC 2025] Tentativa 1/3: Acessando primário (http://internal-api-primary)... [cite: 336]
[Mon Oct 27 19:31:45 UTC 2025] FALHA (Tentativa 1): Primário inacessível. Aguardando 5 segundos (backoff)... 
[Mon Oct 27 19:31:50 UTC 2025] Tentativa 2/3: Acessando primário (http://internal-api-primary)... [cite: 337]
[Mon Oct 27 19:31:50 UTC 2025] FALHA (Tentativa 2): Primário inacessível. Aguardando 10 segundos (backoff)... 
[Mon Oct 27 19:32:00 UTC 2025] Tentativa 3/3: Acessando primário (http://internal-api-primary)... 
[Mon Oct 27 19:32:00 UTC 2025] FALHA GERAL (Primário): Endpoint http://internal-api-primary inacessível após 3 tentativas. 
[Mon Oct 27 19:32:00 UTC 2025] ACIONANDO FALLBACK para http://internal-api-fallback... 
<!DOCTYPE html>
<html lang="pt-br"> [cite: 341]
<head>
<title>Sistema de Contingência</title> [cite: 343]
<style> [cite: 344]
body { [cite: 345, 352, 353, 354, 355, 356]
font-family: Arial, sans-serif;
text-align: center;
margin-top: 100px;
background-color: #fff8e1; /* Um amarelo claro de "alerta" */
color: #333;
}
h1 { [cite: 347, 357]
color: #d9534f; /* Vermelho */
}
</style> [cite: 349]
</head> [cite: 350]
<body> [cite: 351]
<h1>API de Fallback Ativada</h1> [cite: 358]
<p>O servi&ccedil;o prim&aacute;rio (internal-api-primary) falhou em responder.</p> [cite: 359]
<p>Voc&ecirc; foi redirecionado para este endpoint de conting&ecirc;ncia (o Nginx).</p> [cite: 360]
<p><strong>A importa&ccedil;o de dados (Job) foi conclu&iacute;da com SUCESSO usando esta rota.</strong></p> [cite: 361]
</body> [cite: 362]
</html> [cite: 363]
[Mon Oct 27 19:32:00 UTC 2025] SUCESSO: Dados importados via endpoint de FALLBACK. 
Evidência 2: Acessando o Endpoint de Fallback (Localhost)
Podemos verificar a página de contingência diretamente no nosso navegador criando um "túnel" para o serviço.

Abra um novo terminal WSL.

Execute o kubectl port-forward. Usamos a porta local 8081 (pois a 8080 pode estar em uso). 

Bash

kubectl port-forward service/internal-api-fallback 8081:80

Saída Esperada: 

Forwarding from 127.0.0.1:8081 -> 80
Forwarding from [::1]:8081 -> 80
Abra seu navegador: Acesse http://localhost:8081. Você verá a página "API de Fallback Ativada", confirmando que o serviço de contingência está no ar e servindo o HTML customizado.

Arquitetura dos Arquivos YAML
00-custom-page-configmap.yaml: Cria um ConfigMap que armazena nosso index.html customizado de "Sistema de Contingência".


01-secret.yaml: Cria o Secret api-credentials com uma chave API_KEY fictícia.



02-fallback-app.yaml: Cria o Deployment (Nginx) e o Service para o endpoint de fallback (internal-api-fallback). Este arquivo é modificado para montar o ConfigMap e substituir a página padrão do Nginx pela nossa.




03-primary-service-falho.yaml: Cria o Service do endpoint primário (internal-api-primary). Crucialmente, ele não tem Deployment ou Pods associados, garantindo que qualquer conexão com ele falhe.


04-job-com-backoff.yaml: O coração da simulação. Cria o Job que executa um script shell , monta o Secret como variável de ambiente e contém toda a lógica de retry/backoff/fallback.