---
title: 4 Pay - Digital Commerce Web

language_tabs:

toc_footers:
  - <a href='http://4all.com'>4all.com</a>
  - <a href='https://4alltecnologia.github.io/Digital-Commerce/'>Home</a>

includes:

search: true
---

# Digital Commerce Web - 1.0

# 1 Introdução

Com o 4 Pay - Digital Commerce, você pode configurar seu website para aceitar pagamentos de cartões de crédito e débito de maneira fácil e rápida.

Para aceitar pagamentos de Cartão em seu site, você precisa executar os passos:

1. Obter chaves de API no Portal do EC;
2. Incluir o a Janela de Checkout em seu site;
3. Capturar a transação no seu servidor.

## 1.1 Endereços dos servidores de Homologação e de Produção

Os seguintes endpoints devem ser usados para executar as chamadas:

| Ambiente  | URL
|-----------|-------------------
|**Homologação**|conta.homolog.4all.com|
|**Produção**|conta.api.4all.com|
|||

# 2 Chaves de API

Para obter as Chaves de API, acesse o Portal do EC (https://portal.4all.com) e faça seu login. No menu lateral, clique na opção “Chaves de API” no sub-menu “Digital Commerce”.

Nesta página você pode obter pares de chaves no ambiente de Homologação ou no Ambiente de Produção.

Clique no botão “Gerar Chaves” para obter um par de chaves, a Pública (**PublicApiKey**) e a Privada (**MerchantKey**).

# 3 Incluindo a Janela de Checkout em seu Site

A Janela de Checkout é apresentada quando o cliente confirma a compra (no *submit* do formulário de fechar pedido, por exemplo) ou seleciona a forma de pagamento "Cartão de Crédito ou Débito via 4all" (quando o seu site permite mais de uma forma de pagamento).

O *dialog* permite que o usuário informe os dados do cartão ou se autentique com sua Conta 4all para buscar seus dados de pagamento já cadastrados.

Você pode inserir o Dialog no seu site de duas maneiras: via **Embedded Form**, ou via a nossa **Biblioteca Javascript**.

##3.1 Embedded Form

```html
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

|Atributo    |Descrição  |Formato    |Obrigatório
|------------|-----------|-----------|--------------
|**data-public-api-key**| Chave de API pública do Checkout all.|String|Sim
|**data-amount**|Valor da transação em centavos. Ex: "1425" para R$ 14,25.|String|Sim

Se o cliente efetuar o pagamento, um novo campo `<input type="hidden" id="payment_token">` contendo o **payment_token** é adicionado ao seu formulário, e o *submit* é efetuado.

## 3.2 Biblioteca Javascript

```html
<script src="https://lib.4all.com/lib/checkout-lib.js"></script>
```

Para fazer a integração via javascript, você deve primeiramente importar o arquivo da biblioteca na sua página, adicionando ao seu HTML a linha abaixo ou ao lado.

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

Após isso, a biblioteca está disponível em seu escopo global como 'Checkout4all'. Para iniciar o processo de login/pagamento, basta chamar a função **startCheckout** da biblioteca, como demonstrado abaixo ou ao lado (código javascript).

Os parâmetros dessa função são:

|Atributo    |Descrição  |Formato    |Obrigatório
|------------|-----------|-----------|--------------
|**amount**|Valor da transação em centavos. Ex: "1425" para R$ 14,25.|String|Sim
|**publicApiKey**|Chave de API pública do Checkout 4all.|String|Sim
|**successCallback** | Função que será chamada quando o checkout estiver finalizado com sucesso. Recebe o paymentToken como parâmetro. | Função | Sim
|**cancelCallback**| Função que será chamada caso o usuário cancele o processo de pagamento sem concluí-lo. | Função | Não


# 4 Capturando a transação

Com o **payment_token** em mãos, você pode capturar a transação (efetuar a cobrança) através de uma chamada à API Conta 4all.

<aside class="notice">
Nota:  por motivos de segurança, você deve informar o valor total da transação novamente nesta chamada.
</aside>
<aside class="notice">
Em caso de erro de comunicação ao executar esta chamada, você deve consultar o estado da transação utilizando a chamada descrita na seção 4.1 para verificar se a transação foi capturada com sucesso ou não.
</aside>

```shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...","paymentToken":"MDEyM...", "amount": 5000}'
https://conta.api.4all.com/merchant/issueAuthorizedTransaction
```

**Caminho**: `<endpoint>/merchant/issueAuthorizedTransaction`

**Descrição**: Captura uma transação pelo **paymentToken**

```json
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMN...",
	"paymentToken": "MDEyM...",
	"amount": 5000,
	"merchantMetaId": "1259",
	"returnImmediatly": false
}
```

**Requisição**:

|Atributo        |Descrição  |Formato    |Tamanho|Obrigatório
|----------------|-----------|-----------|-------|-------------
|**merchantKey**|Chave de acesso do estabelecimento à API|String|44|Sim
|**paymentToken**|Token da autorização de pagamento, obtido do Checkout.|String|44|Sim
|**amount**|Valor da transação, em centavos.|Number|*|Sim
|**merchantMetaId**|Identificador único, atribuído pelo estabelecimento comercial, para poder pesquisar esta transação em caso de não recebimento da resposta desta chamada (contendo o transactionId). Deve ser um valor numérico inteiro (representado como string).|String|20|Não
|**returnImmediatly**|Quando presente e com valor **true** , a chamada retorna imediatamente. Neste caso, a transação estará com um status pendente (estados 0, 1 ou 2 vide **seção 6 desta documentação**). |Boolean||Não


**Observações**:

 - O parâmetro "transactionAuthorizationToken" deve ser obtido através da Biblioteca Javascript ou Embedded Form.
 - O chamador deve estar preparado para lidar com tempos de resposta longos, de até 30 segundos (a menos que o parâmetro `returnImmediatly` seja passado como `true`, neste caso, a chamada retorna imediatamente).
 - Caso o parâmetro `returnImmediatly` seja passado como `true`, a chamada retorna imediatamente, porém o resultado da transação deve ser consultada posteriormente utilizando a chamada descrita na seção 5.1.

```json
{
	"transactionId": "2181486",
	"status": 3,
	"datetime": "2016-08-31T20:12:31Z"
}
```

**Resposta**:

|Atributo        |Descrição  |Formato    |Tamanho|Presente
|----------------|-----------|-----------|-------|-------------
|**transactionId**|Identificador da transação.|String|20|Sempre
|**status**|Estado da transação (ver **seção 6 desta documentação**).|Number|*|Sempre
|**datetime**|Data e hora UTC em que a transação foi processada pelo servidor 4all. Formato YYYYMMDDThh:mm:ssZ (Formato ISO 8601 https://en.wikipedia.org/wiki/ISO_8601).|String|20|Sempre

**Tratamento de Erro**
Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

```json
{
	"error": {
		"code": 2318,
		"message": "Payment Token inválido."
	}
}
```


## 4.1 Consultando uma transação

Em caso de erro na chamada de captura de transação, você pode consultar o estado da transação através do Meta ID passado na chamada de Captura.

```shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...","transactionId":"73423624"}'
https://conta.api.4all.com/merchant/getTransactionDetails
```

**Caminho**: `<endpoint>/merchant/getTransactionDetails`

**Descrição**:  Retorna os detalhes de uma transação.


```json
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMN...",
	"merchantMetaId": "1259"
}
```

**Requisição**:

|Atributo        |Descrição  |Formato    |Tamanho|Obrigatório
|----------------|-----------|-----------|-------|-------------
|**merchantKey**|Chave de acesso do merchant à API|String|44|Sim
|**transactionId**|Identificador da transação|String|20|Depende
|**merchantMetaId**|Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa.|String|20|Depende


```json
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

