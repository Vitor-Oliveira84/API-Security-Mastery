# 🔍 RECONNAISSANCE & ENUMERATION - API Security

Descobrir, mapear e documentar APIs antes de testar

---

## 1️⃣ DESCOBRINDO APIS

### Google Dorks

```bash
# Encontrar APIs públicas
site:api.example.com
site:api.example.com /v1
site:api.example.com /v2

# Encontrar Swagger/OpenAPI
site:example.com swagger.json
site:example.com openapi.json
site:example.com /api-docs
```

### Padrões Comuns

```
/api
/api/v1
/api/v2
/api/v3
/apis
/graphql
/rest
/soap
/rpc
/oembed
/json
/.well-known/openapi.json
/swagger.json
/openapi.json
```

---

## 2️⃣ MAPEANDO ENDPOINTS

### Com Browser

```
1. Abra site target
2. Abra DevTools (F12)
3. Network tab
4. Faça ações no site (login, comprar, etc)

Todos os requests aparecem:
- POST /api/auth/login
- GET /api/products
- POST /api/orders
- etc
```

### Com Burp Suite

```
1. Proxy → HTTP History
2. Filter: /api/
3. Todos os endpoints visitados

Organize por:
- Método (GET, POST, PUT, DELETE)
- Path
- Authentication required?
```

### Automatizado com FFUF

```bash
# Descobrir endpoints
ffuf -w wordlist.txt -u https://api.example.com/FUZZ -o results.txt

Wordlist comum:
/api/users
/api/products
/api/orders
/api/admin
/api/settings
/api/profile
/api/auth
```

---

## 3️⃣ ENUMERANDO VERSÕES DE API

### Testar Múltiplas Versões

```bash
# V1
curl https://api.example.com/v1/users

# V2
curl https://api.example.com/v2/users

# V3
curl https://api.example.com/v3/users

# SEM versionamento
curl https://api.example.com/users

# Path versionamento
curl https://api.example.com/2021/users
curl https://api.example.com/2022/users
```

### Header Versionamento

```bash
curl -H "API-Version: 1" https://api.example.com/users
curl -H "API-Version: 2" https://api.example.com/users
curl -H "X-API-Version: 1" https://api.example.com/users
```

---

## 4️⃣ DOCUMENTAÇÃO DE API

### Swagger/OpenAPI

```bash
# Encontrar Swagger
curl https://api.example.com/swagger.json
curl https://api.example.com/swagger.yaml
curl https://api.example.com/openapi.json
curl https://api.example.com/.well-known/openapi.json

# Resultado: Schema completo!
# - Todos os endpoints
# - Parâmetros
# - Formatos de response
# - Modelos de dados
```

### RAML/Postman

```bash
# RAML
curl https://api.example.com/api.raml

# Postman collection
curl https://api.example.com/postman.json
```

---

## 5️⃣ GRAPHQL ENUMERATION

### Introspection Query

```bash
curl -X POST https://api.example.com/graphql \
  -H "Content-Type: application/json" \
  -d '{
    "query": "{ __schema { types { name fields { name type } } } }"
  }'

Retorna:
- Todos os tipos
- Todos os campos
- Tipos de dados
- Query/Mutation/Subscription disponíveis
```

### GraphQL Playground

```
Acessar:
https://api.example.com/graphql

Se abrir = Playground ativo!

Você pode:
- Testar queries
- Ver schema
- Explorar campos
- Tudo visualmente
```

---

## 6️⃣ ENUMERANDO PARÂMETROS

### Parâmetros Comuns

```
?id=
?user_id=
?email=
?username=
?search=
?query=
?filter=
?sort=
?page=
?limit=
?offset=
?role=
?admin=
?debug=
?test=
```

### Headers de Autenticação

```
Authorization: Bearer token
X-API-Key: key
X-Auth-Token: token
Authorization-Token: token
X-Access-Token: token
Token: token
```

---

## 7️⃣ RATE LIMIT DETECTION

### Testando Limites

```bash
# Fazer múltiplas requisições
for i in {1..100}; do
  curl https://api.example.com/users
  echo "Request $i"
done

Procurar por:
- 429 Too Many Requests
- 403 Forbidden
- X-RateLimit-Remaining header
- X-RateLimit-Reset header
```

### Headers de Rate Limit

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 50
X-RateLimit-Reset: 1609459200

RateLimit-Limit: 1000
RateLimit-Remaining: 999
RateLimit-Reset: 60
```

---

## 8️⃣ TESTING AUTHENTICATION

### Tipos de Auth

```
# API Key
curl -H "X-API-Key: xxx" https://api.example.com/users

# Bearer Token
curl -H "Authorization: Bearer token" https://api.example.com/users

# Basic Auth
curl -u username:password https://api.example.com/users

# OAuth
OAuth flow com redirect
```

### Sem Autenticação

```bash
# Testar sem header
curl https://api.example.com/users

# Testar com token vazio
curl -H "Authorization: Bearer" https://api.example.com/users

# Testar com token inválido
curl -H "Authorization: Bearer xxx" https://api.example.com/users

Se retorna dados = VULNERÁVEL!
```

---

## 9️⃣ ENUMERANDO DADOS SENSÍVEIS

### Procurar em Responses

```
- password_hash
- api_key
- secret_key
- access_token
- internal_id
- database_version
- server_version
- environment
- debug_info
```

### Procurar em Errors

```
Stack traces (mostram estrutura)
SQL queries (mostram banco)
Database version
Framework version
Internal paths
```

---

## 🔟 DOCUMENTAÇÃO RECON

### Template

```
TARGET: api.example.com

ENDPOINTS DESCOBERTOS:
- GET /api/users
- POST /api/users
- GET /api/users/{id}
- PUT /api/users/{id}
- DELETE /api/users/{id}
- GET /api/products
- POST /api/orders
- GET /api/orders/{id}

AUTENTICAÇÃO:
- Tipo: Bearer Token (JWT)
- Header: Authorization
- Formato: "Bearer token"

VERSÕES:
- /v1/ (deprecated)
- /v2/ (current)
- /v3/ (beta)

RATE LIMITS:
- 100 requests/minute
- Header: X-RateLimit-Remaining

SWAGGER:
✅ /swagger.json (público!)

VULNERABILIDADES ENCONTRADAS:
[ ] Introspection habilitado
[ ] BOLA em /api/users/{id}
[ ] SQLi em /api/search?q=
[ ] Rate limit fraco
[ ] Sem validação CORS

PRÓXIMAS AÇÕES:
[ ] Testar BOLA
[ ] Testar SQLi
[ ] Testar autenticação
[ ] Testar rate limit bypass
```

---

## SCRIPT: RECON AUTOMÁTICO

```bash
#!/bin/bash
# api-recon.sh

TARGET=$1

echo "[*] Iniciando API recon em $TARGET"

# 1. Swagger
echo "[1] Procurando Swagger..."
curl -s https://$TARGET/swagger.json | head -20

# 2. OpenAPI
echo "[2] Procurando OpenAPI..."
curl -s https://$TARGET/openapi.json | head -20

# 3. GraphQL
echo "[3] Procurando GraphQL..."
curl -s -X POST https://$TARGET/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __schema { types { name } } }"}' | head -20

# 4. Versões
echo "[4] Testando versões..."
for v in v1 v2 v3 v4; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://$TARGET/api/$v/users)
  echo "  /api/$v: $STATUS"
done

echo "[+] Recon completo!"
```

---

**Próximo:** Volte para [API-SECURITY-MASTER-README.md](./API-SECURITY-MASTER-README.md)

