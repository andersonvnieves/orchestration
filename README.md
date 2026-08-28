# Orquestração local

Este projeto executa todos os serviços do TC02 em contêineres Docker. O arquivo `compose.yml` inicia SQL Server, RabbitMQ e os quatro serviços: User, Catalog, Payment e Notification.

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