# 🛡️ Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux

Este repositório apresenta a documentação técnica e prática referente ao desafio de projeto da **DIO (Digital Innovation One)** sobre testes de intrusão focados em ataques de força bruta (*Brute Force*), utilizando o **Kali Linux** e a ferramenta **Medusa** em ambientes controlados.

---

## 🎯 Objetivos do Projeto
* Compreender o funcionamento de ataques de força bruta em diferentes protocolos e serviços (**FTP**, **HTTP/Web** e **SMB**).
* Utilizar a ferramenta **Medusa** no Kali Linux para auditoria de credenciais.
* Criar e estruturar *wordlists* personalizadas para testes de segurança.
* Identificar vulnerabilidades de autenticação fraca e propor medidas eficazes de mitigação.

---

## 🖥️ Arquitetura do Ambiente de Testes
Para garantir a segurança e a legalidade dos testes, a simulação foi projetada para um ambiente isolado de laboratório:
* **Máquina Atacante:** Kali Linux (com a ferramenta `medusa` instalada).
* **Máquina Alvo:** Metasploitable 2 (executando serviços vulneráveis propositalmente, como vsftpd, Samba e DVWA).
* **Configuração de Rede:** Rede interna isolada (*Host-Only* no VirtualBox), impedindo qualquer tráfego externo.

---

## 📂 Criação das Wordlists
Para os testes, foram estruturados dois arquivos de texto simples na máquina atacante contendo possíveis usuários e senhas comuns:

**`usuarios.txt`**
```text
admin
user
msfadmin
root
guest
```

**`senhas.txt`**
```text
123456
admin
password
msfadmin
root
12345
```

---

## ⚔️ Cenários de Ataque Simulados

### 1. Ataque de Força Bruta no Serviço FTP
O protocolo FTP (*File Transfer Protocol*) transmite dados sem criptografia e, se não possuir bloqueio de tentativas, fica exposto a testes automatizados de credenciais.

* **Comando executado:**
```bash
medusa -h 192.168.56.101 -U usuarios.txt -P senhas.txt -M ftp -t 4
```
* **Parâmetros utilizados:**
  * `-h`: Endereço IP do alvo.
  * `-U`: Caminho da wordlist de usuários.
  * `-P`: Caminho da wordlist de senhas.
  * `-M`: Módulo do serviço alvo (`ftp`).
  * `-t`: Número de threads simultâneas.
* **Resultado esperado:** Identificação das credenciais válidas (ex: `msfadmin:msfadmin`) após o cruzamento das listas.

---

### 2. Automação de Tentativas em Formulário Web (DVWA)
Utilizando a aplicação vulnerável **DVWA (*Damn Vulnerable Web Application*)**, simulou-se a quebra de autenticação em formulários HTTP.

* **Comando executado:**
```bash
medusa -h 192.168.56.101 -u admin -P senhas.txt -M http -m DIR:/dvwa/login.php -m FORM-DATA:"post&username=&password=&Login=Login" -m DENY-SIGNAL:"Login failed"
```
* **Resultado esperado:** O Medusa analisa a resposta HTTP da página e identifica a senha correta (`password`) assim que a mensagem de erro (`Login failed`) deixa de ser exibida.

---

### 3. Password Spraying e Enumeração em SMB
O protocolo SMB (*Server Message Block*) é amplamente usado para compartilhamento de arquivos em rede. Para evitar o bloqueio de contas por excesso de tentativas em um único usuário, aplica-se a técnica de **Password Spraying** (testar uma mesma senha comum em vários usuários diferentes).

* **Comando executado:**
```bash
medusa -h 192.168.56.101 -U usuarios.txt -p msfadmin -M smbnt
```
* **Resultado esperado:** Mapeamento de contas ativas que utilizam senhas padrão ou fracas sem disparar alertas imediatos de bloqueio por força bruta tradicional.

---

## 🛡️ Medidas de Mitigação e Defesa
Para proteger sistemas e aplicações contra os ataques demonstrados neste projeto, recomenda-se a implementação das seguintes contramedidas:

1. **Autenticação Multifator (MFA):** Exigir um segundo fator de validação além da senha, tornando credenciais vazadas ou descobertas insuficientes para o acesso.
2. **Política de Bloqueio de Conta (*Account Lockout*):** Bloquear temporariamente o acesso após um número limitado de tentativas falhas consecutivas (ex: 3 a 5 tentativas).
3. **Uso de Ferramentas de Prevenção (ex: Fail2Ban):** Monitorar logs de autenticação e banir automaticamente endereços IP com comportamento suspeito.
4. **Política de Senhas Fortes:** Exigir senhas longas, complexas e únicas, evitando senhas padrão de fábrica.
5. **Desativação de Protocolos Inseguros:** Substituir serviços sem criptografia (como FTP) por alternativas seguras (como SFTP ou FTPS).

---

## ⚠️ Aviso Legal
Todo o conteúdo e os comandos documentados neste repositório têm fins **estritamente educacionais**, voltados para o aprendizado de segurança defensiva e ofensiva ética em ambientes controlados.

