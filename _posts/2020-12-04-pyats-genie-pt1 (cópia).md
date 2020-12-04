---
title: "Automatize"
date: 2020/12/04
tags:
 - Python
 - pyATS
 - Genie
category:
 - Python
---

Buenas NetCode… Neste artigo iremos dar os primeiros passos com o pyATS e Genie, iremos desmistificar o conceito por trás dessas duas ferramentas que de certa forma ajuda bastante nos diagnósticos de validação do status da rede.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/WhatsApp Image 2020-12-04 at 08.23.56.jpeg)

## O que é pyATS e Genie?

Com a estrutura de verificação do pyATS podemos simplificar nossa estrutura de redes e otimizar nossas operações diárias. Com o Genie, podemos aplicar filtros para fazer a aprendizagem de inúmeros serviços que estão operando na rede.

Usando essas duas ferramentas em conjunto, podemos fazer uma comparação clara sobre o status da rede antes e depois de determinada configuração foi aplicada.

O genie serve como analisador de rede. Compara o antes e o depois da rede baseado no objeto que você deseja analisar.

Dica: O objeto analisador se refere ao tipo de protocolo que você deseja aprender ou o próprio comando em específico, a saída do comando é baseada em dicionário python, dessa forma podemos manipular os parâmetros armazenados na saída do comando aprendido.

A ideia de usar o genie para automatizar o diagnóstico da rede, é fazer com que, você possa instanciar a grande maioria ou os principais serviços que rodam no seus dispositivos de rede em geral, servidor, router, firewall, switch, entre outros.

Dessa forma podemos pegar a configuração do antes relacionados aos objetos e apenas gravar o depois quando alguma alteração nesses serviços (objetos) instanciados vieram a ter falhas, com isso você pode comparar o antes e o depois da alteração para identificar o erro.

Dica: O genie atua na solução pyats como validador da estrutura de automação, ele valida aquilo que foi feito, por exemplo, uma configuração que não tinha no device.

Já o pyATS pode ser usado para automação de testes, exemplo, desejo validar se a interface Ethernet0/2 está DOWN:
``` python
log.info(banner("Executando parser para filtrar parâmetros..."))
#Executando o parâmetro parse para retornar dados.
pre_output = device.parse("show interface Ethernet0/2")
# Filtrando parâmetros para efetuar possíveis verificações
pre_status = pre_output['Ethernet0/2']['oper_status']
if pre_status == 'down':
log.info("\nPASS: Interface Ethernet0/2 está DOWN\n")
else:
log.error("\nFAIL: Interface Ethernet0/2 está UP\n")
exit()
```

O Genie se baseia no pyATS para fornecer:

* Conectividade de dispositivo, analisadores, apis.
* Modelos de objetos Python independentes de plataforma para recursos como ospf, bgp.
* Conjunto de casos de teste reutilizáveis.
* Mecanismo de teste orientado por YAML

## Validação do estado de configuração

Quando a configuração é aplicada (ou quando a rede entra em desordem), geralmente ocorre uma série de alterações no estado operacional , como:

* Satus da interface, contadores.
* Rotas de rede, vizinhos, próximos saltos.
* Uso da CPU, memória, estados de HA, etc.

Tradicionalmente, a validação de tais mudanças dependia da experiência de cada engenheiro. Mas o pyATS pode aprender e criar um perfil completo da configuração e do status operacional de um recurso, permitindo que os usuários construam a lógica. Por exemplo, com apenas alguns comandos genie, você pode criar um perfil do seu sistema antes e depois de uma alteração na configuração e obter um resumo detalhado do que exatamente mudou.

Para usar a biblioteca pyATS (que engloba também o Genie), basta baixar o pacote da biblioteca com o seguinte comando:
```
pip install pyATS[library]
```

## Criando um perfil da rede

O pyATS e o Genie formam a solução pyATS, para trabalhar com o pyATS precisamos entender dos conceitos de arquivos. O pyATS trabalha com os seguintes arquivos:

* testbed – É o arquivo YAML onde são armazenados dados dos devices que queremos interagir, em comparação, seria o mesmo conceito do arquivo hosts na arquitetura ansible.
* Job file – É o arquivo Python onde é carregado um ou mais arquivos de tarefas a serem feitas.
* Python file – Seria um outro arquivo, esse arquivo estão armazenadas as tarefas que você mesmo irá escrever baseado nas classes e funções do python. É nesse arquivo que você irá definir as verificações que você deseja checar.

