# 🎯 BURP SUITE PARA TESTING DE APIS

Configuração completa e técnicas específicas para explorar APIs

---

## 1️⃣ SETUP BURP PARA API TESTING

### Instalação & Proxy

```bash
# Baixar Burp Suite Community
https://portswigger.net/burp/communitydownload

# Ou usar Professional (mais features)
Licença: $399/ano

# Abrir Burp
Proxy → Options → Listener
Bind to: 127.0.0.1:8080
```

### Configurar Browser para APIs

```
1. Browser → Network Settings
2. Manual proxy configuration
3. HTTP Proxy: 127.0.0.1
4. Port: 8080

5. Check "Use this proxy for all protocols"

6. Certificate: 
   - Proxy → Options → Import CA certificate
   - Instale em Trusted Root
```

---

## 2️⃣ INTERCEPTAR REQUISIÇÕES DE API

### Proxy Listening

```
1. Proxy → Intercept → Is ON
2. Fazer requisição da API:
   curl -H "Authorization: Bearer token" https://api.example.com/users

3. Intercepta em Burp:
   GET /users HTTP/1.1
   Host: api.example.com
   Authorization: Bearer token
   Content-Type: application/json
```

### HTTP vs HTTPS

```
APIs costumam usar HTTPS.

Burp automatically handles se:
1. Usar Burp como proxy (recomendado)
2. Importar CA certificate

Se não funcionar:
- curl --proxy http://127.0.0.1:8080 \
  --cacert /path/to/burp-cert.pem \
  https://api.example.com
```

---

## 3️⃣ REPEATER - TESTE MANUAL

### Configuração

```
1. Capture qualquer requisição API
2. Send to Repeater (Ctrl+R)
3. Agora pode editar e re-enviar quantas vezes quiser

Exemplo:
```

### Teste 1: BOLA (Broken Object Level Authorization)

```
GET /api/users/123/profile
Authorization: Bearer token_user_123

Resposta:
{
  "id": 123,
  "name": "Your Name",
  "email": "your@example.com"
}

TESTE:
Mude ID para 124:
GET /api/users/124/profile
Authorization: Bearer token_user_123

Se retorna dados de outro user = BOLA!
```

### Teste 2: IDOR em POST

```
POST /api/orders/123/update
Authorization: Bearer token

Mude ID:
POST /api/orders/124/update

Se consegue atualizar ordem de outro user = IDOR!
```

---

## 4️⃣ INTRUDER - FUZZING E FORÇA BRUTA

### Setup Intruder

```
1. Capture requisição
2. Send to Intruder (Ctrl+I)
3. Positions → Marque o campo a fuzzear com § §

Exemplo:
GET /api/users/§123§/profile
```

### Ataque 1: IDOR Enumeration

```
Positions:
GET /api/users/§123§/profile

Payload type: Numbers
From: 1
To: 1000

Attack type: Sniper

Resultados:
- Status 200 = usuário existe
- Status 404 = não existe
- Tamanho diferente = dados diferentes

Procure por admin, root, sensitive data
```

### Ataque 2: Força Bruta JWT

```
Intercepte JWT:
Authorization: Bearer eyJhbGci...

Send to Intruder:
Authorization: Bearer §eyJhbGci...§

Payload: Wordlist com senhas comuns

Para cada payload, decodifique e verifique se valid
```

### Ataque 3: Fuzzing de Parâmetros

```
GET /api/users?§id=123§

Payloads:
id=123
id=124
id=999
id=admin
id=root
id=-1
id=0

Attack: Grep Match
Match: "error" ou "success"
```

---

## 5️⃣ DECODER - ANÁLISE DE DADOS

### Base64 Decoding

```
Muitas APIs usam Base64 para:
- Tokens JWT
- Credentials codificadas
- Dados sensíveis

No Decoder:
1. Cole token: eyJhbGc...
2. Decoder → Decode as → Base64
3. Resultado: {"alg":"HS256"...}
```

### URL Encoding

```
Alguns dados em URL:
search=test%20value

Decoder → Decode as → URL

Resultado: search=test value
```

### HTML Entities

```
API pode retornar:
&lt;script&gt;

Decoder → HTML entities

Resultado: <script>
```

---

## 6️⃣ COMPARER - ANÁLISE DE RESPOSTAS

### Comparar Dois Requests

