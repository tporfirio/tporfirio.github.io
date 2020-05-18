---
title: "Automatize sua rede com pyATS (DEMO) Pt 2"
date: 2020/05/18
tags:
 - Python
 - pyATS
 - Genie
category:
 - Python
---

No artigo de hoje, iremos dar continuidade no artigo anterior sobre como utilizar o pyATS e o Genie para monitorar serviços na rede. Pegue seu café, se acomode e vamos nessa.

Iremos utilizar o lab mostrado na imagem abaixo. Este lab encontra-se no meu repositório do [GitHub](https://github.com/tporfirio/network-automation/tree/master/pyats) junto com o código usado neste artigo.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/post-pyats-img-2.jpg)

No arquivo testbed.yaml, irei apenas alocar os dados de conexão do host CORE-SW2. O script que iremos escrever irá capturar registros da feature BGP. 
``` yaml
devices:
  CORE-SW2:
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
        ip: 192.168.36.131
        port: 22
```

Para explorarmos a feature de BGP para verificar o status de vizinhança do protocolo e poder validar se a vizinhança BGP está estabelecida como desejado.

A estrutura do caso de teste inclui as seguintes seções:

* Configuração comum
* Casos de teste

### Configuração comum

Para essa função, iremos apenas fazer a conexão com o device ou grupo de devices que queremos validar o status do BGP.

### Casos de teste

Nessa função é a onde acontece a mágica, iremos aprender o a feature de BGP em formato de dicionário python. Dessa forma será possível capturar palavras chaves para manipular dentro do código.

## Entendendo o código

Para iniciarmos, devemos importar os módulos e as bibliotecas que iremos utilizar no código:
``` python
# #####################################################################
# ####                       arquivo pyATS.py                       ###
# #####################################################################

#!/bin/env python

# Módulo para capturar log durante a execução do script
import logging
import json

# Irá construir a tabela onde será alocado dados da feature BGP
from tabulate import tabulate

# Necessário para o parâmetro aetest
from ats import aetest
from ats.log.utils import banner

# Importando módulos Genie
from genie.conf import Genie
from genie.abstract import Lookup

# Importando a library Genie
from genie.libs import ops 

log = logging.getLogger(__name__)
```

Depois de alocar os módulos e as bibliotecas que iremos utilizar no código, agora devemos definir a estrutura da sessão de configuração comum:
``` python
class common_setup(aetest.CommonSetup):
    """ Common Setup section """

    # CommonSetup pode haver diversas funções alocadas dentro da classe.
    
    # Definindo conexão com cada device descritos na mesa de teste que seria o arquivo testbed.yaml.
    # No caso, temos apenas um device descrito na mesa de teste.
    @aetest.subsection
    def connect(self, testbed):
        genie_testbed = Genie.init(testbed)
        self.parent.parameters['testbed'] = genie_testbed
        device_list = []
        for device in genie_testbed.devices.values():
            log.info(banner(
                "Connect to device '{d}'".format(d=device.name)))
            try:
                device.connect()
            except Exception as e:
                self.failed("Failed to establish connection to '{}'".format(
                    device.name))

            device_list.append(device)

        # Passando a lista de dispositivos para a variável "dev"
        self.parent.parameters.update(dev=device_list)
```

A classe acima irá servir apenas para conectar ao devices. Agora iremos montar a estrutura de casos de teste, é nessa estrutura que iremos manipular os valores que queremos utilizar para aprender e validar a vizinhança BGP dentro do dispositivo CORE-SW2:
``` python
class BGP_Neighbors_Established(aetest.Testcase):
    """ Abrindo sessão de test case """

    # First test section
    @ aetest.test
    def learn_bgp(self):
        """ Montando estrutura """
        
        # all_bgp_sessions se refere ao módulo que faz parte da library pyATS
        self.all_bgp_sessions = {}
        for dev in self.parent.parameters['dev']:
            log.info(banner("Coletando informações da feature BGP {}".format(
                dev.name)))
            abstract = Lookup.from_device(dev)
            bgp = abstract.ops.bgp.bgp.Bgp(dev)
            bgp.learn()
            self.all_bgp_sessions[dev.name] = bgp.info

    @ aetest.test
    def check_bgp(self):
        """ Manipulando dados capturados da feature """

        failed_dict = {}
        mega_tabular = []
        for device, bgp in self.all_bgp_sessions.items():
            # Os valores abaixo definidos como string se refere aos parâmetros que estão alocados dentro dos comandos do BGP
            # A ideia de aprender a feature BGP é fazer com que todos os parâmetros das saída dos comandos 
            relacionados ao BGP sejam armazenados dentro da variável dev.name
            default = bgp['instance']['default']['vrf']['default']
            neighbors = default['neighbor']
            for nbr, props in neighbors.items():
                state = props.get('session_state')
                if state:
                    tr = []
                    tr.append(device)
                    tr.append(nbr)
                    tr.append(state)
                    if state == 'established' or state == 'Established':
                        tr.append('Passed')
                    else:
                        failed_dict[device] = {}
                        failed_dict[device][nbr] = props
                        tr.append('Failed')

                mega_tabular.append(tr)

        log.info(tabulate(mega_tabular,
                          headers=['Device', 'Peer',
                                   'State', 'Pass/Fail'],
                          tablefmt='orgtbl'))

        if failed_dict:
            log.error(json.dumps(failed_dict, indent=3))
            self.failed("Testbed has BGP Neighbors that are not established")

        else:
            self.passed("All BGP Neighbors are established")
```

A estrutura abaixo irá desconectar do dispositivo depois de ser validado o status de vizinhança:
``` python
class common_cleanup(aetest.CommonCleanup):
    """ Common Cleanup """
    @aetest.subsection
    def clean_everything(self):
        """ Common Cleanup Subsection """
        log.info("Aetest Common Cleanup ")

if __name__ == '__main__':  
    aetest.main()
```

Estrutura totalmente montada e pronta para trabalhar em cima do arquivo de mesa (testbed.yaml). O script acima irá rodar dentro de um arquivo python, dentro da estrutura do pyATS esse arquivo é denominado de job file. Você pode dar o nome que quiser para este arquivo. Abaixo é mostrado a estrutura do arquivo:
``` python
# #####################################################################
# ####                       arquivo job  .py                       ###
# #####################################################################

import os
from ats.easypy import run

# A função abaixo irá rodar o script pyTAS.py
def main():
    # Find the location of the script in relation to the job file
    pwd = os.path.dirname(__file__)
    bgp_tests = os.path.join(pwd, 'pyTAS.py')
    # Execute the testscript
    # run(testscript=testscript)
    run(testscript=bgp_tests)
```

Até aqui está tudo ok, agora iremos rodar o arquivo job.py:
```
thiago@thiago-ThinkPad:~/Documentos/Code/Python/Networking/IOS/network-automation/pyats$ pyats run job job.py --testbed-file testbed.yaml
```

Report das tasks executadas:
```
2020-05-18T14:17:31: %EASYPY-INFO: Overall Stats
2020-05-18T14:17:31: %EASYPY-INFO:     Passed     : 3
2020-05-18T14:17:31: %EASYPY-INFO:     Passx      : 0
2020-05-18T14:17:31: %EASYPY-INFO:     Failed     : 0
2020-05-18T14:17:31: %EASYPY-INFO:     Aborted    : 0
2020-05-18T14:17:31: %EASYPY-INFO:     Blocked    : 0
2020-05-18T14:17:31: %EASYPY-INFO:     Skipped    : 0
2020-05-18T14:17:31: %EASYPY-INFO:     Errored    : 0
2020-05-18T14:17:31: %EASYPY-INFO: 
2020-05-18T14:17:31: %EASYPY-INFO:     TOTAL      : 3
2020-05-18T14:17:31: %EASYPY-INFO: 
2020-05-18T14:17:31: %EASYPY-INFO: Success Rate   : 100.00 %
2020-05-18T14:17:31: %EASYPY-INFO: 
2020-05-18T14:17:31: %EASYPY-INFO: +------------------------------------------------------------------------------+
2020-05-18T14:17:31: %EASYPY-INFO: |                             Task Result Summary                              |
2020-05-18T14:17:31: %EASYPY-INFO: +------------------------------------------------------------------------------+
2020-05-18T14:17:31: %EASYPY-INFO: Task-1: pyATS.common_setup                                                PASSED
2020-05-18T14:17:31: %EASYPY-INFO: Task-1: pyATS.BGP_Neighbors_Established                                   PASSED
2020-05-18T14:17:31: %EASYPY-INFO: Task-1: pyATS.common_cleanup                                              PASSED
2020-05-18T14:17:31: %EASYPY-INFO: 
2020-05-18T14:17:31: %EASYPY-INFO: +------------------------------------------------------------------------------+
2020-05-18T14:17:31: %EASYPY-INFO: |                             Task Result Details                              |
2020-05-18T14:17:31: %EASYPY-INFO: +------------------------------------------------------------------------------+
2020-05-18T14:17:31: %EASYPY-INFO: Task-1: pyATS
2020-05-18T14:17:31: %EASYPY-INFO: |-- common_setup                                                          PASSED
2020-05-18T14:17:31: %EASYPY-INFO: |   `-- connect                                                           PASSED
2020-05-18T14:17:31: %EASYPY-INFO: |-- BGP_Neighbors_Established                                             PASSED
2020-05-18T14:17:31: %EASYPY-INFO: |   |-- learn_bgp                                                         PASSED
2020-05-18T14:17:31: %EASYPY-INFO: |   `-- check_bgp                                                         PASSED
2020-05-18T14:17:31: %EASYPY-INFO: `-- common_cleanup                                                        PASSED
2020-05-18T14:17:31: %EASYPY-INFO:     `-- clean_everything                                                  PASSED
2020-05-18T14:17:31: %EASYPY-INFO: Sending report email...
2020-05-18T14:17:31: %EASYPY-INFO: Missing SMTP server configuration, or failed to reach/authenticate/send mail. Result notification email failed to send.
2020-05-18T14:17:31: %EASYPY-INFO: Done!
```

Report da validação do status do BGP:
```
2020-05-18T14:17:31: %AETEST-INFO: The result of section learn_bgp is => PASSED
2020-05-18T14:17:31: %AETEST-INFO: +------------------------------------------------------------------------------+
2020-05-18T14:17:31: %AETEST-INFO: |                          Starting section check_bgp                          |
2020-05-18T14:17:31: %AETEST-INFO: +------------------------------------------------------------------------------+
2020-05-18T14:17:31: %SCRIPT-INFO: | Device   | Peer         | State       | Pass/Fail   |
2020-05-18T14:17:31: %SCRIPT-INFO: |----------+--------------+-------------+-------------|
2020-05-18T14:17:31: %SCRIPT-INFO: | CORE-SW2 | 189.77.20.30 | Established | Passed      |

2020-05-18T14:17:31: %AETEST-INFO: Passed reason: All BGP Neighbors are established
2020-05-18T14:17:31: %AETEST-INFO: The result of section check_bgp is => PASSED
2020-05-18T14:17:31: %AETEST-INFO: The result of testcase BGP_Neighbors_Established is => PASSED
```

Ainda no mesmo diretório, ao executar o comando `pyats logs view` você consegue fazer o monitoramento em tempo real do serviço pela web:

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/report.jpg)

E aí, o que achou dessas ferramentas? Comente aqui embaixo, vai ser legal contar com sua presença por aqui.
