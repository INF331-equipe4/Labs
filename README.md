# Projeto `<Título>`

# Equipe
* `Caio Vitor`
* `Dennis Phillips`
* `Gustavo Nakahara`
* `Manoel Teixeira`
* `Wilson Costa`

# Nível 1

## Diagrama Geral do Nível 1

![Diagrama no nível 1](images/coreografia.png)

### Detalhamento da interação de componentes

O detalhamento deve seguir um formato de acordo com o exemplo a seguir:

* O `componente X` inicia o leilão publicando no barramento a mensagem de tópico "`leilão/<número>/início`" através da interface `Gerente Leilão`, iniciando um leilão.
* O `componente Y` assina no barramento mensagens de tópico "`leilão/+/início`" através da interface `Participa Leilão`. Quando recebe uma mensagem…

* O `componente Usuário` dispõe de uma interface `ISeleçãoProduto` que permite que o usuário selecione um produto de uma lista obtida através da interface `IListaProdutos`.
* O `componente Marketplace` recebe do `componente Usuário` o produto selecionado através da interface `IProduto`.
* O `componente Marketplace` aciona o `componente Leilão` através da interface 'ISolicitaLeilão'.
* O `componente Leilão` publica no barramento no tópico “`leilão/demanda/<produto>`” informando que foi iniciado um leilão para obtenção dos preços do produto selecionado.
* Os componentes do tipo Fornecedor que assinam o tópico “`leilão/demanda/#`" publicam o preço do produto no barramento no tópico “leilão/lance/<produto>/<preço>”
* O componente Leilão assina o tópico “`leilão/lance/#`”, reúne os melhores lances, e passa as informações para o `componente Marketplace` através da interface `IResultadoLeilão`.
* O resultado das consultas feitas pelo `componente Marketplace`, a respeito do produto selecionado, é devolvido para o `componente Usuário` através da interface `IDetalhesProduto`.
* Com as informações obtidas, o cliente realiza seu pedido através da interface `IConfirmaPedido` do `componente Usuário`.
* O pedido é passado para o `componente Marketplace` através da interface `IPedido`. O `componente Marketplace` passa o pedido através da interface `IPedido` para o `componente ControlePedido`.
* O `componente ControlePedido` publica no Barramento B uma mensagem  com o tópico “`confirma/pagamento/<pedido>`” contendo os detalhes do pedido confirmados pelo cliente para o pagamento.
* O `componente Pagamento`, que assina o tópico “`confirma/pagamento/#`”, recebe a mensagem e guarda no `componente TransaçõesPagamento` um histórico da transação financeira, usando a interface `IPagamento`.
* O `componente Pagamento` publica no Barramento B uma mensagem contendo o status do pagamento com o tópico “`status/pagamento/<pedido>/<status>`”.
* O `componente ControlePedido` assina o tópico “`status/#`” e recebe a mensagem publicada pelo `componente Pagamento`. Caso o pagamento seja confirmado, ele publica uma mensagem contendo o pedido com o tópico “`confirma/logistica/<pedido>`”.
* O `componente Logística` assina o tópico “`confirma/logistica/#`” e recebe o pedido para a entrega e guarda no `componente TransaçõesLogística` um histórico da transação financeira, usando a interface `ILogistica`.
* O `componente Logística` publica a mensagem com o tópico “`status/logistica/<pedido>/<status>`” que é recebido pelo `componente ControlePedido` que assina o tópico “`status/#`”.
* O `Componente Usuário` obtém do `componente ControlePedido` a informacão do status do pedido através da interface `IStatusPedido`. É disponibilizada ao cliente a interface `ISolicitaStatus`.
* O componente `Gestão` obtém do `componente ControlePedido` as informações referentes aos pedidos através da interface `IPedidos`. Este componente obtém também do componente Logística informações referentes a entrega através da interface 'IStatusLogistica'.
* O componente `Gestão` disponibiliza duas interfaces aos gestores do sistema, a `ISolicitaLogistica` e `ISolicitaPedidos`.


Para cada componente será apresentado um documento conforme o modelo a seguir:

## Componente Usuário

* Componente para funcionar como View dos clientes. 
* Ele pode ser substituído de acordo com a interface a ser utilizada pelo usuário. 
* Disponibiliza interfaces para que o cliente selecione um produto, confirme o pedido e acompanhe o status do pedido.

![Componente](diagrama-componente-mensagens.png)

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

![Componente](diagrama-componente-mensagens.png)

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

![Componente](diagrama-componente-mensagens.png)

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


![Componente](diagrama-componente-mensagens.png)

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

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* ISolicitaLeilao
* IResultadoLeilao
* ILance
* IDemanda

## Componente Fornecedor

* Componente dos fornecedores que fazem parte do Marketplace.
* Ele possui interfaces para que os fornecedore participem do leilão invertido e façam atualizações da informações de estoque.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* IConfirmaEstoque
* IDemanda
* ILance

## Componente Logística

* Componente responsável pela entrega dos pedidos e registro destas transações.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* IStatus
* IConfirmaPedido
* ILogistica
* IStatusLogistica

## Componente Pagamento

* Componente responsável pelo pagamento dos pedidos.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* IConfirmaPedido
* IStatus
* IPagamento

## Componente Cadastro

* Componente que funciona como Model para o cadastro de usuários.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* IObtemCadastro
* IInsereCadastro

## Componente Estoque

* Componente que funciona como Model para o estoque de produtos do MarketPlace.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* IObtemEstoque
* IInsereEstoque

## Componente Produto

* Componente que funciona como Model para os detalhes dos produtos do MarketPlace.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* IObtemProduto
* IInsereProduto

## Componente Pedido

* Componente que funciona como Model para os pedidos do MarketPlace.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* IObtemPedido
* IInserePedido

## Componente TransaçõesPagamento

* Componente que funciona como Model para as transações de pagamentos.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* IPagamento

## Componente TransaçõesLogística

* Componente que funciona como Model para as transações de logística.

![Componente](diagrama-componente-mensagens.png)

**Interfaces**
* ILogistica



## Detalhamento das Interfaces

### Interface `<nome da interface>`

> Resumo do papel da interface.

**Tópico**: `<tópico que a respectiva interface assina ou publica>`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
<Formato da mensagem JSON associada ao objeto enviado/recebido por essa interface.>
~~~

Detalhamento da mensagem JSON:

Atributo | Descrição
-------| --------
`<nome do atributo>` | `<objetivo do atributo>`

## Exemplo

### Interface DadosPedido

Interface para envio de dados do pedido com itens associados.

**Tópico**: `pedido/{id}/dados`

Classes que representam objetos JSON associados às mensagens da interface:

![Diagrama Classes REST](images/diagrama-classes-rest.png)

~~~json
{
  "number": 16,
  "duoDate": "2009-10-04",
  "total": 1937.01,
  "items": {
    "item": {
       "itemid": "1245",
       "quantity": 1
    },
    "item": {
       "itemid": "1321",
       "quantity": 1
    }
  }  
}
~~~

Detalhamento da mensagem JSON:

**Order**
Atributo | Descrição
-------| --------
number | número do pedido
duoDate | data de vencimento
total | valor total do pedido
items | itens do pedido

**Item**
Atributo | Descrição
-------| --------
itemid | identificador do item
quantity | quantidade do item

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

> Escreva um texto detalhando como seus componentes  podem ser preparados para que seja possível trocar de interface apenas trocando o componente View e mantendo o Model e Controller.
>
> É recomendado a inserção de, pelo menos, um diagrama que deve ser descrito no texto. O formato do diagrama é livre e deve ilustrar a arquitetura proposta.