|Atributo        |Descrição  |Formato    |Tamanho|Presente
|----------------|-----------|-----------|-------|-----------
|**transactionId**|Identificador da transação|String|20|Sempre
|**subscriptionId**|Quando presente, informa o identificador da assinatura que gerou esta transação.|String|20|Depende
|**amount**|Valor da transação, em centavos.|Number||Sempre
|**status**|Estado da transação (ver **seção 6 desta documentação**).|Number||Sempre
|**createdAt**|Data e hora UTC em que a transação foi criada no servidor 4all (formato YYYYMMDDThh:mm:ssZ).|String|20|Sempre
|**paidAt**|Data e hora UTC em que a transação foi paga por uma conta 4all (formato YYYYMMDDThh:mm:ssZ). Presente somente se a transação já foi paga.|String|20|Depende
|**authorizationInfo**|Objeto contendo detalhes de autorização da transação. Presente somente se a transação já foi paga.|Object|*|Depende

***authorizationInfo:***

|Atributo        |Descrição  |Formato    |Tamanho|Presente
|----------------|-----------|-----------|-------|-----------
|**acquirerId**|Identificador do adquirente.|Number||Sempre
|**acquirerUsn**|Identificador/NSU do adquirente para a transação.|String|50|Sempre
|**acquirerTimestamp**|Timestamp de pagamento conforme reportado pelo adquirente. O formato e timezone deste campo segue o formato usado pelo adquirente. Presente somente se o adquirente informar.|String|50|Depende
|**brandId**|Identificador da bandeira do cartão que foi usado no pagamento.|Number||sempre

