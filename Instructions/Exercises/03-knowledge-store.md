---
lab:
  title: Criar um Repositório de Conhecimento com a Pesquisa de IA do Azure
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Criar um Repositório de Conhecimento com a Pesquisa de IA do Azure

A Pesquisa de IA do Azure usa um pipeline de enriquecimento de habilidades de IA para extrair campos gerados por IA de documentos e incluí-los em um índice de pesquisa. Embora o índice possa ser considerado a saída primária de um processo de indexação, os dados enriquecidos que ele contém também podem ser úteis de outras maneiras. Por exemplo:

- Como o índice é essencialmente uma coleção de objetos JSON, cada um representando um registro indexado, pode ser útil exportar os objetos como arquivos JSON para integração em um processo de orquestração de dados usando ferramentas como o Azure Data Factory.
- Talvez você queira normalizar os registros de índice em um esquema relacional de tabelas para análise e relatórios com ferramentas como o Microsoft Power BI.
- Após extrair imagens incorporadas de documentos durante o processo de indexação, convém salvá-las como arquivos.

Neste exercício, você implementará um repositório de conhecimento para a *Margie's Travel*, uma agência de viagens fictícia que usa informações em folhetos e resenhas de hotel para ajudar os clientes a planejarem suas viagens.

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
2. Exiba os **grupos de Recursos** em sua assinatura.
3. Se você estiver usando uma assinatura restrita na qual um grupo de recursos foi fornecido para você, selecione o grupo de recursos para exibir suas propriedades. Caso contrário, crie um novo grupo de recursos com um nome de sua escolha e vá para ele quando ele tiver sido criado.
4. Na página **Visão geral** do seu grupo de recursos, observe a **ID da Assinatura** e **Local**. Você precisará desses valores, juntamente com o nome do grupo de recursos nas etapas subsequentes.
5. No Visual Studio Code, expanda a pasta **Labfiles/03-knowledge-store** e selecione **setup.cmd**. Você usará esse script em lote para executar os comandos da CLI (interface de linha de comando) do Azure necessários para criar os recursos do Azure necessários.
6. Clique com o botão direito do mouse na pasta **03-knowledge-store** e selecione **Abrir no Terminal Integrado**.
7. No painel do terminal, digite o comando a seguir para estabelecer uma conexão autenticada com a sua assinatura do Azure.

    ```powershell
    az login --output none
    ```

8. Quando solicitado, entre em sua assinatura do Azure. Em seguida, retorne para o Visual Studio Code e aguarde a conclusão do processo de entrada.
9. Use o comando a seguir para listar os locais do Azure.

    ```powershell
    az account list-locations -o table
    ```

10. No resultado, localize o valor **Name** que corresponde ao local do seu grupo de recursos (por exemplo, para *Leste dos EUA* o nome correspondente é *eastus*).
11. No script **setup.cmd**, modifique as declarações de variáveis de ** subscription_id**, **resource_group** e **local** com os valores apropriados para sua ID de assinatura, nome do grupo de recursos e nome do local. Em seguida, salve as alterações.
12. No terminal da pasta **03-knowledge-store**, insira o seguinte comando para executar o script:

    ```powershell
    ./setup
    ```
    > **Observação**: o módulo de Pesquisa da CLI está em versão preliminar e pode ficar preso em *– Executando ..* processo. Se isso acontecer por mais de 2 minutos, pressione CTRL+C para cancelar a operação demorada e selecione **N** quando perguntado se deseja encerrar o script. A operação deverá ser concluída com sucesso.
    >
    > Se o script falhar, certifique-se de salvá-lo com os nomes de variáveis corretos e tente novamente.

13. Quando o script for concluído, examine a saída exibida e observe as seguintes informações sobre os recursos do Azure (você precisará desses valores mais tarde):
    - Nome da conta de armazenamento
    - Cadeia de conexão de armazenamento
    - Conta de Serviços de IA do Azure
    - Chave de Serviços de IA do Azure
    - Ponto de extremidade de serviço Pesquisa
    - Chave de administração do serviço Pesquisa
    - Chave de consulta do serviço de pesquisa

