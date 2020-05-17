---
title: "Network Programmability - YAML e XML"
date: 2020-03-26
tags:
 - python
 - YAML
 - XML
category:
 - python
---

No artigo de hoje, iremos falar dos conceitos por trás das linguagens de estrutura e formatos de dados, como YAML e XML. Pegue seu café, se acomode e vamos nessa! Até hoje, nós da área de infraestrutura não precisávamos saber programar. Ok, o conhecimento em linguagens de programação é muito útil para automatizarmos tarefas e rotinas comuns de testes, mas nunca foi um grande requisito no currículo de um CCNP.

Para darmos ênfase neste artigo, precisamos ressaltar alguns conceitos relacionados à estrutura e modelagem de dados.

Ao citar a linguagem XML, o exemplo mais comum no mundo de network é que o JunOS estrutura os dados que o SO utiliza via linguagem XML. Ex - Basta setar o comando > show isis adjacency detail | display xml rpc que ele nos mostrará a saída descrita abaixo:
``` xml 
<rpc-reply xmlns:junos="http://xml.juniper.net/junos/16.1R1/junos">
     <rpc>
        <get-isis-adjacency-information>
            <detail/>
        </get-isis-adjacency-information>
      </rpc>
         <cli>
            <banner></banner>
         </cli>
</rpc-reply>
```

Vale ressaltar que existe inúmeros formatos de estrutura de dados à nossa disposição, devemos entender o propósito de cada modelo de dados, cada um foi criado para um caso diferente.

Depois de entender os formatos de dados nos quais irei detalhar melhor a seguir, é importante compreender quais tipos de dados podem ser representados pelos formatos dos quais serão ditos neste artigo. O objetivo das linguagens de estruturação de dados é fazer com que, números, palavras e até complexos objetos de instância de software se comuniquem.

### String

Indiscutivelmente, é o tipo de dado mais fundamental. Esse parâmetro é a maneira mais comum de representar uma sequência de números, letras ou símbolos. Caso você queira representar alguma frase em um determinado idioma em um dos formatos de estrutura de dados, provavelmente usará uma string para fazer isso. No python, você pode representar uma string através do parâmetro str ou unicode.

### Integer

O parâmetro integer representa tipos de dados que contém valores numéricos, mas para maioria das pessoas, valores numéricos se baseiam em números inteiros. O número inteiro é exatamente aquilo que você aprendeu na escola – Um número inteiro, positivo ou negativo. Existe outro tipo de dado como float que você pode usar ao descrever valores não inteiros. Python utiliza o parâmetro int para declarar valores do tipo inteiro.

### Boolean

Um dos tipos de dados mais simples é o booleano, um valor simples que é verdadeiro ou falso. Esse é um tipo muito popular usado quando um programador deseja saber o resultado de uma operação ou se dois valores são iguais um ao outro, por exemplo. Isso é conhecido como o tipo bool no python.

> Os Network Developers obtém uma estrutura de automação poderosa quando combinam scrpts python com formatos de dados JSON, XML ou YAML junto com as bibliotecas de automação.

## Linguagens de formatação de dados

### YAML

Esse formato de estrutura de dados é muito amigável aos olhos dos seres humanos. Se você comparar o YAML com outros formatos de dados que discutiremos como XML ou JSON, parece fazer a mesma coisa - Representa construções como listas, pares de valores-chave, sequências de caracteres e números inteiros.

No entanto, como você verá em breve, o YAML faz isso de uma maneira excepcionalmente legível para humanos. O YAML é muito fácil de ler e escrever se você entender os tipos de dados básicos discutidos anteriormente.

Esse é um grande motivo pelo qual um número crescente de ferramentas estão utilizando o YAML como um método para definir um fluxo de trabalho de automação ou fornecer um conjunto de dados para trabalhar (como uma lista de VLANs). É muito fácil usar o YAML para ir de zero a um funcional fluxo de trabalho de automação ou para definir os dados que você deseja enviar para um dispositivo de rede.

Na imagem descrita abaixo, percebe-se que no início contém três hifens, isso significa que todo início de um código .yml deve conter os três hifens.
``` yaml
---
- core switch
- 6500
- false
- ['switchport', 'mode', 'access']
```

Os três hifens também servirá para você declarar outras instâncias e assim indicar vários documentos dentro de um arquivo ou fluxo de dados.

Percebe-se que o YAML imita a arquitetura do python, dessa forma obtemos vantagem ao trabalhar com os dois juntos. No exemplo acima, temos uma lista que indica quatro itens, cada item e um tipo totalmente exclusivo.

