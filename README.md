# 🔐 API SECURITY MASTERY

Guia profundo em segurança de APIs com 250+ payloads, técnicas avançadas para GraphQL/gRPC/WebSocket, e 6 scripts de automação

---

## 📚 DOCUMENTAÇÃO (6 Guias)

| # | Documento | Conteúdo |
|---|---|---|
| 02 | [OWASP-API-TOP-10-DETALHADO.md](docs/02-OWASP-API-TOP-10-DETALHADO.md) | 10 vulnerabilidades + Burp passo-a-passo + case studies |
| 03 | [TECNICAS-AVANCADAS-APIS.md](docs/03-TECNICAS-AVANCADAS-APIS.md) | GraphQL, gRPC, WebSocket, OAuth, JWT, Webhooks |
| 04 | [BURP-SUITE-PARA-APIS.md](docs/04-BURP-SUITE-PARA-APIS.md) | Setup Proxy, Repeater, Intruder, Decoder, Scanner, Macros |
| 05 | [PAYLOAD-DATABASE-APIS.md](docs/05-PAYLOAD-DATABASE-APIS.md) | 250+ payloads (REST, GraphQL, gRPC, WebSocket, OAuth, JWT, CORS, SSRF) |
| 06 | [RECON-ENUMERATION-APIS.md](docs/06-RECON-ENUMERATION-APIS.md) | API discovery, endpoint mapping, version enum, rate limit detection |
| 07 | [SCRIPTS-AUTOMACAO-APIS.md](docs/07-SCRIPTS-AUTOMACAO-APIS.md) | 6 scripts Python (discovery, BOLA, JWT, SQLi, rate limit, GraphQL) |

---

## 📊 ESTATÍSTICAS

| Métrica | Valor |
|---|---|
| **Linhas de conteúdo** | 5,500+ |
| **Documentos** | 6 |
| **Vulnerabilidades** | 20+ |
| **Payloads** | 250+ |
| **Scripts prontos** | 6 |
| **Técnicas avançadas** | 6 arquiteturas |
| **Recompensa potencial** | $500-15K/bug |
| **CVSS médio** | 8.0 |

---

## 🎯 VULNERABILIDADES POR SEVERIDADE

### CRÍTICO (9.0-10.0)
- GraphQL Introspection + N+1
- JWT Algorithm Confusion
- Webhook SSRF
- SQL Injection em APIs
- Command Injection

### ALTO (7.0-8.9)
- BOLA/IDOR
- Broken Authentication
- Excessive Data Exposure
- gRPC Reflection
- OAuth Redirect Bypass

### MÉDIO (4.0-6.9)
- Rate Limiting Bypass
- WebSocket CSRF
- CORS Misconfiguration
- Improper Assets Management

---

## 💰 RECOMPENSA POTENCIAL

| Vulnerabilidade | Min | Max | Tempo |
|---|---|---|---|
| BOLA | $500 | $5K | 30min-2h |
| Broken Auth | $1K | $10K | 1-4h |
| GraphQL Injection | $500 | $5K | 1-2h |
| JWT Exploit | $2K | $10K | 1.5h |
| OAuth Bypass | $5K | $15K | 2-3h |
| Webhook SSRF | $2K | $10K | 1h |

---

## 🎓 LEARNING PATH (8 SEMANAS)

### Semana 1-2: Fundações
- HTTP/REST basics
- API architecture
- Burp Suite setup

### Semana 3-4: OWASP API Top 10
- Testar cada vulnerabilidade
- BOLA exploitation
- Authentication testing

### Semana 5-6: Técnicas Avançadas
- GraphQL attacks
- gRPC exploitation
- JWT attacks

### Semana 7-8: Prática
- Usar scripts de automação
- Teste em APIs públicas
- Bug bounty submissions

---

## 🚀 QUICK START

```bash
# 1. Setup Burp
burpsuite &

# 2. Configurar proxy
# Proxy → Options → 127.0.0.1:8080

# 3. Descobrir endpoints
python3 07-SCRIPTS-AUTOMACAO-APIS.md # discover-api.py

# 4. Testar BOLA
python3 bola-tester.py https://api.target.com

# 5. Analisar GraphQL
python3 graphql-introspect.py https://api.target.com/graphql
```

---

## ✅ CHECKLIST

- [ ] Entende arquitetura de APIs
- [ ] Burp Suite configurado
- [ ] OWASP API Top 10 memorizado
- [ ] Testou GraphQL Introspection
- [ ] Testou BOLA em uma API
- [ ] JWT bruteforce bem-sucedido
- [ ] Rate limit bypass explorado
- [ ] Webhook SSRF testado
- [ ] Scripts de automação funcionando
- [ ] Primeira vulnerabilidade encontrada

---

## 🔗 RECURSOS

- [OWASP API Security](https://owasp.org/www-project-api-security/)
- [API Hacking Book](https://www.amazon.com/API-Hacking-Exposed-vulnerabilities-exploitation-ebook/dp/B095SLQSRD)
- [PortSwigger GraphQL](https://portswigger.net/web-security/graphql)
- [HackerOne Reports](https://hackerone.com/reports)

---

## 🤖 Gerado com Claude Code
Este guia foi criado com a ajuda de AI para garantir precisão técnica e cobertura completa.

**Status:** ✅ Completo | **Última atualização:** 2026-10-02 | **Versão:** 1.0
