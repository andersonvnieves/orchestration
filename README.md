# TC03 - Cloud Games — Orquestração

Este repositório contém a infraestrutura de **orquestração e execução dos serviços desenvolvidos no Tech Challenge 03 (TC03)** em um ambiente **Kubernetes local**.
O projeto reúne os manifests e scripts necessários para provisionar e executar os microsserviços que compõem a solução, além dos recursos externos utilizados pela aplicação na **AWS**.

O repositório contém:

- **Stacks e manifests Kubernetes** para implantação dos microsserviços
- Configurações do **Kong**, utilizado como API Gateway
- Stack de **observabilidade**, composta por Prometheus, Loki e Grafana
- Templates **AWS CloudFormation** para provisionamento da infraestrutura necessária na AWS

# Stack Escolhida

Nesta seção são apresentadas as principais decisões arquiteturais relacionadas à adoção de componentes complementares à arquitetura dos microsserviços. Para cada tecnologia, são descritos o contexto de utilização, a motivação para sua escolha e os benefícios esperados dentro da solução.

## Cache do Catálogo de Jogos

O **Redis** foi adotado como camada de cache à frente do catálogo de jogos, utilizando uma estratégia combinada de **Cache-Aside e Write-Through**, com o objetivo de absorver o alto volume de requisições de leitura, reduzir a carga sobre a base de dados principal e manter o cache atualizado de forma proativa durante as operações de escrita.

### Redução de carga (Offloading)

O catálogo possui um volume relativamente pequeno de dados, permitindo que o conjunto de jogos seja mantido integralmente em memória. Dessa forma, as consultas de leitura mais frequentes podem ser atendidas diretamente pelo Redis, reduzindo significativamente o número de acessos à base de dados de origem.

Essa abordagem é especialmente adequada para um cenário em que predominam operações de leitura, permitindo que a base principal seja preservada para operações de escrita e atualizações.

### Atualização proativa do cache

A estratégia adotada não depende de TTL (Time To Live) para garantir a atualização dos dados. Sempre que ocorre uma operação de insert ou update na fonte de dados principal, o cache correspondente é atualizado ou invalidado de forma controlada. Dessa maneira, evita-se que informações antigas permaneçam disponíveis no cache até o vencimento de um TTL. Essa abordagem proporciona maior previsibilidade sobre a consistência dos dados e reduz a possibilidade de disponibilização de informações desatualizadas.

## Biblioteca de Usuários

O MongoDB foi selecionado como mecanismo de persistência para a biblioteca de usuários considerando principalmente a necessidade de flexibilidade na evolução do modelo de dados e a possibilidade de escalabilidade horizontal conforme o crescimento da quantidade de usuários e registros.

### Flexibilidade e evolução do modelo

A biblioteca de um usuário representa uma coleção de jogos adquiridos, podendo receber novos atributos e metadados conforme novas funcionalidades sejam incorporadas ao sistema.

Nesse contexto, o modelo orientado a documentos do **MongoDB** permite utilizar uma abordagem de **modelagem embedded**, na qual os jogos pertencentes à biblioteca podem ser armazenados dentro do próprio documento do usuário. Dessa forma, uma única consulta ao documento é suficiente para recuperar a biblioteca completa, reduzindo a necessidade de múltiplas consultas e relacionamentos entre tabelas.

Em um modelo relacional, à medida que a quantidade de usuários e o volume de jogos associados a cada biblioteca aumentam, operações desse tipo podem exigir consultas envolvendo múltiplas tabelas e JOINs, aumentando a complexidade das consultas e potencialmente o custo de processamento. No modelo orientado a documentos adotado, os dados que são frequentemente acessados em conjunto permanecem agregados, favorecendo o desempenho das operações de leitura da biblioteca.

