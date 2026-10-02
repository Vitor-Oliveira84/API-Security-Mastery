# 🔐 API SECURITY FUNDAMENTOS

Conceitos essenciais, arquiteturas e princípios de segurança em APIs modernas

---

## O QUE É UMA API?

### Definição
```
API = Application Programming Interface

Permite que aplicações se comuniquem entre si
Expõe funcionalidades de forma estruturada
Usa padrões como REST, GraphQL, gRPC, SOAP
```

### REST vs GraphQL vs gRPC

| Aspecto | REST | GraphQL | gRPC |
|---|---|---|---|
| **Protocolo** | HTTP/1.1 | HTTP/1.1 | HTTP/2 |
| **Formato** | JSON/XML | JSON | Protocol Buffers (binário) |
| **Query** | Fixa | Flexível | Pré-definida |
| **Performance** | Moderada | Boa | Excelente |
| **Segurança** | Padrão | Requer hardening | Menos comum |

---

## ANATOMIA DE UMA API REST

### Estrutura de Request

```
GET /api/v1/users/123?role=admin HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGc...
Content-Type: application/json
User-Agent: curl/7.68.0

{
  "filter": "active",
  "page": 1
}
```

**Componentes:**
- **Método HTTP**: GET, POST, PUT, DELETE, PATCH
- **Endpoint**: `/api/v1/users/123`
- **Query Parameters**: `?role=admin`
- **Headers**: Metadados (Auth, Content-Type)
- **Body**: Dados (JSON, XML)

### Estrutura de Response

```
HTTP/1.1 200 OK
Content-Type: application/json
X-Rate-Limit-Remaining: 99

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "admin"
}
```

**Componentes:**
- **Status Code**: 200 (OK), 401 (Unauthorized), 404 (Not Found), 500 (Error)
- **Headers**: Response metadata
- **Body**: Dados retornados

---

## AUTENTICAÇÃO EM APIS

### Tipos Comuns

#### 1. API Key
```
Simple but weak!

GET /api/users HTTP/1.1
X-API-Key: sk_live_xxx

Vulnerabilidades:
- Sem expiração
- Sem rotação automática
- Revogação lenta
```

#### 2. Bearer Token (JWT)
```
Industry standard

GET /api/users HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwicm9sZSI6ImFkbWluIn0.xxx

Vantagens:
- Stateless
- Auto-expiração
- Signed/encrypted

Riscos:
- Secret compromise
- Algorithm confusion
- Token theft
```

#### 3. OAuth 2.0
```
Delegated access

User → App → Authorization Server → API

Fluxo:
1. User clica "Login with Google"
2. Redireciona para Google Auth
3. User autoriza
4. Recebe code
5. App troca code por token
6. Usa token para acessar API
```

---

## VULNERABILIDADES PRINCIPAIS EM APIS

### Top 5 Críticas

| Vulnerabilidade | Impacto | Exemplo |
|---|---|---|
| **BOLA/IDOR** | Acesso dados alheios | GET /api/users/123 → Outro usuário |
| **Broken Auth** | Login bypass | JWT sem assinatura |
| **Excessive Data** | Fuga de info | API retorna password_hash |
| **Rate Limiting** | Força bruta | 1000 login attempts/min |
| **Injection** | RCE/Data breach | `/api/search?q='; DROP TABLE users--` |

---

## METODOLOGIA DE TESTE

### Reconnaissance (Recon)
```
1. Descobrir endpoints
2. Enumerar versões (/v1, /v2)
3. Mapear funcionalidades
4. Encontrar swagger/OpenAPI

Tools: Subfinder, httpx, FFUF, Burp Suite
```

### Autenticação
```
1. Testar sem credenciais
2. Testar credenciais inválidas
3. Testar JWT weaknesses
4. Testar session handling
5. Testar multi-factor auth bypass
```

### Autorização
```
1. BOLA: Mudar IDs em requests
2. IDOR: Enumerar recursos
3. Privilege escalation: Admin access
4. Horizontal escalation: Outro usuário
```

### Data Exposure
```
1. Analisar cada response
2. Procurar PII (emails, SSN)
3. Procurar secrets (API keys)
4. Procurar internals (IDs internos)
```

---

## FERRAMENTAS ESSENCIAIS

### Burp Suite
```
- Proxy: Interceptar requests
- Repeater: Teste manual
- Intruder: Fuzzing/Força bruta
- Scanner: Testes automáticos
```

### Command Line
```
curl - Fazer requests
jq - Parsear JSON
httpx - HTTP probing
Subfinder - Subdomain enum
```

### Programação
```
Python - Scripts de teste
Burp Suite Extensions - Automação
Postman - API testing
```

---

## CHECKLIST DE SEGURANÇA

### Antes de Publicar

- [ ] Autenticação obrigatória
- [ ] Autorização em cada endpoint
- [ ] Rate limiting ativado
- [ ] Input validation
- [ ] Output encoding
- [ ] HTTPS/TLS obrigatório
- [ ] Logging & monitoring
- [ ] Error handling seguro
- [ ] Headers de segurança
- [ ] CORS configurado

### Em Produção

- [ ] WAF (Web Application Firewall)
- [ ] DDoS protection
- [ ] Intrusion detection
- [ ] Penetration testing
- [ ] Security scanning regular
- [ ] Vulnerability disclosure program
- [ ] Incident response plan

---

## PRÓXIMOS PASSOS

Depois de dominar os fundamentos:

1. **[02-OWASP-API-TOP-10-DETALHADO.md](02-OWASP-API-TOP-10-DETALHADO.md)** - Vulnerabilidades específicas
2. **[03-TECNICAS-AVANCADAS-APIS.md](03-TECNICAS-AVANCADAS-APIS.md)** - GraphQL, gRPC, WebSocket
3. **[04-BURP-SUITE-PARA-APIS.md](04-BURP-SUITE-PARA-APIS.md)** - Ferramentas práticas
4. **[05-PAYLOAD-DATABASE-APIS.md](05-PAYLOAD-DATABASE-APIS.md)** - 250+ payloads
5. **[06-RECON-ENUMERATION-APIS.md](06-RECON-ENUMERATION-APIS.md)** - Descoberta avançada
6. **[07-SCRIPTS-AUTOMACAO-APIS.md](07-SCRIPTS-AUTOMACAO-APIS.md)** - Automação

---

**🎯 Início de sua jornada em API Security!**
