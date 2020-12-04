---
title: "Introdução ao Docker para NetDevOps"
date: 2020/12/04
tags:
 - Docker
category:
 - Docker
---

No mundo das Redes, NetDevOps é uma das palavras que estão na moda no período de 1 ano para cá. Mas o que é NetDevOps, ou mesmo DevOps? Nesta série de 3 partes, responderemos a essas perguntas e também mergulharemos em uma implementação do NetDevOps usando Ansible, Github, Jenkins e Vrnetlab.

Buenas NetCode… Neste artigo iremos dar continuidade na série NetDevOps. Neste artigo veremos sobre Docker, iremos aprender sobre os fundamentos desta ferramenta aplicada a Network Infrastructure As Code.

Este artigo é muito importante para construirmos uma casca sólida de conhecimento introdutório sobre Docker, e como implementaremos esta ferramenta para construirmos uma infraestrutura como código (IAC) de testes automatizados para aplicarmos a metodologia CI/CD nos próximos artigos. Pegue seu café e vamos nessa!

## Tópicos

* Introdução
* VM vs Docker
  * VM
  * Docker
* Instalação
* Comandos básicos

## Introdução

O Docker é um serviço de contêiner, uma engine de contêiner. É um software que empacota todas as dependências de sua aplicação ou de uma aplicação que você deseja criar e os alocam em um contêiner. 

A partir do momento que você tem sua aplicação distribuída em contêiner, você pode compartilhar estes contêineres para a sua equipe trabalhar junto. Baseado nisso, não vai ter mais aquele problema de que "na minha máquina funcionou".

Com o Docker, o time pode trabalhar nas aplicações utilizando as mesmas dependências uns aos outros. Isso acontece pelo fato de que as aplicações são baseadas nas dependências de file system ou até mesmo serviços que rodam na máquina do dev.

Versões de aplicações variam muito do uso de pessoas para pessoas, por exemplo, dev-01 está com a versão do PHP "x", o dev-02 está com a versão do PHP "xpto". As vezes, a aplicação pode rodar apenas em uma versão específica do PHP. É devido à estes problemas de versões que as aplicações tem que surgiu aquela famosa frase, "na minha máquina funcionou".

O Docker se propõe para que, sua aplicação independente de onde ela estiver, em qual SO ela estiver alocada, ela sempre funcionará da mesma forma para todos os SO, independente das bibliotecas que os compõe.

O Docker separa a sua aplicação de forma virtualizada, você pode escolher dentro do contêiner as bibliotecas e ferramentas que irão compor sua aplicação. Exemplo:

Temos 03 containeres:

* Container 01 - Aplicação node API
* Container 02 - Aplicação MySql, BD para a aplicação node consultar serviços de BD.
* Container 03 - Página PHP que consome serviços da API node e joga estes serviços para dentro de uma página HTML. 

Em cada um destes contêineres irá rodar uma versão da ferramenta principal, e também irá rodar dentro de um SO base, de acordo com o estilo de aplicação - Exemplo: aplicação node precisa rodar numa distro Linux. Aplicação MySql também precisa rodar numa distro Linux ou Windows. A aplicação PHP, além de uma distro Linux, precisaria indexar junto ao contêiner, o serviço apache para hospedar está página, ou, se fosse uma aplicação .NET, precisaria rodar encima de uma distro Windows e dentro deste SO, instalar o IIS, servidor de aplicação da Microsoft.

Obs: Vale ressaltar que estes três contêineres, em conjunto, eles irão rodarem dentro de um SO específico, por exemplo, o Windows. Isso não interfere pelo fato de que o Docker tem diversas bibliotecas baixadas junto ao software que integra as dependências de ferramentas que você precisa. Óbvio que para as imagens que não vem instaladas juntas ao Docker, ao serem baixadas via repositório Docker hub, irão vir junto as dependências destas imagens.

## VM vs Docker

### VM

Utilizando recurso de VM, irá ter um SO principal encima desta infraestrutura.


O hypervisor é um recurso principal e acaba sendo o mais caro nesta solução, pois, é este software que faz a distribuição de recursos de hardware para serem alocados em cada VM de acordo com a aplicação que você irá rodar. O hypervisor poderia ser VMware ESXI, por exemplo.

Dentro de cada VM seria alocado um SO para servir de base à sua aplicação, em seguida virá as bibliotecas e binários dessa VM, e por fim teremos de fato a aplicação no terceiro bloco de VM.

Obs: Com este tipo de solução, de certa forma acaba tendo muito desperdício de recursos de hardware, além de ficarmos refém do hypervisor para ter este tipo de cenário utilizando VMs para hospedarem as aplicações.

## Docker

