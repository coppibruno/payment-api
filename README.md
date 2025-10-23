# 🏦 Gateway de Pagamentos - API Multi-Métodos

[![CI Pipeline](https://github.com/supero/payment-api/actions/workflows/ci.yml/badge.svg)](https://github.com/supero/payment-api/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/supero/payment-api/branch/main/graph/badge.svg)](https://codecov.io/gh/supero/payment-api)
[![License: ISC](https://img.shields.io/badge/License-ISC-blue.svg)](https://opensource.org/licenses/ISC)

Uma API completa para gateway de pagamentos que suporta PIX, Cartão de Crédito e Boleto, desenvolvida com NestJS, PostgreSQL, Redis, MongoDB e RabbitMQ.

## 🎯 Funcionalidades

- ✅ **Criação de cobranças** para PIX, Cartão de Crédito e Boleto
- ✅ **Consulta de status** com cache Redis para performance
- ✅ **Simulação de notificações** via RabbitMQ
- ✅ **Worker automático** para processar pagamentos
- ✅ **Logs de notificações** armazenados no MongoDB
- ✅ **Documentação Swagger** integrada
- ✅ **Docker Compose** para desenvolvimento
- ✅ **Múltiplos métodos de pagamento** com validações específicas

## 🛠️ Tecnologias Utilizadas

- **NestJS** - Framework Node.js
- **PostgreSQL** - Banco de dados principal (TypeORM)
- **MongoDB** - Armazenamento de logs
- **Redis** - Cache para consultas
- **RabbitMQ** - Mensageria e filas
- **Docker** - Containerização
- **Swagger** - Documentação da API

## 🚀 Como Executar

### Pré-requisitos

- Node.js 18+
- Docker e Docker Compose
- npm ou yarn

### 1. Clone o repositório

```bash
git clone <url-do-repositorio>
cd payment-api
```

### 2. Instale as dependências

```bash
npm install
```

### 3. Configure as variáveis de ambiente

```bash
cp env.example .env
```

Edite o arquivo `.env` com suas configurações:

```env
# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=postgres
DATABASE_NAME=payment_gateway

# MongoDB
MONGODB_URI=mongodb://localhost:27017/payment_gateway_logs

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# RabbitMQ
RABBITMQ_URL=amqp://localhost:5672

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=1h

# Application
PORT=3000
NODE_ENV=development
```

### 4. Execute com Docker (Recomendado)

```bash
# Inicia apenas os serviços de banco de dados
docker-compose -f docker-compose.dev.yml up -d

# Inicialize o banco de dados
docker exec -i payment_gateway_postgres_dev psql -U postgres -d payment_gateway < scripts/init-database.sql

# Instala dependências e executa a aplicação
npm install
npm run start:dev
```

### 5. Ou execute tudo com Docker

```bash
# Inicia todos os serviços incluindo a aplicação
docker-compose up -d
```

## 📚 Documentação da API

Após iniciar a aplicação, acesse:

- **Swagger UI**: http://localhost:3000/api
- **RabbitMQ Management**: http://localhost:15672 (admin/admin)

## 🔧 Endpoints Disponíveis

### POST /charges

Cria uma nova cobrança para PIX, Cartão de Crédito ou Boleto.

**Body (PIX):**

```json
{
  "payer_name": "João Silva",
  "payer_document": "12345678901",
  "amount": 10000,
  "description": "Pagamento de serviços",
  "payment_method": "pix"
}
```

**Body (Cartão de Crédito):**

```json
{
  "payer_name": "João Silva",
  "payer_document": "12345678901",
  "amount": 10000,
  "description": "Pagamento de serviços",
  "payment_method": "credit_card",
  "card_number": "4111111111111111",
  "card_expiry": "12/25",
  "card_cvv": "123",
  "card_holder_name": "João Silva",
  "installments": 1
}
```

**Body (Boleto):**

```json
{
  "payer_name": "João Silva",
  "payer_document": "12345678901",
  "amount": 10000,
  "description": "Pagamento de serviços",
  "payment_method": "bank_slip",
  "due_date": "2024-01-15T10:00:00.000Z"
}
```

**Response (PIX):**

```json
{
  "charge_id": "uuid",
  "pix_key": "pix-abc123",
  "expiration_date": "2024-01-02T10:00:00.000Z",
  "status": "pending",
  "payer_name": "João Silva",
  "payer_document": "12345678901",
  "amount": 10000,
  "description": "Pagamento de serviços",
  "created_at": "2024-01-01T10:00:00.000Z"
}
```

**Response (Cartão de Crédito):**

```json
{
  "charge_id": "uuid",
  "status": "pending",
  "payer_name": "João Silva",
  "payer_document": "12345678901",
  "amount": 10000,
  "description": "Pagamento de serviços",
  "installments": 1,
  "created_at": "2024-01-01T10:00:00.000Z"
}
```

**Response (Boleto):**

```json
{
  "charge_id": "uuid",
  "bank_slip_code": "23791234567890123456789012345678901234567890",
  "bank_slip_url": "https://example.com/boleto/123456",
  "due_date": "2024-01-15T10:00:00.000Z",
  "status": "pending",
  "payer_name": "João Silva",
  "payer_document": "12345678901",
  "amount": 10000,
  "description": "Pagamento de serviços",
  "created_at": "2024-01-01T10:00:00.000Z"
}
```

### GET /charges/:id

Consulta uma cobrança por ID (com cache Redis).

### POST /charges/simulate-payment

Simula uma notificação de pagamento.

**Body:**

```json
{
  "charge_id": "uuid-da-cobranca"
}
```

## 🏗️ Arquitetura

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   API Gateway   │    │   PostgreSQL    │    │     MongoDB     │
│    (NestJS)     │◄──►│   (Charges)     │    │   (Logs)        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       ▲                       ▲
         │                       │                       │
         ▼                       │                       │
┌─────────────────┐              │                       │
│     Redis       │              │                       │
│    (Cache)      │              │                       │
└─────────────────┘              │                       │
         │                       │                       │
         ▼                       │                       │
┌─────────────────┐              │                       │
│   RabbitMQ      │──────────────┘                       │
│  (Messages)     │                                      │
└─────────────────┘                                      │
         │                                              │
         ▼                                              │
┌─────────────────┐                                     │
│  Worker Process │─────────────────────────────────────┘
│ (Payment Consumer)│
└─────────────────┘
```

## 🧪 Testes

```bash
# Testes unitários
npm run test

# Testes com coverage
npm run test:cov

# Testes e2e
npm run test:e2e
```

## 📊 Monitoramento

### Health Check

- **Endpoint**: GET /health
- **Status**: Retorna status dos serviços

### Logs

- Logs estruturados com Winston
- Logs de notificações no MongoDB
- Logs de performance do Redis

## 🔒 Segurança

- Validação de dados com class-validator
- Sanitização de inputs
- Rate limiting (configurável)
- CORS configurado
- Headers de segurança

## 🚀 Deploy

### Produção

```bash
# Build da aplicação
npm run build

# Executa em produção
npm run start:prod
```

### Docker

```bash
# Build da imagem
docker build -t payment-gateway .

# Executa o container
docker run -p 3000:3000 payment-gateway
```

## 📈 Performance

- **Cache Redis**: Consultas de cobranças em < 10ms
- **Worker assíncrono**: Processamento de pagamentos em background
- **Connection pooling**: Otimização de conexões com banco
- **Lazy loading**: Carregamento sob demanda de módulos
- **Múltiplos métodos**: Suporte a PIX, Cartão de Crédito e Boleto

## 🤝 Contribuição

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## 📝 Licença

Este projeto está sob a licença ISC. Veja o arquivo `LICENSE` para mais detalhes.

## 🐰 Teste do RabbitMQ

Para testar a comunicação com RabbitMQ, consulte o guia completo em [TESTE-RABBITMQ.md](./TESTE-RABBITMQ.md).

**Resumo rápido:**

1. Inicie os serviços: `docker-compose -f docker-compose.dev.yml up -d`
2. Teste a conexão: `node test-rabbitmq.js`
3. Inicie a aplicação: `npm run start:dev`
4. Crie uma cobrança e simule o pagamento
5. Monitore no painel: http://localhost:15672 (admin/admin)

## 👨‍💻 Autor

Desenvolvido como teste técnico para avaliação de competências em:

- Modelagem de dados relacionais e não-relacionais
- Mensageria e filas (RabbitMQ)
- Cache e pub-sub com Redis
- APIs REST com NestJS
- Boas práticas de código e segurança
- Transações financeiras (PIX, Cartão de Crédito e Boleto)
