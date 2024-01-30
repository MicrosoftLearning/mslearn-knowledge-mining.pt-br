---
lab:
  title: Criar uma solução de Pesquisa da IA do Azure
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Criar uma solução de Pesquisa da IA do Azure

Todas as organizações dependem de informações para tomar decisões, responder a perguntas e funcionar com eficiência. O problema para a maioria das organizações não é a falta de informações, mas o desafio de encontrar e extrair as informações do grande conjunto de documentos, bancos de dados e outras fontes em que as informações são armazenadas.

Por exemplo, suponha que *Margie's Travel* seja uma agência de viagens especializada em organizar viagens em cidades em todo o mundo. Ao longo do tempo, a empresa reuniu uma enorme quantidade de informações em documentos como folhetos, bem como avaliações de hotéis enviadas pelos clientes. Esses dados são uma fonte valiosa de informações para agentes de viagem e clientes à medida que planejam viagens, mas o enorme volume de dados pode dificultar a localização de informações relevantes para responder a uma pergunta específica do cliente.

Para enfrentar esse desafio, a Margie's Travel pode usar a Pesquisa de IA do Azure para implementar uma solução na qual os documentos são indexados e enriquecidos usando habilidades de IA para torná-los mais fáceis de pesquisar.

## Criar recursos do Azure

A solução que você criará para a Margie's Travel requer os seguintes recursos em sua assinatura do Azure:

- Um recurso do da **Pesquisa de IA do Azure**, que gerenciará a indexação e a consulta.
- Um recurso dos **Serviços de IA do Azure**, que fornece serviços de IA para habilidades que a sua solução de pesquisa pode usar para enriquecer os dados na fonte de dados com insights gerados pela IA.
- Uma **conta de armazenamento** com um contêiner de blob no qual os documentos a serem pesquisados são armazenados.

> **Observação**: seus recursos da Pesquisa de IA do Azure e dos Serviços de IA do Azure precisam estar no mesmo local.

### Criar um recurso do Azure AI Search

1. Em um navegador da web, abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
2. Clique no botão **&#65291;Criar um recurso**, pesquise por *Pesquisa* e crie um recurso de **Pesquisa IA do Azure** com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *crie um novo grupo de recursos (caso esteja usando uma assinatura restrita, você pode não ter permissão para criar um novo grupo de recursos, então use o que foi fornecido)*
    - **Nome do serviço**: *insira um nome exclusivo*
    - **Local**: *selecione um local – observe que seus recursos de Pesquisa de IA do Azure e Serviços de IA do Azure devem estar no mesmo local*
    - **Tipo de preço**: Básico

3. Aguarde a conclusão da implantação e acesse o recurso implantado.
4. Examine a página **Visão geral** no painel do recurso de Pesquisa de IA do Azure no portal do Azure. Aqui, você pode usar uma interface visual para criar, testar, gerenciar e monitorar os vários componentes de uma solução de pesquisa; incluindo fontes de dados, índices, indexadores e conjuntos de habilidades.

### Crie um recurso dos serviços de IA do Azure

Caso ainda não tenha um na sua assinatura, precisará provisionar um recurso dos **Serviços de IA do Azure**. Sua solução de pesquisa usará isso para enriquecer os dados no armazenamento de dados com insights gerados pela IA.

1. Volte para a página inicial do portal do Azure e selecione o botão **&#65291;Criar um recurso**, pesquise *Serviços de IA do Azure* e crie um recurso de **Serviços de IA do Azure** com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *o mesmo grupo de recursos do recurso da Pesquisa de IA do Azure*
    - **Região**: *a mesma localização do recurso da Pesquisa de IA do Azure*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: Standard S0
2. Marque as caixas de seleção necessárias e crie o recurso.
3. Aguarde a conclusão da implantação e veja os detalhes da implantação.

### Criar uma conta de armazenamento

1. Volte à home page do portal do Azure e selecione o botão **&#65291;Criar um recurso**, pesquise *conta de armazenamento* e crie um recurso da **Conta de armazenamento** com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *o mesmo grupo de recursos que o dos recursos da Pesquisa de IA do Azure e Serviços de IA do Azure*.
    - **Nome da conta de armazenamento**: *insira um nome exclusivo*.
    - **Região**: *escolha uma região disponível*
    - **Desempenho**: padrão
    - **Replicação**: LRS (armazenamento com redundância local)
    - Na guia **Avançado**, marque a caixa ao lado de *Permitir habilitação de acesso anônimo em contêineres individuais*
