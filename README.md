# Projeto Componentes MarketPlace

# Equipe
* Caio Vitor
* Dennis Phillips
* Gustavo Nakahara
* Manoel Teixeira
* Wilson Costa

# Nível 1

## Diagrama Geral do Nível 1

![Diagrama no nível 1](images/coreografia.png)

### Detalhamento da interação de componentes

* O `componente Usuário` dispõe de uma interface `ISeleçãoProduto` que permite que o usuário selecione um produto de uma lista obtida através da interface `IListaProdutos`.
* O `componente Marketplace` recebe do `componente Usuário` o produto selecionado através da interface `IProduto`.
* O `componente Marketplace` aciona o `componente Leilão` através da interface 'ISolicitaLeilão'.
* O `componente Leilão` publica no barramento no tópico “`leilão/demanda/<produto>`” informando que foi iniciado um leilão para obtenção dos preços do produto selecionado.
* Os componentes do tipo Fornecedor que assinam o tópico “`leilão/demanda/#`" publicam o preço do produto no barramento no tópico “leilão/lance/<produto>/<preço>”
* O componente Leilão assina o tópico “`leilão/lance/#`”, reúne os melhores lances, e passa as informações para o `componente Marketplace` através da interface `IResultadoLeilão`.
* O resultado das consultas feitas pelo `componente Marketplace`, a respeito do produto selecionado, é devolvido para o `componente Usuário` através da interface `IDetalhesProduto`.
* Com as informações obtidas, o cliente realiza seu pedido através da interface `IConfirmaPedido` do `componente Usuário`.
* O pedido é passado para o `componente Marketplace` através da interface `IPedido`. O `componente Marketplace` passa o pedido através da interface `IPedido` para o `componente ControlePedido`.
* O `componente ControlePedido` publica no Barramento B uma mensagem  com o tópico “`confirma/pedido/{código_pedido}`” contendo os detalhes do pedido confirmados pelo cliente para o pagamento.
* O `componente Pagamento`, que assina o tópico “`confirma/pedido/#`”, recebe a mensagem e guarda no `componente TransaçõesPagamento` um histórico da transação financeira, usando a interface `IPagamento`.
* O `componente Pagamento` publica no Barramento B uma mensagem contendo o status do pagamento com o tópico “`acompanhamento/{id_pedido}`”.
* O `componente ControlePedido` assina o tópico “`acompanhamento/#`” e recebe a mensagem publicada pelo `componente Pagamento`.
* O `componente Logística` assina o tópico “`confirma/pedido/#`” e recebe o pedido para a entrega e guarda no `componente TransaçõesLogística` um histórico da transação de logística, usando a interface `ILogistica`.
* O `componente Logística` publica a mensagem com o tópico “`acompanhamento/{id_pedido}`” que é recebido pelo `componente ControlePedido` que assina o tópico “`acompanhamento/#`”.
* O `Componente Usuário` obtém do `componente ControlePedido` a informacão do status do pedido através da interface `IStatusPedido`. É disponibilizada ao cliente a interface `ISolicitaStatus`.
* O componente `Gestão` obtém do `componente ControlePedido` as informações referentes aos pedidos através da interface `IPedidos`. Este componente obtém também do componente Logística informações referentes a entrega através da interface 'IStatusLogistica'.
* O componente `Gestão` disponibiliza duas interfaces aos gestores do sistema, a `ISolicitaLogistica` e `ISolicitaPedidos`.

## Componente Usuário

* Componente para funcionar como View dos clientes. 
* Ele pode ser substituído de acordo com a interface a ser utilizada pelo usuário. 
* Disponibiliza interfaces para que o cliente selecione um produto, confirme o pedido e acompanhe o status do pedido.

![Componente](images/diagrama-componente-usuario.png)

**Interfaces**
* ISelecaoProduto
* IConfirmaPedido
* ISolicitaStatus
* IStatusPedido
* IProduto
* IListaProdutos
* IDetalhesProduto
* IPedido

## Componente Gestão

