---
title: "Fundamentos NetDevOps"
date: 2020/12/10
tags:
 - NetDevOps
category:
 - NetDevOps
---

Fala NetCoders, no mundo das Redes, NetDevOps é uma das palavras que está na moda no período de 1 ano para cá. Mas o que é NetDevOps, ou mesmo DevOps? Na série sobre NetDevOps falaremos sobre o conceito por trás desta metodologia.

Neste artigo iremos abordar os fundamentos de forma introdutória sobre NetDevOps, Falaremos das ferramentas que completam esta metodologia.

Este artigo é muito importante para construirmos uma casca sólida de conhecimento introdutório sobre o que resume NetDevOps, ou mesmo, a metodologia CI/CD imposta em um ambiente de Network Infrastructure As Code.

Pegue seu café e vamos nessa!

## Tópicos

* Fundamentos sobre IaC
* CI/CD
  * Sistemas CI/CD
  * Ferramentas de deploy
  * Lab para testes automatizados

## Fundamentos sobre IaC

Para construir uma solução NetDevOps baseada em Infraestrutura como Código (IaC) em sua própria rede, precisaremos entender os principais pilares da Infraestrutura como Código.

É essencial entender estes conceitos, como eles interagem e quais ferramentas pertencem a qual pilar, isso permitirá que você crie uma solução IaC poderosa e robusta em seu ambiente de NetOps.

A infraestrutura como código geralmente é construída sobre quatro itens principais:

### 1 - Controle de versão:

  * Os arquivos de controle de versão é essencial para manter e versionar a origem dos arquivos que representam a configuração de elementos de sua infraestrutura. Ou seja, é essencial você descrever a sua infraestrutura em código e armazenar estes códigos em um sistema de controle de versão. 
  
  * Também é essencial documentar a topologia em um arquivo README, não documentar sobre as tecnologias e protocolos que rodam na sua topologia, mas sim, documentar como será o processo CI/CD que sua topologia irá ter.

### 2 - CI/CD:
  
  * CI/CD significa Integração Contínua e Implantação ou Entrega Contínua e descreve os sistemas usados para gerenciar e executar as mudanças ou implantação de sua infraestrutura.

### 3 - Testes:
  
  * Os testes permitem que você possa validar que as alterações executadas por esse processo serão bem-sucedidas e não causarão alterações indesejadas ou não intencionais em sua infraestrutura.

### 4 - Ferramentas de deploy:
  
  * Essas ferramentas são variadas e podem assumir muitas formas, dependendo do sistema IaC que está sendo construído. Para o NetDevOps, frequentemente serão ferramentas ou sistemas que podem se comunicar com o plano de gerenciamento da rede (via SSH ou HTTPS) para implementar mudanças.
  
## CI/CD 
### CI:

  * Foca em escrever o código descrevendo sua topologia em uma plataforma de build, é feito o teste do código e validação daquilo que havia sido feito no código. Dependendo da plataforma de build, ela verifica até as vulnerabilidades que teve no seu código durante o teste.
  
   Ou seja, CI nada mais é que você ter um repositório onde é armazenado códigos da sua infraestrutura, cada alteração é feita uma compilação deste código, ele      será buildado. após os testes e verificações, é gerado uma nova versão de build deste código, dessa forma você consegue verificar as versões que teve este        código baseado nas interações do seu time de NetOps.
  
   Você sempre terá seu código buildavel, sempre quando tiver alguma interação, será executado um fluxo de trabalho onde irá rodar estes testes.

### CD:
  
  * Continuos Delivery foca no deploy. O código após passar pela esteira de testes no ambiente de demo, ele passa por outra esteira, essa esteira foca na validação em ambiente de demo, após essa validação, é passado para o deploy no ambiente de produção. Abaixo segue uma imagem de como seriam as fases CI/CD:
  
  ![alt]({{ site.url }}{{ site.baseurl }}/assets/images/cicd.png)
  
   Os benefícios de CI/CD nada mais é que, se o seu código estiver com erro, ele para no teste (CI), ele não será deployado para o ambiente de produção.
  
   Dica: O ideal é sempre trabalhar com 02 branches, uma será a principal e a outra será para ambiente de demo. Caso o seu código passe na esteira CI e é            validado, será efetuado o merge desta integração continua para o ambiente branch de produção.
  
   Dica: Branches ("ramos") são utilizados para desenvolver funcionalidades isoladas umas das outras. O branch master é o branch "padrão" quando você cria um repositório. Use outros branches para desenvolver e mescle-os (merge) ao branch master após a conclusão.

### Sistemas CI/CD 

Os sistemas CI/CD mais comuns são:

- **[Jenkins](https://jenkins.io/)**
- **[Travis CI](https://travis-ci.com/)**
- **[CircleCI](https://circleci.com/)**
- **[GitLab](https://about.gitlab.com/)**
- **[GitHub](https://github.com/)**
- **[AWS CodePipeline](https://aws.amazon.com/codepipeline/)**
- **[Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/)**
  
### Ferramentas de deploy

Esse pilar é muito diversificado, pois a ferramenta de implantação e configuração que você escolhe depende muito de seus casos de uso. Ele pode ser orientado pelo tipo de infraestrutura que você deseja implantar / configurar, como deseja configurá-la ou até mesmo pela familiaridade existente com uma determinada ferramenta. Mas as mais comuns são:

- **[Terraform](https://www.terraform.io/)**
- **[Fantoche](https://puppet.com/)**
- **[Ansible](https://www.ansible.com/)**
- **[SaltStack](https://www.saltstack.com/)**
- **[AWS CloudFormation](https://aws.amazon.com/cloudformation/)**

### Lab para testes automatizados

* Vrnetlab
* EVE-NG
* GNS3

Bom, por hoje é só. E aí, o que achou deste artigo? Comente aqui embaixo, vai ser legal contar com sua presença por aqui.
 
 
 
 
 
 
