# CyberLab Local 🔐

Ambiente de laboratório para estudos de segurança ofensiva e defensiva.
Ele sobe, com **um único comando**, vários sistemas propositalmente vulneráveis
para você praticar pentest, sem precisar configurar nada manualmente.

Este README foi escrito pensando em quem nunca usou Docker antes. Se você
já é experiente, pode pular direto para [Como usar](#como-usar).

> ⚠️ **Este ambiente é intencionalmente vulnerável.**
> Rode **somente em rede local isolada** ou com firewall ativo, na sua própria
> máquina.
> **NUNCA** suba estes containers numa máquina com IP público, num servidor
> de nuvem exposto à internet, ou numa rede corporativa/compartilhada.

---

## Sumário

- [O que é isso, afinal?](#o-que-é-isso-afinal)
- [O que tem aqui](#o-que-tem-aqui)
- [Pré-requisitos](#pré-requisitos)
- [Como usar](#como-usar)
- [Acessando cada ferramenta](#acessando-cada-ferramenta)
- [Enumeração SNMP e LDAP](#enumeração-snmp-e-ldap)
- [Parar o ambiente](#parar-o-ambiente)
- [Máquina com pouca RAM?](#máquina-com-pouca-ram-suba-só-o-que-precisar)
- [Solução de problemas](#solução-de-problemas)
- [Perguntas frequentes (glossário)](#perguntas-frequentes-glossário)
- [Aviso legal](#aviso-legal)

---

## O que é isso, afinal?

Este repositório usa o **Docker** para ligar, dentro da sua própria máquina,
um conjunto de "computadores virtuais leves" (chamados **containers**) que já
vêm de propósito com falhas de segurança conhecidas. A ideia é simples:

1. Você sobe o ambiente com um comando.
2. Ele cria uma pequena rede isolada, só dentro do seu computador.
3. Você usa ferramentas de segurança (as mesmas usadas profissionalmente,
   como Nmap, Kali Linux, Burp Suite) para **encontrar e explorar** essas
   falhas, do jeito certo, em um ambiente feito para isso.

Isso é chamado de "laboratório de pentest" e é a forma mais segura e comum
de aprender segurança ofensiva: praticar num alvo que você tem permissão
total para atacar, e que não guarda nenhum dado real.

Você **não precisa saber usar Docker de antemão** — este README explica cada
passo. O único pré-requisito real é ter o Docker instalado (próxima seção).

---

## O que tem aqui

| Container | Endereço | Para que serve |
|-----------|----------|-----------------|
| **DVWA** | http://localhost:8080 | Aplicação web vulnerável: SQL Injection, XSS, CSRF, Upload de arquivos malicioso, Command Injection |
| **Juice Shop** | http://localhost:3000 | Aplicação web moderna com o OWASP Top 10 — ótima para praticar com o Burp Suite |
| **WebGoat** | http://localhost:8081/WebGoat | Curso interativo de vulnerabilidades web, com lições passo a passo |
| **Kali Linux** | 172.20.0.5 (rede interna) | A máquina "atacante": já vem com Nmap, enum4linux, snmpwalk, ldapsearch instalados |
| **Metasploitable 2** | 172.20.0.20 (rede interna) | Sistema Linux propositalmente cheio de falhas: FTP, SSH, Telnet, SMB, MySQL |
| **Enum Target** | 172.20.0.21 (rede interna) | Serviços de SNMP e um diretório LDAP, para praticar enumeração de rede |

"Rede interna" quer dizer que esses containers não abrem portas no seu
computador — você só alcança eles de dentro do Kali Linux (ou de outra
ferramenta rodando na mesma rede Docker), como aconteceria num ataque real
dentro de uma rede.

---

## Pré-requisitos

Você só precisa de uma coisa: o **Docker** instalado e rodando. O Docker é o
programa responsável por criar e gerenciar os containers.

### Windows

1. Baixe e instale o **Docker Desktop**: https://www.docker.com/products/docker-desktop/
2. Durante a instalação, mantenha a opção **"Use WSL 2 instead of Hyper-V"**
   marcada (é a recomendada e mais leve).
3. Reinicie o computador se for pedido.
4. Abra o Docker Desktop e espere o ícone da baleia ficar "verde"/estável na
   bandeja do sistema — isso indica que o Docker está rodando.
5. Abra o **PowerShell** ou o **Terminal do WSL** para rodar os comandos deste
   README.

### macOS

1. Baixe o **Docker Desktop** para o seu chip: https://www.docker.com/products/docker-desktop/
   (existe uma versão para Apple Silicon/M1-M2-M3 e outra para Intel — o site
   detecta automaticamente).
2. Arraste o Docker para a pasta Applications e abra.
3. Espere o ícone da baleia aparecer na barra de menu, estável.
4. Abra o app **Terminal** para rodar os comandos deste README.

### Linux

1. Instale o Docker Engine seguindo o guia oficial da sua distribuição:
   https://docs.docker.com/engine/install/
2. Instale também o plugin do Compose (geralmente já vem junto em
   instalações recentes):
   ```bash
   sudo apt-get install docker-compose-plugin   # Debian/Ubuntu
   ```
3. (Recomendado) Adicione seu usuário ao grupo `docker`, para não precisar de
   `sudo` em todo comando:
   ```bash
   sudo usermod -aG docker $USER
   ```
   Depois disso, **saia e entre de novo** na sua sessão (logout/login) para o
   grupo valer.

### Conferindo se está tudo certo

Depois de instalar, abra um terminal e rode:

```bash
docker --version
docker compose version
```

Cada comando deve responder com um número de versão (por exemplo,
`Docker version 27.x.x`). Se aparecer "comando não encontrado" ou erro de
conexão, o Docker não está instalado corretamente ou não está rodando —
volte no passo do seu sistema operacional acima.

> 💡 Este README usa `docker compose` (com espaço, sem hífen), que é o
> comando moderno já embutido no Docker Desktop. Se você tiver uma instalação
> mais antiga, pode ser que o comando seja `docker-compose` (com hífen) — os
> dois funcionam da mesma forma, só troque um pelo outro se precisar.

---

## Como usar

### 1. Baixar o projeto

Se você já usa Git:

```bash
git clone https://github.com/cybersantana83/cyberlab-local.git
cd cyberlab-local
```

Se você **nunca usou Git**, não tem problema: clique no botão verde
**"Code" → "Download ZIP"** na página do repositório no GitHub, extraia o
arquivo em uma pasta e abra um terminal dentro dela.

### 2. Subir o ambiente

Dentro da pasta do projeto, rode:

```bash
docker compose up -d
```

O que esse comando faz:
- `docker compose` chama o Docker para ler o arquivo `docker-compose.yml`
  deste projeto (é ele quem descreve todos os containers).
- `up` diz para "subir" (ligar) tudo o que está descrito nesse arquivo.
- `-d` significa "detached": os containers rodam em segundo plano, e você
  recupera o controle do seu terminal na hora.

Na **primeira vez**, o Docker vai baixar as imagens de cada ferramenta da
internet (cerca de 2 GB no total) — isso pode levar alguns minutos,
dependendo da sua conexão. Nas próximas vezes que você rodar esse comando,
como as imagens já estão salvas na sua máquina, ele sobe tudo em menos de um
minuto.

Alguns containers (o Kali Linux e o Enum Target) instalam ferramentas
adicionais na primeira inicialização — leva mais ou menos 1 minuto depois do
container "Up" para essas ferramentas ficarem prontas. Se você tentar usar o
Kali imediatamente e ver a mensagem "⏳ Aguarde... Instalação de ferramentas
em andamento...", é só isso: espere terminar.

### 3. Verificar se está tudo rodando

```bash
docker compose ps
```

Isso lista todos os containers do projeto e o status de cada um. Você deve
ver todos com status `Up` (ou `Up (healthy)`). Se algum aparecer como
`Exited` ou `Restarting`, veja a seção [Solução de problemas](#solução-de-problemas).

---

## Acessando cada ferramenta

### Kali Linux (a máquina de ataque)

Para "entrar" dentro do Kali e ter um terminal Linux com as ferramentas de
ataque prontas, rode no seu terminal (fora do Docker):

```bash
docker exec -it lab-kali /bin/bash
```

Isso abre um terminal **dentro** do container Kali. A partir daqui, os
comandos a seguir são exemplos do que você pode rodar (dentro do Kali,
não no seu terminal normal):

```bash
# Descobrir serviços e versões abertos no Metasploitable
nmap -sV 172.20.0.20

# Varrer todas as 65535 portas (mais demorado)
nmap -p- -T4 172.20.0.20

# Checar uma vulnerabilidade específica de SMB
nmap -p445 --script smb-vuln-ms17-010 172.20.0.20

# Conectar via SSH no Metasploitable
ssh msfadmin@172.20.0.20   # senha: msfadmin
```

Para sair do terminal do Kali sem desligar o container, digite `exit` ou
aperte `Ctrl+D`.

### Aplicações web (acesse pelo navegador, na sua própria máquina)

| App | Endereço | Usuário | Senha |
|-----|----------|---------|-------|
| DVWA | http://localhost:8080 | `admin` | `password` |
| Juice Shop | http://localhost:3000 | — | Crie sua própria conta na hora |
| WebGoat | http://localhost:8081/WebGoat | `guest` | `guest` |

**Primeira vez no DVWA:** ao acessar http://localhost:8080, você pode cair
numa tela de setup. Clique em **"Create / Reset Database"** para inicializar
o banco de dados antes do primeiro login.

### Metasploitable

O Metasploitable fica isolado na rede interna, no endereço `172.20.0.20` —
ele **não abre porta no seu navegador**. Para "atacá-lo", use o Kali Linux
(veja acima) e aponte as ferramentas para esse IP.

---

## Enumeração SNMP e LDAP

O container `enum-target` (172.20.0.21) sobe com SNMP e um diretório LDAP
próprio, para você praticar enumeração de serviços de rede — uma etapa
clássica de reconhecimento em qualquer pentest.

Rode estes comandos de dentro do Kali (`docker exec -it lab-kali /bin/bash`):

```bash
# Enumeração SMB no Metasploitable
enum4linux -a 172.20.0.20

# Enumeração SNMP — community "public" (sem senha real)
snmpwalk -v2c -c public 172.20.0.21

# Enumeração LDAP — sem precisar de login (bind anônimo)
ldapsearch -x -H ldap://172.20.0.21 -b 'dc=lab,dc=local'
```

O que cada um mostra:
- **`enum4linux`**: usuários, grupos, pastas compartilhadas (shares) e a
  política de senhas do Metasploitable via SMB.
- **`snmpwalk`**: informações do sistema (nome, descrição, processos rodando,
  interfaces de rede) que o SNMP expõe publicamente quando mal configurado —
  é exatamente esse o problema que você está praticando a identificar.
- **`ldapsearch`**: toda a estrutura de um diretório LDAP fictício (usuários,
  grupos, e-mails, cargos), simulando o tipo de informação que um LDAP
  corporativo mal protegido pode vazar sem exigir autenticação.

Detalhes do diretório LDAP simulado:

| Recurso | Detalhe |
|---|---|
| Domínio LDAP | `dc=lab,dc=local` |
| OUs (unidades organizacionais) | `ou=people`, `ou=groups` |
| Grupos | `admins` (jsilva, rsantos), `devs` (mpereira, amoura) |
| Usuários | jsilva, mpereira, rsantos, amoura (senha: `temp123`) |
| Bind admin (opcional, não é necessário para o `ldapsearch` acima) | `cn=admin,dc=lab,dc=local` / `LdapLab2026!` |
| Community SNMP | `public` |

> ⏳ A instalação do SNMP/LDAP roda na **primeira inicialização** desse
> container e leva cerca de 1 minuto. Se o `ldapsearch` ou o `snmpwalk`
> vierem vazios ou derem timeout logo de cara, espere um pouco e tente de
> novo — o serviço provavelmente ainda está configurando.

---

## Parar o ambiente

```bash
# Parar os containers, mas manter tudo salvo para a próxima vez
docker compose stop

# Parar e remover os containers (mantém as imagens já baixadas)
docker compose down

# Reset completo: apaga containers E todos os dados/volumes
docker compose down -v
```

Diferença prática: `stop` é como "desligar o computador" (liga rápido de
novo com `docker compose up -d`); `down -v` é como "formatar" — usado quando
você quer recomeçar do zero (por exemplo, se o banco de dados do DVWA
corromper).

---

## Máquina com pouca RAM? Suba só o que precisar

Você não precisa ligar tudo de uma vez. Dá para subir só os containers que
for usar:

```bash
# Só o DVWA
docker compose up -d dvwa

# Só os alvos web (sem Metasploitable, sem Kali)
docker compose up -d dvwa juiceshop webgoat
```

| Configuração | RAM mínima recomendada |
|---|---|
| Só DVWA | 2 GB |
| Alvos web (DVWA + Juice Shop + WebGoat) | 4 GB |
| Ambiente completo (+ Metasploitable + Kali + Enum Target) | 8 GB |

---

## Solução de problemas

### "Cannot connect to the Docker daemon"
O Docker não está rodando. Abra o Docker Desktop (Windows/Mac) e espere o
ícone da baleia ficar estável, ou no Linux rode `sudo systemctl start docker`.

### "Permission denied" ao rodar `docker` no Linux
Seu usuário ainda não está no grupo `docker`, ou você não fez logout/login
depois de adicionar (veja [Pré-requisitos](#pré-requisitos) → Linux). Como
alternativa rápida, rode os comandos com `sudo` na frente.

### Porta já em uso (`port is already allocated`)
Algum outro programa na sua máquina já está usando aquela porta (por exemplo,
outra aplicação também usa a porta 8080). Edite o `docker-compose.yml` e
troque a porta do lado esquerdo (a do seu computador):

```yaml
ports:
  - "8090:80"   # era "8080:80" — agora acessa em http://localhost:8090
```

Depois rode `docker compose up -d` de novo.

### Um container aparece como "Exited" ou fica reiniciando
Veja os logs dele para entender o motivo:

```bash
docker compose logs dvwa       # troque "dvwa" pelo nome do serviço
docker compose logs kali
docker compose logs enum-target
```

### Baixou pela primeira vez e nada abre no navegador ainda
As imagens grandes (Juice Shop, WebGoat) podem levar alguns minutos para
iniciar de verdade mesmo depois de aparecerem como `Up`. Espere 1-2 minutos
e tente de novo.

### Kali mostra "⏳ Aguarde... Instalação de ferramentas em andamento..."
Normal na primeira subida — ele está instalando Nmap, enum4linux, snmp e
ldap-utils dentro do container. Espere até 1 minuto e o terminal libera
sozinho.

### `snmpwalk` ou `ldapsearch` no `enum-target` não retornam nada
Mesma lógica do Kali: espere cerca de 1 minuto após o `docker compose up -d`
para os serviços SNMP/LDAP terminarem de configurar dentro do container.

### Reset do DVWA (banco de dados corrompido)
```bash
docker compose down -v && docker compose up -d dvwa
# Depois acesse: http://localhost:8080/setup.php → clique em "Create / Reset Database"
```

### Quero apagar tudo e começar do zero
```bash
docker compose down -v
docker compose up -d
```

---

## Perguntas frequentes (glossário)

**O que é Docker?**
Um programa que empacota aplicações inteiras (com tudo que elas precisam
para funcionar) em pacotes isolados chamados containers, para rodarem de
forma idêntica em qualquer computador.

**O que é um container?**
Pense nele como uma "caixinha" isolada rodando dentro do seu computador, com
seu próprio sistema de arquivos e rede — mas muito mais leve que uma máquina
virtual completa. Cada linha da tabela em [O que tem aqui](#o-que-tem-aqui)
é um container.

**O que é pentest (teste de invasão)?**
É o processo profissional de simular ataques reais contra um sistema, com
autorização, para encontrar falhas de segurança antes que alguém mal
intencionado as encontre.

**Preciso saber programar para usar isso?**
Não. Você precisa saber copiar e colar comandos de terminal — este README
explica cada um deles.

**É seguro deixar isso ligado no meu computador?**
Sim, **desde que você siga o aviso do topo deste README**: rode só na sua
máquina local, sem expor as portas para a internet ou para uma rede
compartilhada/corporativa. Os containers são vulneráveis de propósito, então
tratá-los como se estivessem "abertos ao mundo" seria um risco real.

**Posso usar isso para atacar sites de verdade?**
Não. Este ambiente serve exclusivamente para você praticar contra os alvos
inclusos aqui, dentro da sua própria máquina. Atacar sistemas de terceiros
sem autorização é crime.

**O que significa "172.20.0.x"?**
É o endereço IP interno de cada container, dentro da rede virtual que o
Docker cria só para este projeto. Só é possível alcançar esses endereços de
dentro dessa mesma rede — por isso você precisa estar "dentro" do Kali
(veja [Acessando cada ferramenta](#acessando-cada-ferramenta)) para atacá-los.

---

## Aviso legal

Ambiente criado exclusivamente para fins educacionais.
O uso das técnicas aprendidas contra sistemas sem autorização é ilegal.
O autor não se responsabiliza pelo uso indevido deste material.