* Componente para funcionar como View dos gestores. 
* Disponibiliza interfaces para que os gestores acompanhem a entrega e informações de todos os pedidos.

![Componente](images/diagrama-componente-gestao.png)

**Interfaces**
* ISolicitaPedidos
* ISolicitaLogistica
* IStatusLogistica
* IPedidos

## Componente Marketplace

* Componente para funcionar como Controller do Marketplace. 
* Acessa os Models contendo o estoque de produtos, detalhes dos produtos e cadastro dos usuários.
* Interage com o View do usuário, para enviar informações para o cliente realizar a compra e recebe os dados escolhidos do pedido.
* Aciona o componente Leilão para solicitar o leilão invertido e o componente ControlePedido para enviar os pedidos realizados.
* Recebe mensagens dos fornecedores para atualizar os Models.

![Componente](images/diagrama-componente-marketplace.png)

**Interfaces**
* IProduto
* IListaProdutos
* IDetalhesProduto
* IPedido
* ISolicitaLeilao
* IResultadoLeilao
* IConfirmaEstoque
* IInsereProduto
* IObtemProduto
* IInsereEstoque
* IObtemEstoque
* IInsereCadastro
* IObtemCadastro

## Componente ControlePedido

* Componente para funcionar como Controller do andamento dos pedidos. 
* Recebe os pedidos feitos pelos clientes e os distribui pelo barramente para os componentes responsáveis pelo pagamento e entrega.
* Também disponibiliza interfaces para informar os gestores do MarketPlace sobre o andamento dos pedidos.

![Componente](images/diagrama-componente-controle_pedido.png)

**Interfaces**
* IPedidos
* IStatusPedido
* IPedido
* IStatus
* IConfirmaPedido
* IInserePedido
* IObtemPedido

## Componente Leilão

* Componente responsável pelo controle do leilão invertido.
* Publica no barramento a mensagem com o produto demandado e recebe as ofertas dos fornecedores.
* Envia ao componente Marketplace o resultado do leilão, através da seleção dos melhores valores.

![Componente](images/diagrama-componente-leilao.png)

**Interfaces**
* ISolicitaLeilao
* IResultadoLeilao
* ILance
* IDemanda

## Componente Fornecedor

* Componente dos fornecedores que fazem parte do Marketplace.
* Ele possui interfaces para que os fornecedore participem do leilão invertido e façam atualizações da informações de estoque.

![Componente](images/diagrama-componente-fornecedor.png)

**Interfaces**
* IConfirmaEstoque
* IDemanda
* ILance

## Componente Logística

* Componente responsável pela entrega dos pedidos e registro destas transações.

![Componente](images/diagrama-componente-logistica.png)

**Interfaces**
* IStatus
* IConfirmaPedido
* ILogistica
* IStatusLogistica

## Componente Pagamento

* Componente responsável pelo pagamento dos pedidos.

![Componente](images/diagrama-componente-pagamento.png)

**Interfaces**
* IConfirmaPedido
* IStatus
* IPagamento

## Componente Cadastro

* Componente que funciona como Model para o cadastro de usuários.

![Componente](images/diagrama-componente-cadastro.png)

**Interfaces**
* IObtemCadastro
* IInsereCadastro

## Componente Estoque

* Componente que funciona como Model para o estoque de produtos do MarketPlace.

![Componente](images/diagrama-componente-estoque.png)

**Interfaces**
* IObtemEstoque
* IInsereEstoque

## Componente Produto

* Componente que funciona como Model para os detalhes dos produtos do MarketPlace.

![Componente](images/diagrama-componente-produto.png)

**Interfaces**
* IObtemProduto
* IInsereProduto

## Componente Pedido

* Componente que funciona como Model para os pedidos do MarketPlace.

![Componente](images/diagrama-componente-pedido.png)

**Interfaces**
* IObtemPedido
* IInserePedido

## Componente TransaçõesPagamento

* Componente que funciona como Model para as transações de pagamentos.

![Componente](images/diagrama-componente-transacoes_pagamento.png)