14. No portal do Azure, atualize o grupo de recursos que foi criado pelo script de instalação e verifique se ele contém a conta do Armazenamento do Azure, o recurso dos Serviços de IA do Azure e o recurso da Pesquisa de IA do Azure.

## Criar uma solução de pesquisa

Agora que você tem os recursos necessários do Azure, você pode criar uma solução de pesquisa que consiste nos seguintes componentes:

- Uma **fonte de dados** que faz referência aos documentos em seu contêiner de armazenamento do Azure.
- Um **conjunto de habilidades** que define um pipeline de enriquecimento de habilidades para extrair campos gerados por IA dos documentos. O conjunto de habilidades também define as *projeções* que serão geradas em seu *repositório de conhecimento*.
- Um **índice** que define um conjunto pesquisável de registros de documentos.
- Um **indexador** que extrai os documentos da fonte de dados, aplica o conjunto de habilidades e preenche o índice. O processo de indexação também persiste as projeções definidas no conjunto de habilidades no repositório de conhecimento.

Neste exercício, você usará a interface REST da Pesquisa de IA do Azure para criar esses componentes enviando solicitações JSON.

### Preparar JSON para operações REST

Você usará a interface REST para enviar definições JSON para seus componentes de Pesquisa de IA do Azure.

1. No Visual Studio Code, na pasta **03-knowledge-store**, expanda a pasta **create-search** e selecione **data_source.json**. Esse arquivo contém uma definição JSON para uma fonte de dados chamada **margies-knowledge-data**.
2. Substitua o espaço reservado **YOUR_CONNECTION_STRING** pela cadeia de conexão para sua conta de armazenamento do Azure, que deve ser semelhante à seguinte:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Você pode encontrar a cadeia de conexão na página **Chaves de acesso** de sua conta de armazenamento no portal do Azure.*

3. Salve e feche o arquivo JSON atualizado.
4. Na pasta **create-search**, abra **skillset.json**. Esse arquivo contém uma definição JSON para um conjunto de habilidades chamado **margies-knowledge-skillset**.
5. Na parte superior da definição do conjunto de habilidades, no elemento **cognitiveServices**, substitua o espaço reservado **YOUR_COGNITIVE_SERVICES_KEY** por uma das chaves para seus recursos dos Serviços de IA do Azure.

    *Encontre as chaves na página de **Chave e ponto de extremidade** do recurso de Serviços de IA do Azure no portal do Azure.*

