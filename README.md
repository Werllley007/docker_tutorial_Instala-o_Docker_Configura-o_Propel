# Tutorial de Instalação do Docker

Este tutorial vai guiá-lo na instalação do Docker para Ubuntu 24.04.

## Etapa 1: Atualizar o sistema e instalar dependências

Para começar, faça login no seu servidor e atualize o índice de pacotes definido no arquivo `/etc/apt/sources.list` e no diretório `/etc/apt/sources.list.d/`.

```bash
sudo apt update  -y
sudo apt upgrade -y
```

Algumas dependências são necessárias para que a instalação prossiga sem problemas. Execute o comando abaixo para instalá-las.

```bash
sudo apt install curl apt-transport-https ca-certificates software-properties-common
```


Agora que as listas de pacotes estão atualizadas e as dependências necessárias estão instaladas, podemos proceder com a instalação do Docker.

## Etapa 2: Instalar o Docker

Existem duas maneiras de instalar o Docker no Ubuntu. Primeiro, você pode instalá-lo a partir dos repositórios padrão usando o gerenciador de pacotes APT.

Instalar Docker da versão padrão
```bash
sudo apt install docker.io -y

sudo apt install docker.io docker-compose
```

No entanto, a versão instalada não é a mais recente. Para instalar a versão mais atual, você precisará instalá-lo a partir do repositório oficial do Docker.

### Instalar a versão mais recente do Docker

Para isso, baixe a chave GPG do Docker usando o comando curl, conforme mostrado abaixo.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Em seguida, adicione o repositório APT do Docker ao seu sistema. O comando cria um arquivo docker.list no diretório /etc/apt/sources.list.d/.

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Agora, atualize o índice de pacotes local para notificar o sistema sobre o novo repositório adicionado.

```bash
sudo apt update
```

Agora instale o Docker Community Edition (grátis para download e uso) com o seguinte comando. A opção -y permite a instalação não interativa.

```bash
sudo apt install docker-ce -y
```

O serviço do Docker será iniciado automaticamente após a instalação. Você pode verificar seu status executando o comando:

```bash
sudo systemctl enable --now docker

sudo systemctl status docker
```

A saída abaixo confirma que o Docker está funcionando conforme esperado.

```bash
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Fri 2025-08-29 14:48:20 -03; 31s ago
 Invocation: bb66509093e443149ff23146cc2f12f6
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 1344389 (dockerd)
      Tasks: 14
     Memory: 22.4M (peak: 24.6M)
        CPU: 305ms
     CGroup: /system.slice/docker.service
             └─1344389 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Aug 29 14:48:19 werlley-ssd dockerd[1344389]: time="2025-08-29T14:48:19.890919288-03:00" level=info msg="[graphdriver] using prior storage driver: overlay2"
lines 1-14
```

## Install Docker Engine:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

## Deploy Portainer CE:
```bash
sudo docker volume create portainer_data
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

Acesso pelo o link:

```bash
https://localhost:9443/
```

## Etapa 3: Adicionar o usuário ao grupo Docker

Por padrão, o Docker é configurado para rodar como root ou um usuário regular com privilégios elevados (usuário sudo). A implicação disso é que apenas o root ou o usuário sudo pode executar os comandos do Docker. Executar os comandos do Docker como um usuário regular resultará em uma mensagem de "erro de permissão".

Como já temos um usuário sudo chamado werlley configurado, executar o Docker com esse usuário é perfeitamente possível. Uma opção ainda melhor é adicionar o usuário que está logado, neste caso, werlley, ao grupo docker. Isso garante que não precisemos invocar o sudo sempre que executarmos comandos do Docker, já que o usuário já pertence ao grupo docker.

Se você quiser evitar digitar sudosempre que executar o dockercomando, adicione seu nome de usuário ao dockergrupo:

```bash
sudo groupadd docker
```

```bash
sudo usermod -aG docker $USER

groups
```

Para aplicar a nova associação ao grupo, saia do servidor e entre novamente ou digite o seguinte:

```bash
newgrp $USER

su - $USER
```

Confirme se seu usuário foi adicionado ao grupo docker digitando:

```bash
groups
```

Agora, execute o comando groups para verificar se o usuário foi adicionado ao grupo docker:

```bash
groups $USER
```

Agora você pode executar os comandos do Docker sem precisar do sudo. Para verificar a versão do Docker, por exemplo, execute:

```bash
docker version
```
## Como desistalar o docker completamente

- Passo 1:

```bash
dpkg -l | grep -i docker
```
- Passo 2:

```bash
sudo apt-get purge -y docker-engine docker docker.io docker-ce docker-ce-cli docker-compose-plugin

sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce docker-compose-plugin