**Interfaces**
* IPagamento

## Componente TransaçõesLogística

* Componente que funciona como Model para as transações de logística.

![Componente](images/diagrama-componente-transacoes_logistica.png)

**Interfaces**
* ILogistica



## Detalhamento das Interfaces

## Detalhamento das Interfaces

### Interface `IConfirmaEstoque`

> Usada pelo componente Fornecedor para postar mensagens com informações de estoque de determinado produto.

**Tópico**: `estoque/região/confirma/{cod_produto}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/classes-json-estoque.png)

~~~json
{
	"codigo": "112233",
	"nome": "Pendrive",
	"marca": "Kingston",
	"setor": "Informática",
	"subsetor": "dispositivos de armazenamento",
	"detalhes-produto": 
		{
			"modelo": "DataTraveler",
			"capacidade": 16
		},
	"regiao-estoque": 
		{
			"cidade": "Salvador",
			"estado": "Bahia"
		},
	"estoque": 2000
}

~~~

Detalhamento da mensagem JSON:

**Estoque**

Atributo | Descrição
-------| --------
`código` | `Código de identificação do produto no Marketplace`
`nome` | `Nome associado ao produto`
`marca` | `Nome comercial do fabricante`
`setor` | `Setor do Marketplace associado ao produto`
`subsetor` | `Sub-setor do Marketplace associado ao setor do atributo acima`
`detalhes-produto` | `Mensagem JSON com detalhes específicos ligados ao produto em questão. O conteúdo dessa mensagem JSON se adapta ao tipo e setor do produto em questão`
`região-estoque` | `Mensagem JSON com o local do estoque`
`estoque` | `Quantidade de itens em estoque`

**Região do Estoque**

Atributo | Descrição
-------| --------
`cidade` | `Nome da cidade onde está localizado o estoque`
`estado` | `Nome do estado onde está localizado o estoque`

**Detalhes do Produto**

Atributo | Descrição
-------| --------
`modelo` | `Modelo associado ao produto`
`capacidade` | `Capacidade de armazenamento do produto`

### Interface `IDemanda`

> Usada pelo componente Leilão para iniciar o leilão de preços de um determinado produto.

**Tópico**: `leilão/demanda/{cod_produto}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/classes-json-demanda.png)

~~~json
{
	"código": 112233,
	"timestamp": "18-09-2020T11:33:00"
}

~~~

Detalhamento da mensagem JSON:

**Demanda do Leilão**

Atributo | Descrição
-------| --------
`código` | `Código do produto no Marketplace`
`timestamp` | `Horário em que o leilão foi iniciado`

### Interface `ILance`

> Usada pelo componente Fornecedor para publicar mensagens com o preço de determinado produto.

**Tópico**: `leilão/lance/{cod_produto}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/classes-json-lance.png)

~~~json
{
	"código": "112233",
	"preço": 50.00,
	"desconto": 15,
	"timestamp": "18-09-2020T11:34:00"
}

~~~

Detalhamento da mensagem JSON:

**Lance**

Atributo | Descrição
-------| --------
`código` | `Código do produto no Marketplace`
`preço` | `Preço unitário estipulado pelo Fornecedor (em Reais)`
`desconto` | `Desconto oferecido pelo Fornecedor a ser aplicado sobre o preço (em porcentagem)`
`timestamp` | `Data e hora em que a mensagem com o lance foi publicada pelo Fornecedor`

### Interface `IConfirmaPedido`

> Interface usada pelo componente Controle Pedido para publicar mensagens de confirmação de pedido feito por um cliente.

**Tópico**: `confirma/pedido/{id_pedido}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/classes-json-pedido.png)

