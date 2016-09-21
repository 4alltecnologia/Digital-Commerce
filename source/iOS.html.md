---
title: Digital Commerce - iOS - 1.0

language_tabs:

toc_footers:
  - <a href='http://4all.com'>4all.com</a>
  - <a href='https://4alltecnologia.github.io/Digital-Commerce/'>Home</a>

includes:
 
search: true
---

# Digital Commerce - iOS - 1.0

# 1 Introdução

Com o módulo de pagamento para iOS, você pode configurar seu aplicativo para receber pagamentos de cartões de crédito de maneira fácil e rápida.

Para aceitar pagamentos no seu App, você deve seguir os seguintes passos:

1. Obter Chaves de API no Portal do EC;
2. Importar o framework de Mobile Payment 4all;
3. Quando for efetuar um pagamento, obter um paymentToken, usando a chamada Pay4all.getToken().
4. Capturar a transação no seu servidor.

## 1.1 Endereços dos servidores de Homologação e de Produção

Os seguintes endpoints devem ser usados para executar as chamadas:

| Ambiente  | URL
|-----------|-------------------
|Homologação|conta.homolog.4all.com
|Produção|conta.api.4all.com


#2 Chaves de API

Para obter as Chaves de API, acesse o Portal do EC (https://portal.4all.com) e faça seu login. 

No menu lateral, clique na opção “Chaves de API” no sub-menu “Mobile Payment”.

Nesta página você pode obter pares de chaves no ambiente de Homologação ou no Ambiente de Produção.

Clique no botão “Gerar Chaves” para obter um par de chaves, a Pública (**PublicApiKey**) e a Privada (**MerchantKey**).


# 3 Importando a Biblioteca de Mobile Payment

1. Faça o download da [última versão do MobilePayment](https://lib.4all.com/mobile/ios/v1/MobilePayment.framework) e extraia o arquivo zip.
2.  Arraste o arquivo `MobilePayment.framework` extraído para o *File Navigator* do seu projeto no Xcode. Certifique-se de que a opção *Copy items if needed* está selecionada e clique em *Finish*.
3. Clique no seu projeto no *File Navigator* do Xcode. Selecione o Target do seu aplicativo e clique na aba General. Abaixo de Embedded Binaries, clique em +, selecione o `MobilePayment.framework` e clique em Add.

## 3.1 Instalação do MobilePayment em um projeto Swift

Caso o framework esteja sendo utilizado em um projeto Swift, você deve inserir `#import <MobilePayment/MobilePayment.h>` ao *Bridging Header* de seu projeto. Se ainda não existe um *Bridging Header* em seu projeto, siga os seguintes passos após ter executado os 3 passos anteriores:

1. Crie um header acessando *File > New > File > (iOS, watchOS, tvOS ou OS X) > Source > Header File* no Xcode.  Dê ao arquivo o nome do seu *Product Module* seguido de `“-Bridging-Header.h”`.
2.  No header criado, exponha o framework para ser utilizado em Swift adicionando a linha `#import <MobilePayment/MobilePayment.h>.`
3. Clique no seu projeto no *File Navigator* do Xcode. Selecione seu projeto e clique na aba *Build Settings*. Vá para a sessão *Swift Compiler - Code Generation* e, na linha O*bjective-C Bridging Header*, adicione o caminho para seu *Bridging Header* relativo ao seu projeto.

Para maiores informações sobre utilização de **Swift** e **Objective-C** em um mesmo projeto, acesse a documentação fornecida pela Apple em [Swift and Objective-C in the Same Project](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html).
 

# 4 Importando o framework MobilePayment

No início de seus arquivos Objective-C, insira `#import <MobilePayment/MobilePayment.h>` para importar o framework e torná-lo disponível para uso em seu código. No caso de arquivos Swift, não é necessário adicionar declaração para importar o framework uma vez que ele já esteja em seu *Bridging Header*.


# 5 Configurando o MobilePayment para uso

> Exemplo Objective-C

```Objective-C
[[P4APay4all sharedConfiguration] setApiKey:@"I2G5XQu8o4237v82"];
```

> Exemplo Swift

```Swift
P4APay4all.sharedConfiguration().apiKey = "I2G5XQu8o4237v82"
```

Após ter instalado e importado o framework, configure-o com as Chaves de API geradas no Portal do EC no seu *App Delegate.*

# 6 Fazendo a chamada getToken

> Exemplo Objective-C

```Objective-C
NSString *paymentToken = [[[P4APay4all alloc] init]
getTokenWithViewController:self 
  completionBlock:ˆ(NSString *paymentToken, NSError *error) {
    if (error != nil) {
      // Trate aqui o erro ocorrido…
      return;
    }
    // Utilize aqui o paymentToken...
  }];
```

> Exemplo Swift

```Swift
let paymentToken = P4APay4all().getTokenWithViewController(self) {
  paymentToken, error in
  guard error == nil else {
    // Trate aqui o erro ocorrido…
    return
  }
  // Utilize aqui o paymentToken...
}
```

Quando o método **getToken** é chamado pela primeira vez, ele é direcionado para uma tela onde se autentica e cadastra um cartão de crédito a ser usado em todas as compras no seu aplicativo. O *View Controller* passado por parâmetro é utilizado para apresentar esta tela. Nas chamadas subsequentes, o usuário não precisará efetuar a autenticação novamente, e o **paymentToken** é retornado de maneira transparente ao usuário.

Se o cartão de crédito cadastrado é excluído pelo usuário, vence ou de qualquer outra maneira fica inválido, o framework se encarrega de pedir ao usuário que cadastre um novo cartão.

A chamada **getToken** deve ser efetuada a cada compra do usuário visto que este é validado internamente.

# 7 Capturando a Transação no Servidor

Com o **paymentToken** em mãos, ele deve ser transferido ao seu servidor de maneira segura, via conexão **HTTPS**, ou  **SSL/TLS**, e você pode capturar a transação (efetuar a cobrança do usuário) através de uma chamada à API Conta 4all.

Exemplos:

`curl -X POST "https://api.pagar.me/1/transactions/{TOKEN}/capture"
-d 'amount=1000'
-d 'api_key=ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0'`

## 7.1 Consulta de Transação

Em caso de erro na chamada de captura de transação, você pode consultar o estado da transação através do Meta ID passado na chamada de Captura.

`curl -H "Content-Type: application/json" 
-X POST 
-d '{"merchantKey":"MDEyMzQ1Njc4OTAxMjMN...",
"transactionId":"73423624"}' 
https://conta.api.4all.com/merchant/getTransactionDetails`

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
|`merchantKey`|Chave de acesso do merchant à API|String|44|Sim
|`transactionId`|Identificador da transação|String|20|Depende
|`merchantMetaId`|Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa.|String|20|Depende

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

```json
{
	error: {
		"code": 2318,
		"message": "Payment Token inválido."
	}
}
```

Em caso de erro na chamada, o status HTTP retornado será diferente de **200**,  e o corpo da mensagem conterá um objeto no formato JSON contendo um objeto **error** contendo um código(**code**) e uma mensagem(**message**).

# 8 Gerência de Meios de Pagamento
O framework também dispõem os seguintes métodos:

## 8.1 Troca de meio de pagamento

> Exemplo Objective-C

```Objective-C
[[[P4APay4all alloc] init]
  changePaymentMethodWithViewController:self
  completionBlock:ˆ(NSString *paymentToken, NSError *error) {
    if (error != nil) {
      // Trate aqui o erro ocorrido…
      return;
    }
    // Utilize aqui o paymentToken...
  }];
```

> Exemplo Swift

```Swift
P4APay4all().changePaymentMethodWithViewController(self) {
  paymentToken, error in
  guard error == nil else {
    // Trate aqui o erro ocorrido…
    return
	}
	// Utilize aqui o paymentToken...
}
```
Você pode adicionar ao seu aplicativo a opção para o usuário de alterar o cartão de crédito cadastrado para receber pagamentos, através da chamada **changePaymentMethod**, como no exemplo ao lado :
 
O *View Controller* passado por parâmetro é utilizado para apresentar a tela de troca do meio de pagamento.

## 8.2 Logout

> Exemplo Objective-C

```Objective-C
[[[P4APay4all alloc] init] logout];
```

> Exemplo Swift

```Swift
P4APay4all().logout()
```

Quando o usuário do seu aplicativo efetuar logout, você pode apagar os dados de pagamento de usuário através da chamada **logout**, como no exemplo ao lado:

#9 Estados de uma Transação

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
Para realizar transações no ambiente de Homologação, utilize um par de chaves de Homologação, obtidas no Portal do EC.
</aside>

No Ambiente de Homologação, as transações efetuadas não geram cobranças, de modo que você pode testar a integração de seu Site com o Digital Commerce 4all.

No Portal do EC, você pode gerar chaves para o ambiente de Homologação. Quando você usa uma chave de Homologação, aparecerá uma mensagem na Janela de Checkout indicando que você está no ambiente de Homologação e as transações efetuadas não resultarão em cobranças.

## Conta 4all no Ambiente de Homologação

Nas contas criadas no ambiente de Homologação, o envio de SMS não é disparado, e os desafios de SMS podem ser respondidos com "444444".

## Cartões de Teste
No ambiente de Homologação, você pode utilizar cartões de qualquer número de 16 dígitos e qualquer CVV de 3 dígitos para testar compras em seu Site. 

**Cartões de final PAR sempre resultam em compras efetivadas com sucesso.**
**Cartões de final ÍMPAR sempre resultam em transações negadas.**
