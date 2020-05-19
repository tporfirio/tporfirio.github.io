---
title: "Criando um playbook com os módulos ios_config, ios_vlan e ios_l2_interface"
date: 2020-04-28
tags:
 - Ansible
category:
 - Ansible
---

No artigo de hoje, iremos trabalhar com módulos do Ansible para criar um playbook onde irá fazer o provisionamento das seguintes tarefas:

* Habilitar o VTP transparent mode
* Criar VLANS 10, 20, 30 e 40
* Habilitar 802.1q nas interfaces
* Habilitar trunk mode nas interfaces
* Configurar switch virtual interface com HSRP
* Dimensionar HSRP load balancing

Iremos utilizar o lab mostrado na imagem abaixo. Este lab encontra-se no meu repositório do GitHub junto com o código usado neste artigo.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/lab7.jpg)

Bom, iremos montar nosso playbook para executar a primeira e a segunda task:
``` yaml
---
- name: Configuring devices # Nome do manual 
  hosts: all # Irá executar todos os hosts de todos os grupos que estão alocados no arquivo hosts
  gather_facts: false # Recolhe informações do dispositivo e retorna a saída em YAML
  
  vars: # Variável de conexão
    ansible_connection: network_cli
    ansible_network_os: ios
    ansible_user: teste
    ansible_ssh_pass: teste
```

A variável de conexão nada mais é que, o registro de conexão do device remoto, essas variáveis também faz a conexão de usuários cadastrados no tacacs ou radius.

Dica: Ao declarar minhas variáveis de comunicação, utilizei o parâmetro network_cli que se refere no modo de conexão CLI sobre SSH. Vale ressaltar que não são todos os módulos de redes que suportam esse tipo de protocolo de comunicação.

Iremos descrever a task para enviar a configuração que queremos executar no device remoto:
``` yaml
tasks:
    - name: Configurando VTP Transparent mode e NTP em todos os switches 
      ios_config: # Módulo de configuração       
        lines:
          - vtp domain ansible
          - vtp mode transparent         
        
      register: print_output # Armazenando os dados executados no módulo acima

    -  debug: var=print_output.stdout_lines # Imprimindo os dados armazenados 
    
    # Criar VLANS 10, 20, 30 e 40
    - name: Criando VLANS 10, 20, 30 e 40
      ios_vlan: # Módulos contendo apenas parametrização de vlan
        aggregate: # Executa a lista de itens ordenados pelo parâmetro vlan_id
          - vlan_id: 10              
            name: VLAN 10          
            state: active

          - vlan_id: 20              
            name: VLAN 20          
            state: active 

          - vlan_id: 30              
            name: VLAN 30          
            state: active

          - vlan_id: 40              
            name: VLAN 40          
            state: active          
          
      register: print_output
```

Bom, com poucas linhas de código, podemos configurar o VTP e vlan id e replicar em todos os devices da topologia.

Agora iremos montar a terceira task:
``` yaml
# 802.1q nas interfaces
- name: Configuring 802.1q in the interfaces in core switches
  hosts: ansible_core
  gather_facts: false

  vars:
    ansible_connection: network_cli
    ansible_network_os: ios
    ansible_user: teste
    ansible_ssh_pass: teste

  tasks:    
    - name: Habilitando 802.1q nas interfaces dos switches core
      ios_config:        
        parents: "interface {{ item.interface }}" # Entra nas int listadas no dicio with_items
        lines: # Executa comandos dentro do modo config-if
          - switchport trunk encapsulation dot1q
      with_items:
        - { interface : Ethernet0/0 }
        - { interface : Ethernet0/1 }
        - { interface : Ethernet0/2 }
        - { interface : Ethernet0/3 }      

      register: print_output
```
Essa task consiste em habilitar o 802.1q nas interfaces dos core que conecta com os switches de acessos. Agora iremos executar a qaurta task, a próxima task irá habilitar o trunk mode para ocorrer distribuição de vlans pelo link:
``` yaml
    - name: HABILITANDO TRUNK MODE NAS INTERFACES ATIVAS DOS SWITCHES CORE
      ios_l2_interface:
        aggregate:
          - name: Ethernet0/0
            mode: trunk
            trunk_allowed_vlans: 1-4094
          - name: Ethernet0/1
            mode: trunk
            trunk_allowed_vlans: 1-4094
          - name: Ethernet0/2
            mode: trunk
            trunk_allowed_vlans: 1-4094
          - name: Ethernet0/3
            mode: trunk
            trunk_allowed_vlans: 1-4094
            
      register: print_output
```
A quarta task foi descrita dentro da variável de conexão da task anterior, pois, a quarta task irá executar comandos no mesmo grupo de devices.

Iremos executar a terceira e a quarta task novamente, porém, alterando as interfaces e o grupo de devices, iremos aplicar as configurações no grupo de switches de acessos:
``` yaml
# 802.1q nas interfaces
- name: Configuring 802.1q in the interfaces in access switches
  hosts: ansible_access
  gather_facts: false

  vars:
    ansible_connection: network_cli
    ansible_network_os: ios
    ansible_user: teste
    ansible_ssh_pass: teste

  tasks:    
    - name: Habilitando 802.1q nas interfaces dos switches access
      ios_config:        
        parents: "interface {{ item.interface }}"
        lines:
          - switchport trunk encapsulation dot1q
      with_items:
        - { interface : Ethernet1/0 }
        - { interface : Ethernet1/1 }  

      register: print_output

      - name: HABILITANDO TRUNK MODE NAS INTERFACES ATIVAS DOS SWITCHES ACCESS
        ios_l2_interface:
          aggregate:
           - name: Ethernet1/0
             mode: trunk
             trunk_allowed_vlans: 1-4094
           - name: Ethernet1/1
             mode: trunk
             trunk_allowed_vlans: 1-4094          
            
       register: print_output

```
A quinta task faz junção com a sexta task. Essas duas tasks irão utilizar o módulo ios_config para criar dois grupos de HSRP seguidos de SVIs para fazer load balancing de tráfego entre as vlans.

