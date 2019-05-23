# Rapid Designer

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

A URL do iframe deve ser deve seguir o seguinte formato:

```
https://designer.rapidmockup.net/designer
```

E deverá ter os seguintes parâmetros enviados:
'api_token' => 'required|string',
            'remote_id' => 'required|string',
            'remote_type' => 'required|string',
            'rapid_product' => 'required',
            'designer_id' => 'sometimes|exists:designs,id'
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
        // "event.data.design". No entanto o id do design não muda, 
        // mesmo após a edição.

    }

}, false);
```

## 4 - Obtendo a Thumbnail do design que foi desenhado

## 5 - Modo de Edição

## 6 - Obtendo resposta 

## Questions Collection [/questions]

### List All Questions [GET]

+ Response 200 (application/json)

        [
            {
                "question": "Favourite programming language?",
                "published_at": "2015-08-05T08:40:51.620Z",
                "choices": [
                    {
                        "choice": "Swift",
                        "votes": 2048
                    }, {
                        "choice": "Python",
                        "votes": 1024
                    }, {
                        "choice": "Objective-C",
                        "votes": 512
                    }, {
                        "choice": "Ruby",
                        "votes": 256
                    }
                ]
            }
        ]

### Create a New Question [POST]

You may create your own question using this action. It takes a JSON
object containing a question and a collection of answers in the
form of choices.

+ Request (application/json)

        {
            "question": "Favourite programming language?",
            "choices": [
                "Swift",
                "Python",
                "Objective-C",
                "Ruby"
            ]
        }

+ Response 201 (application/json)

    + Headers

            Location: /questions/2

    + Body

            {
                "question": "Favourite programming language?",
                "published_at": "2015-08-05T08:40:51.620Z",
                "choices": [
                    {
                        "choice": "Swift",
                        "votes": 0
                    }, {
                        "choice": "Python",
                        "votes": 0
                    }, {
                        "choice": "Objective-C",
                        "votes": 0
                    }, {
                        "choice": "Ruby",
                        "votes": 0
                    }
                ]
            }