Além disso, a estrutura flexível do MongoDB facilita a evolução do modelo sem exigir alterações estruturais frequentes, como a criação ou alteração de múltiplas tabelas e relacionamentos. Novos atributos e metadados podem ser incorporados aos documentos conforme novas funcionalidades sejam adicionadas ao sistema.

### Agilidade no desenvolvimento

A estrutura orientada a documentos reduz a complexidade do mapeamento entre as entidades da aplicação e o modelo de persistência, especialmente para uma entidade como a biblioteca, na qual as informações relacionadas a um usuário podem ser representadas de forma agregada.

Essa característica facilita a implementação de novas funcionalidades e permite que o modelo de persistência acompanhe a evolução do domínio com menor impacto sobre a estrutura existente.

## Stack Open Source de Observabilidade — Prometheus, Loki e Grafana

Para a camada de observabilidade, foi adotada uma stack open source e self-hosted, composta por Prometheus, Loki e Grafana. A escolha considera a necessidade de centralizar métricas e logs dos serviços, possibilitando acompanhar a saúde da aplicação e correlacionar eventos de infraestrutura e aplicação.

Além disso, a adoção de uma solução self-hosted proporciona maior controle sobre os dados de observabilidade e reduz a dependência de plataformas externas de monitoramento.

### Prometheus — Métricas

O Prometheus é responsável pela coleta e armazenamento das métricas dos serviços e da infraestrutura. Seu modelo baseado em time series permite acompanhar indicadores como utilização de CPU e memória, quantidade de requisições, latência, taxas de erro e disponibilidade dos serviços.

### Loki — Logs

O Loki é utilizado como solução centralizada para armazenamento e consulta dos logs da aplicação.

### Grafana — Visualização e correlação

O Grafana atua como camada de visualização da solução de observabilidade, consolidando em uma única interface as informações provenientes do Prometheus e do Loki. Por meio dos dashboards, é possível acompanhar métricas de infraestrutura e aplicação e, simultaneamente, consultar os logs associados aos mesmos serviços.

Essa integração permite correlacionar diferentes sinais de observabilidade. Por exemplo, um aumento na taxa de erros identificado nos logs pode ser analisado em conjunto com métricas de CPU, memória, latência ou volume de requisições, facilitando a identificação da causa de problemas.

### Soberania e controle dos dados

A utilização de uma stack open source e self-hosted mantém as métricas e os logs dentro da infraestrutura controlada por nós.

Essa abordagem reduz a dependência de provedores externos de observabilidade e proporciona maior previsibilidade sobre armazenamento, retenção e custos de ingestão, além de facilitar o atendimento a requisitos internos de governança e controle dos dados.

# Execução local

Seguindo as instruções apresentadas neste README, é possível configurar toda a infraestrutura necessária e executar o projeto em um **cluster Kubernetes local**, utilizando soluções como **Kind** ou **Minikube**.

O processo de inicialização contempla desde a preparação do cluster e provisionamento dos recursos AWS até a implantação dos microsserviços, da camada de observabilidade e do API Gateway.

A arquitetura resultante permite reproduzir localmente um ambiente distribuído composto por **microsserviços, comunicação assíncrona, persistência, cache, observabilidade e API Gateway**, utilizando Kubernetes como plataforma de orquestração.

> **Pré-requisitos:** antes de iniciar a execução, certifique-se de possuir Docker, Kubernetes, `kubectl`, Kind ou Minikube e AWS CLI v2 instalados e configurados.

## Clonar os repositórios

Para começar, clone o repositorio dos microserviços na mesma pasta que contem o reposititório atual:

```bash
git clone https://github.com/andersonvnieves/user-service.git
```

```bash
git clone https://github.com/andersonvnieves/catalog-service.git
```

```bash
git clone https://github.com/andersonvnieves/payment-service.git
```

```bash
git clone https://github.com/andersonvnieves/notification-service.git
```

## AWS — SQS (CloudFormation)

Após configurar a AWS CLI, devem ser provisionadas as stacks responsáveis pelos recursos externos utilizados pela aplicação.

