# CyberLab Local 🔐

Ambiente de laboratório para o seus estudos de segurança ofensiva e defensiva.
Ambiente com alvos vulneráveis para prática de pentest — sobe com um comando.

> ⚠️ **Este ambiente é intencionalmente vulnerável.**
> Rode **somente em rede local isolada** ou com firewall ativo.
> **NUNCA** suba estes containers numa máquina com IP público ou rede corporativa.

---

## O que tem aqui

| Container | Porta | Uso |
|-----------|-------|-----|
| **DVWA** | http://localhost:8080 | SQLi, XSS, CSRF, File Upload |
| **Juice Shop** | http://localhost:3000 | OWASP Top 10, Burp Suite |
| **WebGoat** | http://localhost:8081/WebGoat | Lições interativas OWASP |
| **Metasploitable 2** | 172.20.0.20 (interno) | Nmap, Metasploit, Hydra |

---

## Pré-requisitos

- Docker Desktop instalado (docker.com/get-started)
- Windows: Docker Desktop com WSL2 habilitado
- Mac: Docker Desktop (Intel ou Apple Silicon)
- Linux: Docker Engine + docker-compose

```bash
docker --version
docker-compose --version
```

---

## Como usar

### 1. Fazer fork e clonar

```bash
git clone https://github.com/SEU-USUARIO/cyberlab-local.git
cd cyberlab-local
```

### 2. Subir o ambiente

```bash
docker-compose up -d
```

Aguarde o download das imagens na primeira vez (~2 GB).
Após o download, os containers sobem em menos de 1 minuto.

### 3. Verificar se está rodando

```bash
docker-compose ps
```

Todos devem aparecer com status `Up`.

### 4. Acessar os alvos web

| App | URL | Usuário | Senha |
|-----|-----|---------|-------|
| DVWA | http://localhost:8080 | admin | password |
| Juice Shop | http://localhost:3000 | — | Crie na hora |
| WebGoat | http://localhost:8081/WebGoat | guest | guest |

### 5. Acessar o Metasploitable
```bash
docker exec -it lab-metasploitable /bin/bash
```


O Metasploitable fica isolado na rede interna `172.20.0.20`.
Para atacá-lo, aponte as ferramentas para esse IP:

```bash
# Exemplos de uso no Kali:
nmap -sV 172.20.0.20
nmap -p- -T4 172.20.0.20
nmap -p445 --script smb-vuln-ms17-010 172.20.0.20
ssh msfadmin@172.20.0.20   # senha: msfadmin
```

---

## Parar o ambiente

```bash
# Parar (dados preservados)
docker-compose stop

# Parar e remover containers
docker-compose down

# Reset completo (apaga tudo)
docker-compose down -v
```

---

## Máquina com pouca RAM? Suba só o que precisar

```bash
# Só o DVWA
docker-compose up -d dvwa

# Só os alvos web (sem Metasploitable)
docker-compose up -d dvwa juiceshop webgoat
```

| Configuração | RAM mínima |
|---|---|
| Só DVWA | 2 GB |
| Alvos web (3) | 4 GB |
| Todos (+ Metasploitable) | 8 GB |

---

## Solução de problemas

**Porta já em uso:**
```bash
# Edite o docker-compose.yml e troque a porta do host
# "8080:80" → "8090:80"
```

**Container não sobe:**
```bash
docker-compose logs dvwa
```

**Reset do DVWA (banco corrompido):**
```bash
docker-compose down -v && docker-compose up -d dvwa
# Acesse: http://localhost:8080/setup.php → Create / Reset Database
```

---

## Aviso legal

Ambiente criado exclusivamente para fins educacionais.
O uso das técnicas aprendidas contra sistemas sem autorização é ilegal.
O autor não se responsabiliza pelo uso indevido deste material.