2. Aguarde a conclusão da implantação e acesse o recurso implantado.
3. Na página **Visão geral**, observe a **ID da assinatura** – ela identifica a assinatura em que a conta de armazenamento é provisionada.
4. Na página **Chaves de acesso**, observe que duas chaves foram geradas para sua conta de armazenamento. Em seguida, selecione **Mostrar chaves** para exibir as chaves.

    > **Dica**: mantenha a folha **Conta de armazenamento** aberta – você precisará do ID da assinatura e de uma das chaves no próximo procedimento.

## Preparar para desenvolver um aplicativo no Visual Studio Code

Você desenvolverá seu aplicativo de pesquisa usando o Visual Studio Code. Os arquivos de código para seu aplicativo foram fornecidos em um repositório do GitHub.

> **Dica**: Se você já clonou o repositório **mslearn-knowledge-mining**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
1. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` em uma pasta local (não importa qual pasta).
1. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
1. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**.

## Carregar documentos no Armazenamento do Azure

Agora que você tem os recursos necessários, pode carregar alguns documentos em sua conta de Armazenamento do Azure.

1. No Visual Studio Code, no painel do **Explorer**, expanda a pasta **Labfiles\01-azure-search** e selecione **UploadDocs.cmd**.
2. Edite o arquivo em lotes para substituir os espaços reservados **YOUR_SUBSCRIPTION_ID**, **YOUR_AZURE_STORAGE_ACCOUNT_NAME ** e **YOUR_AZURE_STORAGE_KEY** com a ID de assinatura apropriada, o nome da conta de armazenamento do Azure e os valores de chave da conta de armazenamento do Azure para a conta de armazenamento criada anteriormente.
3. Salve suas alterações e clique com o botão direito do mouse na pasta **01-azure-search** e abra um terminal integrado.
4. Insira o comando a seguir para entrar em sua assinatura do Azure ao usar a CLI do Azure.

    ```
    az login
    ```

    Uma guia do navegador da Web será aberta e solicitará que você entre no Azure. Faça isso, em seguida feche a guia do navegador e volte para o Visual Studio Code.

5. Insira o comando a seguir para executar o arquivo em lote. Isso criará um contêiner de blob em sua conta de armazenamento e carregará os documentos na pasta de **dados** para ele.

    ```
    UploadDocs
    ```

## Indexar os documentos

Agora que os documentos estão no lugar, você pode criar uma solução de pesquisa indexando-os.

1. No portal do Azure, procure pelo seu recurso da Pesquisa de IA do Azure. Em seguida, na página **Visão geral**, selecione **Importar dados**.
2. Na página **Conectar-se aos seus dados**, na lista **Fonte de Dados**, escolha **Armazenamento de Blobs do Azure**. Em seguida, preencha os detalhes do armazenamento de dados com os seguintes valores:
    - **Fonte de dados**: Armazenamento de Blobs do Azure
    - **Nome da fonte de dados**: margies-data
    - **Dados para extração**: Conteúdo e metadados
    - **Modo de análise**: Padrão
    - **Cadeia de conexão**: *selecione **Escolher uma conexão existente**. Em seguida, selecione sua conta de armazenamento e, finalmente, selecione o contêiner **margies** que foi criado pelo script UploadDocs.cmd.*
    - **Autenticação da identidade gerenciada**: Nenhuma
    - **Nome do contêiner**: margies
    - **Pasta do blob**: *mantenha essa opção em branco*
    - **Descrição**: brochuras e comentários no site da Margie's Travel.
3. Prossiga para a próxima etapa, (*Adicionar habilidades cognitivas*).
4. Na seção **Anexar Serviços de IA do Azure**, selecione o recurso de Serviços de IA do Azure.
5. Na seção **Adicionar enriquecimentos**:
    - Altere o **Nome do conjunto de habilidades** para **margies-skillset**.
    - Selecione a opção **Habilitar OCR e mesclar todo o texto no campo merged_content**.
    - Verifique se o **campo Dados de origem** está definido **como merged_content**.
    - Deixe o **nível de granularidade do Enriquecimento** como ** campo Origem**, que é definido por todo o conteúdo do documento que está sendo indexado, mas observe que você pode alterar isso para extrair informações em níveis mais granulares, como páginas ou frases.
    - Selecione os seguintes campos enriquecidos:

        | Habilidade cognitiva | Parâmetro | Nome do campo |
        | --------------- | ---------- | ---------- |
        | Extrair nomes de localização | | Locais |
        | Extrair frases-chave | | keyphrases |
        | Detectar o idioma | | linguagem |
        | Gerar marcas com base em imagens | | imageTags |
        | Gerar legendas com base em imagens | | imageCaption |

6. Verifique as seleções mais uma vez (pode ser difícil alterá-las mais tarde). Em seguida, prossiga para a próxima etapa (*Personalizar índice de destino*).
7. Altere o **Nome do índice** para **margies-index**.
8. Verifique se a **Chave** está definida como **metadata_storage_path** e mantenha o **Nome do sugestor** em branco e o **Modo de pesquisa** em seu padrão.
9. Faça as seguintes alterações nos campos de índice, deixando todos os outros campos com suas configurações padrão (**IMPORTANTE**: talvez seja necessário rolar para a direita para ver a tabela inteira):

    | Nome do campo | Recuperável | Filtrável | Classificável | Com faceta | Pesquisável |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | metadata_author | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | Locais | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | linguagem | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | |

10. Verifique novamente suas seleções, prestando atenção em especial para garantir que as opções corretas **Recuperável**, **Filtrável**, **Classificável**, **Com faceta** e **Pesquisável** sejam selecionadas para cada campo (pode ser difícil alterá-las mais tarde). A seguir, prossiga para a próxima etapa (*Criar um indexador*).
11. Altere o **Nome do indexador** para **margies-indexer**.
12. Mantenha o **Agendamento** definido como **Uma vez**.
13. Expanda as opções **Avançadas** e verifique se a opção **Chaves de codificação de Base 64** está selecionada (geralmente, as chaves de codificação tornam o índice mais eficiente).
14. Escolha **Enviar** para criar a fonte de dados, o conjunto de habilidades, o índice e o indexador. O indexador é executado automaticamente e executa o pipeline de indexação, que:
    1. Extrai os campos de metadados do documento e o conteúdo da fonte de dados
    2. Executa o conjunto de habilidades cognitivas para gerar campos enriquecidos adicionais
    3. Mapeia os campos extraídos para o índice.
15. Na metade inferior da página **Visão geral** do recurso de Pesquisa de IA do Azure, veja a guia **Indexadores**, que deve mostrar o **margies-indexer** recém-criado. Aguarde alguns minuto e clique em **&orarr; Atualizar** até que o **Status** indique êxito.

## Pesquisar o índice

Agora que você tem um índice, você pode pesquisá-lo.

1. Na parte superior da página **Visão geral** do seu recurso de Pesquisa de IA do Azure, selecione **Gerenciador de pesquisa**.
2. No Gerenciador de pesquisa, na caixa **Cadeia de consulta**, digite `*` (um único asterisco) e selecione **Pesquisar**.

    Esta consulta recupera todos os documentos no índice no formato JSON. Examine os resultados e observe os campos de cada documento, que contêm conteúdo do documento, metadados e dados enriquecidos extraídos pelas habilidades cognitivas selecionadas.

3. No menu **Exibição**, selecione o **modo de exibição JSON** e observe que a solicitação JSON para a pesquisa é mostrada, da seguinte maneira:

    ```json
    {
      "search": "*"
    }
    ```

1. Modifique a solicitação JSON para incluir o parâmetro **count** conforme mostrado aqui:

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. Envie a pesquisa modificada. Desta vez, os resultados incluem um campo **@odata.count** na parte superior dos resultados que indica o número de documentos retornados pela pesquisa.

4. Experimente a seguinte consulta:

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,metadata_author,locations"
    }
    ```

    Desta vez, os resultados incluem apenas o nome do arquivo, o autor e quaisquer locais mencionados no conteúdo do documento. O nome do arquivo e o autor estão nos campos **metadata_storage_name** e **metadata_author**, que foram extraídos do documento de origem. O campo **locais** foi gerado por uma habilidade cognitiva.