**Observações**

 Apenas um dos dois parâmetros, ***transactionId*** ou ***merchantMetaId***, pode ser usado; se ambos
estiverem presentes, esta chamada falha com um erro específico.

* Adquirentes:
	1. Stone
	2. Pagar.me

* Bandeiras:
	1. Visa
	2. Mastercard

**Tratamento de Erro**
Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

```json
{
	"error": {
		"code": 2318,
		"message": "Payment Token inválido."
	}
}
```
## 4.2. Cancelamento uma transação

Também é possível efetuar o cancelamento de uma transação previamente aprovada

```shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...","transactionId":"73423624"}'
https://conta.api.4all.com/merchant/refundTransaction
```

**Caminho**: `<endpoint>/merchant/refundTransaction`

**Descrição**: Funcionalidade para estornar/cancelar uma transação.

Em alguns casos, o cancelamento da transação pode não ser mais possível pois o prazo limite para realizar a operação já se esgotou.

```json
{
  "merchantKey": "MDEyMzQ1Njc4OTAxMjMNT...",
  "transactionId": "2181486"
}
```

**REQUISIÇÃO:**

|Campo |Descrição |Formato |Tam. |Obrigatório
|------|----------|--------|-----|-------
|**merchantKey**|Chave de acesso do merchant à API.|String|44|sim|
|**transactionId**|Identificador da transação.|String|20|sim|
||||||

**RESPOSTA:** HTTP 204

#6. Estados de uma Transação

|Estado | Descrição
|-------|-----------
|0 | Transação aguardando pagamento.
|1 | Transação em processamento.
|2 | Transação não aprovada.
|3 | Transação aprovada.
|4 | Transação em cancelamento.
|5 | Transação cancelada.
|6 | Transação aprovada (cancelamento falhou).
|7 | Transação contestada.
|8 | Transação reapresentada.
|9 | Transação aguardando confirmação.

# Homologação

<aside class="notice">
Para realizar transações no ambiente de Homologação, utilize um par de chaves de Homologação, obtidas no Portal do EC.
</aside>

No Ambiente de Homologação, as transações efetuadas não geram cobranças, de modo que você pode testar a integração de seu Site com o Digital Commerce 4all.

No Portal do EC, você pode gerar chaves para o ambiente de Homologação. Quando você usa uma chave de Homologação, aparecerá uma mensagem na Janela de Checkout indicando que você está no ambiente de Homologação e as transações efetuadas não resultarão em cobranças.

## Conta 4all no Ambiente de Homologação

Nas contas criadas no ambiente de Homologação, o envio de SMS não é disparado, e os desafios de SMS podem ser respondidos com "444444".

## Cartões de Teste
No ambiente de Homologação, você pode utilizar cartões de gerados e qualquer CVV de 3 dígitos para testar compras em seu Site.

**Cartões de final PAR sempre resultam em compras efetivadas com sucesso.**
**Cartões de final ÍMPAR sempre resultam em transações negadas.**

Você pode utilizar o seguinte website para gerar cartões de teste: http://credit-card-generator.2-ee.com/

<aside class="notice">
Nota:  a 4all não permite que o mesmo cartão seja utilizado em duas contas diferentes. Este comportamento também é aplicado no ambiente de homologação.
</aside>
