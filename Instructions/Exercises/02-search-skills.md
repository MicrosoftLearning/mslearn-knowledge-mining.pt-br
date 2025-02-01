---
lab:
  title: Criar uma habilidade personalizada para a Pesquisa de IA do Azure
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Criar uma habilidade personalizada para a Pesquisa de IA do Azure

A Pesquisa de IA do Azure usa um pipeline de enriquecimento de habilidades de IA para extrair campos gerados por IA de documentos e incluí-los em um índice de pesquisa. Há um conjunto abrangente de habilidades embutidas que você pode usar, mas se você tiver um requisito específico que não seja atendido por essas habilidades, poderá criar uma habilidade personalizada.

Neste exercício, você criará uma habilidade personalizada que tabula a frequência de palavras individuais em um documento para gerar uma lista das cinco palavras mais usadas e adicioná-la a uma solução de pesquisa para Margie's Travel – uma agência de viagens fictícia.

## Preparar para desenvolver um aplicativo no Visual Studio Code

Você desenvolverá seu aplicativo de pesquisa usando o Visual Studio Code. Os arquivos de código para seu aplicativo foram fornecidos em um repositório do GitHub.

> **Dica**: Se você já clonou o repositório **mslearn-knowledge-mining**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
1. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` em uma pasta local (não importa qual pasta).
1. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
1. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar os ativos necessários para compilar e depurar, selecione **Agora não**.

## Criar recursos do Azure

> **Observação**: se você tiver concluído anteriormente o exercício **[Criar uma solução de Pesquisa de IA do Azure](01-azure-search.md)** e ainda tiver esses recursos do Azure em sua assinatura, poderá ignorar esta seção e começar na seção **Criar uma solução de pesquisa**. Caso contrário, siga as etapas abaixo para provisionar os recursos necessários do Azure:

1. Em um navegador da web, abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
2. Na barra de pesquisa superior, pesquise *Serviços de IA do Azure*, clique em **Conta multisserviço dos Serviços de IA do Azure** e crie um recurso de conta multisserviço dos serviços de IA do Azure com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se você estiver usando uma assinatura restrita, talvez não tenha permissão para criar um novo grupo de recursos; use o que foi fornecido)*
    - **Região**: *Escolha entre regiões disponíveis geograficamente próximas a você*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: Standard S0
1. Depois de implantado, vá para o recurso e, na página de **visão geral**, anote a **ID da assinatura** e o **Local**. Você precisará desses valores, juntamente com o nome do grupo de recursos nas etapas subsequentes. 
1. No Visual Studio Code, expanda a pasta **Labfiles/02-search-skill** e selecione **setup.cmd**. Você usará esse script em lote para executar os comandos da CLI (interface de linha de comando) do Azure necessários para criar os recursos do Azure necessários.
1. Clique com o botão direito do mouse na pasta **02-search-skill** e selecione **Abrir no terminal integrado**.
1. No painel do terminal, digite o comando a seguir para estabelecer uma conexão autenticada com a sua assinatura do Azure.

    ```powershell
    az login --output none
    ```

8. Quando solicitado, selecione ou entre em sua assinatura do Azure. Em seguida, retorne para o Visual Studio Code e aguarde a conclusão do processo de entrada.
9. Use o comando a seguir para listar os locais do Azure.

    ```powershell
    az account list-locations -o table
    ```

10. No resultado, localize o valor **Name** que corresponde ao local do seu grupo de recursos (por exemplo, para *Leste dos EUA* o nome correspondente é *eastus*).
11. No script **setup.cmd**, modifique as declarações de variáveis de ** subscription_id**, **resource_group** e **local** com os valores apropriados para sua ID de assinatura, nome do grupo de recursos e nome do local. Em seguida, salve as alterações.
12. No terminal da pasta **02-search-skill**, insira o seguinte comando para executar o script:

    ```powershell
    ./setup
    ```

    > **Observação**: Se o script falhar, certifique-se de salvá-lo com os nomes de variáveis corretos e tente novamente.

13. Quando o script for concluído, examine a saída exibida e observe as seguintes informações sobre os recursos do Azure (você precisará desses valores mais tarde):
    - Nome da conta de armazenamento
    - Cadeia de conexão de armazenamento
    - Ponto de extremidade de serviço Pesquisa
    - Chave de administração do serviço Pesquisa
    - Chave de consulta do serviço de pesquisa

14. No portal do Azure, atualize o grupo de recursos que foi criado pelo script de instalação e verifique se ele contém a conta do Armazenamento do Azure, o recurso dos Serviços de IA do Azure e o recurso da Pesquisa de IA do Azure.

## Criar uma solução de pesquisa

Agora que você tem os recursos necessários do Azure, você pode criar uma solução de pesquisa que consiste nos seguintes componentes:

- Uma **fonte de dados** que faz referência aos documentos em seu contêiner de armazenamento do Azure.
- Um **conjunto de habilidades** que define um pipeline de enriquecimento de habilidades para extrair campos gerados por IA dos documentos.
- Um **índice** que define um conjunto pesquisável de registros de documentos.
- Um **indexador** que extrai os documentos da fonte de dados, aplica o conjunto de habilidades e preenche o índice.

Neste exercício, você usará a interface REST da Pesquisa de IA do Azure para criar esses componentes enviando solicitações JSON.

1. No Visual Studio Code, na pasta **02-search-skill**, expanda a pasta **create-search** e selecione **data_source.json**. Esse arquivo contém uma definição JSON para uma fonte de dados chamada **margies-custom-data**.
2. Substitua o espaço reservado **YOUR_CONNECTION_STRING** pela cadeia de conexão para sua conta de armazenamento do Azure, que deve ser semelhante à seguinte:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Você pode encontrar a cadeia de conexão na página **Chaves de acesso** de sua conta de armazenamento no portal do Azure.*

3. Salve e feche o arquivo JSON atualizado.
4. Na pasta **create-search**, abra **skillset.json**. Esse arquivo contém uma definição JSON para um conjunto de habilidades chamado **margies-custom-skillset**.
5. Na parte superior da definição do conjunto de habilidades, no elemento **cognitiveServices**, substitua o espaço reservado **YOUR_AI_SERVICES_KEY** por uma das chaves para seus recursos dos Serviços de IA do Azure.

    *Encontre as chaves na página de **Chave e ponto de extremidade** do recurso de Serviços de IA do Azure no portal do Azure.*

6. Salve e feche o arquivo JSON atualizado.
7. Na pasta **create-search**, abra **index.json**. Esse arquivo contém uma definição JSON para um índice chamado **margies-custom-index**.
8. Revise o JSON para o índice e, em seguida, feche o arquivo sem fazer alterações.
9. Na pasta **create-search**, abra **indexer.json**. Esse arquivo contém uma definição JSON para um indexador chamado **margies-custom-indexer**.
10. Revise o JSON do indexador e feche o arquivo sem fazer alterações.
11. Na pasta **create-search**, abra **create-search.cmd**. Esse script em lote usa o utilitário cURL para enviar as definições JSON para a interface REST para seu recurso de Pesquisa de IA do Azure.
12. Substitua os espaços reservados das variáveis **YOUR_SEARCH_URL** e **YOUR_ADMIN_KEY** pela **URL** e uma das **chaves de administrador** do recurso de Pesquisa de IA do Azure.

    *Você poderá encontrar esses valores na página **Visão geral** e **Chaves** do seu recurso de Pesquisa de IA do Azure no portal do Azure.*

13. Salve o arquivo em lotes atualizado.
14. Clique com o botão direito do mouse no nome da pasta **create-search** e selecione **Abrir no terminal integrado**.
15. No painel do terminal da pasta **create-index**, digite o seguinte comando para executar o script em lote.

    ```powershell
    ./create-search
    ```

16. Quando o script for concluído, no portal do Azure, na página do recurso de Pesquisa de IA do Azure, selecione a página **Indexadores** e aguarde a conclusão do processo de indexação.

    *Você pode selecionar **Atualizar** para acompanhar o progresso da operação de indexação. Pode levar cerca de um minuto para ser concluído.*

## Pesquisar o índice

Agora que você tem um índice, você pode pesquisá-lo.

1. Na parte superior do painel do seu recurso de Pesquisa de IA do Azure, selecione **Gerenciador de pesquisa**.
2. No Gerenciador de pesquisa, na caixa **Cadeia de consulta**, digite a seguinte cadeia de consulta e selecione **Pesquisar**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    Essa consulta recupera a **url**, o ** sentimento** e as **frases-chave** de todos os documentos que mencionam *Londres* de autoria do *Revisor* que têm uma etiqueta de **sentimento** positivo (em outras palavras, comentários positivos que mencionam Londres)

## Criar uma função do Azure para uma habilidade personalizada

A solução de pesquisa inclui uma série de habilidades de IA integradas que enriquecem o índice com informações dos documentos, como as pontuações de sentimento e listas de frases-chave vistas na tarefa anterior.

Você pode melhorar ainda mais o índice criando habilidades personalizadas. Por exemplo, pode ser útil identificar as palavras que são usadas com mais frequência em cada documento, mas nenhuma habilidade interna oferece essa funcionalidade.

Para implementar a funcionalidade de contagem de palavras como habilidade personalizada, você criará uma Função do Azure na linguagem de sua preferência.

> **Observação**: neste exercício, você criará uma função Node.JS simples usando os recursos de edição de código no portal do Azure. Em uma solução de produção, você normalmente usaria um ambiente de desenvolvimento, como o Visual Studio Code, para criar um aplicativo de função em sua linguagem preferida (por exemplo, C#, Python, Node.JS ou Java) e publicá-lo no Azure como parte de um processo de DevOps.

1. No Portal do Azure, na **página inicial** , crie um novo recurso **Aplicativo de funções** com as seguintes configurações:
    - **Plano de hospedagem**: consumo
    - **Assinatura**: *Sua assinatura*
    - **Grupo de recursos**: *o mesmo grupo de recursos do recurso que o seu recurso de Pesquisa de IA do Azure*
    - **Nome do aplicativo de funções**: *um nome exclusivo*
    - **Pilha de runtime** : Node.js
    - **Versão**: 18 LTS
    - **Região**: *a mesma região que o recurso de Pesquisa de IA do Azure*
    - **Sistema operacional**: Windows

2. Aguarde a conclusão da implantação e vá até o recurso de aplicativo de funções implantado.
3. Na página **Visão geral**, selecione **Criar função** na parte inferior da página para criar uma nova função com as seguintes configurações:
    - **Selecionar um modelo**
        - **Modelo**: gatilho de HTTP    
    - **Detalhes do modelo**:
        - **Nome da função**: wordcount
        - **Nível de autorização**: função

    > **Observação**: se você receber um erro de Criação de Função, atualize a página e o recurso será criado conforme o esperado.

4. Aguarde até a função *contagem de palavras* ser criada. Em seguida, em sua página, selecione a guia **Código + Teste**.
5. Substitua o código padrão da função pelo seguinte código:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. Salve a função e então abra o painel **Teste/execução**.
7. No painel **Testar/Executar**, substitua o **Corpo** existente pelo seguinte JSON, que reflete o esquema esperado por uma habilidade da Pesquisa de IA do Azure na qual os registros que contêm dados de um ou mais documentos são enviados para processamento:

    ```json
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```

8. Clique em **Executar** e veja o conteúdo da resposta HTTP retornado pela função. Ele reflete o esquema esperado pela Pesquisa de IA do Azure ao consumir uma habilidade, em que uma resposta para cada documento é retornada. Nesse caso, a resposta é composta por até dez termos em cada documento na ordem decrescente da frequência com que aparecem:

    ```json
    {
        "values": [
        {
            "recordId": "a1",
            "data": {
                "text": [
                "tiger",
                "burning",
                "bright",
                "darkness",
                "night"
                ]
            }
        },
        {
            "recordId": "a2",
            "data": {
                "text": [
                    "rain",
                    "spain",
                    "stays",
                    "mainly",
                    "plains",
                    "thats",
                    "youll",
                    "find"
                ]
            }
        }
        ]
    }
    ```

9. Feche o painel **Test/Run** e, na folha da função **wordcount**, clique em **Obter URL da Função**. Em seguida, copie a URL da chave padrão para a área de transferência. Você precisará disso no próximo procedimento.

## Adicionar a habilidade personalizada à solução de pesquisa

Agora você precisa incluir sua função como uma habilidade personalizada no conjunto de habilidades da solução de pesquisa e mapear os resultados que ela produz para um campo no índice. 

1. No Visual Studio Code, na pasta **02-search-skill/update-search**, abra o arquivo **update-skillset.json**. Ele contém a definição JSON de um conjunto de habilidades.
2. Examine a definição do conjunto de habilidades. Ele inclui as mesmas habilidades que antes, bem como uma nova habilidade **WebApiSkill** chamada **get-top-words**.
3. Edite a definição da habilidade **get-top-words** para definir o valor do **url** como a URL de sua função do Azure (que você copiou para a área de transferência no procedimento anterior), substituindo **YOUR-FUNCTION-APP-URL**.
4. Na parte superior da definição do conjunto de habilidades, no elemento **cognitiveServices**, substitua o espaço reservado **YOUR_AI_SERVICES_KEY** por uma das chaves para seus recursos dos Serviços de IA do Azure.

    *Encontre as chaves na página de **Chave e ponto de extremidade** do recurso de Serviços de IA do Azure no portal do Azure.*

5. Salve e feche o arquivo JSON atualizado.
6. Na pasta **update-search**, abra **update-index.json**. Esse arquivo contém a definição JSON para o índice **margies-custom-index**, com um campo adicional chamado **top_words** na parte inferior da definição do índice.
7. Revise o JSON para o índice e, em seguida, feche o arquivo sem fazer alterações.
8. Na pasta ** update-search**, abra **update-indexer.json**. Esse arquivo contém uma definição JSON para o **margies-custom-indexer**, com um mapeamento adicional para o campo **top_words**.
9. Revise o JSON do indexador e feche o arquivo sem fazer alterações.
10. Na pasta **update-search**, abra **update-search.cmd**. Esse script em lote usa o utilitário cURL para enviar as definições JSON atualizadas para a interface REST para seu recurso de Pesquisa de IA do Azure.
11. Substitua os espaços reservados das variáveis **YOUR_SEARCH_URL** e **YOUR_ADMIN_KEY** pela **URL** e uma das **chaves de administrador** do recurso de Pesquisa de IA do Azure.

    *Você poderá encontrar esses valores na página **Visão geral** e **Chaves** do seu recurso de Pesquisa de IA do Azure no portal do Azure.*

12. Salve o arquivo em lotes atualizado.
13. Clique com o botão direito do mouse na pasta **update-search** e selecione **Abrir em terminal integrado**.
14. No painel de terminal da pasta **update-search**, digite o seguinte comando executar o script em lote.

    ```powershell
    ./update-search
    ```

15. Quando o script for concluído, no portal do Azure, na página do recurso de Pesquisa de IA do Azure, selecione a página **Indexadores** e aguarde a conclusão do processo de indexação.

    *Você pode selecionar **Atualizar** para acompanhar o progresso da operação de indexação. Pode levar cerca de um minuto para ser concluído.*

## Pesquisar o índice

Agora que você tem um índice, você pode pesquisá-lo.

1. Na parte superior do painel do seu recurso de Pesquisa de IA do Azure, selecione **Gerenciador de pesquisa**.
2. No Gerenciador de pesquisa, altere o modo de exibição para o **modo de exibição JSON** e envie a seguinte consulta de pesquisa:

    ```json
    {
      "search": "Las Vegas",
      "select": "url,top_words"
    }
    ```

    Esta consulta recupera os campos **url** e **top_words** de todos os documentos que mencionam *Las Vegas*.

## Limpar

Agora que você concluiu o exercício, exclua todos os recursos de que não precisa mais. Exclua os recursos do Azure:

1. No **portal do Azure**, selecione Grupos de recursos.
1. Selecione o grupo de recursos que você não precisa e, em seguida, selecione **Excluir grupo de recursos**.

## Mais informações

Para saber mais sobre como criar habilidades personalizadas para a Pesquisa de IA do Azure, consulte a [documentação da Pesquisa de IA do Azure.](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)