6. No final da coleção de habilidades em seu conjunto de habilidades, localize a **habilidade Microsoft.Skills.Util.ShaperSkill** chamada **define-projection**. Esta habilidade define uma estrutura JSON para os dados enriquecidos que serão usados para as projeções que o pipeline persistirá no repositório de conhecimento para cada documento processado pelo indexador.
7. Na parte de baixo do arquivo do conjunto de habilidades, observe que o conjunto de habilidades também inclui uma definição de **knowledgeStore**, que inclui uma cadeia de conexão para a conta de Armazenamento do Azure em que o repositório de conhecimento deve ser criado e uma coleção de **projeções**. Esse conjunto de habilidades inclui três *grupos de projeções*:
    - Um grupo contendo uma projeção de *objeto* baseada na saída **knowledge_projection** da habilidade shaper no conjunto de habilidades.
    - Um grupo contendo uma projeção de *arquivo* baseada na coleção **normalized_images** de dados de imagem extraídos dos documentos.
    - Um grupo contendo as seguintes projeções de *tabela*:
        - **KeyPhrases**: contém uma coluna de chave gerada automaticamente e uma coluna **keyPhrase** mapeada para a saída da coleção **knowledge_projection/key_phrases/** da habilidade de shaper.
        - **Locations**: contém uma coluna de chave gerada automaticamente e uma coluna **location** mapeada para a saída da coleção **knowledge_projection/key_phrases/** da habilidade de shaper.
        - **ImageTags**: contém uma coluna de chave gerada automaticamente e uma coluna **tag** mapeada para a saída da coleção **knowledge_projection/image_tags/** da habilidade de shaper.
        - **Docs**: contém uma coluna de chave gerada automaticamente e todos os valores de saída de **knowledge_projection** da habilidade de shaper que ainda não foram atribuídos a uma tabela.
8. Substitua o espaço reservado **YOUR_CONNECTION_STRING ** pelo valor **storageConnectionString** com cadeia de conexão para sua conta de armazenamento.
9. Salve e feche o arquivo JSON atualizado.
10. Na pasta **create-search**, abra **index.json**. Esse arquivo contém uma definição JSON para um índice chamado **margies-knowledge-index**.
11. Revise o JSON para o índice e, em seguida, feche o arquivo sem fazer alterações.
12. Na pasta **create-search**, abra **indexer.json**. Esse arquivo contém uma definição JSON para um indexador chamado **margies-knowledge-indexer**.
13. Revise o JSON do indexador e feche o arquivo sem fazer alterações.

### Enviar solicitações REST

Agora que você preparou os objetos JSON que definem os componentes da solução de pesquisa, pode enviar os documentos JSON para a interface REST para criá-los.

1. Na pasta **create-search**, abra **create-search.cmd**. Esse script em lote usa o utilitário cURL para enviar as definições JSON para a interface REST para seu recurso de Pesquisa de IA do Azure.
2. Substitua os espaços reservados das variáveis **YOUR_SEARCH_URL** e **YOUR_ADMIN_KEY** pela **URL** e uma das **chaves de administrador** do recurso de Pesquisa de IA do Azure.

    *Você poderá encontrar esses valores na página **Visão geral** e **Chaves** do seu recurso de Pesquisa de IA do Azure no portal do Azure.*

3. Salve o arquivo em lotes atualizado.
4. Clique com o botão direito do mouse no nome da pasta **create-search** e selecione **Abrir no terminal integrado**.
5. No painel do terminal da pasta **create-index**, digite o seguinte comando para executar o script em lote.

    ```powershell
    ./create-search
    ```

6. Quando o script for concluído, no portal do Azure, na página do recurso de Pesquisa de IA do Azure, selecione a página **Indexadores** e aguarde a conclusão do processo de indexação.

    *Você pode selecionar **Atualizar** para acompanhar o progresso da operação de indexação. Pode levar cerca de um minuto para ser concluído.*

    > **Dica**: se o script falhar, verifique os espaços reservados adicionados nos arquivos **data_source.json** e **skillset.json**, bem como o arquivo **create-search.cmd**. Depois de corrigir quaisquer erros, talvez seja necessário usar a interface do usuário do portal do Azure para excluir quaisquer componentes que foram criados em seu recurso de pesquisa antes de executar novamente o script.

## Exibir o repositório de conhecimento

Depois de executar um indexador que usa um conjunto de habilidades para criar um repositório de conhecimento, os dados enriquecidos extraídos pelo processo de indexação são persistidos nas projeções do repositório de conhecimento.

### Exibir projeções de objeto

As projeções de *objeto* definidas no conjunto de habilidades da Margie's Travel consistem em um arquivo JSON para cada documento indexado. Esses arquivos são armazenados em um contêiner de blob na conta de Armazenamento do Azure especificada na definição do conjunto de habilidades.

1. No portal do Azure, exiba a conta de armazenamento do Azure criada anteriormente.
2. Selecione a guia do **navegador de armazenamento** (no painel à esquerda) para ver a conta de armazenamento na interface do gerenciador de armazenamento no portal do Azure.
3. Expanda **contêineres de blob** para ver os contêineres na conta de armazenamento. Além do contêiner **margies**, onde os dados de origem estão armazenados, deve haver dois novos contêineres: **margies-images** e **margies-knowledge**. Eles foram criados pelo processo de indexação.
4. Selecione o contêiner **margies-knowledge**. Ele deve conter uma pasta para cada documento indexado.
5. Abra qualquer uma das pastas, então baixe a abra o arquivo **knowledge-projection.json** contido nela. Cada arquivo JSON contém uma representação de um documento indexado, incluindo os dados enriquecidos extraídos pelo conjunto de habilidades, como mostrado aqui.

```json
{
    "file_id":"abcd1234....",
    "file_name":"Margies Travel Company Info.pdf",
    "url":"https://store....blob.core.windows.net/margies/...pdf",
    "language":"en",
    "sentiment":0.83164644241333008,
    "key_phrases":[
        "Margie’s Travel",
        "Margie's Travel",
        "best travel experts",
        "world-leading travel agency",
        "international reach"
        ],
    "locations":[
        "Dubai",
        "Las Vegas",
        "London",
        "New York",
        "San Francisco"
        ],
    "image_tags":[
        "outdoor",
        "tree",
        "plant",
        "palm"
        ]
}
```

A capacidade de criar projeções de *objeto* como essa permite que você gere objetos de dados enriquecidos que podem ser incorporados em uma solução de análise de dados empresariais, por exemplo, ingerindo os arquivos JSON em um pipeline do Azure Data Factory para processamento adicional ou carregamento em um data warehouse.

### Exibir projeções de arquivo

As projeções de *arquivo* definidas no conjunto de habilidades criam arquivos JPEG para cada imagem que foi extraída dos documentos durante o processo de indexação.

1. Na interface do *Navegador de armazenamento* no portal do Azure, selecione o contêiner de blobs **margies-images**. Ele contém uma pasta para cada documento que continha imagens.
2. Abra qualquer uma das pastas e veja o conteúdo – cada pasta contém pelo menos um arquivo \*. jpg.
3. Abra qualquer um dos arquivos de imagem para verificar se eles contêm imagens extraídas dos documentos.

A capacidade de gerar projeções de *arquivo* faz da indexação uma forma eficiente de extrair imagens incorporadas de um grande volume de documentos.

### Exibir projeções de tabela

As projeções de *tabela* definidas no conjunto de habilidades formam um esquema relacional de dados enriquecidos.

1. Na interface do *Navegador de armazenamento* no portal do Azure, expanda **Tabelas**.
2. Selecione a tabela **Docs** para ver suas colunas. Elas incluem algumas colunas padrão da tabela de Armazenamento do Azure – para ocultá-las, modifique as **Opções de Coluna** de modo a selecionar apenas as seguintes colunas:
    - **document_id** (a coluna de chave gerada automaticamente pelo processo de indexação)
    - **file_id** (a URL do arquivo codificado)
    - **file_name** (o nome do arquivo extraído dos metadados do documento)
    - **language**: (o idioma no qual o documento está escrito)
    - **sentimento** a pontuação de sentimento calculada para o documento.
    - **URL** a URL para o blob de documentos no armazenamento do Azure.
3. Exiba as outras tabelas criadas pelo processo de indexação:
    - **ImageTags** (contém uma linha para cada marca de imagem individual com a **document_id** do documento em que a marca aparece).
    - **KeyPhrases** (contém uma linha para cada frase-chave individual com a **document_id** para o documento em que a frase aparece).
    - **Locations** (contém uma linha para cada localização individual com a **document_id** para o documento no qual a localização aparece).

A capacidade de criar projeções de *tabela* permite que você crie soluções analíticas e de relatório que consultam o esquema relacional, por exemplo, usando o Microsoft Power BI. As colunas de chave geradas automaticamente podem ser usadas para unir as tabelas em consultas – por exemplo, para retornar todos os locais mencionados em um documento específico.

## Mais informações

Para saber mais sobre como criar repositórios de conhecimento com a Pesquisa de IA do Azure, consulte a [documentação da Pesquisa de IA do Azure](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro).
