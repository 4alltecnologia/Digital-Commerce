---
title: Digital Commerce - Android - 1.0

language_tabs:

toc_footers:
  - <a href='http://4all.com'>4all.com</a>
  - <a href='https://4alltecnologia.github.io/Digital-Commerce/'>Home</a>

includes:
  - _homologacao.md

search: true
---

# Digital Commerce - Android - 1.0

# 1 Introdução

Com o módulo de pagamento para Android, você pode configurar seu aplicativo para receber pagamentos de cartões de crédito de maneira fácil e rápida.

Para aceitar pagamentos no seu App, você deve seguir os seguintes passos:

1. Obter Chaves de API no Portal do EC;
2. Importar o framework de Mobile Payment 4all;
3. Quando for efetuar um pagamento, obter um paymentToken, usando a chamada Pay4all.getToken().
4. Capturar a transação no seu servidor.


#2 Chaves de API

Para obter as Chaves de API, acesse o Portal do EC (https://portal.4all.com) e faça seu login. 

No menu lateral, clique na opção “Chaves de API” no sub-menu “Mobile Payment”.

Nesta página você pode obter pares de chaves no ambiente de Homologação ou no Ambiente de Produção.

Clique no botão “Gerar Chaves” para obter um par de chaves, a Pública (**PublicApiKey**) e a Privada (**MerchantKey**).


# 3 Importando a Biblioteca de Mobile Payment

A inclusão da biblioteca no projeto é através da inclusão de sua dependência no projeto via gradle. Desta maneira, inclui-se arquivo build.gradle do projeto inclui-se os seguintes respositórios ao projeto:

 build.gradle(Projeto:`<nome do projeto>`) : 

```Gradle
allprojects {
  repositories {
    jcenter()

    maven {
      url 's3://utilitarios/android/libs'
   }
}
```

No arquivo build.gradle referente ao módulo da aplicação, basta adicionar a linha abaixo:

build.gradle(Module:app) :

```Gradle
compile 'com.4all.libs:digital_commerce:1.0.0'
```


# 4 Fazendo a chamada Pay4all.getToken()

Para obter o **payment_token**, crie uma instância o objeto Pay4all e faça a chamada getToken no exemplo abaixo:

```Java
FourAll_DigitalCommerce pay4allObject =
FourAll_DigitalCommerce.newInstance(MainActivity.this, <<APIKEY>>);
 
pay4allObject.getToken();
```

Quando a função `Pay4all.getToken()` é chamada pela primeira vez, ele é direcionado para uma tela onde se autentica e cadastra um cartão de crédito a ser usado em todas as compras no seu aplicativo.  Devido a natureza assíncrona da chamada,  a Activity a exibir DialogFragment para a inserção de dados do usuário deve implementar a interface desta:   OneTouch_WebPayFragment.OnOneTouchPaymentListener. A interface possui dois métodos: onSucess e onFail. Em caso de sucesso, o método onSucess será chamado, recebendo como parâmetro a chave do usuário, que também será salva na persistência do aplicativo para futuras utilizações. Já a chamada onFail, utilizada em caso de falhas na operação ou cancelamento pelo usuário não possui nenhum tipo de parâmetro.  

Nas chamadas subsequentes, o usuário não precisará efetuar a autenticação novamente, e o **payment_token** é retornado de maneira transparente.

Se o cartão de crédito cadastrado é excluido pelo usuário, vence ou de qualquer outra maneira fica inválido, o framework se encarrega de pedir ao usuário que cadastre um novo cartão.

A chamada `Pay4all.getToken()` deve ser efetuada a cada compra do usuário.


# 5 Capturando a transação

Com o **payment_token** em mãos, você pode capturar a transação (efetuar a cobrança) através de uma chamada à API Conta 4all.

<aside class="notice">
Em caso de erro de comunicação ao executar esta chamada, você deve consultar o estado da transação utilizando a chamada descrita na seção 5.1 para verificar se a transação foi capturada com sucesso ou  não.
</aside>

```shell
curl -H "Content-Type: application/json" 
-X POST 
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...",
"paymentToken":"MDEyM...", "amount": 5000}' 
https://conta.api.4all.com/merchant/issueAuthorizedTransaction
```

**Caminho**: https://conta.api.4all.com/merchant/issueAuthorizedTransaction
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
|`merchantKey`|Chave privada de acesso do estabelecimento à API|String|44|Sim
|`paymentToken`|Token da autorização de pagamento, obtido da biblioteca mobile. |String|44|Sim
|`amount`|Valor da transação, em centavos.|Number|*|Sim
|`merchantMetaId`|Identificador único, atribuído pelo estabelecimento comercial, para poder pesquisar esta transação em caso de não recebimento da resposta desta chamada (contendo o transactionId). Deve ser um valor numérico inteiro (representado como string).|String|20|Não
|`returnImmediatly`|Quando presente e com valor **true** , a chamada retorna imediatamente. Neste caso, a transação estará com um status pendente (estados 0, 1 ou 2 vide **seção 6 desta documentação**). |Boolean||Não


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
|`transactionId`|Identificador da transação.|String|20|Sempre
|`status`|Estado da transação (ver **seção 8 desta documentação**).|Number|*|Sempre
|`datetime`|Data e hora UTC em que a transação foi processada pelo servidor 4all. Formato YYYYMMDDThh:mm:ssZ (Formato ISO 8601 https://en.wikipedia.org/wiki/ISO_8601).|String|20|Sempre

**Tratamento de Erro**
Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

```json
{
	error: {
		"code": 2318,
		"message": "Payment Token inválido."
	}
}
```

## 5.1 Consultando uma transação
Em caso de erro na chamada de captura de transação, você pode consultar o estado da transação através do Meta ID passado na chamada de Captura.

`shell
curl -H "Content-Type: application/json" 
-X POST 
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...",
"transactionId":"73423624"}' 
https://conta.api.4all.com/merchant/getTransactionDetails
`

**Caminho**: `https://conta.api.4all.com/merchant/getTransactionDetails`

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
|`merchantKey`|Chave de acesso do merchant à API|String|44|Sim
|`transactionId`|Identificador da transação|String|20|Depende
|`merchantMetaId`|Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa.|String|20|Depende


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

|Atributo        |Descrição  |Formato    |Tamanho|Presente
|----------------|-----------|-----------|-------|-----------
|`transactionId`|Identificador da transação|String|20|Sempre
|`subscriptionId`|Quando presente, informa o identificador da assinatura que gerou esta transação.|String|20|Depende
|`amount`|Valor da transação, em centavos.|Number||Sempre
|`status`|Estado da transação (ver **seção 8 desta documentação**).|Number||Sempre
|`createdAt`|Data e hora UTC em que a transação foi criada no servidor 4all (formato YYYYMMDDThh:mm:ssZ).|String|20|Sempre
|`paidAt`|Data e hora UTC em que a transação foi paga por uma conta 4all (formato YYYYMMDDThh:mm:ssZ). Presente somente se a transação já foi paga.|String|20|Depende
|`authorizationInfo`|Objeto contendo detalhes de autorização da transação. Presente somente se a transação já foi paga.|Object|*|Depende

**authorizationInfo**:

|Atributo        |Descrição  |Formato    |Tamanho|Presente
|----------------|-----------|-----------|-------|-----------
|`acquirerId`|Identificador do adquirente.|Number||Sempre
|`acquirerUsn`|Identificador/NSU do adquirente para a transação.|String|50|Sempre
|`acquirerTimestamp`|Timestamp de pagamento conforme reportado pelo adquirente. O formato e timezone deste campo segue o formato usado pelo adquirente. Presente somente se o adquirente informar.|String|50|Depende
|`brandId`|Identificador da bandeira do cartão que foi usado no pagamento.|Number||sempre

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
	error: {
		"code": 2318,
		"message": "Payment Token inválido."
	}
}
```

# 6 Gerência de Meios de Pagamento

A Biblioteca Mobile Payment também dispõem os seguintes métodos:

## 6.1 Troca de meio de Pagamento

Você pode adicionar ao seu aplicativo a opção para o usuário de alterar o cartão de crédito cadastrado para realizar pagamentos, através da chamada `Pay4all.changePaymentMethod()`, como no exemplo abaixo:

```Java
FourAll_DigitalCommerce pay4allObject = 
FourAll_DigitalCommerce.newInstance(MainActivity.this, <<APIKEY>>);

pay4allObject.changePaymentMethod();
```

Para esta chamada, deve-se implementar a mesma interface já descrita para a operação de obter o token do usuário.  Os mesmos métodos serão utilizados para a comunicação do DialogFragment com a Activity que a exibe.

## 6.2 Logout

Quando o usuário do seu aplicativo efetuar logout, você pode apagar os dados de pagamento de usuário através da chamada Pay4all.logout(), como no exemplo abaixo:

```Java
FourAll_DigitalCommerce pay4allObject =
FourAll_DigitalCommerce.newInstance(MainActivity.this, <<APIKEY>>);

pay4allObject.logout();
```

#7. Estados de uma Transação

|Estado | Descrição
|-------|-----------
|0 | Transação criada. Aguardando pagamento por uma conta 4all.
|1 | Transação em processo de pagamento por uma conta 4all.
|2 | Última tentativa de pagamento falhou. Aguardando pagamento por uma conta 4all.
|3 | Transação paga (capturada).
|4 | Transação em processo de cancelamento.
|5 | Transação cancelada.
|6 | Transação paga ­ cancelamento falhou.
|7 | Transação contestada pelo portador do cartão (chargeback).
|8 | Transação paga ­ reapresentada após contestação (chargeback refund).
|9 | Transação em processo de pagamento por débito