### Criar as filas SQS

Na pasta cloudformation temos o arquivo sqs-stack.yaml responsável por criar as duas filas sqs dos evnetos que a aplicação utiliza: user-created-queue e payment-processed-queue. Execute o comando abaixo para criar as duas filas na sua conta.

```bash
aws cloudformation deploy --template-file .\cloudformation\sqs-stack.yaml --stack-name sqs-stack --capabilities CAPABILITY_IAM
```

## Parâmetros e Segredos do AWS Systems Manager (Parameter Store)

Abaixo estão listados todos os parâmetros utilizados pela aplicação FGC, divididos por categoria, com suas respectivas descrições, tipos e o comando de exemplo para criação/atualização via AWS CLI.

### 1. JWT & Autenticação

* **`/fgc/jwt/secret`**
  * **Descrição:** Segredo JWT utilizado pelo Kong e pelos microsserviços.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/jwt/secret' --type 'SecureString' --value 'SEU_SEGREDO_JWT' --description 'Segredo JWT utilizado pelo Kong e pelos microsserviços' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/jwt/issuer`**
  * **Descrição:** Issuer JWT para os tokens.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/jwt/issuer' --type 'String' --value 'fgcapi' --description 'Issuer JWT para os tokens' --tier 'Standard' --overwrite
    ```

* **`/fgc/jwt/audience`**
  * **Descrição:** Audience JWT para os tokens.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/jwt/audience' --type 'String' --value 'fgcapi' --description 'Audience JWT para os tokens' --tier 'Standard' --overwrite
    ```

* **`/fgc/jwt/token-ttl`**
  * **Descrição:** Tempo de vida do token JWT em minutos.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/jwt/token-ttl' --type 'String' --value '10' --description 'Tempo de vida do token JWT em minutos' --tier 'Standard' --overwrite
    ```

### 2. Kong API Gateway

* **`/fgc/kong/ssl-cert`**
  * **Descrição:** Certificado SSL utilizado pelo Kong API Gateway.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/kong/ssl-cert' --type 'String' --value '-----BEGIN CERTIFICATE-----\n...' --description 'Certificado SSL utilizado pelo Kong API Gateway' --tier 'Standard' --overwrite
    ```

* **`/fgc/kong/ssl-key`**
  * **Descrição:** Chave privada SSL utilizada pelo Kong API Gateway.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/kong/ssl-key' --type 'SecureString' --value '-----BEGIN PRIVATE KEY-----\n...' --description 'Chave privada SSL utilizada pelo Kong API Gateway' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

### 3. Observabilidade (Grafana, Prometheus, Loki)

