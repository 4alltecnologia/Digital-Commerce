---
title: Digital Commerce - Android - 1.0

language_tabs:

toc_footers:
  - <a href='http://4all.com'>4all.com</a>
  - <a href='https://4alltecnologia.github.io/Digital-Commerce/'>Home</a>

includes:

search: true
---

# Digital Commerce - Android - 1.2

# 1 Introdução

Com o módulo de pagamento para Android, você pode configurar seu aplicativo para receber pagamentos de cartões de crédito de maneira fácil e rápida.

A biblioteca Digital Commerce Android é compatível com as versões de Android 4.4 em diante (API LEVEL 19 ou superior)

Para aceitar pagamentos no seu App, você deve seguir os seguintes passos:

1. Obter Chaves de API no Portal do EC;
2. Importar o framework de Mobile Payment 4all;
3. Quando for efetuar um pagamento, obter um ***paymentToken***, usando a chamada `Pay4all.getToken()`.
4. Capturar a transação no seu servidor.

## 1.1 Endereços dos servidores de Homologação e de Produção

Os seguintes endpoints devem ser usados para executar as chamadas:

| Ambiente  | URL
|-----------|-------------------
|**Homologação**|conta.homolog.4all.com
|**Produção**|conta.api.4all.com

##1.2. Postback
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
	    url "s3://libs-externas"
        credentials(AwsCredentials) {
	        accessKey "AKIAJMX53DI7VWGFCSNA"
            secretKey "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"
	    }
	}
}
```

No arquivo build.gradle referente ao módulo da aplicação, basta adicionar a linha abaixo:

build.gradle(Module:app) :

```Gradle
    compile 'com.4all.libs:4all_digitalCommerce:1.1.2'
```


# 4 Instanciando um objeto DigitalCommerce
O objeto para as chamadas da biblioteca de DigitalCommerce da 4all possui seu construtor de forma privada. A fim de instanciar um objeto deste tipo, deve-se utilizar a chamada de método estático FourAll_DigitalCommerce.newInstance();
Tal chamada possui sobrecarga em sua assinatura, de forma a aceitar três formatos diferentes de parâmetros, sendo estes:

:: newInstance(AppCompatActivity activity) : O objeto será instanciado a utilizando suas configurações padrão. Recebe apenas a activity onde este é chamado a fim de utilizar desta seu fragmentManager e contexto

```Java
FourAll_DigitalCommerce digiCommerce = 	FourAll_DigitalCommerce.newInstance(MainActivity.this);        
```


:: newInstance(AppCompatActivity activity, String APIKEY) : Neste caso, recebe, além da activity onde está sendo chamado, também a APIKEY do cliente que a utiliza. A partir desta string, gerará uma nova instância de suas configurações.

```Java
FourAll_DigitalCommerce digiCommerce = 	FourAll_DigitalCommerce.newInstance(MainActivity.this, <<APIKEY do cliente>>);        
```

:: newInstance(AppCompatActivity activity, DigitalCommerce_Config config) : Ao invés de receber a APIKEY, recebe já uma instância da classe config com as configurações a serem utilizadas;

```Java
FourAll_DigitalCommerce digiCommerce = 	FourAll_DigitalCommerce.newInstance(MainActivity.this, DigitalCommerce_Config.getDefaultConfig());        
```


##4.1 Configurações da classe DigitalCommerce
A biblioteca possui uma classe de configurações para os atributos a serem utilizados pela classe principal da biblioteca. Embora ela possua um construtor, também possui métodos auxiliares para a geração de uma configuração padrão, a ser utilizados por várias instâncias da biblioteca.

As o objeto com as configurações padrão pode ser acessado através do método estático DigitalCommerce_Config.getDefaultConfig(). Porém, a fim de ter acesso a este, deve-se primeiro definir uma chave de API para ela. Esta definição é feita através da chamada DigitalCommerce_Config.setDefaultAPIKey(String APIKey) que recebe a String com a chave de API.

É possível também gerar novos objetos do tipo Config através da chamada do construtor estática DigitalCommerce_Config.newInstance(String APIKey), que recebe diretamente a String com a APIKEY




# 5 Fazendo a chamada DigitalCommerce.getToken()

Para obter o **payment_token**, crie uma instância o objeto Pay4all e faça a chamada getToken no exemplo abaixo:

```Java
FourAll_DigitalCommerce pay4allObject =
FourAll_DigitalCommerce.newInstance(MainActivity.this);

