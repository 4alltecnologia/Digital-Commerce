---
title: 4 Pay - Digital Commerce

language_tabs:
  - HTML
  - javascript
  - shell

toc_footers:
  - <a href='http://4all.com'>4ll.com</a>

includes:
  - errors

search: true
---

**4 Pay - Digital Commerce**
----------------------------
    Manual de Uso - Web/Javascript - Versão 0.1

# 1 Introdução

Com o 4 Pay - Digital Commerce, você pode configurar seu website para aceitar pagamentos de cartões de crédito e débito de maneira fácil e rápida.

Para aceitar pagamentos de Cartão em seu site, você precisa executar os passos:
1. Obter chaves de API no Portal do EC;
2. Incluir o a Janela de Checkout em seu site;
3. Capturar a transação no seu servidor.


#2 Chaves de API

Para obter as Chaves de API, acesse o Portal do EC (https://portal.4all.com) e faça seu login. No menu lateral, clique na opção “Chaves de API” no sub-menu “Digital Commerce”.

Nesta página você pode obter pares de chaves no ambiente de Homologação ou no Ambiente de Produção.

Clique no botão “Gerar Chaves” para obter um par de chaves, a Pública (**PublicApiKey**) e a Privada (**MerchantKey**).

# 3 Incluindo a Janela de Checkout em seu Site

A Janela de Checkout é apresentada quando o cliente confirma a compra (no *submit* do formulário de fechar pedido, por exemplo) ou seleciona a forma de pagamento "Cartão de Crédito ou Débito via 4all" (quando o seu site permite mais de uma forma de pagamento). 

O *dialog* permite que o usuário informe os dados do cartão ou se autentique com sua Conta 4all para buscar seus dados de pagamento já cadastrado. 

Você pode inserir o Dialog no seu site de duas maneiras: via **Embedded Form**, ou via a nossa **Biblioteca Javascript**.

##3.1 Embedded Form

> Inclua o código do checkout na sua página, como no exemplo:

```HTML
<form action="pedido_concluido.php" method="POST">

  <!-- … demais inputs do formulário … -->
  
  <script
    src="https://lib.4all.com/lib/checkout-express.js"
    data-public-api-key="pk_test_6pRNASCoBOKtIshFeQd4XMUh"
    data-amount="5999"
    >
  </script>
</form>
```

**Como funciona:** Nesta modalidade de checkout, você adiciona à sua página um formulário que efetua, de forma segura, todo o processo de captura e validação dos dados de pagamento do cliente, assim, as informações sensíveis do seu cliente nunca entram em contato com ambientes não-seguros. Após a captura e validação dos dados, é adicionado ao formulário um token de pagamento (válido para apenas **uma** transação) que deverá ser utilizado para efetuar a confirmação na API de pagamentos 4all.

No evento de *submit* do seu formulário, a Janela de Checkout será apresentada para o cliente inserir os dados de pagamento.

Substitua o valor de cada atributo de acordo com o pagamento a ser efetuado. Os atributos aceitos são:

Atributo    |Descrição  |Formato    |Obrigatório 
------------|-----------|-----------|--------------
`data-public-api-key`| Chave de API pública do Checkout all.|String|Sim
`data-amount`|Valor da transação em centavos. Ex: "1425" para R$ 14,25.|String|Sim

Se o cliente efetuar o pagamento, um novo campo `<input type="hidden" id="payment_token">` contendo o **payment_token** é adicionado ao seu formulário, e o submit é efetuado.

## 3.2 Biblioteca Javascript

```HTML
<script src="https://lib.4all.com/lib/checkout-lib.js"></script>
```

Para fazer a integração via javascript, você deve primeiramente importar o arquivo da biblioteca na sua página, adicionando a seguinte linha ao seu HTML:

```javascript
	function onSuccess(paymentToken) {
	  //esta função será chamada ao completar o checkout
	}

	function onCancel() {
	  // esta funcão será chamada caso o cliente cancele o checkout
	}

	Var options = {
	  amount: 2500,
	  publicApiKey: "pk_test_6pRNASCoBOKtIshFeQd4XMUh",
	  successCallback: onSuccess,
	  cancelCallback: onCancel
	};

Checkout4all.startCheckout(options);
```

Após isso, a biblioteca está disponível em seu escopo global como 'Checkout4all'. Para iniciar o processo de login/pagamento, basta chamar a função **startCheckout** da biblioteca:

Os parâmetros dessa função são:

Atributo    |Descrição  |Formato    |Obrigatório 
------------|-----------|-----------|--------------
`amount`|Valor da transação em centavos. Ex: "1425" para R$ 14,25.|String|Sim
`publicApiKey`|Chave de API pública do Checkout 4all.|String|Sim
`successCallback` | Função que será chamada quando o checkout estiver finalizado com sucesso. Recebe o paymentToken como parâmetro. | Função | Sim
`cancelCallback` | Função que será chamada caso o usuário cancele o processo de pagamento sem concluí-lo. | Função | Não


# 4 Capturando a transação

Com o **payment_token** em mãos, você pode capturar a transação (efetuar a cobrança) através de uma chamada à API Conta 4all.

**Nota:** por motivos de segurança, você deve informar o valor total da transação novamente nesta chamada.

> Exemplo:
```shell
curl -X POST "https://api.pagar.me/1/transactions/{TOKEN}/capture"
  -d 'amount=1000'
  -d 'api_key=ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0'
```

## 4.1 Consultando uma transação
Em caso de erro na chamada de captura de transação, você pode consultar o estado da transação através do Meta ID passado na chamada de Captura.

## 4.2 Issue Authorized Transaction
**Caminho**: <endpoint>/merchant/issueAuthorizedTransaction
**Descrição**: Emite uma transação previamente autorizada

> Exemplo de requisição:

```javascript
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMN...",
	"transactionAuthorizationToken": "MDEyM...",
	"amount": 5000,
	"merchantMetaId": "1259",
	"waitForTransaction": true,
	"postbackURL": "www.example.com/postback/"
}
```

**Requisição**:

Atributo        |Descrição  |Formato    |Tamanho|Obrigatório
----------------|-----------|-----------|-------|-------------
`merchantKey`|Chave de acesso do estabelecimento à API|String|44|Sim
`transactionAuthorizationToken`|Token da autorização prévia|String|44|Sim
`amount`|Valor da transação, em centavos.|Number|*|Sim
`merchantMetaId`|Identificador único, atribuído pelo estabelecimento comercial, para poder pesquisar esta transação em caso de nãorecebimento da resposta desta chamada (contendo o transactionId). Deve ser um valor numérico inteiro (representado como string).|String|20|Não
`waitForTransaction`|Quando presente e com valor **true** , a chamada não retorna enquanto a transação estiver com status pendente de pagamento (estados 0, 1 ou 2 vide **Anexo B Tabela B.2**). O chamador deve estar preparado para lidar com tempos de resposta longos, de até 30 segundos. Existe um timeout de 30 segundos quando a chamada retorna independente do estado da transação.|Boolean||Não
`postbackURL`|Endereço a ser chamado quando o estado da transação mudar (vide seção 1.2 deste documento).|URL|250|Não

**Observações**:

 - O parâmetro "transactionAuthorizationToken" deve ser obtido através da biclioteca de pagamentos
 - O sistema deve registrar, com base na sessão do usuário que está fazendo o pagamento, a partir de
qual aplicativo o pagamento foi executado.

> Exemplo de resposta:

```javascript
{
	"transactionId": "2181486",
	"status": 3,
	"datetime": "2016-08-31T20:12:31Z"
}
```

**Resposta**:

Atributo        |Descrição  |Formato    |Tamanho|Presente
----------------|-----------|-----------|-------|-------------
`transactionId`|Identificador da transação.|String|20|Sempre
`status`|Estado da transação (conforme Anexo B Tabela B.2).|Number|*|Sempre
`datetime`|Data e hora UTC em que a transação foi processada pelo servidor 4all. Formato YYYYMMDDThh:mm:ssZ.|String|20|Sempre

## 4.3 Get Transaction Details

**Caminho**: `<endpoint>/merchant/getTransactionDetails`
**Descrição**:  Retorna os detalhes de uma transação.

> Exemplo de requisição:

```javascript
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMNT...",
	"transactionId": "2181486"
}
```

**Requisição**:

Atributo        |Descrição  |Formato    |Tamanho|Obrigatório
----------------|-----------|-----------|-------|-------------
`merchantKey`|Chave de acesso do merchant à API|String|44|Sim
`transactionId`|Identificador da transação|String|20|Depende
`merchantMetaId`|Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa.|String|20|Depende

> Exemplo de resposta:

```javascript
{
	"transactionId": "854383518",
	"amount": 5000,
	"status": 2,
	"createdAt": "2016-01-31T20:12:31Z",
	"paidAt": "2016-01-31T20:33:12Z",
	"authorizationInfo": {
	"acquirerId": 1,
	"acquirerUsn": "18318318",
	"acquirerTimestamp": "20161231125959",
	"brandId": 1
	}
}
```

**Resposta:**

Atributo        |Descrição  |Formato    |Tamanho|Presente
----------------|-----------|-----------|-------|-----------
`transactionId`|Identificador da transação|String|20|Sempre
`subscriptionId`|Quando presente, informa o identificador da assinatura que gerou esta transação.|String|20|Depende
`amount`|Valor da transação, em centavos.|Number||Sempre
`status`|Estado da transação (conforme Anexo B Tabela B.2).|Number||Sempre
`createdAt`|Data e hora UTC em que a transação foi criada no servidor 4all (formato YYYYMMDDThh:mm:ssZ).|String|20|Sempre
`paidAt`|Data e hora UTC em que a transação foi paga por uma conta 4all (formato YYYYMMDDThh:mm:ssZ). Presente somente se a transação já foi paga.|String|20|Depende
`authorizationInfo`|Objeto contendo detalhes de autorização da transação. Presente somente se a transação já foi paga.|Object|*|Depende
`transactionToken`|Chave para consulta da transação pela chamada "Wait For Transaction".|String|44|Sempre

**authorizationInfo**:

Atributo        |Descrição  |Formato    |Tamanho|Presente
----------------|-----------|-----------|-------|-----------
`acquirerId`|Identificador do adquirente.|Number||Sempre
`acquirerUsn`|Identificador/NSU do adquirente para a transação.|String|50|Sempre
`acquirerTimestamp`|Timestamp de pagamento conforme reportado pelo adquirente. O formato e timezone deste campo segue o formato usado pelo adquirente. Presente somente se o adquirente informar.|String|50|Depende
`brandId`|Identificador da bandeira do cartão que foi usado no pagamento.|Number||sempre

**Observações**

 - Apenas um dos dois parâmetros, ***transactionId*** ou ***merchantMetaId***, pode ser usado; se ambos
estiverem presentes, esta chamada falha com um erro específico.

- Adquirentes:
1. Stone
2. Pagar.me

- Bandeiras:
1. Visa
2. Mastercard

# 5 Ambiente de Homologação

No Ambiente de Homologação, as transações efetuadas não geram cobranças, de modo que você pode testar a integração de seu Site com o Digital Commerce 4all.

No Portal do EC, você pode gerar chaves para o ambiente de Homologação. Quando você usa uma chave de Homologação, aparecerá uma mensagem na Janela de Checkout indicando que você está no ambiente de Homologação e as transações efetuadas não resultarão em cobranças.

##5.1 Cartões de Teste
No ambiente de Homologação, você pode utilizar cartões de qualquer número de 16 dígitos e qualquer CVV de 3 dígitos para testar compras em seu Site. 

**Cartões de final PAR sempre resultam em compras efetivadas com sucesso.**
**Cartões de final ÍMPAR sempre resultam em transações negadas.**