O primeiro item, core switch, é um tipo de string. O segundo, 6500, é interpretado como um número inteiro. O terceiro é interpretado como um booleano. Essa “interpretação" é executada por um intérprete YAML, como PyYAML. PyYAML, em específico, faz um bom trabalho ao se referir no tipo de dados que o usuário está tentando comunicar.

Por exemplo, você pode escrever False, como no exemplo acima, ou você poderia escrever não, desligado ou simplesmente n. Todos eles acabam significando a mesma coisa: um valor booleano falso. Esta é uma grande razão pela qual o YAML é frequentemente usado como uma interface humana para muitos softwares.

O exemplo a seguir iremos nos referir nos formatos de dicionário que contém nessa linguagem.
``` yaml
---
- core switch
- 6500
- false
- ['switchport', 'mode', 'access']

---
{juniper: EX9200, cisco: 6500,
VMware: ['esxi', 'vcenter']}
```

Na imagem acima representamos dois tipos de dicionário, a maioria dos analisadores interpretará esses dois documentos YAML exatamente da mesma forma, mas o primeiro é obviamente muito mais legível. Se você estiver procurando por um mais legível, utilize as opções mais detalhadas.

Caso contrário, você provavelmente nem deseja usar o YAML e talvez queira algo como JSON ou XML. Por exemplo, em uma API, a legibilidade é quase irrelevante, a ênfase está na velocidade e no amplo suporte de software.

Para o conhecimentos sobre os conceitos que envolve esse formato de dados, irei utilizar python para ler um arquivo .yml e nos retornar um dicionário, essa é uma maneira poderosa de representar determinados tipos de dados.
``` yaml
---
- juniper: EX9200
- cisco:   6500
- VMware:
   - esxi
   - vcenter
   
# script python    
import yaml
with open("/home/thiago/yaml.yml") as f:
  result = yaml.load(f)
  print(result)
  type(result)
```

O primeiro script está descrito em código YAML, abaixo dele, foi atribuído alguns parâmetros em python para abrir este arquivo .yml e nos retornar um dicionário ao ser compilado.

Na segunda linha indicamos o caminho no qual será carregado o arquivo, o arquivo está sendo representado pelo atributo “f”.

Na terceira linha foi criada uma variável chamada “result’, dentro dessa variável, inserimos a função load(), essa função carrega o módulo YAML e nos permite carregar a saída deste arquivo em um dicionário que está alocado na variável “result”. Abaixo mostra o arquivo .py sendo compilado e retornando um dicionário baseado na estrutura YAML.
```
[{'juniper': 'EX9200'}, {'cisco', '6500'}, {'VMware': ['esxi', 'vcenter']}]
```

### XML

Um overview rápido, o XML compartilha algumas semelhanças com o que vimos com o YAML. Por exemplo, é inerentemente hierárquico, podemos facilmente incorporar dados em uma construção pai:
``` xml
<device>
   <vendor>Cisco</vendor>
   <model>Nexus 7700</model>
   <osver>NXOS 6.1</osver>
</device>
```

Neste exemplo, o elemento "device" é considerado a raiz. Embora o espaçamento e o recuo não seria relevante para a validade do XML. É também o pai dos elementos dentro dele, são eles, vendor, model e osver.

Eles são chamados de filhos do elemento "device" e são considerados irmãos um do outro. Isso é muito conveniente para armazenar metadados sobre dispositivos de rede.

Em um documento XML, pode haver várias instâncias da tag "device" (ou vários elementos "device"), talvez aninhadas em uma tag "devices" mais ampla.

> Os elementos XML também podem ser alocados como atributos, isso quer dizer que, quando uma informação contém alguns metadados associados, pode não ser apropriado utilizar elemento filho e sim, associá-lo como atributo.

``` xml
<devide type=”datacenter-switch” />
```

Desde o início deste artigo, descrevemos os formatos de dados como permitindo que aplicativos - ou dispositivos, como dispositivos de rede - troquem informações de maneiras padronizadas.

O XML é uma dessas maneiras padronizadas para troca de informações. No entanto, formatos de dados como XML não impõe que tipo de dados estão contidos nos vários campos e valores. Para garantir que o tipo certo de dados estejam alocados nos elementos XML corretos, temos a definição do esquema XML (XSD).

O conceito deste esquema nada mais é que, cria-se um arquivo .xsd, gera um código python a partir deste arquivo, ao compilar, teremos o XML que precisamos.

Esse é apenas o primeiro passo de um mundo de coisas que podemos fazer no segmento de automação com base no avanço sobre o tema e conforme vão surgindo demandas.