~~~json
{
	"id": 999,
	"cliente": "wilsoncosta",
	"fornecedor": "magazineluiza",
	"pedidos":
	{
		"pedido":
			{
				"codigo": "112233",
				"quantidade": 2,
				"valor": 60
			},
		"pedido":
			{
				"codigo": "223344",
				"quantidade": 1,
				"valor": 100
			}
	},
	"timestamp": "18-09-2020T11:40:00",
	"endereço":
		{
			"cidade": "Salvador",
			"estado": "Bahia",
			"rua": "Rua Asdrubal",
			"numero": 10
		},
	"tipo-pagamento": "Cartão de crédito",
	"detalhes-pagamento":
		{
			"número": "0011223344556677",
			"parcelas": 3
		}
}

~~~

Detalhamento da mensagem JSON:

**Pedido**

Atributo | Descrição
-------| --------
`id` | `Número de identificação do pedido` 
`cliente` | `Login do cliente na plataforma do Marketplace`
`fornecedor` | `Login do fornecedor na plataforma do Marketplace`
`pedidos` | `Mensagens JSON com identificação e quantidade dos produtos inclusos no pedido e valor da compra`
`timestamp` | `Data e hora em que a compra foi realizada`
`endereço` | `Mensagem JSON com o endereço para entrega do pedido`
`tipo-pagamento` | `Forma de pagamento utilizada no pedido`
`detalhes-pagamento` | `Mensagem JSON com detalhes da forma de pagamento`

**Produto incluso no pedido**

Atributo | Descrição
-------| --------
`código` | `Código de identificação do produto no Marketplace`
`quantidade` | `Quantidade de itens do produto inclusos no pedido`
`valor` | `Valor do pedido para esse produto`

**Endereço de entrega**

Atributo | Descrição
-------| --------
`cidade` | `Nome da cidade do local de entrega`
`estado` | `Nome do estado do local de entrega`
`rua` | `Nome da rua do local de entrega`
`número` | `Número do endereço de entrega`

**Detalhes do pagamento**

Atributo | Descrição
-------| --------
`número` | `Número do cartão de crédito`
`parcelas` | `Número de parcelas escolhidas para o pagamento`

### Interface `IOrdem`

> Interface usada para publicar mensagens com a criação de uma nova entrega.

