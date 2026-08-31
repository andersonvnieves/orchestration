# Orquestração local

Este projeto executa todos os serviços do TC03 em em um cluster Kubernetes.

## Pré-requisitos

- Docker Desktop em execução;
- portas locais `1433`, `5672`, `15672`, `8080`, `8081` e `8082` disponíveis.

## Executar localmente

Na pasta `orquestration`:

```powershell
docker compose up --build
```

Para executar em segundo plano, use `docker compose up --build -d`. A primeira execução cria as imagens e os volumes persistentes de SQL Server e RabbitMQ.

| Componente | Endereço local |
| --- | --- |
| User Service | `http://localhost:8080` |
| Catalog Service | `http://localhost:8081` |
| Payment Service | `http://localhost:8082` |
| RabbitMQ Management | `http://localhost:15672` |
| SQL Server | `localhost,1433` |

As APIs expõem Swagger somente no ambiente `Development`. O compose atual não define esse ambiente para elas; para consultar a documentação durante desenvolvimento, execute os serviços diretamente conforme seus READMEs.

## Verificar e encerrar

```powershell
docker compose ps
docker compose logs -f
docker compose down
```

`docker compose down` preserva os volumes. Para remover também os dados locais de SQL Server e RabbitMQ, execute `docker compose down -v`.



aws cloudformation deploy --template-file fgc-template-cloudformation.yaml --stack-name fgc-stack --capabilities CAPABILITY_IAM

aws cloudformation delete-stack  --stack-name fgc-stack


aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 257500279765.dkr.ecr.us-east-1.amazonaws.com


1. Redis Cache em Frente ao Catálogo de Jogos
O Redis foi adotado na camada de acesso ao catálogo de jogos para absorver o alto volume de requisições de leitura e proteger a fonte de dados principal.

Mitigação de Carga (Offloading): Como o catálogo possui um volume total pequeno — permitindo que todo o conjunto de dados seja carregado integralmente em memória —, a aplicação elimina consultas recorrentes à base de origem, concentrando as leituras diretamente no Redis.

Consistência por Atualização Proativa (Write-Through/Cache Aside dinâmico): A estratégia de invalidação e atualização não depende de TTL. Sempre que ocorre uma operação de update ou insert na base principal, o cache é imediatamente atualizado de forma síncrona ou assíncrona, garantindo dados sempre frescos sem expirar por tempo.

Garantia de Resiliência e Throughput: Mesmo operando com latências na faixa de milissegundos devido a camadas de rede e serialização, o Redis suporta uma densidade massiva de conexões simultâneas, blindando o banco principal de picos de tráfego no catálogo.


2. MongoDB para a Biblioteca de Usuários
O MongoDB foi selecionado para persistir os dados da biblioteca de usuários devido à natureza semi-estruturada e altamente dinâmica dos dados de perfil, inventário e progresso.

Flexibilidade de Schema: Diferente de modelos relacionais rígidos, o modelo orientado a documentos do MongoDB acomoda facilmente variações nos dados dos usuários (como preferências customizadas, diferentes tipos de itens na biblioteca e metadados que evoluem com novas funcionalidades do sistema).

Escalabilidade Horizontal: O suporte nativo do MongoDB a sharding e replicação permite que a base de usuários escale de forma eficiente conforme a base de clientes cresce, distribuindo a carga de escrita e leitura de forma balanceada.

Agilidade no Desenvolvimento: A representação dos dados em formato JSON/BSON nativo simplifica o mapeamento objeto-documento na aplicação, acelerando o ciclo de entrega das funcionalidades da conta do usuário.


3. Stack Open Source de Observabilidade: Prometheus, Loki e Grafana
A escolha por uma stack open source composta por Prometheus, Loki e Grafana atende à necessidade de monitoramento centralizado, correlacionado e de baixo custo operacional, tendo como premissa fundamental a soberania sobre os dados.

Soberania e Controle Total dos Dados: A preferência por uma stack open source on-premise/self-hosted garante que todas as métricas e logs permaneçam estritamente sob o nosso domínio e infraestrutura. Isso evita a dependência de plataformas de terceiros (SaaS), elimina custos imprevisíveis de ingestão ou retenção por volume e atende a rigorosos critérios de privacidade, governança e conformidade de dados.

Prometheus (Métricas): Responsável pela coleta e armazenamento de métricas de infraestrutura e aplicação baseadas em séries temporais. Sua arquitetura pull e a eficiência do TSDB (Time Series Database) garantem alertas rápidos e visibilidade sobre a saúde dos containers e serviços.

Loki (Logs): Adotado para a centralização de logs por sua eficiência de armazenamento. Diferente de soluções tradicionais que indexam o conteúdo completo dos logs, o Loki indexa apenas os metadados (labels), reduzindo drasticamente o consumo de armazenamento e facilitando a busca agregada.

Grafana (Visualização): Atua como a camada unificada de dashboards. Ele consolida as métricas do Prometheus e os logs do Loki em uma única interface, permitindo a correlação visual de eventos (ex: identificar se um pico de erro nos logs coincide com a saturação de CPU monitorada pelo Prometheus).