* **`/fgc/grafana/admin-user`**
  * **Descrição:** Usuário administrador do Grafana.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/grafana/admin-user' --type 'SecureString' --value 'admin' --description 'Usuário administrador do Grafana' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/grafana/admin-password`**
  * **Descrição:** Senha do administrador do Grafana.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/grafana/admin-password' --type 'SecureString' --value 'admin' --description 'Senha do administrador do Grafana' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/observability/prometheus-url`**
  * **Descrição:** URL interna do serviço Prometheus.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/observability/prometheus-url' --type 'String' --value '[http://prometheus-service.observability.svc.cluster.local:9090](http://prometheus-service.observability.svc.cluster.local:9090)' --description 'URL interna do serviço Prometheus' --tier 'Standard' --overwrite
    ```

* **`/fgc/observability/loki-url`**
  * **Descrição:** URL interna do serviço Loki.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/observability/loki-url' --type 'String' --value '[http://loki-service.observability.svc.cluster.local:3100](http://loki-service.observability.svc.cluster.local:3100)' --description 'URL interna do serviço Loki' --tier 'Standard' --overwrite
    ```

### 4. Mensageria & Infraestrutura

* **`/fgc/aws/region`**
  * **Descrição:** Região AWS da aplicação.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/aws/region' --type 'String' --value 'sa-east-1' --description 'Região AWS da aplicação' --tier 'Standard' --overwrite
    ```

* **`/fgc/aws/access-key-id`**
  * **Descrição:** AWS Access Key ID para serviços AWS.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/aws/access-key-id' --type 'SecureString' --value 'CHANGE_ME' --description 'AWS Access Key ID para serviços AWS' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/aws/secret-access-key`**
  * **Descrição:** AWS Secret Access Key para serviços AWS.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/aws/secret-access-key' --type 'SecureString' --value 'CHANGE_ME' --description 'AWS Secret Access Key para serviços AWS' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/sqs/user-created-queue-url`**
  * **Descrição:** URL da fila SQS `user-created-queue`.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/sqs/user-created-queue-url' --type 'String' --value 'CHANGE_ME' --description 'URL da fila SQS user-created-queue' --tier 'Standard' --overwrite
    ```

* **`/fgc/sqs/payment-processed-queue-url`**
  * **Descrição:** URL da fila SQS `payment-processed-queue`.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/sqs/payment-processed-queue-url' --type 'String' --value 'CHANGE_ME' --description 'URL da fila SQS payment-processed-queue' --tier 'Standard' --overwrite
    ```

### 5. User Service

* **`/fgc/users/connection-string`**
  * **Descrição:** Connection string do banco de dados do User Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/users/connection-string' --type 'SecureString' --value 'Server=users-db-service;Database=FGC_User;User Id=sa;Password=rohPav-juKmeb-DIhku9!;TrustServerCertificate=True' --description 'Connection string do banco de dados do User Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/users/root-password`**
  * **Descrição:** Senha inicial do usuário root do User Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/users/root-password' --type 'SecureString' --value 'Admin@123' --description 'Senha inicial do usuario root do User Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/users/root-first-name`**
  * **Descrição:** Nome do usuário root inicial.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/users/root-first-name' --type 'String' --value 'FirstName' --description 'Nome do usuario root inicial' --tier 'Standard' --overwrite
    ```

* **`/fgc/users/root-last-name`**
  * **Descrição:** Sobrenome do usuário root inicial.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/users/root-last-name' --type 'String' --value 'LastName' --description 'Sobrenome do usuario root inicial' --tier 'Standard' --overwrite
    ```

* **`/fgc/users/root-email`**
  * **Descrição:** E-mail do usuário root inicial.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/users/root-email' --type 'String' --value 'admin@fgc.com' --description 'E-mail do usuario root inicial' --tier 'Standard' --overwrite
    ```

### 6. Catalog Service

* **`/fgc/catalog/connection-string`**
  * **Descrição:** Connection string do banco de dados SQL do Catalog Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/connection-string' --type 'SecureString' --value 'Server=catalog-db-service;Database=FGC_Catalog;User Id=sa;Password=rohPav-juKmeb-DIhku9!;TrustServerCertificate=True' --description 'Connection string do banco de dados SQL do Catalog Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/rabbitmq-uri`**
  * **Descrição:** URI de conexão com RabbitMQ para o Catalog Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/rabbitmq-uri' --type 'SecureString' --value 'amqp://admin:admin123@rabbitmq-service:5672/' --description 'URI de conexao com RabbitMQ para o Catalog Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/mongodb-uri`**
  * **Descrição:** URI de conexão com MongoDB para o Catalog Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/mongodb-uri' --type 'SecureString' --value 'mongodb://mongoadmin:zessuq-8gyzda-jazhyN@mongodb-service:27017/catalog_service?authSource=admin&guidRepresentation=Standard' --description 'URI de conexao com MongoDB para o Catalog Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/mongodb-dbname`**
  * **Descrição:** Nome da base MongoDB do Catalog Service.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/mongodb-dbname' --type 'String' --value 'catalog_service' --description 'Nome da base MongoDB do Catalog Service' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/redis-connection`**
  * **Descrição:** Connection string do Redis para o Catalog Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/redis-connection' --type 'SecureString' --value 'redis-service:6379,password=datgef-6zadci-nyZbud' --description 'Connection string do Redis para o Catalog Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/redis-password`**
  * **Descrição:** Senha do Redis do Catalog Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/redis-password' --type 'SecureString' --value 'datgef-6zadci-nyZbud' --description 'Senha do Redis do Catalog Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/order-created-exchange`**
  * **Descrição:** Exchange RabbitMQ para evento Order Created[span_20](start_span)[span_20](end_span).
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/order-created-exchange' --type 'String' --value 'fgc' --description 'Exchange RabbitMQ para evento Order Created' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/order-created-routing-key`**
  * **Descrição:** Routing Key RabbitMQ para evento Order Created.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/order-created-routing-key' --type 'String' --value 'order.created' --description 'Routing Key RabbitMQ para evento Order Created' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/order-created-queue-name`**
  * **Descrição:** Nome da fila RabbitMQ para evento Order Created.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/order-created-queue-name' --type 'String' --value 'order.created' --description 'Nome da fila RabbitMQ para evento Order Created' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/payment-processed-exchange`**
  * **Descrição:** Exchange RabbitMQ para evento Payment Processed.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/payment-processed-exchange' --type 'String' --value 'fgc' --description 'Exchange RabbitMQ para evento Payment Processed' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/payment-processed-routing-key`**
  * **Descrição:** Routing Key RabbitMQ para evento Payment Processed.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/payment-processed-routing-key' --type 'String' --value 'payment.processed' --description 'Routing Key RabbitMQ para evento Payment Processed' --tier 'Standard' --overwrite
    ```

* **`/fgc/catalog/payment-processed-queue-name`**
  * **Descrição:** Nome da fila RabbitMQ para evento Payment Processed.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/catalog/payment-processed-queue-name' --type 'String' --value 'catalog.payment.processed' --description 'Nome da fila RabbitMQ para evento Payment Processed' --tier 'Standard' --overwrite
    ```

### 7. Payment Service

* **`/fgc/payment/connection-string`**
  * **Descrição:** Connection string do banco de dados SQL do Payment Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/payment/connection-string' --type 'SecureString' --value 'Server=payment-db-service;Database=FGC_Payment;User Id=sa;Password=rohPav-juKmeb-DIhku9!;TrustServerCertificate=True' --description 'Connection string do banco de dados SQL do Payment Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/payment/rabbitmq-uri`**
  * **Descrição:** URI de conexão com RabbitMQ para o Payment Service.
  * **Tipo:** `SecureString` (Segredo)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/payment/rabbitmq-uri' --type 'SecureString' --value 'amqp://admin:admin123@rabbitmq-service:5672/' --description 'URI de conexao com RabbitMQ para o Payment Service' --key-id 'alias/aws/ssm' --tier 'Standard' --overwrite
    ```

* **`/fgc/payment/order-created-exchange`**
  * **Descrição:** Exchange RabbitMQ Order Created no Payment Service.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/payment/order-created-exchange' --type 'String' --value 'fgc' --description 'Exchange RabbitMQ Order Created no Payment Service' --tier 'Standard' --overwrite
    ```

* **`/fgc/payment/order-created-routing-key`**
  * **Descrição:** Routing Key RabbitMQ Order Created no Payment Service.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/payment/order-created-routing-key' --type 'String' --value 'order.created' --description 'Routing Key RabbitMQ Order Created no Payment Service' --tier 'Standard' --overwrite
    ```

* **`/fgc/payment/payment-processed-exchange`**
  * **Descrição:** Exchange RabbitMQ Payment Processed no Payment Service.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/payment/payment-processed-exchange' --type 'String' --value 'fgc' --description 'Exchange RabbitMQ Payment Processed no Payment Service' --tier 'Standard' --overwrite
    ```

* **`/fgc/payment/payment-processed-routing-key`**
  * **Descrição:** Routing Key RabbitMQ Payment Processed no Payment Service.
  * **Tipo:** `String` (Parâmetro)
  * **Exemplo AWS CLI:**
    ```bash
    aws ssm put-parameter --name '/fgc/payment/payment-processed-routing-key' --type 'String' --value 'payment.processed' --description 'Routing Key RabbitMQ Payment Processed no Payment Service' --tier 'Standard' --overwrite
    ```

### Sincronizar com o Kubernetes via External Secrets Operator

Para integrar os valores salvos no AWS Parameter Store com o Kubernetes, aplique a stack do External Secrets Operator:

```bash
kubectl apply -f .\k8s\external-secrets-stack.yaml
```

### Deploy Notifications Lambda

Vá para a pasta do projeto da lambda e execute o comando de deploy da lambda:

```bash
cd ..\notification-service\br.com.fiap.cloudgames.Notification.Lambda
```

```bash
dotnet lambda deploy-function notification-service
```

Substitua os arns das filas criadas pela stack do CloudFormation no comando abaixo para adicionar os triggers que acionaram a lambda:

```bash
aws lambda create-event-source-mapping --function-name notification-service --event-source-arn <ARN> --batch-size 10
```
Não esqueça de executar um para cada fila SQS.


## Kubernetes — Stack de Observabilidade

Neste repositório na pasta k8s temos observability com os arquivos para subir e configurar o grafana, prometheus e loki:

```bash
kubectl apply -f  k8s\observability\prometheus-service-stack.yaml 
```

```bash
kubectl apply -f  k8s\observability\loki-service-stack.yaml
```

```bash
kubectl apply -f  k8s\observability\promtail-daemon-stack.yaml
```

```bash
kubectl apply -f  k8s\observability\grafana-service-stack.yaml
```

## Kubernetes — Build das imagens dos microserviços

Com todos os repositórios clonados na pasta indicada, execute os comandos a seguir para gerar as imagens dos microserviços:

```bash
docker build -t fgc-catalog-service:latest -f ..\catalog-service\Dockerfile ..\catalog-service
```

```bash
docker build -t fgc-payment-service:latest -f ..\payment-service\Dockerfile ..\payment-service
```

```bash
docker build -t fgc-user-service:latest -f ..\user-service\Dockerfile ..\user-service
```

## Kubernetes — Ajustar parametros nas stask dos microserviços

Alguns parametros tem que ser ajustados nas stacks de cada mciroserviço antes de aplicar a stack no cluster,
procure o README.md de cada projeto para mais detalhes e substitua as credenciais do AWS, a arn das filas sqs criadas.

## Kubernetes — Stacks dos microserviços

Aplique a stack de cada mciroserviço com os comandos abaixo:

```bash
kubectl apply -f ..\user-service\k8s\user-service-stack.yml
```

```bash
kubectl apply -f ..\catalog-service\k8s\catalog-service-stack.yml
```

```bash
kubectl apply -f ..\payment-service\k8s\payment-service-stack.yml
```

## Kubernetes — Stacks do Kong (apigateway)

Rode o comando abaixo para aplicar a stack do kong:

```bash
kubectl apply -f .\k8s\api-gateway\kong-service-stack.yml
```

## Kubernetes — Port Forward

Dependendo do seu ambiente e do seu cluster k8s, pode ser necessário fazer encaminhamento de porta para acessar os serviços, execute os comandos abaixo de acordo com sua necessidade:

### Kong (api gateway)
```bash
kubectl port-forward svc/kong-proxy-service -n fgc 30443:443
```

### Grafana
```bash
kubectl port-forward svc/grafana-service -n observability 3000:3000
```

### Prometheus
```bash
kubectl port-forward svc/prometheus-service -n observability 9090:9090
```

### RabbitMQ (management)
```bash
kubectl port-forward svc/rabbitmq-service -n fgc 15672:15672
```