**Tópico**: `confirma/entrega/{id_entrega}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/classes-json-entrega.png)

~~~json
{
	"id": 22,
	"cliente": "wilsoncosta",
	"fornecedor": "magazineluiza",
	"transportadora": "loggi",
	"nota-fiscal": "000001155",
	"endereço":
		{
			"cidade": "Salvador",
			"estado": "Bahia",
			"rua": "Rua Asdrubal",
			"numero": 10
		},
	"frete": 15.00,
	"prazo": "25-09-2020T00:00:00"
}
~~~

Detalhamento da mensagem JSON:

**Entrega**

Atributo | Descrição
-------| --------
`id` | `Número de identificação da entrega`
`cliente` | `Login do cliente na plataforma Marketplace`
`fornecedor` | `Login do fornecedor na plataforma Marketplace`
`transportadora` | `Nome comercial da empresa responsável pela entrega`
`nota-fiscal` | `Número da nota fiscal emitida para o pedido que será entregue`
`endereço` | `Mensagem JSON com o endereço para entrega`
`frete` | `Valor do frete cobrado para a entrega`
`prazo` | `Data estimada de conclusão da entrega`

**Endereço de entrega**

Atributo | Descrição
-------| --------
`cidade` | `Nome da cidade do local de entrega`
`estado` | `Nome do estado do local de entrega`
`rua` | `Nome da rua do local de entrega`
`número` | `Número do endereço de entrega`

### Interface `IStatus`

> Interface para mensagens de acompanhamento de status de pagamento e das entregas de um pedido.

**Tópico**: `acompanhamento/{id_pedido}`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/classes-json-status.png)

~~~json
{
	"id-pedido": 999,
	"status-entregas":
	{
		"status-entrega":
		{
			"id": 22,
			"status": "A caminho"
		},
		"status-entrega":
		{
			"id": 23,
			"status": "Preparando para entrega"
		}
	},
	"status-pagamento":
	{
		"id": 15,
		"status": "Aprovado"
	},
	"timestamp": "19-09-2020T08:30:00"
}
~~~

Detalhamento da mensagem JSON:

**Status**

Atributo | Descrição
-------| --------
`id-pedido` | `Número de identificação do pedido`
`status-entrega` | `Mensagens JSON com o status das entregas`
`status-pagamento` | `Mensagem JSON com o status do pagamento`
`timestamp` | `Data e hora da atualização`

**Status da Entrega**

Atributo | Descrição
-------| --------
`id` | `Número de identificação da entrega`
`status` | `Atualização sobre o status da entrega`

**Status**

Atributo | Descrição
-------| --------
`id` | `Número de identificação do pagamento`
`status` | `Atualização sobre o status do pagamento`


# Nível 2

Apresente aqui o detalhamento do Nível 2 conforme detalhado na especificação com, no mínimo, as seguintes subseções:

## Diagrama do Nível 2

Apresente um diagrama conforme o modelo a seguir:

> ![Modelo de diagrama no nível 2](images/diagrama-subcomponentes.png)

### Detalhamento da interação de componentes

O detalhamento deve seguir um formato de acordo com o exemplo a seguir:

* O componente `Entrega Pedido Compra` assina no barramento mensagens de tópico "`pedido/+/entrega`" através da interface `Solicita Entrega`.
  * Ao receber uma mensagem de tópico "`pedido/+/entrega`", dispara o início da entrega de um conjunto de produtos.
* Os componentes `Solicita Estoque` e `Solicita Compra` se comunicam com componentes externos pelo barramento:
  * Para consultar o estoque, o componente `Solicita Estoque` publica no barramento uma mensagem de tópico "`produto/<id>/estoque/consulta`" através da interface `Consulta Estoque` e assina mensagens de tópico "`produto/<id>/estoque/status`" através da interface `Posição Estoque` que retorna a disponibilidade do produto.

Para cada componente será apresentado um documento conforme o modelo a seguir:

## Componente `<Nome do Componente>`

> <Resumo do papel do componente e serviços que ele oferece.>

![Componente](images/diagrama-componente.png)

**Interfaces**
> * Listagem das interfaces do componente.

As interfaces listadas são detalhadas a seguir:

## Detalhamento das Interfaces

### Interface `<nome da interface>`

> ![Diagrama da Interface](images/diagrama-interface-itableproducer.png)

> <Resumo do papel da interface.>

Método | Objetivo
-------| --------
`<id do método>` | `<objetivo do método e descrição dos parâmetros>`

## Exemplos:

### Interface `ITableProducer`

![Diagrama da Interface](images/diagrama-interface-itableproducer.png)

Interface provida por qualquer fonte de dados que os forneça na forma de uma tabela.

Método | Objetivo
-------| --------
`requestAttributes` | Retorna um vetor com o nome de todos os atributos (colunas) da tabela.
`requestInstances` | Retorna uma matriz em que cada linha representa uma instância e cada coluna o valor do respectivo atributo (a ordem dos atributos é a mesma daquela fornecida por `requestAttributes`.

### Interface `IDataSetProperties`

![Diagrama da Interface](images/diagrama-interface-idatasetproperties.png)

Define o recurso (usualmente o caminho para um arquivo em disco) que é a fonte de dados.

Método | Objetivo
-------| --------
`getDataSource` | Retorna o caminho da fonte de dados.
`setDataSource` | Define o caminho da fonte de dados, informado através do parâmetro `dataSource`.

# Multiplas Interfaces

Em um sistema de comércio eletrônico do tipo shopping virtual, consideramos que existem duas jornadas de usuário de suma importância, do comprador e do lojista. Para cada uma dessas jornadas, existem inúmeras ações e operações envolvidas, distribuídas em setores como Pedido, Pagamento, Entrega e Produto. Como um sistema vivo, cada componente tem sua devida importancia, e caso algum deles apresente falhas, pode comprometer o sistema como um todo.

Do ponto de vista de escalabilidade, Marketplaces podem crescer significantemente em pouco tempo, ou também ter picos de acessos em datas específicas, como a Black Friday por exemplo. 

Um dos principais requisitos não funcionais de qualquer sistema online atualmente, é ter a capacidade de ser acessado tanto em dispositivos móveis como em desktops e notebooks.

Com estes pontos em evidência, torna-se imprescindivel optar por um ecossistema que suporte tais necessidades. Abaixo, detalharemos a escolha das linguagens e frameworks.

## Padrão de Projeto MVC

De acordo com a escolha do grupo pelo padrão arquitetural MVC, utilizaremos a linguagem Javascript em toda stack. 

Embora atualmente no ambíto online as linguagens campeãs de mercado ainda sejam .NET, PHP e Java, optamos por utilizar Javascript (Versão ECMAScript 2018) para facilitar no entendimento de código por qualquer profissional envolvido, entendemos que todos os responsáveis pela manutenção do código tenham o perfil *Full Stack*, ou seja, capaz de prover soluções nas três camadas de projeto.

### Model e Controller 
Para as camadas Model e Controller, ou *backend*, será utilizado o runtime de Javascript **Node.JS** e seu framework **Express.js**.

Além da linguagem utilizada e não exigir muitos recursos computacionais, Node.JS possui arquitetura assíncrona baseada em eventos, e apesar de *single thread*, é capaz de executar múltiplas conexões em concorrência. Possui também o repositório mais utilizado de bibliotecas web, o NPM (*Node Package Manager*).

### View (Web)
Para a camada View em ambientes web, foi escolhida a biblioteca **React** e o framework **Gatsby**.

Mantido pelo Facebook e uma comunidade de desenvolvedores individuais. React tem se provado ser extramamente flexível e agil, sendo utilizado nos sites da Netflix, Airbnb, SeatGeek, HelloSign, Walmart e outros.

Também são utilizados HTML 5 e CSS 3.

### View (App mobile Android e iOS)
Após o sucesso do React, o Facebook anunciou em 2015 o lançamento do **React Native**, sendo também uma biblioteca Javascript. Segue a filosofia *Learn once, write anywhere*.

Utiliza componentes de interface de usuário nativos tanto de Android como iOS. Oferece grandes vantagens em relação a web views em termos de performance pois não necessita de plugins para se comunicar com as APIs dos sistemas operacionais mobile.

Mesmo em apenas 5 anos da data de lançamento e atualmente na versão 0.63, o React Native já tem arsenal completo para desenvolver qualquer tipo de aplicação, não deixando nada a desejar em relação a Java (Swift) ou Objective-C

### Banco de Dados

Na disciplina anterior, INF 325, concluímos que nosso Marketplace seguirá a tendência de persistência poliglota de bancos de dados. Identificamos que a utilização de tecnologias de bancos de dados específicas para cada finalidade do sistema servirá de diferencial competitivo ao explorar o melhor de cada uma. O arcabouço de dados será composto por:

- Sistema Marketplace: MySQL

- Monitoramento e Logs: MongoDB ou Elasticsearch

- Recomendação de produtos, priorização de fornecedores: Neo4J

- Armazenamento de sessão: Redis ou Cassandra

## Comunicação Entre Camadas
O principal desafio do ponto de vista arquitetural é projetar a aplicação para utilizar os mesmos componentes independente da interface de usuário web ou mobile.

Para atingir este objetivo, em ambas views React e React Native, a comunicação será feita utilizando a mesma API desenvolvida em Node.JS, ou seja, os mesmos dados serão recuperados tanto via web como mobile. Apenas algumas diferenças do ponto de vista de detalhamento de interface serão consideradas e tratadas de acordo com cada interface.

Por exemplo, ao acessar a página inicial são exibidos os produtos em destaque do marketplace através do resultado da endpoint `/produto/exibe/destaques/{quantidade:opcional}`

Para a View Mobile, será passado o parâmetro `quantidade` para limitar a quantidade de resultados a serem retornados no formato *json*

## Diagrama da Arquitetura

[![Diagrama da Arquitetura](images/diagrama_plataformas_mvc.png)](images/diagrama_plataformas_mvc.png)
