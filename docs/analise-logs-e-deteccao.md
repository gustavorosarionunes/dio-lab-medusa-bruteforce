# 🔎 Guia de Análise de Logs e Detecção de Força Bruta

Identificar e responder a ataques de força bruta, dicionário e password spraying requer monitoramento contínuo dos logs do sistema operacional e das aplicações.

---

## 1. Onde Localizar os Logs Relevantes

| Serviço | Local Padrão no Linux | Tipo de Evento Registrado |
| :--- | :--- | :--- |
| **Autenticação Geral (PAM/SSH)** | `/var/log/auth.log` (Debian/Ubuntu) ou `/var/log/secure` (RHEL/CentOS) | Sucessos e falhas de login no sistema operacional. |
| **Servidor FTP (vsftpd)** | `/var/log/vsftpd.log` | Conexões, autenticações e transferências. |
| **Servidor Web (Apache/Nginx)** | `/var/log/apache2/access.log` ou `/var/log/nginx/access.log` | Métodos HTTP, URIs solicitadas, códigos de resposta e User-Agents. |
| **Samba / SMB** | `/var/log/samba/log.smbd` | Tentativas de logon de rede e sessões SMB. |

---

## 2. Padrões Típicos de Ataque

### Cenário 1: Força Bruta em FTP
Em uma investida automatizada, observa-se alta densidade de mensagens de falha em poucos segundos:

```bash
# Monitorar em tempo real as tentativas de login
tail -f /var/log/vsftpd.log
```
Exemplo de entrada:
```text
Mon Nov  2 21:30:15 2026 [pid 4120] [admin] FAIL LOGIN: Client "192.168.56.10"
Mon Nov  2 21:30:16 2026 [pid 4125] [admin] FAIL LOGIN: Client "192.168.56.10"
Mon Nov  2 21:30:17 2026 [pid 4130] [admin] FAIL LOGIN: Client "192.168.56.10"
```

**Como extrair IPs com excesso de erros:**
```bash
grep "FAIL LOGIN" /var/log/vsftpd.log | awk '{print $NF}' | tr -d '"' | sort | uniq -c | sort -nr
```

---

### Cenário 2: Ataque em Formulário Web (DVWA)
Em `/var/log/apache2/access.log`:

```text
192.168.56.10 - - [02/Nov/2026:21:40:01 -0300] "POST /dvwa/login.php HTTP/1.1" 200 4528 "-" "Mozilla/5.0"
192.168.56.10 - - [02/Nov/2026:21:40:02 -0300] "POST /dvwa/login.php HTTP/1.1" 200 4528 "-" "Mozilla/5.0"
```

**Comando analítico para identificar taxa de requisições por segundo:**
```bash
awk '{print $4}' /var/log/apache2/access.log | cut -d: -f2,3,4 | sort | uniq -c
```

---

## 3. Regra Conceitual de Detecção (Suricata / Snort)

Para ambientes que utilizam sistemas de detecção de intrusão (IDS/IPS), assinaturas baseadas em limites de frequência (*thresholding*) podem alertar ou bloquear conexões suspeitas:

```text
# Exemplo de regra para alertar sobre excesso de falhas FTP no Suricata
alert ftp any any -> any 21 (msg:"POSSÍVEL FORÇA BRUTA FTP - Múltiplas Falhas de Login"; \
    flow:from_server,established; \
    content:"530 Login incorrect"; \
    threshold:type both, track by_src, count 5, seconds 30; \
    classtype:suspicious-login; sid:1000001; rev:1;)
```

---

## 4. Métricas de Resposta a Incidentes

Ao documentar incidentes em relatórios corporativos, considere incluir:
- **Tempo Médio de Detecção (MTTD)**: Intervalo entre o início das requisições anômalas e a emissão do alerta pelo SIEM/monitor.
- **Tempo Médio de Resposta (MTTR)**: Duração até o bloqueio efetivo do IP de origem (via Fail2ban, WAF ou firewall).