```
1. Faça request normal:
GET /api/users/123

2. Faça request alterada:
GET /api/users/124

3. Selecione ambas
4. Tools → Compare (Ctrl+Shift+C)

Resultado mostra:
- Dados diferentes
- Campos adicionais
- Tamanho diferente
```

### Encontrar Segredos

```
Resposta 1 (seu usuário):
{"id":123,"name":"You","email":"you@example.com"}

Resposta 2 (outro usuário):
{"id":124,"name":"Other","email":"other@example.com","api_key":"sk_xxx"}

Comparer destaca:
- api_key foi exposto!
```

---

## 7️⃣ SCANNER - TESTE AUTOMÁTICO

### Active Scanning

```
1. Proxy → Intercept requisição
2. Do Active Scan (Ctrl+Shift+A)

Testa automaticamente:
- SQL Injection
- XSS
- CSRF
- XXE
- Misconfigurations
```

### Target Scope

```
1. Target → Scope
2. Add: api.example.com

3. Scanner → Scan this host (Ctrl+Shift+T)

Resultado:
- Vulnerabilidades encontradas
- Confiança de cada um
- CVSS score
```

---

## 8️⃣ MACROS - AUTENTICAÇÃO AUTOMÁTICA

### Problema

```
Teste manual é tedioso:
1. Login para pegar token
2. Usar token em requests
3. Token expira
4. Login novamente

Solução: Macros automatizam
```

### Criar Macro

```
1. Proxy → Options → Session Handling Rules
2. New → Add
3. Select "Run a macro"

4. Macro Recorder → Start recording
5. Fazer login:
   POST /api/auth/login
   {"username":"test","password":"test"}
   
   Resposta:
   {"token":"xxx"}

6. Stop recording

7. Extract token do response:
   Session handling → Extract Session Cookie
   Look in: Response body
   Parameter: token
   Name: token
```

### Usar Macro

```
Agora em qualquer teste:
- Repeater automaticamente:
  1. Faz login
  2. Extrai token
  3. Usa em requisição

Nunca precisa fazer login manualmente de novo!
```

---

## 9️⃣ EXTENSIONS - FERRAMENTAS AVANÇADAS

### Active Scan++ (Gratuito)

```
Adiciona mais checks que Community não tem:
- Bypass de autenticação
- Lógica de negócio
- GraphQL injection
- Cripto issues
```

### Autorize (Gratuito)

```
Testa Broken Access Control automaticamente:

1. Instale extension
2. Faça requisições normal
3. Autorize automaticamente captura
4. Testa sem autenticação
5. Mostra o que é acessível sem auth
```

### Logger++ (Gratuito)

```
Loga TODAS as requisições:
- Request/Response
- Headers
- Payloads
- Status codes

Útil para:
- Auditar o que foi feito
- Replicar vulnerabilidades
- Documentação
```

---

## 🔟 WORKFLOW COMPLETO DE API TESTING

### 1. Reconnaissance

```
- Descobrir endpoints: /api, /api/v1, etc
- Mapear funcionalidades
- Encontrar parâmetros
```

### 2. Autenticação

```
- Testar login
- Pegar token
- Configurar Burp para usar token automaticamente
```

### 3. BOLA/IDOR

```
- Alterar IDs em requisições
- Usar Intruder para enumerar
- Procurar por dados de outros usuários
```

### 4. Autenticação Broken

```
- Testar sem token
- JWT sem assinatura
- Tokens que não expiram
```

### 5. Data Exposure

```
- Analisar cada resposta
- Procurar campos que não deveriam estar lá
- API keys, senhas, tokens
```

### 6. Rate Limiting

```
- Força bruta com Intruder
- Sem delay between requests
- Ver se consegue fazer 1000 tentativas
```

### 7. Injection

```
- SQL em parâmetros
- Command injection
- XXE em XML
- SSTI em templates
```

---

## 📊 CHECKLIST BURP PARA APIS

```
☐ Proxy setup (localhost:8080)
☐ Certificate installed
☐ Capture primeiro request
☐ Repeater - teste manual básico
☐ Intruder - enumeration
☐ Scanner - scan automático
☐ Macros - autenticação automática
☐ Decoder - analise tokens
☐ Comparer - compare respostas
☐ Extensions instaladas
☐ Documentar achados
```

---

**Próximo:** Volte para [API-SECURITY-MASTER-README.md](./API-SECURITY-MASTER-README.md)

