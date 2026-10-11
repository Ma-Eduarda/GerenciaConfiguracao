# GerenciaConfiguracao

## Configuração de SSH e UFW no Ubuntu 24.04

Este documento apresenta os passos para configurar o acesso remoto por SSH e o firewall UFW em um container Ubuntu 24.04 utilizando Docker.

### 1. Criar o container
No terminal do Ubuntu, crie o container com o seguinte comando:

```bash
docker run -d --name ubuntu2404 -p 2222:22 --privileged ubuntu:24.04 bash -c "tail -f /dev/null"
```

O comando cria um container chamado `ubuntu2404` usando a imagem do Ubuntu 24.04. A opção `-p 2222:22` permite acessar, pela porta 2222 do computador, a porta 22 do container. A opção `--privileged` garante que o firewall UFW tenha as permissões necessárias para funcionar no Docker.

### 2. Acessar o container
**Onde executar:** terminal do Ubuntu, fora do container.

```bash
docker exec -it ubuntu2404 /bin/bash
```

Após executar o comando, o terminal estará dentro do container. Os próximos comandos desta documentação devem ser executados nele, até que seja indicado o contrário.

### 3. Instalar os pacotes necessários
**Ainda dentro do container:** Primeiro, atualize a lista de pacotes disponíveis:

```bash
apt update
```

Em seguida, instale os programas necessários:

```bash
apt install openssh-server openssh-client ufw nano sudo iproute2 -y
```

Os pacotes instalados são:
- `openssh-server`: permite que o container receba conexões SSH.
- `openssh-client`: permite iniciar conexões SSH com outros computadores ou containers.
- `ufw`: ferramenta para configurar regras de firewall.
- `nano`: editor de texto usado para editar arquivos de configuração.
- `sudo`: permite executar comandos com privilégios administrativos quando configurado.
- `iproute2`: disponibiliza ferramentas para consultar e administrar informações de rede.

A opção `-y` confirma automaticamente a instalação dos pacotes.

> **Observação:** No meio da instalação do SSH, o sistema pedirá a área geográfica. Selecione `2. America` e, para a região, aperte Enter para rolar a lista e selecione o número da que mais se aproxima da sua (ex: `135. Sao_paulo`).

### 4. Criar o usuário
Crie o usuário `gcsi`:

```bash
useradd -m gcsi
```

Defina a senha do usuário:

```bash
passwd gcsi
```

Adicione o usuário ao grupo sudo:

```bash
usermod -aG sudo gcsi
```

Defina o Bash como shell do usuário:

```bash
chsh -s /bin/bash gcsi
```

### 5. Configurar o SSH
Abra o arquivo de configuração do SSH:

```bash
nano /etc/ssh/sshd_config
```

Verifique se a opção abaixo está configurada desta forma:

```text
PermitRootLogin yes
PasswordAuthentication yes
```

Salve o arquivo e feche o editor. Crie o diretório necessário:

```bash
mkdir -p /run/sshd
```

Verifique se a configuração apresenta erros:

```bash
/usr/sbin/sshd -t
```

Se nenhum erro for exibido, execute:

```bash
service ssh start
service ssh status
```

O último comando permite verificar se o serviço SSH está em execução.

### 6. Configurar o firewall UFW
Defina as regras iniciais:

```bash
ufw default deny incoming
ufw default allow outgoing
```

Esses comandos bloqueiam conexões de entrada por padrão e permitem conexões de saída. Libere a porta utilizada pelo SSH:

```bash
ufw allow 22/tcp
```

Ative o firewall:

```bash
ufw enable
```

Para consultar as regras:

```bash
ufw status verbose
```

> **Observação:** O funcionamento do UFW dentro de um container depende das permissões e da configuração de rede do Docker. A execução desses comandos não garante, por si só, que o acesso publicado pelo Docker esteja protegido pelas regras do UFW (por isso usamos a flag `--privileged` na criação).

### 7. Testar o acesso remoto
Abra outro terminal do Ubuntu, fora do container. Execute:

```bash
ssh -p 2222 gcsi@127.0.0.1
```

Informe a senha criada para o usuário `gcsi`. Na primeira conexão, pode ser necessário confirmar a chave do servidor. Após conectar, execute:

```bash
whoami
hostname
```

O primeiro comando deve mostrar `gcsi`. O segundo mostra o nome do container ao qual você se conectou. Para encerrar a conexão SSH:

```bash
exit
```

### 8. Verificar a configuração
Verifique se o container está em execução:

```bash
docker ps
```

Consulte o estado do SSH:

```bash
docker exec -it ubuntu2404 service ssh status
```

Consulte o estado do UFW:

```bash
docker exec -it ubuntu2404 ufw status verbose
```

### Resultado
Ao final, o container terá os pacotes necessários instalados, o usuário `gcsi` criado e o serviço SSH configurado. O acesso remoto poderá ser testado pela porta 2222 do computador. O estado do UFW deve ser conferido no ambiente utilizado, considerando as limitações de rede dos containers Docker.
