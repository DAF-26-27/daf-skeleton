# DAF — Skeleton do Projecto

Repositório esqueleto para a disciplina **Desenvolvimento de Aplicações com Frameworks (DAF)**.

**Stack:** Docker · PHP 8.4 · Apache · MySQL 8 · Redis 7 · Symfony 8.1 · Pest 4 · PHPStan 2

---

## Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado e a correr
- Git

---

## Setup inicial

### 1. Clonar o repositório

```bash
git clone <URL do repositório do grupo>
cd <nome-do-projecto>
```

### 2. Instalar as dependências PHP

```bash
docker run --rm \
  -v "$(pwd)":/app \
  -w /app \
  composer:2 \
  composer install --no-interaction
```

### 3. Subir o ambiente

```bash
docker compose up -d
```

Aguardar até os três contentores ficarem `healthy`:

```bash
docker compose ps
```

```
NAME           STATUS
<proj>_php     running (healthy)
<proj>_db      running (healthy)
<proj>_redis   running (healthy)
```

---

## Comandos do dia-a-dia

```bash
# Abrir shell no contentor PHP
docker compose exec php bash

# Correr os testes
docker compose exec php vendor/bin/pest

# Análise estática
docker compose exec php vendor/bin/phpstan analyse

# Limpar a cache do Symfony
docker compose exec php bin/console cache:clear

# Ver rotas registadas
docker compose exec php bin/console debug:router

# Criar uma migração
docker compose exec php bin/console doctrine:migrations:diff

# Aplicar migrações
docker compose exec php bin/console doctrine:migrations:migrate
```

---

## Verificar o ambiente

### Testar o Redis

```bash
docker compose exec php php -r "
\$r = new Redis();
\$r->connect('redis', 6379);
echo \$r->ping() . PHP_EOL;
"
```

Deve imprimir `+PONG`.

### PHPStan sem erros

```bash
docker compose exec php vendor/bin/phpstan analyse
# [OK] No errors
```

---

## Estrutura do projecto

```
.
├── config/                 # Configuração do Symfony
│   ├── packages/           # Configuração de bundles
│   └── routes.yaml         # Ponto de entrada de rotas
├── docker/
│   ├── apache/vhost.conf   # Virtual host Apache
│   └── php/Dockerfile      # Imagem PHP 8.4 + extensões
├── migrations/             # Migrações Doctrine
├── public/
│   └── index.php           # Front controller (não editar)
├── src/
│   ├── Controller/         # Controllers HTTP
│   ├── Entity/             # Entidades Doctrine (write side)
│   └── Kernel.php
├── tests/
│   ├── Feature/            # Testes de integração/feature
│   ├── Unit/               # Testes unitários
│   ├── Pest.php            # Configuração do Pest
│   └── bootstrap.php
├── .env                    # Variáveis de ambiente (não commitar segredos)
├── docker-compose.yml
├── phpstan.neon
└── composer.json
```

---

## Conventional Commits

Formato: `<tipo>(<âmbito>): <descrição>`

| Tipo | Quando usar |
|------|-------------|
| `feat` | Nova funcionalidade |
| `fix` | Correcção de bug |
| `chore` | Manutenção (deps, config) |
| `docs` | Documentação |
| `test` | Testes |
| `refactor` | Refactorização sem mudança de comportamento |

Exemplos:
```
feat(members): add POST /members endpoint
fix(loans): prevent double-borrowing of same copy
test(email): add invariant tests for Email value object
```

---

## Configuração do banco de dados

O `docker-compose.yml` cria automaticamente a base de dados `app` com o utilizador `app`/`app`.

Para personalizar (nome do projecto), editar em `docker-compose.yml`:

```yaml
db:
  environment:
    MYSQL_DATABASE: <nome-do-projecto>
    MYSQL_USER: <nome-do-projecto>
    MYSQL_PASSWORD: <nome-do-projecto>
```

E actualizar o `.env`:

```
DATABASE_URL=mysql://<nome>:<password>@db:3306/<nome>?serverVersion=8.0&charset=utf8mb4
```