Aqui está um exemplo que faz uso rápido do pyATS ou Genie na sua linha de comando. Iremos criar um snapshot do estado atual da rede baseado nos objetos que queremos ter em nosso perfil, desejo que o pyATS aprende tudo que for relacionado a vlan baseado nos devices alocados dentro do arquivo testbed.yaml.

Arquivo testbed.yaml:
``` yaml
devices:

  SW_CORE_1:
    type: switch
    os: ios
    platform: iosl2
    alias: sw1_dev
    tacacs:
    login_prompt: 'login:'
    password_prompt: 'Password:'
    username: teste
    passwords:
    tacacs: teste
    connections:
      cli:
        protocol: ssh
        ip: 192.168.36.214
        port: 22
        
  SW_CORE_2:
    type: switch
    os: ios
    platform: iosl2
    alias: sw2_dev
    tacacs:
    login_prompt: 'login:'
    password_prompt: 'Password:'
    username: teste
    passwords:
    tacacs: teste
    connections:
      cli:
        protocol: ssh
        ip: 192.168.36.215
        port: 22

  SW_ACCESS_1:
    type: switch
    os: ios
    platform: iosl2
    alias: utt
    tacacs:
    login_prompt: 'login:'
    password_prompt: 'Password:'
    username: teste
    passwords:
    tacacs: teste
    connections:
      cli:
        protocol: ssh
        ip: 192.168.36.210
        port: 22

```

Aprendendo o estado da feature de vlan para criação de objeto como um dos perfis de objetos da rede:
```
$ genie parse "sh vlan" --testbed testbed.yaml  --devices SW_ACCESS_1 --output profile-devices/SW_ACCESS_1/vlan/atual/

$ genie parse "sh vlan" --testbed testbed.yaml  --devices SW_CORE_1 --output profile-devices/SW_CORE_1/vlan/atual/

$ genie parse "sh vlan" --testbed testbed.yaml  --devices SW_CORE_2 --output profile-devices/SW_CORE_2/vlan/atual/
```

Quando ocorrer um desastre ou quando você fizer alterações na configuração, o genie pode ser capaz de identificar exatamente o que você está procurando.

Para simular uma possível configuração que o estado atual do perfil de vlan ainda não tem, irei criar a vlan 110 nos três devices que temos no nosso arquivo testbed:
```
SW_CORE_1(config)# vlan 110
SW_CORE_1(config-vlan)# name Vlan 110

SW_CORE_2(config)# vlan 110
SW_CORE_2(config-vlan)# name Vlan 110

SW_ACCESS_1(config)# vlan 110
SW_ACCESS_1(config-vlan)# name Vlan 110
```

Agora iremos configurar um novo perfil da rede tendo como objeto novamente a feature de vlan:
```
$ genie parse "sh vlan" --testbed testbed.yaml  --devices SW_ACCESS_1 --output profile-devices/SW_ACCESS_1/vlan/depois/

$ genie parse "sh vlan" --testbed testbed.yaml  --devices SW_CORE_1 --output profile-devices/SW_CORE_1/vlan/depois/

$ genie parse "sh vlan" --testbed testbed.yaml  --devices SW_CORE_2 --output profile-devices/SW_CORE_2/vlan/depois/
```

Agora que criamos um novo perfil, iremos efetuar a comparação da feature de vlan dos dois perfis criados:
```
$ diff  profile-devices/SW_ACCESS_1/vlan/atual/ profile-devices/SW_ACCESS_1/vlan/depois/

>     "110": {
>       "mtu": 1500,
>       "said": 100110,
>       "trans1": 0,
>       "trans2": 0,
>       "type": "enet"
>     },

$ diff  profile-devices/SW_CORE_1/vlan/atual/ profile-devices/SW_CORE_1/vlan/depois/

>     "110": {
>       "mtu": 1500,
>       "said": 100110,
>       "trans1": 0,
>       "trans2": 0,
>       "type": "enet"
>     },

$ diff  profile-devices/SW_CORE_2/vlan/atual/ profile-devices/SW_CORE_2/vlan/depois/

>     "110": {
>       "mtu": 1500,
>       "said": 100110,
>       "trans1": 0,
>       "trans2": 0,
>       "type": "enet"
>     },
```

Dessa forma podemos fazer um diagnóstico rápido do estado de configuração antes e depois de ter feito algo. O parâmetro "diff" serve para comparar o perfil da feature de vlan antes e depois de ter ocorrido uma mudança, percebe-se acima que foi criada uma nova vlan (110).

Ficamos por aqui, no próximo post iremos nos aprofundar ainda mais na solução pyATS. Até breve!!!
