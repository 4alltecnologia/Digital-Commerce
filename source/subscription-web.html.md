---
title: 4 Pay - Subscription Web

language_tabs:

toc_footers:
  - <a href='http://4all.com'>4all.com</a>
  - <a href='https://4alltecnologia.github.io/Subscription/'>Home</a>

includes:

search: true
---

# Subscription - Web - 1.0.1

# 1 Introdução

Com o Subscription 4all, você pode configurar seu website para captar clientes de maneira fácil e rápida.

Para que seus clientes contratem assinaturas em seu site, você precisa executar os passos:

1. Obter chaves de API no Portal do EC;
2. Incluir o a Janela de Checkout de Subscription em seu site;
3. Capturar a assinatura no seu servidor.

## 1.1 Endereços dos servidores de Homologação e de Produção

Os seguintes endpoints devem ser usados para executar as chamadas:

| Ambiente  | URL
|-----------|-------------------
|**Homologação**|conta.homolog.4all.com|
|**Produção**|conta.api.4all.com|

## 1.2. Postback
O postback é um mecanismo de comunicação com o merchant para informar sobre mudanças ocorridas no status de um pagamento ou assinatura.
Quando um pagamento ou assinatura são criados pelo merchant, uma URL pode ser informada, para onde as mensagens de postback serão direcionadas. Esta URL deve estar num formato válido, apontar para um domínio autorizado no cadastro do merchant, e pode possuir até 250 caracteres.


```json
{
  "type": 1,
  "id": "17999999999999999999",
  "status": 1
}
```

Sempre que ocorrer uma alteração no status de um pagamento ou assinatura que possuam as informações de postback configuradas, uma requisição POST será feita para a URL informada, contendo um corpo JSON (Contenttype:application/json) como pode ser visto no exemplo a seguir:

O campo ***type*** pode assumir os valores 1 (para postback referente a pagamento) ou 2 (para assinatura).

Para que seja considerado que a comunicação ocorreu com sucesso, o retorno dado à mensagem de postback deve possuir status 200 ou 204. Do contrário, serão efetuadas retentativas intervaladas entre si em pelo menos um minuto até um  máximo de cinco vezes. Caso ainda não se obtenha sucesso, NÃO serão feitas tentativas adicionais de comunicação da mudança de status.

O recurso de postback é apenas uma conveniência para melhorar a integração com o sistema 4all, e está sujeito a falhas de comunicação, indisponibilidade de alguma das partes, etc. Desta forma, não elimina a necessidade de consultas periódicas ao status do pagamento ou da assinatura que se deseja monitorar.

Para garantir a autenticidade da mensagem, é enviado no seu cabeçalho um campo **Content-signature**. Este contém um *hash* em **HMAC-SHA-256** em hexadecimal calculado sobre o corpo da mensagem, usando como chave a *signatureKey* do estabelecimento.


# 2 Chaves de API

