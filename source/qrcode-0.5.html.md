# Pagamento Integrado via QR Code - 0.5

# 1. Introdução

## 1.1. Chave de Autenticação
O acesso à API é restrito a parceiros comerciais previamente cadastrados, sendo necessário informar, em cada conexão, a chave de autenticação fornecida na interface de gerenciamento. A duração da chave é indeterminada, somente perdendo a validade caso a conta do EC seja desativada na interface de gerenciamento.

### 1.2. GET sobre HTTPS
Com capacidade de 1024 caracteres, este método é utilizado quando se quer passar poucas ou pequenas informações para realizar uma pesquisa ou simplesmente passar uma informação para outra página através da URL (de Uniform Resource Locator).
Exemplo: https://api.4all.com/qrcode/payment?apiKey=&amount=3000

### 1.3. Parâmetros das Requisições e Respostas
O tamanho dos parâmetros pode ser fixo ou variável e segue a seguintes regras:
  * **String, URL e Datetime: tamanho medido em caracteres UTF-8;**
  * **Number: Valores numéricos;**
  * **Object: Não possui tamanho.**

### 1.4. Respostas de Sucesso
O iframe será redirecionado para uma url `endpoint/qrcode/paymentSuccess`. Na tela irá aparecer informações sobre o pagamento.

## 1.5. Respostas de Erro
O iframe será redirecionado para uma url `endpoint/qrcode/paymentFail`. Na tela irá aparecer informações sobre a falha do pagamento.

### 1.6. Endereços dos servidores de testes e de produção
Os seguintes endpoints devem ser usados para executar as chamadas:

| Ambiente  | URL
|-----------|-------------------
|**Testes**| [https://test.api.4all.com](https://test.api.4all.com)
|**Produção**| [https://api.4all.com](https://api.4all.com)

Os serviços sempre devem ser acessados usando TLS, na porta 443. Na prática, os endpoints acima sempre serão precedidos de "https://".


## 1.3. Endereços dos servidores de Homologação e de Produção

Os seguintes endpoints devem ser usados para executar as chamadas:

| Ambiente  | URL
|-----------|-------------------
|**Homologação**| [https://loyalty.homolog.4all.com/api/](https://loyalty.homolog.4all.com/api/)
|**Produção**| [https://loyalty.4all.com/api/](https://loyalty.homolog.4all.com/api/)


# 2. Chamadas de API

## 2.1. QrCode

### 2.1.1. Get IFrame
**Caminho**: `<endpoint>/qrcode/payment`

**Método HTTP**: GET

**Descrição**: Retorna uma página web desenvolvida na 4all para controlar todo o fluxo de geração de QRCode e confirmação do pagamento. Tamanho ideal do iframe 375 px x 400 px.

**Fluxo**: Após definido o valor do pagamento e as parcelas,deverá ser chamado                
 `<endpoint>/qrcode/payment`, será gerado um QR Code e o iframe ficará fazendo requisições de 3 em 3 segundos para o servidor da 4all verificando se o pagamento ja foi efetuado se confirmado o pagamento ele será redirecionado para `endpoint/qrcode/paymentSucess`(item 1.4.) caso haja um pagamento mas ele não seja autorizado será redirecionado para `endpoint/qrcode/paymentFail`(item 1.5).

**Observação**:
* **Todos os parâmetros devem ser devidamente escapados para serem enviados como querystring.**

**Requisição**:

|Campo               |Descrição                            |Formato    |Tam |Obrigatório
|--------------------|-------------------------------------|-----------|----|-----------
|**apiKey**          |Tipo de credenciais que serão geradas|String     |44  |Sim
|**documentNumber**  |Tipo de credenciais que serão geradas|String     |11  |Sim
|**terminalId**      |Tipo de credenciais que serão geradas|String     |44  |Não
|**amount**          |Tipo de credenciais que serão geradas|String     |44  |Sim
|**installment**     |Tipo de credenciais que serão geradas|String     |2   |Sim
|**metaData**        |Tipo de credenciais que serão geradas|String     |    |Não
|**metaId**          |Tipo de credenciais que serão geradas|String     |44  |Não

**Descrição**: Retorna uma página web desenvolvida na 4all para controlar todo o fluxo de geração de QR Code e confirmação do pagamento. No endpoint ‘paymentSuccess’ vai constar no querystring seguintes dados da transação: 

**Resposta**:

|Campo               |Descrição                                       |Formato    |Tam |Obrigatório
|--------------------|------------------------------------------------|-----------|----|-----------
|**message**         |Mensagem de confirmação.|String                 |String     |20  |Sim
|**brandId**         |Identificador da bandeira do cartão             |String     |2   |Sim
|**acquirerUsn**     |Identificador/NSU do adquirente para a transação|String     |20  |Sim
|**issuerAuthCode**  |Código de autorização adquirente                |String     |20  |Sim
|**transactionId**   |id da transação na 4all                         |String     |20  |Sim

**Exemplo**:
    `<endpoint>/qrcode/payment?apiKey=0123456789ABCDEF01&documentNumber=19201989000120&terminalId=145&amount=10000&installment=6&metaData=%7B%22UID%22%3A1523%7D&metaId=abc123`

### 2.1.2. Get data
**Caminho**: `<endpoint>/qrcode/getData`

**Método HTTP**: POST

**Descrição**: Retorna um base64 para ser gerado o QR Code e transactionId da 4all.Utilizado quando parceiro deseja gerar o QR Code nativo em seu sistema.

**Fluxo**: Após definido o valor do pagamento e as parcelas,deverá ser chamado                
 `<endpoint>/qrcode/getData`, será gerado um base64 que deverá ser gerado um QR Code e id de transação da 4all, e deverá ser desenvolvido um pulling com a 4all para ficar validando o status da transação.

**Requisição**:

|Campo               |Descrição                            |Formato    |Tam |Obrigatório
|--------------------|-------------------------------------|-----------|----|-----------
|**apiKey**          |Chave única do EC                    |String     |44  |Sim
|**documentNumber**  |CNPJ do estabelecimento comercial    |String     |11  |Sim
|**terminalId**      |Identificador do terminal PDV        |String     |44  |Não
|**amount**          |VAlor da transação, em centavos      |Number     |44  |Sim
|**installment**     |Número de parcelas                   |Number     |2   |Sim
|**metaData**        |Dados do pagamento no EC             |Json/String|    |Não
|**metaId**          |Usn do pagamento no EC               |String     |44  |Não

**Resposta**: Retorna um json com base 64 para gerar o QR Code e id da transação.

|Campo               |Descrição                                       |Formato    |Tam |Obrigatório
|--------------------|------------------------------------------------|-----------|----|-----------
|**qrCode**          |QR Code para gerar imagem                       |String     |    |Sim
|**transactionId**   |Id da transação na 4all                         |Int        |11  |Sim

**Exemplo**:

**Requisição**:

```json
{
  "apiKey": "0123456789ABCDEF01...",
  "documentNumber": "19201989000120",
  "terminalId": "145",
  "amount": 10000,
  "installments": 6,
  "metaData": {"UID":1523},
  "metaId": "145"
}
```

**Resposta**:

```json
{
  "qrCode": "0123456789ABCDEF01...",
  "transactionId": 3070
}
```

### 2.1.3. Get Image
**Caminho**: `<endpoint>/qrcode/getImage`

**Método HTTP**: POST

**Descrição**: Retorna um base64 para ser gerado o QR Code e transactionId da 4all.

**Fluxo**: Após definido o valor do pagamento e as parcelas,deverá ser chamado                
 `<endpoint>/qrcode/getImage`, será gerado uma imagem contendo um QR Code e id de transação da 4all, e deverá ser desenvolvido um pulling com a 4all para ficar validando o status da transação.

**Requisição**:

|Campo               |Descrição                            |Formato    |Tam |Obrigatório
|--------------------|-------------------------------------|-----------|----|-----------
|**apiKey**          |Chave única do EC                    |String     |44  |Sim
|**documentNumber**  |CNPJ do estabelecimento comercial    |String     |11  |Sim
|**terminalId**      |Identificador do terminal PDV        |String     |44  |Não
|**amount**          |VAlor da transação, em centavos      |Number     |44  |Sim
|**installment**     |Número de parcelas                   |Number     |2   |Sim
|**metaData**        |Dados do pagamento no EC             |Json/String|    |Não

**Resposta**: Retorna um json com base 64 para gerar o QR Code e id da transação.

|Campo               |Descrição                                       |Formato    |Tam |Obrigatório
|--------------------|------------------------------------------------|-----------|----|-----------
|**qrCode**          |QR Code para gerar imagem                       |String     |    |Sim
|**transactionId**   |Id da transação na 4all                         |Int        |11  |Sim

**Exemplo**:

**Requisição**:

```json
{
  "apiKey": "0123456789ABCDEF01...",
  "documentNumber": "19201989000120",
  "terminalId": "145",
  "amount": 10000,
  "installments": 6,
  "metaData": {"UID":1523},
  "metaId": "145"
}
```

**Resposta**:

```json
{
  "qrCodeImage": "endpoint/qrcode/image/1641651681",
  "transactionId": 3070
}
```

### 2.1.4. Get IFrame - Refund
**Caminho**: `<endpoint>/qrcode/refund`

**Método HTTP**: GET

**Descrição**: Retorna uma página web desenvolvida na 4all para controlar todo o fluxo de geração de QRCode e confirmação do cancelamento. Tamanho ideal do iframe 375 px x 400 px.

**Fluxo**: Deverá ser chamado `<endpoint>/qrcode/refund`, será gerado um QR Code e o iframe ficará fazendo requisições de 3 em 3 segundos para o servidor da 4all verificando se o pagamento foi cancelado se confirmado o cancelamento ele será redirecionado para `endpoint/qrcode/paymentSucess`(item 1.4.) caso haja um cancelamento mas ele não seja autorizado será redirecionado para `endpoint/qrcode/paymentFail`(item 1.5).

**Observação**:
* **Todos os parâmetros devem ser devidamente escapados para serem enviados como querystring.**

**Requisição**:

|Campo             |Descrição                                                                                            |Formato    |Tam |Obrigatório
|------------------|-----------------------------------------------------------------------------------------------------|-----------|----|-----------
|**apiKey**        |Chave única do EC                                                                                    |String     |44  |Sim
|**transcationId** |Id da transação na 4all                                                                              |String     |20  |Depende
|**merchantMetaId**|Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa |String     |44  |Depende


**Resposta**: Retorna uma página web desenvolvida na 4all para controlar todo o fluxo de geração de QR Code e confirmação do pagamento. No endpoint ‘paymentSuccess’ vai constar no querystring seguintes dados da transação:

|Campo               |Descrição                                       |Formato    |Tam |Obrigatório
|--------------------|------------------------------------------------|-----------|----|-----------
|**message**         |Mensagem de confirmação.|String                 |String     |20  |Sim
|**transactionId**   |id da transação na 4all                         |String     |20  |Sim
|**amount**          |Valor da transação, em centavos                 |String     |20  |Sim
|**merchantName**    |Nome do estabelecimento                         |String     |20  |Sim

**Exemplo**:
    `<endpoint>/qrcode/refund?apiKey=3%2FQIfSKOAK%2FR2mKFZSJNpUh0ozOuo89xThINGzrGKmE%3D&transactionId=49747`

### 2.1.5. Get data - Refund
**Caminho**: `<endpoint>/qrcode/getDataRefund`

**Método HTTP**: POST

**Descrição**: Retorna um base64 para ser gerado o QR Code e transactionId da 4all.Utilizado quando parceiro deseja gerar o QR Code nativo em seu sistema.

**Fluxo**: Deverá ser chamado `<endpoint>/qrcode/getDataRefund`, será gerado um base64 que deverá ser gerado um QR Code e id de transação da 4all, e deverá ser desenvolvido um pulling com a 4all para ficar validando o status da transação.

**Requisição**:

|Campo              |Descrição                                                                                            |Formato |Tam |Obrigatório
|-------------------|-----------------------------------------------------------------------------------------------------|--------|----|-----------
|**apiKey**         |Chave única do EC                                                                                    |String  |44  |Sim
|**transcationId**  |Id da transação na 4all                                                                              |String  |11  |Depende
|**merchantMetaId** |Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa |String  |44  |Depende

**Resposta**: Retorna um json com base 64 para gerar o QR Code e id da transação.

|Campo               |Descrição                                       |Formato    |Tam |Obrigatório
|--------------------|------------------------------------------------|-----------|----|-----------
|**qrCode**          |QR Code para gerar imagem                       |String     |    |Sim
|**transactionId**   |Id da transação na 4all                         |Int        |11  |Sim

**Exemplo**:

**Requisição**:

```json
{
  "apiKey": "0123456789ABCDEF01...",
  "transactionId": "49747",
}

ou

{
  "apiKey": "0123456789ABCDEF01...",
  "merchantMetaId": "1259",
}

```

**Resposta**:

```json
{
  "qrCode": "0123456789ABCDEF01...",
  "transactionId": 49747
}
```

### 2.1.6. Get Image
**Caminho**: `<endpoint>/qrcode/getImageRefund`

**Método HTTP**: POST

**Descrição**: Retorna um base64 para ser gerado o QR Code e transactionId da 4all.

**Fluxo**: Deverá ser chamado `<endpoint>/qrcode/getImageRefund`, será gerado uma imagem contendo um QR Code e id de transação da 4all, e deverá ser desenvolvido um pulling com a 4all para ficar validando o status da transação.

**Requisição**:

|Campo              |Descrição                                                                                            |Formato |Tam |Obrigatório
|-------------------|-----------------------------------------------------------------------------------------------------|--------|----|-----------
|**apiKey**         |Chave única do EC                                                                                    |String  |44  |Sim
|**transcationId**  |Id da transação na 4all                                                                              |String  |11  |Depende
|**merchantMetaId** |Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa |String  |44  |Depende

**Resposta**: Retorna um json com base 64 para gerar o QR Code e id da transação.

|Campo               |Descrição                                       |Formato    |Tam |Obrigatório
|--------------------|------------------------------------------------|-----------|----|-----------
|**qrCode**          |QR Code para gerar imagem                       |String     |    |Sim
|**transactionId**   |Id da transação na 4all                         |Int        |11  |Sim

**Exemplo**:

**Requisição**:

```json
{
  "apiKey": "0123456789ABCDEF01...",
  "transactionId": "49747",
}

ou

{
  "apiKey": "0123456789ABCDEF01...",
  "merchantMetaId": "1259",
}

```

**Resposta**:

```json
{
  "qrCodeImage": "endpoint/qrcode/image/1641651681",
  "transactionId": 49747
}
```

# 3. Consulta de transações

## 3.1. Get Transaction Details

**Caminho homolog**: `https://conta.homolog-interna.4all.com/merchant/getTransactionDetails`

**Caminho produção**: `https://conta.api.4all.com/merchant/getTransactionDetails`

**Método HTTP**: POST

**Descrição**: Retorna os detalhes de uma transação.

**Fluxo**: Está chamada pode ser utilizada após realizar o fluxo 2.1.2 ou 2.1.3.

**Requisição**:

|Campo              |Descrição                                                                                            |Formato |Tam |Obrigatório
|-------------------|-----------------------------------------------------------------------------------------------------|--------|----|-----------
|**merchantKey**    |Chave de acesso do merchant à API                                                                    |String  |44  |Sim
|**transactionId**  |Identificador da transação                                                                           |String  |11  |Depende
|**merchantMetaID** |Identificador único, atribuído pelo estabelecimento comercial, que será usado como chave na pesquisa |String  |44  |Depende

**Resposta**:
Retorna um json com imagem e id da transação.

|Campo                 |Descrição                                                                                                                                  |Formato |Tam   |Obrigatório
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------|--------|------|-----------
|**transactionId**     |Chave de acesso do merchant à API                                                                                                          |String  |      |Sim
|**subscriptionId**    |Quando presente, informa o identificador da assinatura que gerou esta transação                                                            |String  |20    |Depende
|**amount**            |Valor da transação, em centavos                                                                                                            |Number  |      |Sim
|**fullAmount**        |Valor original da transação antes de aplicar o desconto, em centavos (Presente se foi aplicado alguma forma de desconto)                   |Number  |      |Depende
|**installments**      |Número de parcelas                                                                                                                         |Number  |      |Sim
|**InstallmentType**   |Tipo de parcelamento (1- à vista, 2 - parcelado lojista, 3 - parcelado emissor)                                                            |Number  |      |Sim
|**status**            |Estado da transação (conforme anexo A)                                                                                                     |Number  |      |Sim
|**reasonMessage**     |Mensagem de autorização/recusa da transação (Presente se disponivel)                                                                       |String  |1-1024|Depende
|**createdAt**         |Data e hora UTC em que a transação foi criada no servidor 4all. Formato:YYYY-MM-DDThh:mm:ssZ                                               |String  |20    |Sim
|**paidAt**            |Data e hora UTC em que a transação foi paga por uma conta 4all (formato YYYY-MM-DDThh:mm:ssZ). Presente somente se a transação já foi paga |String  |20    |Depende
|**authorizationInfo** |Objeto contendo detalhes de autorização da transação. Presente somente se a transação já foi paga (ver abaixo)                             |Object  |*     |Depende
|**transactionToken**  |Chave para consulta da transação pela chamada “Wait For Transaction”                                                                       |String  |44    |Sim
|**loyalty**           |Dados do sistema de fidelidade (Presente se foi aplicado alguma forma de desconto)                                                         |Object  |*     |Depende
|**receipt**           |Recibo do comprovante de pagamento / estorno. Retorna apenas se na requisição o indicador receipt for igual a true e se a transação encontra-se nos estados 3 - Pago, 5 - Estornado/Cancelado, e 20 - Autorizado |String  |999  |Depende

**AuthorizationInfo**:

|Campo                 |Descrição                                                                                            |Formato |Tam |Obrigatório
|----------------------|-----------------------------------------------------------------------------------------------------|--------|----|-----------
|**acquirerId**        |Identificador do adquirente (Tabela D)                                                               |Number  |    |Sim
|**acquirerUsn**       |Identificador/NSU do adquirente para a transação                                                     |String  |50  |Sim
|**acquirerTimestamp** |Timestamp de pagamento conforme reportado pelo adquirente. O formato e timezone deste campo segue o formato usado pelo adquirente. Presente somente se o adquirente informar |String  |50  |Depende
|**brandId**           |Identificador da bandeira do cartão que foi usado no pagamento (Tabela C)                            |        |    |Sim

**Loyalty**:

|Campo             |Descrição                                                               |Formato|Tam  |Obrigatório
|------------------|------------------------------------------------------------------------|-------|-----|-----------
|**programId**     |Identificador do adquirente                                             |Number |     |Sim
|**campaignUUID**  |Identificador único da campanha                                         |String |1-64 |Sim
|**couponUUID**    |Identificador único do cupom                                            |String |1-64 |Sim
|**code**          |Código do cupom                                                         |String |1-64 |Sim
|**discountType**  |Tipo do desconto, sendo percentual ('PERCENT’) ou valor fixo ('AMOUNT’) |String |1-64 |Sim
|**discountValue** |Valor do desconto aplicado                                              |String |1-64 |Sim
|**message**       |Mensagem descritiva do cupom                                            |String |1024 |Sim
|**redeemAt**      |Data e hora em que o cupom foi utilizado (YYYY-MM-DDThh:mm:ssZ)         |String |20   |Sim

**Observação**:
* **Apenas um dos dois parâmetros, `transactionId` ou `merchantMetaId`, pode ser usado; se ambos estiverem presentes, esta chamada falha com um erro específico.**

**Exemplo**:

**Requisição**:

```json
{
  "apiKey": "0123456789ABCDEF01...",
  "documentNumber": "19201989000120",
  "terminalId": "145",
  "amount": 10000,
  "installments": 6,
  "metaData": {"UID":1523},
  "metaId": "145"
}
```

**Resposta**:

```json
{
  "qrCodeImage": "endpoint/qrcode/image/1641651681",
  "transactionId": 3070
}
```

## 3.2. Wait For Transaction

**Caminho homolog**: `<endpoint>/merchant/waitForTransaction`

**Caminho produção**: `<endpoint>/merchant/waitForTransaction`

**Método HTTP**: POST

**Descrição**: Esta rotina não retorna enquanto a transação estiver com status pendente de pagamento (estadis 0, 1 ou 2 - vide Anexo B tabela B.2). O chamador desta rotina deve estar preparado para lidar com tempos de resposta longos, de 30 segundos ou mais. O objetivo desta chamada é reduzir o overhead de múltiplas conexões fazendo polling enquanto aguardam uma transação ser paga.

**Fluxo**: Deverá ser chamado `<endpoint>/merchant/waitForTransaction` e deverá ser retornado o status se a transação foi paga ou não.

**Requisição**:

|Campo                  |Descrição                                                                                             |Formato |Tam |Obrigatório
|-----------------------|------------------------------------------------------------------------------------------------------|--------|----|-----------
|**merchantKey**        |Chave de acesso do merchant à API                                                                     |String  |44  |Depende
|**transactionId**      |Identificador da transação                                                                            |String  |20  |Depende
|**transactionToken**   |Chave de identificação da transação, retornado pela chamada createTransaction                         |String  |44  |Depende
|**ignoreRefusedStatus**|Indicador para continuar a espera e se o pagamento tiver sido negado. Por padrão, é considerado false |Boolean |*   |Não

**Observação**:
* **Esta chamada pode ser executada usando o `merchantKey` e o `transactionId` (ambos são necessários), e neste caso o campo `transactionToken` não deve ser usado.**
* **Quando `transactionToken` for usado, o `merchantKey` e o `transactionId` não devem ser informados.**


**Resposta**:
Retorna um json com imagem e id da transação.

|Campo      |Descrição           |Formato |Tam |Obrigatório
|-----------|--------------------|--------|----|-----------
|**status** |Status da transação |Number  |*   |Sempre

**Exemplo**:

**Requisição**:

```json
{
 "transactionToken": "NGFsbDRhbGw0YWxsNGFs...",
  "ignoreRefusedStatus": false
}
```

**Resposta**:

```json
{
  "status": 3
}
```

# Anexo - Tabelas de Domínio

## Tabela A - Estados de uma Transação

|Estado  |Descrição                                                               
|--------|-------------------------------------------------------------------
| 0      |Transação em processo de autorização
| 1      |Transação recusada
| 2      |Transação autorizada
| 3      |Transação em processo de captura
| 4      |Transação capturada (paga)
| 5      |Transação em processo de cancelamento
| 6      |Transação cancelada
| 7      |Transação paga - cancelamento falhou
| 8      |Transação contestada pelo portador do cartão (chargeback)
| 9      |Transação paga - reapresentada após contestação (chargeback refund)
| 10     |Transação falhou
| 11     |Transação com falha no processo de cancelamento                   

## Tabela B - Forma de Pagamento

|paymentMode  |Modalidade do pagamento
|-------------|-----------------------
| 1           |Crédito 
| 2           |Débito

## Tabela C - Bandeiras de Cartão

|brandId |Nome da Bandeira                                                               
|--------|----------------
| 0      |Desconhecido
| 1      |Visa
| 2      |Mastercard
| 3      |Diners
| 4      |Elo
| 5      |Amex
| 6      |Discover
| 7      |Aura
| 8      |Jcb
| 9      |Hipercard
| 10     |Maestro

## Tabela D - Adquirentes

|acquirerId |Nome     |Observação
|-----------|---------|---------------
|1          |Stone    |Somente Crédito
|2          |Pagar me |Somente Crédito
|3          |GetNet   |Somente Crédito
|4          |Cielo    |Somente Crédito
|5          |Rede     |Somente Crédito
|6          |Cielo v3 |Somente Crédito