O Docker não depende de um hypervisor para rodar seus contêineres, ele utiliza as próprias features do kerner do próprio host onde ele está instalado. Dessa forma a inicialização é mais rápida e em poucos segundos a sua aplicação já está no ar com todas as dependências que ela precisa para rodar encima do Docker e de qualquer outra máquina, desde que esteja também com o Docker instalado:


Para que as sua aplicações possam consumir menos recursos de hardware, o seu host físico precisaria rodar em uma distribuição Linux para que o Docker consiga consumir os recursos do kernel de seu SO. Todos os recursos de file system seria compartilhado, é essa magia que torna o Docker incrível de se implementar.

## Instalação

A instalação do Docker e suas dependências é bem fácil de instalar mas primeiro, precisamos atualizar o índice do pacote e preparar o apt para instalar repositórios HTTP:

``` bash
thiago@thiago:~$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    Software-properties-common
```
Em seguida, precisamos adicionar a chave GPG para apt e verificarmos se a chave corresponde à impressão digital `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`:

``` bash
thiago@thiago:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
thiago@thiago:~$ sudo apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ desconhecida] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

thiago@thiago:~$
``` 
Depois de puxarmos dados do Docker via comando `curl`, agora iremos adicionar um repositório Docker estável ao apt:
``` bash
thiago@thiago:~$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
``` 

Por fim, atualizaremos o índice do pacote e em seguida iremos instalar o Docker Engine e o containerd:
``` bash
thiago@thiago:~$ sudo apt-get update
thiago@thiago:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io
``` 

Para validarmos que o Docker está instalado, faremos a execução de um contêiner básico para confirmar se a instalação foi bem-sucedida:
``` bash
thiago@thiago:~$ sudo docker run hello-world
``` 

Opcionalmente, poderá ser adicionado seu nome de usuário ao Docker group, caso queira utilizar o Docker como um usuário não root. Costumo fazer em ambiente de laboratório, mas isso pode não ser apropriado para seu ambiente de produção, portanto, verifique com sua equipe de segurança antes de fazer isso na produção:
``` bash
thiago@thiago:~$ sudo usermod -aG docker `id -un`
``` 

Pronto, o Docker agora já está instalado!

## Comandos básicos

* `docker` -  executando só esta palavra, do lado client irá ser listado os seguintes comandos:
``` bash
thiago@thiago:~$ docker 
Usage:	docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/home/thiago/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and default context set with "docker
                           context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/home/thiago/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/home/thiago/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/home/thiago/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  engine      Manage the docker engine
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes
``` 

* `docker image ls` -  Lista as imagens que eu tenho pré carregadas dentro do ambiente de repositório Docker local:
``` bash
thiago@thiago:~$ docker image ls
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
vr-xcon                     latest              9d2298bb92b0        6 days ago          147MB
vrnetlab/vr-xcon            latest              9d2298bb92b0        6 days ago          147MB
veos                        4.18.10M            12915790e700        6 days ago          889MB
debian                      stretch             b33ba41eae78        11 days ago         101MB
ciscotestautomation/pyats   latest              4872b666b62f        7 months ago        609MB
hello-world                 latest              bf756fb1ae65        11 months ago       13.3kB
busybox                     latest              b534869c81f0        12 months ago       1.22MB
``` 

* `#cd /var/lib/docker/` - Diretório onde estão todas as dependências do Docker:
``` bash
root@thiago:/var/lib/docker# l
builder/  buildkit/  containers/  image/  network/  overlay2/  plugins/  runtimes/  swarm/  tmp/  trust/  volumes/
root@thiago:/var/lib/docker# 
root@thiago:/var/lib/docker# alias l="ls -la"
root@thiago:/var/lib/docker# l
total 56
drwx--x--x 14 root root 4096 nov 23 08:17 .
drwxr-xr-x 75 root root 4096 nov 23 08:04 ..
drwx------  2 root root 4096 dez 12  2019 builder
drwx--x--x  4 root root 4096 dez 12  2019 buildkit
drwx------  8 root root 4096 nov 23 15:30 containers
drwx------  3 root root 4096 dez 12  2019 image
drwxr-x---  3 root root 4096 dez 12  2019 network
drwx------ 31 root root 4096 nov 23 15:30 overlay2
drwx------  4 root root 4096 dez 12  2019 plugins
drwx------  2 root root 4096 nov 23 08:17 runtimes
drwx------  2 root root 4096 dez 12  2019 swarm
drwx------  2 root root 4096 nov 23 09:48 tmp
drwx------  2 root root 4096 dez 12  2019 trust
drwx------  2 root root 4096 dez 12  2019 volumes
``` 