Para obter as Chaves de API, acesse o **Portal do EC** (https://portal.4all.com) e faça seu login. No menu lateral, clique na opção “Chaves de API” no sub-menu “*Subscription*”.

Nesta página você pode obter pares de chaves no ambiente de Homologação ou no Ambiente de Produção.

Clique no botão “Gerar Chaves” para obter um par de chaves, a Pública (**PublicApiKey**) e a Privada (**MerchantKey**).

# 3 Incluindo a Janela de Checkout de Subscription em seu Site

A Janela de Checkout de Subscription é apresentada para que o cliente faça a adesão ao seu plano de assinatura.

O *dialog* permite que o usuário informe os dados do cartão ou se autentique com sua Conta 4all para buscar seus dados de pagamento já cadastrados, e faça a adesão à sua assinatura, de maneira segura.

Você pode inserir o Dialog no seu site de duas maneiras: via **Embedded Form**, ou via a nossa **Biblioteca Javascript**.

##3.1 Embedded Form

```html
<form action="assinatura_contratada.php" method="POST">

  <!-- … demais inputs do formulário … -->

  <script
    src="https://lib.4all.com/lib/subscription-embed.js"
    data-public-api-key="homolog_gR83LcKFXFOdZJgRjiv0FhEKFzsuYB0uAojq9PQcnV4="
    data-upfront-amount="3000"
    data-start-date="2018-12-20"
    data-recurring-amount="6000"
    data-recurrence-count="0"
    data-interval-type="2"
    data-interval-value="1"
    data-payment-tolerance="5"
    >
  </script>
</form>
```

**Como funciona:** Nesta modalidade de checkout, você adiciona à sua página um formulário que efetua, de forma segura, todo o processo de captura e validação dos dados de pagamento do cliente, assim, as informações sensíveis do seu cliente nunca entram em contato com ambientes não-seguros.

No evento de *submit* do seu formulário, a Janela de Checkout será apresentada para o cliente inserir os dados de pagamento e concordar com os termos da assinatura.

Além do **subscription_token**, os seguintes campos `<input type="hidden">` também são adicionados ao formulário:

|ID    |Descrição  |Formato    |Presente
|------|-----------|-----------|---------
|**subscription_fullname**| Nome completo do usuário. | string | Sempre
|**subscription_emailaddress**| Endereço de email informado pelo usuário. | string | Sempre
|**subscription_phonenumber**| Telefone celular informado pelo usuário. Com ddi e ddd, exemplo: "5511992392322". | string | Sempre
|**subscription_cpfcnpj**| CPF ou CNPJ informado pelo usuário. Sem formatação. |string| Sempre
|**subscription_birthdate**| Data de nascimento informado pelo usuário. Sem formatação. |string|Sempre

Substitua o valor de cada atributo de acordo com a assinatura a ser contratada. Os atributos aceitos são:

|Atributo    |Descrição  |Formato    |Obrigatório
|------------|-----------|-----------|--------------
|**data-public-api-key**| Chave de API pública de Subscription 4all.|String|Sim
|**data-upfront-amount**|Valor que deve ser pago imediatamente pela conta 4all, no momento em que aceitar a assinatura (taxa de adesão). Em centavos. Ex: "1425" para R$ 14,25. Passe "0" em caso que não há taxa de adesão.|String|Sim
|**data-start-date**|Data em que os pagamentos periódicos terão início. Formato YYYY-MM-DD. É a data em que o primeiro pagamento recorrente será efetuado.|String|Sim
|**data-recurring-amount**|Valor das cobranças periódicas. Em centavos. Ex: "1425" para R$ 14,25.|String|Sim
|**data-recurrence-count**|Indica quantas cobranças periódicas serão feitas no total. O valor 0 indica infinitas cobranças (até o cancelamento da assinatura).|String|Sim
|**data-interval-type**|Indica como o intervalo entre cobranças recorrentes é medido: 0-Dias 1-Semanas 2-Meses 3-Indefinido.|String|Sim
|**data-interval-value**|Indica quantos dias/semanas/meses (dependendo do parâmetro **data-interval-type**) existem entre dois pagamentos recorrentes consecutivos.|String|Sim
|**data-payment-tolerance**|Indica quantos dias a assinatura pode permanecer com pagamento em atraso antes de ser automáticamente cancelada.|String|Sim

**Observações:**
- Se o parâmetro `intervalType` possuir valor igual a `3`, então a assinatura não terá intervalo definido e o valor definido para as recorrências é `R$ 0,00`. Isto é, o estabelecimento deverá controlar quando a assinatura deverá ser cobrada, quanto será cobrado naquela recorrência e qual a data base a ser contabilizada para a recorrência cobrada.

## 3.2 Biblioteca Javascript

```html
<script src="https://lib.4all.com/lib/subscription-lib.js"></script>
```

Para fazer a integração via javascript, você deve primeiramente importar o arquivo da biblioteca na sua página, adicionando ao seu HTML a linha abaixo ou ao lado.

```javascript
	function onSuccess(data) {
	  /*
	  esta função será chamada quando o cliente efetuar a contratação, recebendo os dados do cliente e o subscription_token no objeto data
	  */
	}

	function onCancel() {
	  /*
	  esta funcão será chamada caso o cliente feche o dialog sem efetuar a contratação
	  */
	}

	var options = {
	  publicApiKey: "homolog_gR83LcKFXFOdZJgRjiv0FhEKFzsuYB0uAojq9PQcnV4=",
	  upfrontAmount: 3000,
	  startDate: "2018-12-20",
	  recurringAmount: 6000,
	  recurrenceCount: 0,
	  intervalType: 2,
	  intervalValue: 1,
	  paymentTolerance: 5,
	  successCallback: onSuccess,
	  cancelCallback: onCancel,

		//se você já tem dados do usuário, você pode informar à biblioteca para acelerar o processo de cadastro
		input: {
			name: 'Fulano de tal',
			phone: '51999998888', //ddd e telefone
			email: 'fulano@gmail.com',
			document: '00000000000', //cpf do usuário
			birthdate: '10/02/1992' //data de nascimento no formato DD/MM/AAAA
		},

		//adicione este treixo se você deseja que a autenticação de usuário seja somente via código SMS
		rules: {
			smsOnly: true
		}
	};

Subscription4all.startSubscription(options);
```

Após isso, a biblioteca está disponível em seu escopo global como 'Subscription4all'. Para iniciar o processo de login/pagamento, basta chamar a função **startSubscription** da biblioteca, como demonstrado abaixo ou ao lado (código javascript).

Os parâmetros dessa função (atributos do objeto *options*) são:

|Atributo    |Descrição  |Formato    |Obrigatório
|------------|-----------|-----------|--------------
|**publicApiKey**|Chave de API pública de Subscription 4all.|String|Sim
|**upfrontAmount**|Valor que deve ser pago imediatamente pela conta 4all, no momento em que aceitar a assinatura (taxa de adesão). Em centavos. Ex: "1425" para R$ 14,25. Passe "0" em caso que não há taxa de adesão.|Number|Sim
|**startDate**|Data em que os pagamentos periódicos terão início. Formato YYYY-MM-DD. É a data em que o primeiro pagamento recorrente será efetuado.|String|Sim
|**recurringAmount**|Valor das cobranças periódicas. Em centavos. Ex: "1425" para R$ 14,25.|Number|Sim
|**recurrenceCount**|Indica quantas cobranças periódicas serão feitas no total. O valor 0 indica infinitas cobranças (até o cancelamento da assinatura).|Number|Sim
|**intervalType**|Indica como o intervalo entre cobranças recorrentes é medido: 0-Dias 1-Semanas 2-Meses 3-Indefinido.|Number|Sim
|**intervalValue**|Indica quantos dias/semanas/meses (dependendo do parâmetro **intervalType**) existem entre dois pagamentos recorrentes consecutivos.|String|Sim
|**paymentTolerance**|Indica quantos dias a assinatura pode permanecer com pagamento em atraso antes de ser automáticamente cancelada.|String|Sim
|**successCallback** | Função que será chamada quando o checkout estiver finalizado com sucesso. Recebe o objeto *data* como parâmetro. | Função | Sim
|**cancelCallback**| Função que será chamada caso o usuário cancele o processo de contratação sem concluí-lo, ou aconteça um erro no pagamento. | Função | Não
|**debug**|Parâmetro usado para ativar os logs no console, útil para validação dos parâmetros passados à biblioteca. |Boolean||Não
|**input**| Caso já tenha coletado dados do usuário, você pode acelerar o processo de login/cadastro fornecendo os dados do usuário neste objeto. | Objeto (olhar abaixo) | Não
|**rules**| Objeto com opções para o funcionamento da biblioteca. | Objeto (olhar abaixo) | Não

Objeto *input*:

|Atributo    |Descrição  |Formato    |Obrigatório
|------------|-----------|-----------|--------------
|**name**| Nome completo do usuário.| String | Não
|**phone**| Telefone do usuário, com DDD. | String | Não
|**email**| Email do usuário, com DDD. | String | Não
|**document**| CPF do usuário. | String | Não
|**birthdate**| Data de nascimento do usuário, formato DD/MM/AAAA. | String | Não

Objeto *rules*:

|Atributo    |Descrição  |Formato    |Obrigatório
|------------|-----------|-----------|--------------
|**smsOnly**| Caso configurado como *true*, não será solicitada senha ao usuário, e a autenticação é feita somente por código SMS enviado ao telefone do usuário. | Boolean | Não

Caso o cliente efetue o aceite na assinatura, o *successCallback* é chamado, recebendo um objeto *data* contendo as seguintes propriedades:

|Atributo   |Descrição  |Formato    | Presente
|------|-----------|-----------|---------
|**subscriptionToken**| Token que deve ser passado na chamada **issueAuthorizedSubscription** para confirmar a contratação da assinatura. Presente somente quando o atributo **capture** é passado como *false*. | string | Depende
|**fullName**| Nome completo do usuário. | string | Sempre
|**emailAddress**| Endereço de email informado pelo usuário. | string | Sempre
|**phoneNumber**| Telefone celular informado pelo usuário. Com ddi e ddd, exemplo: "5511992392322". | string | Sempre
|**cpfCnpj**| CPF ou CNPJ informado pelo usuário. Sem formatação. |string| Sempre
|**birthDate**| Data de nascimento informado pelo usuário. Sem formatação. |string|Sempre

**Observações:**
- Se o parâmetro `intervalType` possuir valor igual a `3`, então a assinatura não terá intervalo definido e o valor definido para as recorrências é `R$ 0,00`. Isto é, o estabelecimento deverá controlar quando a assinatura deverá ser cobrada, quanto será cobrado naquela recorrência e qual a data base a ser contabilizada para a recorrência cobrada.

# 4 Chamadas de API

##4.1 Confirmando uma contratação

Após o cliente autorizar a contratação da assinatura, o estabelecimento deverá realizar uma chamada através de seu servidor backend para confirmar a assinatura e obter o identificador da mesma. Será utilizado o **subscriptionToken** para identificar a autorização.

Os dados fornecidos nesta chamada devem ser os mesmos fornecidos para o cliente. Caso venham a divergir, a confirmação da assinatura falhará. Isto evita problemas em caso de divergencia.

<aside class="notice">
Em caso de erro de comunicação ao executar esta chamada, você deve consultar o estado da assinatura utilizando a chamada descrita na seção 4.2 para verificar se a assinatura foi confirmada com sucesso ou não.
</aside>

```shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey": "MDEyMzQ1Njc4OTAxMjMN...","subscriptionToken": "MDEyM...","upfrontAmount": 3000,"startDate": "2016-12-20","recurringAmount": 6000,"recurrenceCount": 0,"intervalType": 3,"intervalValue": 1,"paymentTolerance": 5}'
https://conta.api.4all.com/merchant/issueAuthorizedSubscription
```

**Caminho**: `<endpoint>/merchant/issueAuthorizedSubscription`

**Descrição**: Confirma a contratação de uma assinatura pelo **subscriptionToken**

```json
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMN...",
	"subscriptionToken": "MDEyM...",
	"upfrontAmount": 3000,
	"startDate": "2016-12-20",
	"recurringAmount": 6000,
	"recurrenceCount": 0,
	"intervalType": 3,
	"intervalValue": 1,
	"paymentTolerance": 5
}
```

**Requisição**:

|Atributo        |Descrição  |Formato    |Tamanho|Obrigatório
|----------------|-----------|-----------|-------|-------------
|**merchantKey**|Chave de acesso do estabelecimento à API|String|44|Sim
|**subscriptionAuthorizationToken**|Token da autorização de assinatura, obtido do Checkout.|String|44|Sim
|**upfrontAmount**|Valor que deve ser pago imediatamente pela conta 4all, no momento em que aceitar a assinatura (taxa de adesão). Em centavos. Ex: "1425" para R$ 14,25. Passe "0" em caso que não há taxa de adesão.|Number|Sim
|**startDate**|Data em que os pagamentos periódicos terão início. Formato YYYY-MM-DD. É a data em que o primeiro pagamento recorrente será efetuado.|String|Sim
|**recurringAmount**|Valor das cobranças periódicas. Em centavos. Ex: "1425" para R$ 14,25.|Number|Sim
|**recurrenceCount**|Indica quantas cobranças periódicas serão feitas no total. O valor 0 indica infinitas cobranças (até o cancelamento da assinatura).|Number|Sim
|**intervalType**|Indica como o intervalo entre cobranças recorrentes é medido: 1-Dias 2-Semanas 3-Meses.|Number|Sim
|**intervalValue**|Indica quantos dias/semanas/meses (dependendo do parâmetro **intervalType**) existem entre dois pagamentos recorrentes consecutivos.|String|Sim
|**paymentTolerance**|Indica quantos dias a assinatura pode permanecer com pagamento em atraso antes de ser automáticamente cancelada.|String|Sim
|**merchantMetaId**|Identificador único, atribuído pelo estabelecimento comercial, para poder pesquisar esta assinatura em caso de não recebimento da resposta desta chamada. Deve ser um valor numérico inteiro (representado como string).|String|20|Não
|**postbackURL**| Endereço a ser chamado quando o estado da assinatura mudar.	|URL	|1 - 250	|Não|
|**returnImmediatly**|Quando presente e com valor **true** , a chamada retorna imediatamente. Neste caso, a assinatura estará com um status pendente (estados 0, 1 ou 2 vide **seção 6 desta documentação**). |Boolean||Não


**Observações**:

 - O parâmetro "subscriptionToken" deve ser obtido através da Biblioteca Javascript ou Embedded Form.
 - O chamador deve estar preparado para lidar com tempos de resposta longos, de até 30 segundos (a menos que o parâmetro `returnImmediatly` seja passado como `true`, neste caso, a chamada retorna imediatamente).
 - Caso o parâmetro `returnImmediatly` seja passado como `true`, a chamada retorna imediatamente, porém o resultado da assinatura deve ser consultada posteriormente utilizando a chamada descrita na seção 4.2.
 - O parâmetro `merchantMetaId` poderá ou não aceitar duplicidade de valor, esta configuração será definida por Estabelecimento.

```json
{
	"subscriptionId": "2181486",
	"status": 3,
	"datetime": "2016-08-31T20:12:31Z"
}
```

**Resposta**:

|Atributo        |Descrição  |Formato    |Tamanho|Presente
|----------------|-----------|-----------|-------|-------------
|**subscriptionId**|Identificador da assinatura.|String|20|Sempre
|**status**|Estado da transação (ver **seção 6 desta documentação**).|Number|*|Sempre
|**datetime**|Data e hora UTC em que a transação foi processada pelo servidor 4all. Formato YYYYMMDDThh:mm:ssZ (Formato ISO 8601 https://en.wikipedia.org/wiki/ISO_8601).|String|20|Sempre

**Tratamento de Erro**
Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

```json
{
	"error": {
		"code": 2318,
		"message": "Subscription Token inválido."
	}
}
```


## 4.2 Consultando uma assinatura

Em caso de erro na chamada de confirmação, ou caso você queira consultar o status de uma assinatura, você pode consultar o estado da assinatura através do campo **merchantMetaId**, fornecido na chamada de confirmação. Também será possivel utilizar esta chamada para conferir o andamento da assinatura, e verificar se o pagamento esta em dia.

```shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...","transactionId":"73423624"}'
https://conta.api.4all.com/merchant/getSubscriptionDetails
```

**Caminho**: `<endpoint>/merchant/getSubscriptionDetails`

**Descrição**:  Retorna os detalhes de uma assinatura.


> Exemplo de requisição:

```json
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMNT...",
	"subscriptionId": "872364"
}
```

**Requisição**:

|Campo |Descrição |Formato |Tam. |Obrigatório
|------|----------|--------|-----|-------
|**merchantKey**|Chave de acesso do merchant à API.|String|44|sim|
|**subscriptionId**|Identificador da assinatura.|String|20|depende|
|**merchantMetaId**|Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa.|String|20|depende|


```json
{
	"subscriptionId": "872364",
	"upfrontAmount": 2500,
	"startDate": "2016-02-25",
	"recurringAmount": 5000,
	"recurrenceCount": 0,
	"intervalType": 3,
	"intervalValue": 1,
	"paymentTolerance": 10,
	"status": 3,
	"customerId": "1259",
	"createdAt": "2016-02-24",
	"acceptedAt": "2016-02-24",
	"lastTranactionId": "120893",
	"lastPaymentDate": "2016-02-25",
	"nextPaymentDate": "2016-03-25",
	"subscriptionToken": "M0T1RBeE1qTU5ULi4u..."
}
```

**Resposta:**

|Campo |Descrição |Formato |Tam. |Presente
|------|----------|--------|-----|-------
|**subscriptionId**|Identificador da assinatura.|String|20|sempre|
|**upfrontAmount**|Valor que deve ser pago imediatamente pela conta 4all, no momento em que aceitar a assinatura.|Number||sempre|
|**startDate**|Data de início dos pagamentos periódicos. Formato YYYY-MM-DD.|String|10|sempre|
|**recurringAmount**|Valor dos pagamentos periódicos.|Number||sempre|
|**recurrenceCount**|Indica quantos pagamentos periódicos serão feitos no total. O valor 0 indica infinitos pagamentos (até o cancelamento da assinatura).|Number||sempre|
|**intervalType**|Indica como o intervalo entre pagamentos recorrentes é medido: 1-Dias 2-Semanas 3-Meses |Number||sempre|
|**intervalValue**|Indica quantos dias/semanas/meses (dependendo do parâmetro ***intervalType***) existem entre dois pagamentos recorrentes consecutivos.|Number||sempre|
|**paymentTolerance**|ndica quantos dias a assinatura pode permanecer sem receber pagamento antes de ser automaticamente cancelada.|Number||sempre|
|**status**|Estado da transação (conforme `Anexo B Tabela B.3`).|Number||sempre|
|**customerId**|Quando presente, contém a identificação única da conta 4all para a qual esta assinatura foi direcionada.|String|20|depende|
|**createdAt**|Data e hora UTC em que a transação foi criada no servidor 4all. Formato:YYYY-MM-DDThh:mm:ssZ|String|20|sempre|
|**acceptedAt**|Data e hora UTC em que a transação foi aceita por uma conta 4all (formato YYYY-MM-DDThh:mm:ssZ). Presente somente se a transação já foi aceita.|String|20|depende|
|**lastTransactionAmount**|Valor da última transação gerada por esta assinatura. Presente somente quando a assinatura já tiver gerado um pagamento.|Number|*|depende|
|**lastTransactionId**|Identificador da última transação gerada por esta assinatura. Presente somente quando a assinatura já tiver gerado um pagamento.|String|20|depende|
|**lastPaymentDate**|Data do último pagamento gerado através desta assinatura. Presente somente quando a assinatura já tiver gerado um pagamento. Formato YYYY-MM-DD.|String|10|depende|
|**nextPaymentDate**|Data do próximo pagamento que será gerado por esta assinatura. Formato YYYY-MM-DD.|String|10|sempre|
|**subscriptionToken**|Chave para consulta do estado da assinatura, através da chamada "Wait For Subscription".|String|44|sempre|


**Observações**

 Apenas um dos dois parâmetros, ***subscriptionId*** ou ***merchantMetaId***, pode ser usado; se ambos
estiverem presentes, esta chamada falha com um erro específico.

**Tratamento de Erro**
Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

```json
{
	"error": {
		"code": 2318,
		"message": "Subscription ID inválido."
	}
}
```

##4.3 Alterando o valor recorrente de uma assinatura

Utilize esta chamada para alterar o valor de cobrança recorrente de uma assinatura.

```shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...","transactionId":"73423624"}'
https://conta.api.4all.com/merchant/changeSubscriptionAmount
```

**Caminho**: `<endpoint>/merchant/changeSubscriptionAmount`

**Descrição**: Esta chamada permite ao merchant alterar o valor do pagamento recorrente de uma assinatura.

> Exemplo de requisição:

```json
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMNT...",
	"subscriptionId": "872364",
	"newAmount": 1250
}
```

**Requisição:**

|Campo |Descrição |Formato |Tam. |Obrigatório
|------|----------|--------|-----|-------
|**merchantKey**|Chave de acesso do merchant à API.|String|44|sim|
|**subscriptionId**|Identificador da assinatura.|String|20|sim|
|**newAmount**|Novo valor a ser cobrado periodicamente (em centavos).|Number||sim|


**Resposta:** HTTP 204

**Observações**:

● Esta chamada afeta todos os valores cobrados periodicamente, a partir do momento em que a chamada é executada. Valores anteriores não são afetados. Se existe um pagamento em aberto associado a esta assinatura, o seu valor também não é alterado, mesmo que o pagamento em si ocorra depois desta chamada ter sido executada.

**Tratamento de Erro**
Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

```json
{
	"error": {
		"code": 2318,
		"message": "Subscription ID inválido."
	}
}
```

#4.4 Cancelando uma assinatura
Utilize esta chamada para cancelar uma assinatura. Ela permanecerá com estado "em dia" até a data da próxima cobrança recorrente.

```shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...","subscriptionId":"73423624"}'
https://conta.api.4all.com/merchant/cancelSubscription
```

**Caminho**: `<endpoint>/merchant/cancelSubscription`

**Descrição**: Esta chamada permite ao merchant cancelar uma assinatura.


> Exemplo de requisição:

```json
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMNT...",
	"subscriptionId": "872364"
}
```

**Requisição:**

|Campo |Descrição |Formato |Tam. |Obrigatório
|------|----------|--------|-----|-------
|**merchantKey**|Chave de acesso do merchant à API.|String|44|sim|
|**subscriptionId**|Identificador da assinatura.|String|20|sim|


**Resposta:** HTTP 204

**Tratamento de Erro**
Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

```json
{
	"error": {
		"code": 2318,
		"message": "Subscription ID inválido."
	}
}
```


##4.5 Efetuando uma baixa manual

Utilize esta chamada para registrar o pagamento de uma assinatura, quando este não foi feito pelo sistema da 4all (exemplo: pagamento feito em dinheiro). Ela atualizará a data do próximo pagamento de acordo com o número de recorrências dado como entrada. Caso existam recorrências em atraso, o estado da assinatura só irá mudar para "em dia" se o número de recorrências for maior que o número de recorrências atrasadas. Se não informado o número de recorrências, será registrado o pagamento de uma só recorrência. Também é possível dar baixa na cobrança da taxa de adesão.
```shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...", "subscriptionId": "872364", "paymentMethod": "3", "numberOfRecurrences": "3", "includeUpFrontAmount": false}'
https://conta.api.4all.com/merchant/payUpSubscription
```

**Caminho**: `<endpoint>/merchant/payUpSubscription`

**Descrição**: Esta chamada permite ao merchant fazer o pagamento manual de recorrências uma assinatura.


> Exemplo de requisição:

```json
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMNT...",
	"subscriptionId": "872364",
	"paymentMethod": "3",
	"numberOfRecurrences": "3",
	"includeUpFrontAmount": false
}
```

**Requisição:**

|Campo |Descrição |Formato |Tam. |Obrigatório
|------|----------|--------|-----|-------
|**merchantKey**|Chave de acesso do merchant à API.|String|44|sim|
|**subscriptionId**|Identificador da assinatura.|String|20|sim|
|**paymentMethod**|Forma de pagamento.|String|20|sim|
|**numberOfRecurrences**|Número de recorrências à serem marcados como pagas.|String|20|não|
|**includeUpFrontAmount**| Caso seja passado como *true*, a taxa de adesão da assinatura será baixada. A taxa de adesão não conta como uma recorrência (exemplo, se for passado *0* em **numberOfRecurrences** e *true* em **includeUpFrontAmount**, será feita somente a baixa da taxa de adesão).  |Bool|não|


**Resposta:** HTTP 204

**Tratamento de Erro**
Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

```json
{
	"error": {
		"code": 2318,
		"message": "Subscription ID inválido."
	}
}
```

#5. Estados de uma Assinatura
|Estado|Descrição
|------|---------
|0|Assinatura criada. Aguardando apropriação por uma conta 4all.|
|1|Assinatura em processo de apropriação por uma conta 4all.|
|2|Última tentativa de apropriação falhou. Aguardando apropriação por uma conta 4all.|
|3|Pagamento em dia.|
|4|Aguardando pagamento recorrente.|
|5|Assinatura em atraso.|
|6|Assinatura chegou ao final.|
|7|Assinatura cancelada automaticamente por falta de pagamento.|
|8|Assinatura cancelada pelo *customer*.|
|9|Assinatura cancelada pelo merchant|
|10|Pagamento em dia, mas será cancelada no próximo vencimento por solicitação do customer.|


#6. Estados de uma Transação

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

# Homologação

<aside class="notice">
Para realizar operações no ambiente de Homologação, utilize um par de chaves de Homologação, obtidas no Portal do EC.
</aside>

No Ambiente de Homologação, as transações efetuadas não geram cobranças, de modo que você pode testar a integração de seu Site com o Subscription 4all.

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
