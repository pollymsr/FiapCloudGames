# 🎮 Fiap Cloud Games (FCG) - Orquestração e Infraestrutura

Bem-vindo ao repositório central de **Infraestrutura e Orquestração** do projeto Fiap Cloud Games (Fase 2 do Tech Challenge). 

Este repositório tem como objetivo orquestrar a execução de toda a arquitetura de microsserviços do e-commerce de jogos da FIAP, utilizando **Docker Compose** e manifestos **Kubernetes**.

---

## 🏛️ Arquitetura do Sistema

Na Fase 2, evoluímos a nossa aplicação monolítica para uma arquitetura distribuída em **Microsserviços**, orientada a eventos. O ecossistema é composto por 4 APIs independentes que se comunicam de forma assíncrona através de um Message Broker (RabbitMQ):

1. **[UsersAPI](https://github.com/pollymsr/UsersAPI)**: Responsável pela autenticação (JWT) e gestão de usuários (Roles: Admin / User).
2. **[CatalogAPI](https://github.com/pollymsr/CatalogAPI)**: Core do e-commerce. Gerencia o catálogo de jogos, promoções e a efetivação de compras.
3. **[PaymentsAPI](https://github.com/pollymsr/PaymentsAPI)**: Worker assíncrono isolado que consome a fila de pedidos e processa os pagamentos.
4. **[NotificationsAPI](https://github.com/pollymsr/NotificationsAPI)**: Worker assíncrono que escuta eventos globais do sistema e simula o envio de e-mails para os usuários.

---

## 🚀 Como Executar o Projeto Localmente

Requisitos: Ter o **Docker Desktop** e o **Git** instalados na sua máquina.

### 1. Clonando os Repositórios
Para que o `docker-compose` funcione corretamente, todos os repositórios devem ser clonados dentro de uma mesma pasta "pai" (ex: `fiap-repos`), mantendo a seguinte estrutura de pastas:
```text
/fiap-repos
  ├── Tech-Challenge-Fiap-Cloud-Games (Este repositório)
  ├── UsersAPI
  ├── CatalogAPI
  ├── PaymentsAPI
  └── NotificationsAPI
```

### 2. Subindo os Containers
Acesse a pasta deste repositório de infraestrutura via terminal e execute o comando:
```bash
docker-compose up -d --build
```
Isso iniciará o RabbitMQ e os 4 microsserviços. Os bancos de dados SQLite de cada API serão criados e **povoados automaticamente** (Data Seeding) na primeira execução.

---

## 🧪 Passo a Passo para Testes (Banca Avaliadora)

Para testar o fluxo completo de negócio (Orientação a Eventos), siga os passos abaixo:

### Passo 1: Autenticação (Gerando o Token)
1. Acesse o Swagger da **UsersAPI**: `http://localhost:5001/swagger`
2. Vá na rota `POST /api/auth/login`.
3. O banco de dados já possui dois usuários criados por padrão para testes:
   - **Administrador**: `admin@fiap.com.br` | Senha: `Admin123!`
   - **Usuário Comum**: `user@fiap.com.br` | Senha: `User123!`
4. Faça o login com o Administrador e **copie o Token JWT** gerado na resposta.

### Passo 2: O Catálogo de Jogos
1. Acesse o Swagger da **CatalogAPI**: `http://localhost:5002/swagger`
2. Clique no botão verde **Authorize** (no topo direito), digite a palavra `Bearer `, cole o seu Token e clique em *Authorize*.
3. Use a rota `GET /api/games` e clique em *Execute*. O banco de dados já subirá com 3 jogos incríveis pré-cadastrados (Elden Ring, Black Myth Wukong e Hellblade II).
4. Copie o **Id** de um dos jogos.

### Passo 3: O Fluxo de Compra e Eventos em Background
1. Ainda no Swagger da **CatalogAPI**, vá na rota `POST /api/games/{id}/buy`.
2. Cole o **Id** do jogo que você copiou e execute.
3. Você receberá um `202 Accepted` ("Pedido de compra recebido com sucesso!").

Neste exato momento, a mágica dos Microsserviços acontece:
- A `CatalogAPI` publicou um evento `OrderPlacedEvent` no RabbitMQ.
- A `PaymentsAPI` interceptou o pedido, processou o pagamento e publicou o evento `PaymentProcessedEvent`.
- A `CatalogAPI` interceptou o pagamento e adicionou o jogo na Biblioteca do usuário.
- A `NotificationsAPI` escutou o sucesso e disparou o E-mail de confirmação.

### Passo 4: Validando o Sucesso
1. Verifique os logs do Docker para ver os Workers trabalhando em background:
```bash
docker logs payments-api
docker logs notifications-api
```
2. No Swagger da `CatalogAPI`, execute a rota `GET /api/games/library`. Você verá que o jogo comprado já está disponível na biblioteca do usuário!

---

Projeto desenvolvido por Pollyana para a fase 2 da pós-graduação.