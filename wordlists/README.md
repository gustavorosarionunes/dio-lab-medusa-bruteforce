# 📚 Wordlists em Testes de Auditoria de Senhas

## O que é uma Wordlist?
Uma *wordlist* (ou lista de palavras/dicionário) é uma coleção estruturada de termos e senhas comuns utilizada por equipes de segurança e auditores para avaliar a robustez das políticas de credenciais de uma organização.

---

## Tipos Comuns de Listas

1. **Listas de Senhas Padrão (*Default Passwords*)**:
   - Senhas de fábrica configuradas por fabricantes (ex: `admin/admin`, `root/toor`, `cisco/cisco`).
2. **Listas Baseadas em Vazamentos Públicos**:
   - Arquivos como o conhecido `rockyou.txt`, contendo termos extraídos de brechas históricas.
3. **Listas Sintéticas/Customizadas**:
   - Dicionários construídos com base no perfil da organização auditada (ex: nome da empresa + ano atual), simulando cenários realistas de *Password Spraying*.

---

## Por que Usar Listas Curtas em Laboratório?
Em exercícios educacionais (como o laboratório da DIO), o uso de dicionários compactos e controlados permite:
- **Agilidade na Validação**: O teste conclui rapidamente, permitindo focar na verificação dos logs e na eficácia da mitigação.
- **Redução de Carga**: Evita exaustão de recursos de CPU e memória nas máquinas virtuais.
- **Previsibilidade**: Facilita a comprovação de que o sistema detectou e barrou o IP após um número exato de tentativas configurado na política defensiva.