pay4allObject.getToken();
```

Quando a função `Pay4all.getToken()` é chamada pela primeira vez, ele é direcionado para uma tela onde se autentica e cadastra um cartão de crédito a ser usado em todas as compras no seu aplicativo.  Devido a natureza assíncrona da chamada,  a Activity a exibir DialogFragment para a inserção de dados do usuário deve implementar a interface desta:   OneTouch_WebPayFragment.OnOneTouchPaymentListener. A interface possui dois métodos: onSucess e onFail. Em caso de sucesso, o método onSucess será chamado, recebendo como parâmetro a chave do usuário, que também será salva na persistência do aplicativo para futuras utilizações. Já a chamada onFail, utilizada em caso de falhas na operação ou cancelamento pelo usuário não possui nenhum tipo de parâmetro.  

```Java
public class MainActivity extends AppCompatActivity implements DigitalCommerce_WebPayFragment.OnOneTouchPaymentListener {

...

@Override
public void onPaymentSuccess(String key) {
    Toast.makeText(this, "Sucesso ao realizar a operação 4all " + key, Toast.LENGTH_LONG).show();
	  }

@Override
public void onPaymentFail() {
    Toast.makeText(this, "Falha ao realizar a operação 4all", Toast.LENGTH_LONG).show();
	}
 }
```

Nas chamadas subsequentes, o usuário não precisará efetuar a autenticação novamente, e o **payment_token** é retornado de maneira transparente.

Se o cartão de crédito cadastrado é excluido pelo usuário, vence ou de qualquer outra maneira fica inválido, o framework se encarrega de pedir ao usuário que cadastre um novo cartão.

A chamada `Pay4all.getToken()` deve ser efetuada a cada compra.  As operações com o servidor 4all solicitam os dados de geolocalização do usuário.  Desta maneira, deve-se adicionar ao manifesto da aplicação as solicitações de uso do GPS do sistema. Para tal, é necessário que sejam acrescentadas as seguintes linhas ao documento:

```xml
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

A partir da API de Android lvl 23, o sistema deve implementar também a verificação de permissões ao usuário durante runtime.  Ao invés de solicitar ao usuário a permissão aos recursos do aparelho no momento da instalação, os sistemas mais atuais fazem a requisição no momento de uso destas.
Desta maneira, a activity onde será feita a chamada para a busca do token deverá implementar o código para o request de permissões, bem como o handler para a resposta do usuário, conforme o exemplo abaixo, retirado do site https://developer.android.com/training/permissions/requesting.html

Para checar se há permissão para o uso da localização:
```Java

// Assume thisActivity is the current activity
int permissionCheckCoarse = ContextCompat.checkSelfPermission(thisActivity,
        Manifest.ACCESS_COARSE_LOCATION);

int permissionCheckFine = ContextCompat.checkSelfPermission(thisActivity,
        Manifest.ACCESS_FINE_LOCATION);
```

  Para definir a solicitação do uso dos recursos de GPS do aparelho pela aplicação:
```Java
      // Verifica se deve mostrar motivo de utilização do recurso de geolocalização
      if (ActivityCompat.shouldShowRequestPermissionRationale(this,
          Manifest.permission.ACCESS_COARSE_LOCATION)) {

		// Mostra uma explicação assíncrona para o usuário. Ou seja, não bloqueia a thread principal.
		// Após o usuário verificar a explicação apresentada, chama novamente o request para a permissão de uso do recurso
      } else {
        // Não exige explicação. Faz o request direto para a permissão do uso.
        ActivityCompat.requestPermissions(this,
            new String[]{Manifest.permission.ACCESS_FINE_LOCATION, Manifest.permission.ACCESS_COARSE_LOCATION},
            FourAll_Lib4all.MY_PERMISSIONS_REQUEST_LOCATION);
       // MY_PERMISSIONS_REQUEST_LOCATION é um valor int constante na aplicaçnao. Este método de callback utiliza o resultado do request
      }
```

Handler para o resultado obtido pela permissão:
```java
  @Override
  public void onRequestPermissionsResult(int requestCode,
                                         String permissions[], int[] grantResults) {
    switch (requestCode) {
      case FourAll_Lib4all.MY_PERMISSIONS_REQUEST_LOCATION: {
        // Se o request for cancelado, o vetor será vazio.
        if (grantResults.length > 0
            && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
          FourAll_Lib4all.getInstance().initLocationManager(this);
          // Foi aprovada a permissão. Faz as ações solicitadas.
        } else {
          // A permissão não foi aprovada. Faz as ações de acordo.
          }
        }
        return;
      }

      // outros casos
      // utilize para chegar outras permissões
    }
  }
```


# 6 Capturando a transação

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

**Caminho**: `<endpoint>/merchant/issueAuthorizedTransaction`

**Descrição**: Captura uma transação pelo **paymentToken**

```json
{
	"merchantKey": "MDEyMzQ1Njc4OTAxMjMN...",
	"paymentToken": "MDEyM...",
	"amount": 5000,
	"merchantMetaId": "1259",
	"returnImmediatly": true,
	"postbackURL": "www.example.com/postback/"
}
```

**Requisição**:

|Atributo        |Descrição  |Formato    |Tamanho|Obrigatório
|----------------|-----------|-----------|-------|-------------
|**merchantKey**|Chave privada de acesso do estabelecimento à API|String|44|Sim
|**paymentToken**|Token da autorização de pagamento, obtido da biblioteca mobile. |String|44|Sim
|**amount**|Valor da transação, em centavos.|Number|*|Sim
|**merchantMetaId**|Identificador único, atribuído pelo estabelecimento comercial, para poder pesquisar esta transação em caso de não recebimento da resposta desta chamada (contendo o transactionId). Deve ser um valor numérico inteiro (representado como string).|String|20|Não
|**returnImmediatly**|Quando presente e com valor **true** , a chamada retorna imediatamente. Neste caso, a transação estará com um status pendente (estados 0, 1 ou 2 vide **seção 6 desta documentação**). |Boolean||Não
|**postbackURL**|Endereço a ser chamado quando o estado da transação mudar (vide seção 1.2 deste documento).|URL|250|não|
||||||


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
|**status**|Estado da transação (ver **seção 8 desta documentação**).|Number|*|Sempre
|**datetime**|Data e hora UTC em que a transação foi processada pelo servidor 4all. Formato YYYYMMDDThh:mm:ssZ (Formato ISO 8601 https://en.wikipedia.org/wiki/ISO_8601).|String|20|Sempre

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

## 6.1 Consultando uma transação
Em caso de erro na chamada de captura de transação, você pode consultar o estado da transação através do Meta ID passado na chamada de Captura.

`shell
curl -H "Content-Type: application/json"
-X POST
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...",
"transactionId":"73423624"}'
https://conta.api.4all.com/merchant/getTransactionDetails
`

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
|**transactionId**|Identificador da transação|String|20|Sempre
|**subscriptionId**|Quando presente, informa o identificador da assinatura que gerou esta transação.|String|20|Depende
|**amount**|Valor da transação, em centavos.|Number||Sempre
|**status**|Estado da transação (ver **seção 8 desta documentação**).|Number||Sempre
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

 Apenas um dos dois parâmetros, ***transactionId*** ou ***merchantMetaId***, pode ser usado; se ambos estiverem presentes, esta chamada falha com um erro específico.

* **Adquirentes**:
	1. Stone
	2. Pagar.me

* **Bandeiras**:
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

# 7 Gerência de Meios de Pagamento

A Biblioteca Mobile Payment também dispõem os seguintes métodos:

## 7.1 Obtendo informações do cartão

Você pode exibir em seu aplicativo qual é a bandeira e os últimos 4 dígitos do cartão de crédito configurado pelo usuário utilizando a chamada `pay4allObject.getUserData()`, como no exemplo abaixo:

```Java
FourAll_DigitalCommerce pay4allObject =
FourAll_DigitalCommerce.newInstance(MainActivity.this);

DigitalCommerce_CreditCardInfo cardInfo =  pay4allObject.getUserData();

//pode ser null caso não tenha configurado meio de pagamento
if (cardInfo != null) {
	String brand = cardInfo.getBrandName(); // "Visa"
	String lastDigits = cardInfo.getLastDigits(); // "1234"
}

```

## 7.2 Troca de meio de Pagamento

Você pode adicionar ao seu aplicativo a opção para o usuário de alterar o cartão de crédito cadastrado para realizar pagamentos, através da chamada `pay4allObject.changePaymentMethod()`, como no exemplo abaixo:

```Java
FourAll_DigitalCommerce pay4allObject =
FourAll_DigitalCommerce.newInstance(MainActivity.this);

pay4allObject.changePaymentMethod();
```

Para esta chamada, deve-se implementar a mesma interface já descrita para a operação de obter o token do usuário.  Os mesmos métodos serão utilizados para a comunicação do DialogFragment com a Activity que a exibe.

## 7.3 Logout

Quando o usuário do seu aplicativo efetuar logout, você pode apagar os dados de pagamento de usuário através da chamada `pay4allObject.logout()`, como no exemplo abaixo:

```Java
FourAll_DigitalCommerce pay4allObject =
FourAll_DigitalCommerce.newInstance(MainActivity.this);

pay4allObject.logout();
```

# 8 Estados de uma Transação

|Estado | Descrição
|-------|-----------
|0 | Transação criada. Aguardando pagamento por uma conta 4all.
|1 | Transação em processo de pagamento por uma conta 4all.
|2 | Última tentativa de pagamento falhou. Aguardando pagamento por uma conta 4all.
|3 | Transação paga (capturada).
|4 | Transação em processo de cancelamento.
|5 | Transação cancelada.
|6 | Transação paga ­ cancelamento falhou.
|7 | Transação contestada pelo portador do cartão (chargeback).
|8 | Transação paga ­ reapresentada após contestação (chargeback refund).
|9 | Transação em processo de pagamento por débito

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