sudo apt remove docker-buildx-plugin
```

Os comandos acima não removerão imagens, contêineres, volumes ou arquivos de configuração criados pelo usuário no seu host. Se desejar excluir todas as imagens, contêineres e volumes, execute os seguintes comandos:


```bash
sudo rm -rf /var/lib/docker /etc/docker
sudo rm /etc/apparmor.d/docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
sudo rm -rf /var/lib/containerd
sudo rm -r ~/.docker
sudo apt autoremove
```

Removido o Docker completamente do sistema.


# Configuração do Lattice Propel em Ambiente Docker

O software **Lattice Propel v2025.1** apresentou incompatibilidades com a versão atual do sistema operacional Ubuntu, principalmente relacionadas a bibliotecas gráficas e ao sistema de licenciamento **FlexNet**. A solução implementada foi a criação de uma imagem Docker personalizada, baseada em uma versão compatível do **Ubuntu (22.04 LTS)**, com todas as dependências necessárias pré-instaladas.

O desafio principal foi o sistema de licença **node-locked**, que amarra o software a um endereço MAC (Host ID) específico. A solução foi gerar uma nova licença atrelada ao endereço MAC de um contêiner Docker padrão, garantindo que o ambiente seja funcional e portátil.

---

## 1 Dockerfile Definitivo

Após um longo processo de depuração, o Dockerfile abaixo foi finalizado. Ele utiliza o Ubuntu 22.04 para resolver incompatibilidades de bibliotecas C++ (GLIBCXX) e instala um conjunto completo de dependências gráficas e de sistema, identificadas através da ferramenta ldd.

- Crie o arquivo Dockerfile no caminho 
```bash
mkdir -p ~/docker

nano Dockerfile
```
- Cole dentro do arquivo:

```bash
# BASE: Ubuntu 22.04 - Versão Final e Definitiva
FROM ubuntu:22.04

# Evita interações manuais
ENV DEBIAN_FRONTEND=noninteractive

# Instala TODAS as dependências, incluindo as encontradas pelo diagnóstico final do ldd.
RUN apt-get update && apt-get install -y \
    sudo \
    lsb-core \
    locales \
    # Bibliotecas Gráficas e X11
    libx11-6 \
    libx11-xcb1 \
    libxext6 \
    libxrender1 \
    libxtst6 \
    libglx0 \
    libopengl0 \
    libegl1 \
    libglib2.0-0 \
    # Dependências do Plugin Qt/XCB
    libxcb-icccm4 \
    libxcb-image0 \
    libxcb-keysyms1 \
    libxcb-randr0 \
    libxcb-render-util0 \
    libxcb-xinerama0 \
    libxcb-xfixes0 \
    libxcb-xkb1 \
    libxkbcommon-x11-0 \
    libxcb-cursor0 \
    libxcb-shape0 \
    # Biblioteca de Teclado
    libxkbcommon0 \
    # Bibliotecas Boost
    libboost-filesystem1.74.0 \
    libboost-system1.74.0 \
    # App de teste e ferramentas de rede
    x11-apps \
    iproute2 \
