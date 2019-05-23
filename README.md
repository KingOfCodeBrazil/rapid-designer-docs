# Rapid Designer - API e Integração v1

Utilizando a API de integração do Rapid Designer, é possível integrá-lo à sites e plataformas de e-commerce, possibilitando a customização de produtos e a visualização dos mesmos em 3D.

O Rapid Designer foi projetado para funcionar inteiramente através da plataforma **https://designer.rapidmockup.net**, 
portanto a API no momento cobre apenas os elementos principais necessários para a comunicação e bom uso dentro do site
ou loja virtual.

Caso precise de algum recurso específico na plataforma para a sua integração, por gentileza entre em contato pelo
e-mail **contato@kingofcode.com.br**

## Autenticação

O cliente da plataforma pode acessar a área de autorização para criar um token de acesso. O token deve ser enviado na
requisição através do parâmetro **api_token**. Por exemplo:

```
designer.rapidmockup.net/api/orders?api_token=seu-token-aqui
```

É recomendável a criação de uma área onde esse token possa ser informado e guardado no banco de dados do seu site, para que
seja possível ao cliente fazer requisições futuras à plataforma do Rapid Designer.

## Endereço da API

O endereço atual da API é:

```
designer.rapidmockup.net/api/
```

## 1 - Obtendo Produtos

O primeiro passo para a integração é ter uma referência do produto cadastrado dentro da plataforma do Rapid Designer
salva no produto do seu site/loja. 

Por exemplo, se você tem o produto **Caneca de Porcelana Customizável** na loja, pode colocar
uma "Caixa de Seleção" na tela de cadastro desse produto onde seleciona à qual produto do Rapid Designer ele está relacionado,
salvando o ID retornado pela api no seu banco de dados.

Um mesmo produto do Rapid Designer pode estar relacionado à vários produtos na loja. Por exemplo, é possível cadastrar uma Caneca Personalizável
com arte total, e utilizá-la tanto para a caneca de porcelana quanto para a de plástico, etc.

### Listar Produtos [GET /products]

Retorna todos os produtos associados ao **api_token** informado.

+ Response 200 (application/json)

        [
            {
                "products": [
                    {
                        "id": "the-product-uuid",
                        "name": "The product name",
                        "type": "product-type"
                    }
                ]
            }
        ]



## 2 - Redirecionando ao Designer

Tendo o ID do Rapid Designer salvo no seu produto, você pode colocar um botão "Customizar" na página de compra, que deve redirecionar para uma outra página dentro da sua loja que contenha um iframe.

```html
<div id="rapidmockup-block">
    <iframe id="rapidmockup-iframe" frameborder="0"></iframe>
</div>
```

A URL do iframe deve ter o seguinte formato:

```
https://designer.rapidmockup.net/designer
```

E deverá ter os seguintes parâmetros enviados:

- api_token - Obrigatório - Veja a seção de autenticação
- currency - Opcional - Apenas se você quiser especificar como a moeda será mostrada, ex: R\$, $, etc
- remote_id - Obrigatório - O ID do produto **na sua loja**, utilizado para fins de identificação se necessário
- remote_type - Obrigatório - Pode enviar aqui a url do seu site/plataforma. É utilizado para identificar o tipo de integração que gerou o design.
- rapid_product - Obrigatório - Aqui você enviar o ID do produto do Rapid Designer associado ao produto da sua loja, que foi obtido previamente
- designer_id - Opcional - Enviado apenas em **MODO DE EDIÇÃO** para identificar qual design está sendo editado.

A url resultante será parecida com essa abaixo, preenchidos seus devidos valores.

```
https://designer.rapidmockup.net/designer?api_token=&currency=&product_name=&product_price=&remote_type=woocommerce&remote_id=&rapid_product=&rapid_quantity=&design_id=
```

Recomendamos adicionar o seguinte código CSS na página para que o iframe comporte-se corretamente em todos os dispositivos, incluindo mobile:

```css
#rapidmockup-block {
    width: 100%;
    height: 100vh;
}

#rapidmockup-iframe {
    border: 0;
    overflow: hidden;
    width: 100%;
    height: 100%;
}

@media (max-width: 992px) {
    * {
        visibility: hidden;
    }

    #rapidmockup-block {
        visibility: visible;
    }

    #rapidmockup-iframe {
        z-index: 9999999999;
        visibility: visible;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
        position: fixed;
    }
}
```

## 3 - Redirecionando para o Carrinho

Quando o usuário do Designer clica em "Adicionar", o desenho é salvo na plataforma e em seguida é emitida uma mensagem para a página. Você pode utilizar o seguinte código javascript na página do Designer para obter a mensagem:

```js
window.addEventListener('message', function (event) {
    let enabledEvents = [
        'designerImagesSaved',
        'designerImagesUpdated'
    ];

    // SEGURANÇA - Nunca remova essas duas linhas. Elas garantem
    // que apenas a plataforma do Rapid Designer envie mensagens
    // para a página do designer
    if(event.origin != 'https://designer.rapidmockup.net') return;
    if(!enabledEvents.includes(event.data.type)) return;

    if(event.data.type == 'designerImagesSaved') {
        // Se chegou aqui as imagens foram salvas e você pode obter
        // o ID do design e realizar as ações necessárias, por exemplo,
        // redirecionar para o carrinho de compras ou fazer uma request 
        // AJAX
        // Ex: goToCart(event.data.design);
    }

    if(event.data.type == 'designerImagesUpdated') {
        // Essa mensagem é emitida sempre que um desenho editado
        // é salva pelo usuário clicando em "Salvar Desenho". Você
        // pode aqui fazer um redirect para o carrinho ou qualquer
        // outra ação. Também retorna o id do design em 
        // "event.data.design". No entanto o id do design não muda nunca, 
        // mesmo após a edição.

    }

}, false);
```

Recomendamos que o ID do design, retornado via "event.data.design", seja guardado no item do carrinho, na Session ou qualquer outro meio que esteja utilizando para guardá-lo. 

Dessa forma é possível utilizar este ID para editar o design se o usuário desejar. 

Também é altamente recomendável guardar esta informação no banco de dados, pois a mesma deverá ser enviada na finalização do pedido, e também poderá ser utilizada futuramente caso seja necessário refazer o envio do pedido por falha de comunicação ou algo parecido.

## 4 - Obtendo a Thumbnail do design que foi desenhado

É provável que queira mostrar uma amostra do desenho que o usuário fez no carrinho ou na página de compra da loja. Para isso, você pode fazer uma requisição para a seguinte URL (ou utilizá-la diretamente na tag img).

```
https://designer.rapidmockup.net/designs/id-do-seu-design?type=thumb
```

Exemplo direto na tag img

```html
<img src="https://designer.rapidmockup.net/designs/id-do-seu-design?type=thumb">
```

## 5 - Modo de Edição

É altamente recomendável que seja adicionado um botão ou link "Editar Desenho" ou algo similar nos itens do carrinho ou na página de compra. Dessa forma o usuário pode retornar e editar o desenho antes de finalizar a compra.

Para acessar o modo de edição, é bastante simples. Basta passar o parâmetro **design_id** on iframe do designer (veja mais detalhes na seção 2). 

Recomendamos inclusive reaproveitar a mesma página que foi criada para o Designer para a edição, apenas passando o design_id quando necessário. 

## 6 - Finalizando o Pedido [POST /orders]

Após a finalização/pagamento do pedido, é necessário enviar alguns dados do mesmo para a plataforma. 

Apenas após o envio desses dados as imagens ficam disponíveis na dashboard do Rapid Designer para download.

O corpo da requisição deve conter os seguintes parâmetros obrigatórios:

- remote_id - Obrigatório - ID do Pedido no seu banco de dados
- remote_type - Obrigatório - Pode ser informado a url da sua loja/plataforma
- currency - Obrigatório - Código da moeda utilizada na finalização do pedido, de acordo com a [ISO 4217](https://pt.wikipedia.org/wiki/ISO_4217). Ex: BRL, EUR, USD
- price - obrigatório - Sub-Total do pedido em centavos (não é necessário informar o valor total incluindo frete, taxas, etc. Apenas o valor da compra dos produtos, incluindo descontos),

- items - Obrigatório - Array de itens do pedido, sendo obrigatório informar os próximos três parâmetros para cada item
- items.\*.design_id - Obrigatório - ID do design salvo previamente no item
- items.\*.description - Obrigatório - Pode ser enviado o nome do produto, facilitando a identificação na Dashboard,
- items.\*.quantity - Obrigatório - Quantidade de produtos no item do pedido,

E também pode conter os seguintes parâmetros opcionais:

- address - opcional - Array com dados do endereço
- address.address_1 - Obrigatório apenas se **address** for enviado - Primeira linha do endereço
- address.address_2 - Opcional - Segunda linha do endereço
- address.city - Obrigatório apenas se **address** for enviado - Cidade do endereço
- address.country - Obrigatório apenas se **address** for enviado - País do endereço

Exemplo:

+ Request (application/json)

        {
            "remote_id": 123,
            "remote_type": "meusite.com",
            "currency": "BRL",
            "price": 1500,
            "items": [
                {
                    "design_id": "id-do-design",
                    "description": "caneca-de-porcelana"
                    "quantity": 1
                }
            ]
        }

Havendo sucesso em salvar o pedido, será retornado:

+ Response 200 (application/json)

    + Body

            {
                "created": true,
                "order": {
                    "id": "id-do-pedido"
                }
            }

Recomendamos que o ID do pedido retornado seja guardado no banco de dados, relacionado ao pedido da loja/site, facilitando o reenvio ou suporte caso necessário, além de possibilitar gerar um link direto para a Dashboard de Pedidos do Rapid Designer.

## 7 - Link do Pedido

Geralmente, após o pedido, será enviado um e-mail para o usuário reponsável pela loja cadastrado na plataforma. No entanto, é possível gerar um link direto para o Pedido no seguinte formato:

```
https://designer.rapidmockup.net/orders/id-do-pedido
```

Na posse deste link, estando devidamente autenticado na Dashboard, é possível visualizar os detalhes do pedido.