---
title: 4 Pay - Digital Commerce

language_tabs:
  - HTML
  - Javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

**4 Pay - Digital Commerce**

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

Clique no botão “Gerar Chaves” para obter um par de chaves, a Pública e a Privada.

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
data-public-api-key| Chave de API pública do Checkout 4all.|String|Sim
data-amount|Valor da transação em centavos. Ex: "1425" para R$ 14,25.|String|Sim

Se o cliente efetuar o pagamento, um novo campo `<input type="hidden" id="payment_token">` contendo o **payment_token** é adicionado ao seu formulário, e o submit é efetuado.

## 3.2 Biblioteca Javascript

```HTML
<script src="https://lib.4all.com/lib/checkout-lib.js"></script>
```
Para fazer a integração via javascript, você deve primeiramente importar o arquivo da biblioteca na sua página, adicionando a seguinte linha ao seu HTML:

```Javascript
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
amount|Valor da transação em centavos. Ex: "1425" para R$ 14,25.|String|Sim
publicApiKey|Chave de API pública do Checkout 4all.|String|Sim
successCallback | Função que será chamada quando o checkout estiver finalizado com sucesso. Recebe o paymentToken como parâmetro. | Função | Sim
cancelCallback | Função que será chamada caso o usuário cancele o processo de pagamento sem concluí-lo. | Função | Não


# 4 Capturando a transação

Com o **payment_token** em mãos, você pode capturar a transação (efetuar a cobrança) através de uma chamada à API Conta 4all.

**Nota:** por motivos de segurança, você deve informar o valor total da transação novamente nesta chamada.

Exemplo:
`
curl -X POST "https://api.pagar.me/1/transactions/{TOKEN}/capture"
  -d 'amount=1000'
  -d 'api_key=ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0'
`
#4.1 Consultando uma transação
Em caso de erro na chamada de captura de transação, você pode consultar o estado da transação através do Meta ID passado na chamada de Captura.

#5 Ambiente de Homologação