&& rm -rf /var/lib/apt/lists/*

# Configura o locale
RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8

# Cria o usuário
ARG UID=1000
ARG GID=1000
RUN groupadd -g $GID werlley && \
    useradd -u $UID -g $GID -m -s /bin/bash werlley && \
    usermod -aG sudo werlley && \
    echo "werlley ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Define o usuário padrão
USER werlley
WORKDIR /home/werlley
WORKDIR /home/werlley/docker
COPY . .

# Comando padrão
CMD ["/bin/bash"]
```


### 1.1 Baixar o programa Propel_2025.1_lin.run

- Na pagina https://www.latticesemi.com/LatticePropel
- deve baixar em:
    - Download: Choose and download software from the Software Downloads & Documentation table below
- Coloque o arquivo realize a extração do arquivo Propel_2025.1_lin.run
- Cole o arquivo dentro pasta:

```bash
cd ~/docker/
```

### 1.2 Gerar a Licença com um Host ID Fixo
- Escolha um endereço MAC fixo para o seu contêiner. Um bom exemplo é 02:42:ac:11:00:02.

- Acesse o portal de licenciamento da Lattice https://www.latticesemi.com/Support/Licensing/DiamondAndiCEcube2SoftwareLicensing/PropelLicense

- Use este endereço MAC formatado como **0242ac110002** (sem os separadores) para gerar a sua licença.

- Baixe o arquivo de licença (.dat) e salve-o em um local na sua máquina, por exemplo, /home/werlley/docker/license.dat.

```bash
cd ~/docker/
```

### O Host ID (Endereço MAC)

Dentro do contêiner, o pacote iproute2 foi instalado para permitir a inspeção da rede. O comando ip a revelou o endereço MAC da interface eth0.

```bash
# Comando executado dentro do contêiner
sudo apt-get update && sudo apt-get install -y iproute2
ip a
```

### 1.3 Executar o Propel Builder com a Licença
Agora, vamos combinar o comando de execução com o mapeamento da licença. Você precisa usar o docker run para:

Iniciar um novo contêiner a partir da imagem propel-pronto.

Fixar o endereço MAC do contêiner com --mac-address.

Mapear o arquivo de licença local para dentro do contêiner usando a flag -v.


## 2 Construir a Imagem Base

O comando a seguir foi usado para construir a imagem propel-env a partir do Dockerfile:

```bash
docker build --build-arg UID=$(id -u) --build-arg GID=$(id -g) -t propel-env .
```
## Parte 3: Instalação do Propel e Criação da Imagem Final

Com a imagem base pronta, o software Propel foi instalado e o resultado foi salvo em uma nova imagem final e portátil.

### 3.1 Iniciar Contêiner de Instalação

Um contêiner temporário foi iniciado a partir da imagem propel-env.

```bash
sudo docker run -it --name propel-install-session -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v /home/werlley/docker:/home/werlley/docker propel-env
```

Caso tenha erro no comando acima, o comando xhost +local:docker deve ser executado, permitindo que o Docker acesse a interface gráfica do host. Agora, você pode tentar novamente executar o contêiner Docker com o Lattice Propel e verificar se o erro relacionado ao X11 (QXcbConnection: Could not connect to display) foi resolvido.
Tente os seguintes passos:

```bash
#Saia do Conteiner
exit

$ xhost +local:docker 

non-network local connections being added to access control list
```


Dentro do contêiner, verifique a variável de ambiente DISPLAY. Se não estiver corretamente configurada, defina-a manualmente:

```bash
$ echo $DISPLAY
:1
```
```bash
$ export DISPLAY=:1
```

```bash
# Comandos executados dentro do contêiner
cd /home/werlley/docker
sudo ./Propel_2025.1_lin.run
sudo mv /root/lscc /opt/
sudo chown -R werlley:werlley /opt/lscc
exit
```

### 3.2 Salvar a Imagem Final (propel-pronto)

O estado do contêiner, agora com o Propel instalado e configurado corretamente, foi salvo em uma nova imagem chamada propel-pronto.
```bash
docker commit propel-install-session propel-pronto
```

### 4 Iniciar o Ambiente Propel

O comando a seguir inicia o contêiner propel-pronto, compartilhando a interface gráfica, o hardware da placa de vídeo e apontando para os dois arquivos de licença.

```bash
docker run -it --rm \
    --name=propel-run \
    -e DISPLAY=$DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /home/werlley/docker/license.dat:/opt/lscc/propel/2025.1/builder/rtf/license/license.dat \
    --mac-address="02:42:ac:11:00:02" \
    propel-pronto \
    /bin/bash
```
### Executar o Propel Builder

Dentro do contêiner, o programa é iniciado sem a necessidade de sudo.

```bash
# Comandos executados dentro do contêiner
cd /opt/lscc/propel/2025.1/
./launch_builder.sh
```

## No seu $USER 

```bash
cd ~
```
## Cole dentro do bashrc

```bash
alias propeldocker='docker run -it --rm \
					--name=propel-run \
					-e DISPLAY=$DISPLAY \
					-v /tmp/.X11-unix:/tmp/.X11-unix \
					-v /home/werlley/docker/license.dat:/opt/lscc/propel/2025.1/builder/rtf/license/license.dat \
					--mac-address="02:42:ac:11:00:02" \
					--workdir /opt/lscc/propel/2025.1/ \
					propel-pronto \
					./launch_builder.sh'
```

### Algumas soluções para o problema de COLAR arquivos do Host para o Docker

Caso o arquivo o arquivo Propel_2025.1_lin.run não esteja no conteiner.

- Copie para o conteiner docker

```bash
$ sudo docker ps -a

CONTAINER ID   IMAGE        COMMAND       CREATED         STATUS         PORTS     NAMES
cdf23a8401bc   propel-env   "/bin/bash"   4 minutes ago   Up 4 minutes             propel-install-session
```
- Pegue o ID do conteiner **ee7cd8b8816d**

```bash
sudo docker cp /home/werlley/docker/Propel_2025.1_lin.run cdf23a8401bc:/home/werlley/docker/

Successfully copied 2.2GB to cdf23a8401bc:/home/tech03/propel_container
```

- Apagar todos os conteiners e Volumes não usados

```bash
docker system prune -a -f --volumes
```

## Parte 5: Gerenciamento e Backup
### 5.1 Fazer Backup da Imagem Final

A imagem propel-pronto pode ser salva em um arquivo .tar para backup.

```bash
docker save -o propel-pronto-backup.tar propel-pronto
```
### 5.2 Restaurar a Imagem do Backup

O backup pode ser restaurado em qualquer máquina com Docker.

```bash
docker load -i propel-pronto-backup.tar
```

# Referências

[Tutorial Video 1](https://www.youtube.com/watch?v=B0bacur-C0Q&ab_channel=PalomitaTV)


[Tutorial Video 2](https://www.youtube.com/watch?v=xkWuK5OVFPM&ab_channel=EdcarlosNeves)



Agora você pode salvar esse conteúdo em um arquivo `.md` e usá-lo conforme necessário.

# docker_tutorial_Instala-o_Docker_Configura-o_Propel
