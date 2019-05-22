FORMAT: 1A
HOST: http://polls.apiblueprint.org/
https://app.apiary.io/rapiddesigner1/editor
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

Tendo o ID do Rapid Designer salvo no seu produto, você pode colocar um botão "Customizar" na página de compra, que deve redirecionar para uma outra página dentro da sua loja que contenha um iframe

## 3 - Redirecionando para o Carrinho

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