5. Agora tente a seguinte cadeia de caracteres de consulta:

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    Esta pesquisa localiza documentos que mencionam "New York" em qualquer um dos campos pesquisáveis e retorna o nome do arquivo e as frases principais no documento.

6. Vamos tentar mais uma consulta:

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name",
      "filter": "metadata_author eq 'Reviewer'"
    }
    ```

    Essa consulta retorna o nome do arquivo de quaisquer documentos de autoria do * Revisor* que mencionem "New York".

## Explorar e modificar definições de componentes de pesquisa

Os componentes da solução de pesquisa são baseados em definições JSON, que você pode exibir e editar no portal do Azure.

Embora você possa usar o portal para criar e modificar soluções de pesquisa, geralmente é desejável definir os objetos de pesquisa em JSON e usar a interface REST do Serviço de IA do Azure para criá-los e modificá-los.

### Obter o ponto de extremidade e a chave para o seu recurso da Pesquisa de IA do Azure

1. No portal do Azure, retorne à página ** Visão geral** do seu recurso de Pesquisa de IA do Azure e, na seção superior da página, localize a **URL** do seu recurso (que se parece com **https://resource_name.search.windows.net**) e copie-a para a área de transferência.
2. No Visual Studio Code, no painel do Explorer, expanda a pasta **01-azure-search** e sua subpasta **modify-search** e selecione **modify-search.cmd** para abri-la. Você usará esse arquivo de script para executar comandos *cURL* que enviam JSON para a interface REST do Serviço de IA do Azure.
3. Em **modify-search.cmd**, substitua o espaço reservado **YOUR_SEARCH_URL** pela URL copiada para a área de transferência.
4. No portal do Azure, exiba a página **Chaves** do seu recurso de Pesquisa de IA do Azure e copie a **chave de administrador primária** para a área de transferência.
5. No Visual Studio Code, substitua o espaço reservado **YOUR_ADMIN_KEY** pela chave que você copiou para a área de transferência.
6. Salve as alterações em **modify-search.cmd** (mas não execute ainda!)

### Revisar e modificar o conjunto de habilidades

1. No Visual Studio Code, na pasta ** modify-search**, abra **skillset.json**. Isso mostra uma definição JSON para **margies-skillset**.
2. Na parte superior da definição do conjunto de habilidades, observe o objeto **cognitiveServices**, que é usado para conectar seu recurso dos Serviços de IA do Azure ao conjunto de habilidades.
3. No portal do Azure, abra seu recurso de Serviços de IA do Azure (<u>não</u> seu recurso de Pesquisa de IA do Azure!) e exiba sua página **Chaves**. Em seguida copie a **Chave 1** para a área de transferência.
4. No Visual Studio Code, em **skillset.json**, substitua o espaço reservado **YOUR_COGNITIVE_SERVICES_KEY** pela chave dos Serviços de IA do Azure que você copiou para a área de transferência.
5. Percorra o arquivo JSON, observando que ele inclui definições para as habilidades que você criou usando a interface do usuário da Pesquisa de IA do Azure no portal do Azure. Na parte inferior da lista de habilidades, uma habilidade adicional foi adicionada com a seguinte definição:

    ```
    {
        "@odata.type": "#Microsoft.Skills.Text.V3.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "sentiment",
                "targetName": "sentimentLabel"
            }
        ]
    }
    ```

    A nova habilidade é chamada de ** get-sentiment**, e para cada nível de **documento** em um documento, ele avaliará o texto encontrado no campo **merged_content** do documento que está sendo indexado (que inclui o conteúdo de origem, bem como qualquer texto extraído de imagens no conteúdo). Ele usa o **idioma** extraído do documento (inglês por padrão) e avalia uma etiqueta para o sentimento do conteúdo. Os valores para a etiqueta de sentimento podem ser "positivos", "negativos", "neutros" ou "mistos". Essa etiqueta é então gerada como um novo campo chamado **sentimentLabel**.

6. Salve as alterações que você fez no **skillset.json**.

### Revisar e modificar o índice

1. No Visual Studio Code, na pasta **modify-search**, abra **index.json**. Isso mostra uma definição JSON para **margies-index**.
2. Percorra o índice e exiba as definições de campo. Alguns campos são baseados em metadados e conteúdo no documento de origem, e outros são resultados de habilidades no conjunto de habilidades.
3. No final da lista de campos que você definiu no portal do Azure, observe que dois campos adicionais foram adicionados:

    ```
    {
        "name": "sentiment",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. O campo **sentimento** será usado para adicionar os resultados da habilidade ** get-sentiment** que foi adicionada ao conjunto de habilidades. O campo **url** será usado para adicionar a URL de cada documento indexado ao índice, com base no valor **metadata_storage_path** extraído da fonte de dados. Observe que o índice já inclui o campo ** metadata_storage_path**, mas ele é usado como a chave do índice e codificado em Base-64, tornando-o eficiente como uma chave, mas exigindo que os aplicativos cliente o decodifiquem se quiserem usar o valor real da URL como um campo. Adicionar um segundo campo para o valor não codificado resolve esse problema.

### Revisar e modificar o indexador

1. No Visual Studio Code, na pasta **modify-search**, abra **indexer.json**. Isso mostra uma definição JSON para **margies-indexer** que mapeia campos extraídos do conteúdo do documento e metadados (na seção **fieldMappings**) e valores extraídos por habilidades no conjunto de habilidades (na seção **outputFieldMappings**) para campos no índice.
2. Na lista **fieldMappings**, observe o mapeamento do valor **metadata_storage_path** para o campo de chave codificada em base 64. Isso foi criado quando você atribuiu o **metadata_storage_path** como a chave e selecionou a opção para codificar a chave no portal do Azure. Além disso, um novo mapeamento mapeia explicitamente o mesmo valor para o campo **url**, mas sem a codificação Base-64:

    ```
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }
    
    ```

    Todos os outros campos de metadados e conteúdo no documento de origem são mapeados implicitamente para campos de mesmo nome no índice.

3. Revise a seção **ouputFieldMappings**, que mapeia os resultados das habilidades no conjunto de habilidades para os campos de índice. A maioria deles reflete as escolhas feitas na interface do usuário, mas o mapeamento a seguir foi adicionado para mapear o valor **sentimentLabel** extraído por sua habilidade de sentimento para o campo **sentimento** adicionado por você ao índice:

    ```
    {
        "sourceFieldName": "/document/sentimentLabel",
        "targetFieldName": "sentiment"
    }
    ```

### Usar a API REST para atualizar a solução de pesquisa

1. Clique com o botão direito do mouse na pasta **modify-search** e abra um terminal integrado.
2. No painel de terminal da pasta ** modify-search**, digite o seguinte comando para executar o script **modify-search.cmd**, que envia as definições JSON para a interface REST e inicia a indexação.

    ```
    ./modify-search
    ```

3. Quando o script terminar, retorne à página **Visão geral** do recurso de Pesquisa de IA do Azure no portal do Azure e exiba a página **Indexadores**. Selecione periodicamente **Atualizar** para acompanhar o progresso da operação de indexação. Isso poderá levar alguns minutos para ser concluído.

    *Pode haver alguns avisos para alguns documentos que são muito grandes para avaliar o sentimento. Muitas vezes, a análise de sentimento é realizada no nível da página ou da frase, em vez do documento completo. Mas neste cenário, a maioria dos documentos, em especial as avaliações de hotéis, são curtas o suficiente para que as pontuações de sentimento úteis em nível de documento sejam avaliadas.*

### Consultar o índice modificado

1. Na parte superior do painel do seu recurso de Pesquisa de IA do Azure, selecione **Gerenciador de pesquisa**.
2. No Gerenciador de pesquisa, na caixa **Cadeia de caracteres de consulta**, envie a seguinte consulta JSON:

    ```json
    {
      "search": "London",
      "select": "url,sentiment,keyphrases",
      "filter": "metadata_author eq 'Reviewer' and sentiment eq 'positive'"
    }
    ```

    Essa consulta recupera a **url**, o ** sentimento** e as **frases-chave** de todos os documentos que mencionam *Londres* de autoria do *Revisor* que têm uma etiqueta de **sentimento** positivo (em outras palavras, comentários positivos que mencionam Londres)

3. Feche a página **Gerenciador de pesquisa** para retornar à página **Visão geral**.

## Criar um aplicativo cliente de pesquisa

Agora que você tem um índice útil, você pode usá-lo a partir de um aplicativo cliente. Você pode fazer isso consumindo a interface REST, enviando solicitações e recebendo respostas no formato JSON sobre HTTP; ou você pode usar o SDK (kit de desenvolvimento de software) para sua linguagem de programação preferida. Neste exercício, usaremos o SDK.

> **Observação**: você pode optar por usar o SDK para **C#** ou **Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

### Obter o ponto de extremidade e chaves para seu recurso de pesquisa

1. No portal do Azure, na página **Visão geral** do seu recurso de Pesquisa de IA do Azure, observe o valor de **Url**, que deve ser semelhante a **https://* your_resource_name.search.windows.net***. Este é o ponto de extremidade para seu recurso de pesquisa.
2. Na página **Chaves**, observe que há duas chaves de **administrador** e uma única chave de **consulta**. Uma chave de *administrador* é usada para criar e gerenciar recursos de pesquisa, uma chave de *consulta* é usada por aplicativos cliente que só precisam executar consultas de pesquisa.

    *Você precisará do ponto de extremidade e da chave de consulta para seu aplicativo cliente.*

### Preparar-se para usar o SDK de Pesquisa de IA do Azure

1. No Visual Studio Code, no painel do **Explorer**, navegue até a pasta **01-azure-search** e expanda a pasta **C-Sharp** ou **Python**, dependendo da sua preferência de idioma.
2. Clique com o botão direito do mouse na pasta **margies-travel** e abra um terminal integrado. Em seguida, instale o pacote do SDK da Pesquisa de IA do Azure executando o comando apropriado para sua linguagem de escolha:

    **C#**

    ```
    dotnet add package Azure.Search.Documents --version 11.1.1
    ```

    **Python**

    ```
    pip install azure-search-documents==11.0.0
    ```

3. Exiba o conteúdo da pasta **margies-travel** e observe que ela contém um arquivo para definições de configuração:
    - **C#**: appsettings.json
    - **Python**: .env

    Abra o arquivo de configuração e atualize os valores de configuração que ele contém para refletir o **ponto de extremidade** e **chave de consulta** para seu recurso de Pesquisa de IA do Azure. Salve suas alterações.

### Explorar código para pesquisar um índice

A pasta **margies-travel** contém arquivos de código para um aplicativo Web (um aplicativo Web Microsoft C# *ASP.NET Razor* ou um aplicativo Python *Flask* ), que inclui a funcionalidade de pesquisa.

1. Abra o seguinte arquivo de código no aplicativo Web, dependendo da sua escolha de linguagem de programação:
    - **C#**:Pages/Index.cshtml.cs
    - **Python**: app.py
2. Na parte superior do arquivo de código, localize o comentário **Importar namespaces de pesquisa** e observe os namespaces que foram importados para funcionar com o SDK de Pesquisa de IA do Azure:
3. Na função **search_query**, localize o comentário **Criar um cliente de pesquisa** e observe que o código cria um objeto **SearchClient** usando o ponto de extremidade e a chave de consulta para seu recurso de Pesquisa de IA do Azure:
4. Na função **search_query**, localize o comentário **Enviar consulta de pesquisa** e revise o código para enviar uma pesquisa para o texto especificado com as seguintes opções:
    - Um *modo de pesquisa* que requer que **todas** as palavras individuais no texto de pesquisa sejam encontradas.
    - O número total de documentos encontrados pela pesquisa está incluído nos resultados.
    - Os resultados são filtrados para incluir apenas documentos que correspondam à expressão de filtro fornecida.
    - Os resultados são classificados na ordem de classificação especificada.
    - Cada valor discreto do campo **campo metadata_author** é retornado como uma *faceta* que pode ser usada para exibir valores predefinidos para filtragem.
    - Até três extrações dos campos **merged_content** e **imageCaption** com os termos de pesquisa realçados são incluídos nos resultados.
    - Os resultados incluem apenas os campos especificados.

### Explorar código para renderizar resultados de pesquisa

O aplicativo Web já inclui código para processar e renderizar os resultados da pesquisa.

1. Abra o seguinte arquivo de código no aplicativo Web, dependendo da sua escolha de linguagem de programação:
    - **C#**:Pages/Index.cshtml
    - **Python**: templates/search.html
2. Examine o código, que renderiza a página na qual os resultados da pesquisa são exibidos. Observe que:
    - A página começa com um formulário de pesquisa que o usuário pode usar para enviar uma nova pesquisa (na versão Python do aplicativo, esse formulário é definido no modelo **base.html**), que é referenciado no início da página.
    - Um segundo formulário é renderizado, permitindo que o usuário refine os resultados da pesquisa. O código para este formulário:
        - Recupera e exibe a contagem de documentos dos resultados da pesquisa.
        - Recupera os valores de faceta para o campo **metadata_author** e os exibe como uma lista de opções para filtragem.
        - Cria uma lista suspensa de opções de classificação para os resultados.
    - Em seguida, o código itera pelos resultados da pesquisa, renderizando cada resultado da seguinte maneira:
        - Exibe o campo **metadata_storage_name** (nome do arquivo) como um link para o endereço no campo **url**.
        - Exibição de *destaques* para termos de pesquisa encontrados nos campos **merged_content** e **imageCaption** para ajudar a mostrar os termos de pesquisa no contexto.
        - Exiba os campos **metadata_author**, **metadata_storage_size**, **metadata_storage_last_modified** e **linguagem**.
        - Exiba a etiqueta de **sentimento** do documento. Pode ser positivo, negativo, misto ou neutro.
        - Exiba as cinco primeiras **frases-chave** (se houver).
        - Exiba os cinco primeiros **locais** (se houver).
        - Exiba as cinco primeiras **imageTags** (se houver).

### Executar o aplicativo Web

 1. Retorne ao terminal integrado da pasta **margies-travel** e digite o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. Na mensagem exibida quando o aplicativo é iniciado com êxito, siga o link para o aplicativo Web em execução (*http://localhost:5000/* ou *http://127.0.0.1:5000/*) para abrir o site Margies Travel em um navegador da Web.
3. No site da Margie's Travel, digite **hotel Londres** na caixa de pesquisa e clique em **Pesquisar**.
4. Examinar os resultados da pesquisa. Eles incluem o nome (com um hiperlink para a URL do arquivo), uma extração do conteúdo do arquivo com os termos de pesquisa (*Londres* e *hotel*) enfatizados, além de outros atributos do arquivo dos campos de índice.
5. Observe que a página de resultados inclui alguns elementos da interface do usuário que permitem refinar os resultados. Estão incluídos:
    - Um *filtro* baseado em um valor de faceta para o campo **metadata_author**. Isso demonstra como você pode usar estes campos *com faceta* para retornar uma lista de *facetas* – campos com um pequeno conjunto de valores discretos que podem ser exibidos como valores de filtro potenciais na interface do usuário.
    - A capacidade de *ordenar* os resultados com base em um campo especificado e na direção de classificação (ascendente ou descendente). A ordem padrão é baseada na *relevância*, que é calculada como um valor **search.score()** com base em um *perfil de pontuação* que avalia a frequência e a importância dos termos de pesquisa nos campos de índice.
6. Selecione o filtro **Revisor** e a opção de classificação **Positiva para negativa** e selecione **Refinar resultados**.
7. Observe que os resultados são filtrados para incluir apenas avaliações e classificados com base na etiqueta de sentimento.
8. Na caixa **Pesquisar**, insira uma nova pesquisa de **hotel tranquilo em Nova York** e analise os resultados.
9. Tente os seguintes termos de pesquisa:
    - **Torre de Londres** (observe que este termo é identificado como uma *frase-chave* em alguns documentos).
    - **arranha-céu** (observe que essa palavra não aparece no conteúdo real de nenhum documento, mas é encontrada nas *legendas de imagem * e *tags de imagem* que foram geradas para imagens em alguns documentos).
    - **Deserto de Mojave** (observe que este termo é identificado como um *local* em alguns documentos).
10. Feche a guia do navegador que contém o site da Margie's Travel e volte para o Visual Studio Code. Em seguida, no terminal do Python da pasta **margies-travel** (em que o aplicativo dotnet flask está em execução), pressione Ctrl+C para interromper o aplicativo.

## Mais informações

Para saber mais sobre o serviço de Pesquisa IA do Azure, confira a [documentação da Pesquisa de IA do Azure](https://docs.microsoft.com/azure/search/search-what-is-azure-search).
