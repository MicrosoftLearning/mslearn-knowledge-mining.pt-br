---
lab:
  title: Use a API REST para executar consultas de busca em vetores
---

# Use a API REST para executar consultas de busca em vetores

Neste exercício, você configurará seu projeto, criará um índice, carregará seus documentos e executará consultas.

Você precisará dos seguintes itens para realizar este exercício com êxito:

- O aplicativo [Postman](https://www.postman.com/downloads/)
- Uma assinatura do Azure
- Serviço da Pesquisa de IA do Azure
- A coleção de amostras do Postman localizada neste repositório - *Vector-Search-Quickstart.postman_collection v1.0 json*.

> **Observação**: se necessário, você poderá obter mais informações sobre o aplicativo Postman [aqui](https://learn.microsoft.com/en-us/azure/search/search-get-started-rest).

## Configurar o seu projeto

Primeiro, configure seu projeto executando as etapas a seguir:

1. Observe a **URL** e a **Chave** do seu serviço Pesquisa de IA do Azure.

    ![Ilustração do local para o nome e as chaves do serviço.](../media/vector-search/search keys.png)

1. Faça o download da coleção de amostras [Postman](https://github.com/MicrosoftLearning/mslearn-knowledge-mining/blob/main/Labfiles/10-vector-search/Vector%20Search.postman_collection%20v1.0.json).
1. Abra o Postman e importe a coleção selecionando o botão **Importar** e arraste e solte a pasta da coleção na caixa.

    ![Imagem da caixa de diálogo Importar](../media/vector-search/import.png)

1. Selecione o botão **Bifurcação** para criar uma bifurcação da coleção e adicionar um nome exclusivo.
1. Clique com o botão direito do mouse no nome da coleção e selecione **Editar**.
1. Selecione a guia **Variáveis** e insira os seguintes valores usando o serviço de pesquisa e os nomes de índice do seu serviço Pesquisa de IA do Azure:

    ![Diagrama mostrando um exemplo de configurações variáveis](../media/vector-search/variables.png)

1. Salve suas alterações selecionando o botão **Salvar**.

Você está pronto para enviar suas solicitações para o serviço Pesquisa de IA do Azure.

## Criar um índice

Em seguida, crie seu índice no Postman:

1. Selecione **PUT Criar/Atualizar Índice** no menu lateral.
1. Atualize a URL com **search-service-name**, **index-name** e **api-version** que você anotou anteriormente.
1. Selecione a guia **Corpo** para ver a resposta.
1. Defina **index-name** com o valor do nome do índice da sua URL e selecione **Enviar**.

Você deverá ver um código de status do tipo **200**, que indica uma solicitação bem-sucedida.

## Carregar documentos

Há 108 documentos incluídos na solicitação Carregar Documentos, cada um com um conjunto completo de inserções para os campos **titleVector** e **contentVector**.

1. Selecione **POST Carregar Documentos** no menu lateral.
1. Atualize a URL com **search-service-name**, **index-name** e **api-version** como antes.
1. Selecione a guia **Corpo** para ver a resposta e selecione **Enviar**.

Você deverá ver um código de status do tipo **200** para mostrar que sua solicitação foi bem-sucedida.

## Executar consultas

1. Agora, tente executar as seguintes consultas no menu lateral. Para fazer isso, certifique-se de atualizar a URL sempre como antes e envie uma solicitação selecionando **Enviar**:

    - Busca em Vetores única
    - Busca em Vetores única com filtro
    - Pesquisa híbrida simples
    - Pesquisa híbrida simples com filtro
    - Pesquisa entre campos
    - Pesquisa com várias consultas

1. Selecione a guia **Corpo** para ver a resposta e exibir os resultados.

Você deverá ver um código de status do tipo **200** para uma solicitação bem-sucedida.