Configurando SVIs e HSRP load balancing no SW_CORE_1:
``` yaml
#Configuring HSRP group 1 in the SW_CORE_1
- name: Configuring switch virtual interface with HSRP in the Core SW_CORE_1
  hosts: SW_CORE_1
  gather_facts: false

  vars:
    ansible_connection: network_cli
    ansible_network_os: ios
    ansible_user: teste
    ansible_ssh_pass: teste

  tasks:    
    - name: Configurando switch virtual interface com HSRP grupo 1 no SW_CORE_1
      ios_config:        
        parents: "interface {{ item.interface }}"
        lines:
          - "ip address {{ item.address }} {{ item.mask }}" 
          - "no shutdown"

          - "standby 1 ip 192.168.11.3"
          - "standby 1 preempt"
          - "standby 1 priority 120"

          - "standby 1 ip 192.168.11.67"
          - "standby 1 preempt"
          - "standby 1 priority 120"

          - "standby 1 ip 192.168.11.131"
          - "standby 1 preempt"
          - "standby 1 priority 120"

          - "standby 1 ip 192.168.11.195"
          - "standby 1 preempt"
          - "standby 1 priority 120"
          
      with_items:
        - { interface : vlan 10, address : 192.168.11.1, mask : 255.255.255.192 }
        - { interface : vlan 20, address : 192.168.11.65, mask : 255.255.255.192 }
        - { interface : vlan 30, address : 192.168.11.129, mask : 255.255.255.192 }
        - { interface : vlan 40, address : 192.168.11.193, mask : 255.255.255.192 }

      register: print_output

    #Configuring HSRP group 2 in the SW_CORE_1
    - name: Configurando switch virtual interface com HSRP grupo 2 no SW_CORE_1
      ios_config:        
        parents: "interface {{ item.interface }}"
        lines:          
          - "standby 2 ip 192.168.11.4"        
          - "standby 2 preempt"
          - "standby 2 priority 110"

          - "standby 2 ip 192.168.11.68"
          - "standby 2 preempt"
          - "standby 2 priority 110"

          - "standby 2 ip 192.168.11.132"
          - "standby 2 preempt"
          - "standby 2 priority 110"

          - "standby 2 ip 192.168.11.196"
          - "standby 2 preempt"
          - "standby 2 priority 110"

      with_items:
        - { interface : vlan 10 }
        - { interface : vlan 20 }
        - { interface : vlan 30 }
        - { interface : vlan 40 }      

      register: print_output
```

Configurando SVIs e HSRP load balancing no SW_CORE_2:
``` yaml
#Configuring HSRP group 2 in the SW_CORE_2
- name: Configuring switch virtual interface with HSRP Core SW_CORE_2
  hosts: SW_CORE_2
  gather_facts: false

  vars:
    ansible_connection: network_cli
    ansible_network_os: ios
    ansible_user: teste
    ansible_ssh_pass: teste

  tasks:    
    - name: Configurando switch virtual interface com HSRP grupo 2 no SW_CORE_2
      ios_config:        
        parents: "interface {{ item.interface }}"
        lines:
          - "ip address {{ item.address }} {{ item.mask }}" 
          - "no shutdown"

          - "standby 2 ip 192.168.11.4"   
          - "standby 2 preempt"
          - "standby 2 priority 120"

          - "standby 2 ip 192.168.11.68"
          - "standby 2 preempt"
          - "standby 2 priority 120"

          - "standby 2 ip 192.168.11.132"
          - "standby 2 preempt"
          - "standby 2 priority 120"

          - "standby 2 ip 192.168.11.196"
          - "standby 2 preempt"
          - "standby 2 priority 120"
          
      with_items:
        - { interface : vlan 10, address : 192.168.11.2, mask : 255.255.255.192 }
        - { interface : vlan 20, address : 192.168.11.66, mask : 255.255.255.192 }
        - { interface : vlan 30, address : 192.168.11.130, mask : 255.255.255.192 }
        - { interface : vlan 40, address : 192.168.11.194, mask : 255.255.255.192 }

      register: print_output

    #Configuring HSRP group 1 in the SW_CORE_2
    - name: Configurando switch virtual interface com HSRP grupo 1 no SW_CORE_2
      ios_config:        
        parents: "interface {{ item.interface }}"
        lines:          
          - "standby 1 ip 192.168.11.3"       
          - "standby 1 preempt"
          - "standby 1 priority 110"

          - "standby 1 ip 192.168.11.67"
          - "standby 1 preempt"
          - "standby 1 priority 110"

          - "standby 1 ip 192.168.11.131"
          - "standby 1 preempt"
          - "standby 1 priority 110"

          - "standby 1 ip 192.168.11.195"
          - "standby 1 preempt"
          - "standby 1 priority 110"

      with_items:
        - { interface : vlan 10 }
        - { interface : vlan 20 }
        - { interface : vlan 30 }
        - { interface : vlan 40 }      
        
      register: print_output
```

Bom, esses são alguns exemplos de como trabalhar com módulos ansible playbook. E aí, o que achou deste artigo? Dá um pulo lá no nosso [Linkedin](https://www.linkedin.com/company/ccna-student/?viewAsMember=true) para ficar por dentro de novas publicações, vai ser legal contar com sua presença por lá. Ficamos por aqui e nos vemos no próximo post.
