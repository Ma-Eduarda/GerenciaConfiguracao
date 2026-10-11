# GerenciaConfiguracao

## Configuração de SSH e UFW no Ubuntu 24.04

Este documento apresenta os comandos utilizados para instalar e configurar o SSH e o UFW em um container Ubuntu 24.04 usando Docker.

### 1. Criar o container
No terminal, execute:
```bash
docker run -d --name ubuntu2404 -p 2222:22 ubuntu:24.04 bash -c "tail -f /dev/null"
```

Para acessar o container:
```bash
docker exec -it ubuntu2404 /bin/bash
```

### 2. Instalar os pacotes
Atualize a lista de pacotes e instale os programas necessários:
```bash
apt update
apt install openssh-server openssh-client ufw nano sudo iproute2 -y
```

### 3. Criar o usuário
Crie o usuário `gcsi`, defina uma senha e adicione-o ao grupo sudo:
```bash
useradd -m gcsi
passwd gcsi
usermod -aG sudo gcsi
chsh -s /bin/bash gcsi
```

### 4. Configurar o SSH
Abra o arquivo de configuração:
```bash
nano /etc/ssh/sshd_config
```
Verifique as seguintes opções:
```text
PermitRootLogin yes
PasswordAuthentication yes
```
Salve o arquivo e execute:
```bash
/usr/sbin/sshd -t
mkdir -p /run/sshd
service ssh start
service ssh status
```

### 5. Configurar o UFW
Execute os comandos para definir as regras do firewall e liberar a porta do SSH:
```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw enable
ufw status verbose
```
Confirme a ativação quando solicitado.

> **Observação:** O UFW dentro de um container Docker pode apresentar limitações, pois o Docker também controla as regras de rede (através do iptables). É necessário verificar se as regras foram aplicadas corretamente na prática.

### 6. Testar a conexão SSH
Abra outro terminal no computador e execute:
```bash
ssh -p 2222 gcsi@127.0.0.1
```
Informe a senha do usuário `gcsi`. Após conectar, execute:
```bash
whoami
hostname
```
Para encerrar a conexão:
```bash
exit
```

### 7. Verificar os serviços
Para verificar se o container está em execução:
```bash
docker ps
```
Para verificar o SSH:
```bash
docker exec -it ubuntu2404 service ssh status
```
Para verificar o UFW:
```bash
docker exec -it ubuntu2404 ufw status verbose
```

### Resultado
Foram apresentados os procedimentos para criar o container, instalar os pacotes, criar o usuário, configurar o SSH e definir as regras do UFW. A conexão SSH pode ser testada pela porta 2222 do computador anfitrião. O funcionamento do firewall deve ser verificado considerando as limitações da manipulação de rede do Docker.