* `root@thiago:/var/lib/docker# docker image pull <nome_imagem>` - Este comando pega a imagem do ubuntu no repositório remoto do Docker, seria o Docker hub, isso acontece pelo fato de que, dentro das dependências do Docker existe uma API que pega a imagem que você solicitou:
``` bash
root@thiago:/var/lib/docker# docker image pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
da7391352a9b: Pull complete 
14428a6d4bcd: Pull complete 
2c2d948710f2: Pull complete 
Digest: sha256:c95a8e48bf88e9849f3e0f723d9f49fa12c5a00cfc6e60d2bc99d87555295e4c
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
root@thiago:/var/lib/docker# 
root@thiago:/var/lib/docker# 
root@thiago:/var/lib/docker#
``` 

* `root@thiago:/var/lib/docker# docker image rm ubuntu` - Faz a exclusão de uma imagem específica:
```bash
root@thiago:/var/lib/docker# docker image rm ubuntu
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:c95a8e48bf88e9849f3e0f723d9f49fa12c5a00cfc6e60d2bc99d87555295e4c
Deleted: sha256:f643c72bc25212974c16f3348b3a898b1ec1eb13ec1539e10a103e6e217eb2f1
Deleted: sha256:9386795d450ce06c6819c8bc5eff8daa71d47ccb9f9fb8d49fe1ccfb5fb3edbe
Deleted: sha256:3779241fda7b1caf03964626c3503e930f2f19a5ffaba6f4b4ad21fd38df3b6b
Deleted: sha256:bacd3af13903e13a43fe87b6944acd1ff21024132aad6e74b4452d984fb1a99a
root@thiago-ThinkPad-T430:/var/lib/docker#
``` 

Obs: Quando carregamos uma imagem ao Docker, vale ressaltar que estamos carregando apenas uma camada dessa imagem, o Docker não carrega a imagem completa, isso é bom por que você só baixa as imagens de acordo com o que você quer mexer dentro da imagem, ou seja, você baixa pacotes da imagem.

* `root@thiago:/var/lib/docker# docker container ls` - Lista todos os contêineres em status de execução:
```bash
root@thiago:/var/lib/docker# docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                PORTS                                                                  NAMES
b6869525935a        veos:4.18.10M       "/launch.py"        6 days ago          Up 6 days (healthy)   22/tcp, 80/tcp, 443/tcp, 830/tcp, 5000/tcp, 10000-10099/tcp, 161/udp   veos2
812a511d67ac        veos:4.18.10M       "/launch.py"        6 days ago          Up 6 days (healthy)   22/tcp, 80/tcp, 443/tcp, 830/tcp, 5000/tcp, 10000-10099/tcp, 161/udp   veos1
root@thiago:/var/lib/docker#
``` 

* `root@thiago:/var/lib/docker# docker container ls -a` - Visualiza todos os contêineres em execução e parados:
```bash
root@thiago:/var/lib/docker# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                                                                  NAMES
b6869525935a        veos:4.18.10M       "/launch.py"             6 days ago          Up 6 days (healthy)        22/tcp, 80/tcp, 443/tcp, 830/tcp, 5000/tcp, 10000-10099/tcp, 161/udp   veos2
812a511d67ac        veos:4.18.10M       "/launch.py"             6 days ago          Up 6 days (healthy)        22/tcp, 80/tcp, 443/tcp, 830/tcp, 5000/tcp, 10000-10099/tcp, 161/udp   veos1
b783ca1bbb0c        vr-xcon             "/xcon.py --p2p veos…"   6 days ago          Exited (0) 6 days ago                                                                             bridge-veos1-2-veos2-2
2e7522fd9a82        vr-xcon             "/xcon.py --p2p veos…"   6 days ago          Exited (0) 6 days ago                                                                             vr-xcon2
347b1a433650        hello-world         "/hello"                 6 days ago          Exited (0) 6 days ago                                                                             hardcore_spence
542664784e9d        busybox             "sh"                     11 months ago       Exited (0) 11 months ago                                                                          gracious_curran
root@thiago:/var/lib/docker
```

* `root@thiago:/var/lib/docker# docker container stop veos1 veos2` - Stop para todos os contêineres em execução:
```bash
root@thiago:/var/lib/docker# docker container stop veos1 veos2
veos1
veos2
```

* `root@thiago:/home/thiago/netdev/vrnetlab# docker container exec -it veos1 bash` - Este comando entra no bash do contêiner, desta forma você pode inserir comandos para manipular um contêiner específico.
```bash
root@thiago:/home/thiago/netdev/vrnetlab# docker container exec -it veos1 bash
root@3b0d6974ba32:/#
```

Bom, por hoje é só. E aí, o que achou deste artigo? Comente aqui embaixo, vai ser legal contar com sua presença por aqui.
