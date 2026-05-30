# Auth-Service

Microsserviço responsável por autenticação e geração de tokens JWT. Gerencia o fluxo de login e registro de usuários, emite tokens via cookie `httpOnly`, e fornece endpoint de validação de token utilizado pelo `gateway-service`.

## Tecnologias

- Java 25
- Spring Boot 4.0.3
- Spring Cloud 2025.1.0 (OpenFeign)
- JJWT (JSON Web Tokens com HMAC-SHA256)
- Micrometer / Prometheus
- Lombok

## Endpoints

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/auth/login` | Autentica usuário e retorna JWT em cookie |
| `POST` | `/auth/register` | Registra novo usuário |
| `GET` | `/auth/whoiam` | Retorna dados do usuário (header `id-account`) |
| `POST` | `/auth/solve` | Valida e decodifica um token JWT |
| `GET` | `/auth/logout` | Encerra sessão (limpa o cookie) |
| `GET` | `/auth/health-check` | Health check |

### POST `/auth/login`

**Request body:**
```json
{
  "email": "joao@email.com",
  "password": "senha123"
}
```

**Response:** `200 OK` com `Set-Cookie: __store_jwt_token=<jwt>; HttpOnly; Secure; SameSite=None; Path=/`

### POST `/auth/register`

**Request body:**
```json
{
  "name": "João Silva",
  "email": "joao@email.com",
  "password": "senha123"
}
```

**Response:** `201 Created`

### POST `/auth/solve`

Usado internamente pelo `gateway-service` para validar tokens.

**Request body:**
```json
{
  "token": "<jwt>"
}
```

**Response:**
```json
{
  "idAccount": "uuid",
  "role": "USER"
}
```

## Token JWT

| Campo | Descrição |
|-------|-----------|
| `id` | ID da conta |
| `subject` | Nome do usuário |
| `email` | E-mail (claim customizado) |
| `role` | Perfil do usuário (claim customizado) |
| `issuer` | `Insper::PMA` |
| `expiration` | Tempo atual + duração (padrão: 24h) |
| Algoritmo | HMAC-SHA256 |

## Comunicação entre serviços

| Tipo | Serviço | Detalhe |
|------|---------|---------|
| Feign (saída) | `account-service` | Cria conta no registro, valida credenciais no login, busca dados no whoiam |
| Feign (entrada) | `gateway-service` | Chama `/auth/solve` para validar tokens antes de rotear requisições |

## Variáveis de ambiente

| Variável | Descrição | Padrão |
|----------|-----------|--------|
| `JWT_SECRET_KEY` | Chave secreta HMAC em Base64 | — |
| `JWT_HTTP_ONLY` | Define flag `httpOnly` no cookie | `true` |

## Executando localmente

### Com Docker (recomendado)

```bash
docker compose up
```

### Build manual

```bash
mvn clean package -DskipTests
java -jar target/auth-service-1.0.0.jar
```

O serviço sobe na porta `8080`.

## Métricas

Prometheus disponível em `/auth/actuator/prometheus`.

## Dependências externas

- **`store:auth:1.0.0`** — biblioteca de contratos
- **`store:account:1.0.0`** — biblioteca de contratos do account
- **`account-service`** — gerenciamento de contas e verificação de credenciais
