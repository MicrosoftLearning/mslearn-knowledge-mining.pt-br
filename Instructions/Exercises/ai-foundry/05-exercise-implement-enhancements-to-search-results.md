---
lab:
  title: Implementar aprimoramentos nos resultados da pesquisa
---

# Implementar aprimoramentos nos resultados da pesquisa

Você já tem um serviço de pesquisa usado por um aplicativo de reservas de pacotes de feriados. Você viu que a relevância dos resultados da pesquisa está afetando o número de reservas que você está recebendo. Recentemente, você também adicionou hotéis em Portugal, portanto, deseja oferecer o português como um idioma com suporte.

Neste exercício, você adicionará um perfil de pontuação para aprimorar a relevância dos resultados da pesquisa. Em seguida, você usará os Serviços de IA do Azure para adicionar descrições em português de todos os hotéis.

> **Observação**: para concluir este exercício, você precisará de uma assinatura do Microsoft Azure. Caso ainda não tenha uma, inscreva-se em uma avaliação gratuita em [https://azure.com/free](https://azure.com/free?azure-portal=true).

## Criar recursos do Azure

Você criará um serviço de Pesquisa de IA do Azure e importará amostras de dados de hotéis.

1. Entre no [portal do Azure](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true).
1. Selecione **+ Criar um recurso**.
1. Procure por **pesquisa** e, em seguida, selecione **Pesquisa de IA do Azure**.
1. Selecione **Criar**.
1. Selecione **Criar novo** em Grupo de recursos, nomeie-o como **learn-advanced-search**.
1. Em **Nome do serviço**, insira **advanced-search-service-12345**. O nome precisa ser globalmente exclusivo, portanto, adicione números aleatórios ao final dele.
1. Selecione uma região com suporte perto de você.
1. Use os valores padrão em **Tipo de preço**.
1. Selecione **Examinar + criar**.
1. Selecione **Criar**.
1. Aguarde a implantação dos recursos e selecione **Ir para o recurso**.

### Importar os dados de exemplo para o serviço de pesquisa

Importe os dados de exemplo.

1. No painel **Visão geral**, selecione **Importar dados**.

    ![Uma captura de tela que mostra o menu Importar dados.](../media/05-media/import-data-new.png)
1. No painel **Importar dados**, na lista de seleção **Fonte de dados**, selecione **Amostras**.
1. Selecione **hotels-sample**.

1. Na guia **Adicionar habilidades cognitivas (Opcional)**, expanda **Anexar Serviços de IA** e selecione **Criar novo recurso dos Serviços de IA**.

    ![Uma captura de tela mostrando a seleção e adição dos Serviços de IA do Azure.](../media/05-media/add-cognitive-services-new.png)

### Criar um Serviço de IA do Azure para dar suporte a traduções

1. Em uma nova guia, entre no portal do Azure.
1. Em **Grupo de recursos**, selecione **learn-advanced-search**.
1. Em **Região**, selecione a mesma região que escolheu para o serviço de pesquisa.
1. Em **Nome**, insira **learn-cognitive-translator-12345** ou qualquer nome que preferir. O nome precisa ser globalmente exclusivo, portanto, adicione números aleatórios ao final dele.
1. Em **Tipo de preço**, selecione **Standard S0**.
1. Marque **Ao marcar esta caixa, confirmo que li e compreendi todos os termos abaixo**.
1. Selecione **Examinar + criar**.
1. Selecione **Criar**.
1. Quando os recursos tiverem sido criados, feche a guia.

### Adicionar um enriquecimento de tradução

1. Na guia **Adicionar habilidades cognitivas (opcional)**, selecione Atualizar.
1. Selecione o novo serviço, **learn-cognitive-translator-12345**.
1. Expanda a seção **Adicionar enriquecimentos**.
    ![Uma captura de tela que mostra a adição de tradução em português.](../media/05-media/add-translation-enrichment-new.png)
1. Selecione **Traduzir texto**, altere a **Linguagem de Destino** para **Português** e o **Nome do campo** para **Description_pt**.
1. Selecione **Avançar: Personalizar índice de destino**.

### Alterar o campo para armazenar o texto traduzido

1. Na guia **Personalizar índice de destino**, role a página até a parte inferior da lista de campos e altere o **Analisador** para **Português (Portugal) – Microsoft** no campo **Description_pt**.
1. Selecione **Próximo: Criar um indexador**.
1. Selecione **Enviar**.

    O índice será criado, o indexador será executado e 50 documentos contendo exemplos de dados de hotéis serão importados.
1. No painel **Visão geral**, selecione **Índices** e escolha **hotels-sample-index**.
1. Selecione **Pesquisar** para ver o JSON de todos os documentos no índice.
1. Procure por **Description_pt** (você pode usar **CTRL + F** para isso) nos resultados e observe que essa não é uma tradução para o português da descrição em inglês, mas parece com algo assim:

    ```json
    "Description_pt": "45",
    ```

O portal do Azure pressupõe que o primeiro campo do documento precisa ser traduzido. Portanto, ele está usando a habilidade de tradução para traduzir a `HotelId`.

### Atualizar o conjunto de habilidades para traduzir o campo correto no documento

1. Na parte superior da página, selecione o serviço de pesquisa, link **advanced-search-service-12345 |Indexes**.
1. Selecione **Conjuntos de Habilidades** em Gerenciamento de pesquisa no painel esquerdo e selecione **hotels-sample-skillset**.
1. Edite o documento JSON, alterando a linha 11 para:

    ```json
    "context": "/document/Description",
    ```

1. Altere o padrão de idioma para o inglês na linha 12:

    ```json
    "defaultFromLanguageCode": "en",
    ```

1. Altere o campo de origem na linha 18 para:

    ```json
    "source": "/document/Description"
    ```

1. Clique em **Salvar**.
1. Na parte superior da página, selecione o serviço de pesquisa, link **advanced-search-service-12345 | Skillsets**.
1. No painel **Visão geral**, selecione **Indexadores** e **hotels-sample-indexer**.
1. Selecione **Definição do Indexador (JSON)**.
1. Altere o nome do campo de origem na linha 21 para:

    ```json
    "sourceFieldName": "/document/Description/Description_pt",
    ```

1. Clique em **Salvar**.
1. Selecione **Redefinir**, depois **Sim**.
1. Selecione **Executar** e depois selecione **Sim**.

### Testar o índice atualizado

1. Na parte superior da página, selecione o serviço de pesquisa, link **advanced-search-service-12345 | Indexers**.
1. No painel **Visão geral**, selecione **Índices** e escolha **hotels-sample-index**.
1. Selecione **Pesquisar** para ver o JSON de todos os documentos no índice.
1. Procure **Description_pt** nos resultados e observe que agora há uma descrição em português.

    ```json
    "Description_pt": "O maior resort durante todo o ano da área oferecendo mais de tudo para suas férias – pelo melhor valor!  O que você pode desfrutar enquanto estiver no resort, além das praias de areia de 1,5 km do lago? Confira nossas atividades com certeza para excitar tanto os jovens quanto os jovens hóspedes do coração. Temos tudo, incluindo ser chamado de \"Propriedade do Ano\" e um \"Top Ten Resort\" pelas principais publicações.",
    ```

1. Agora você buscará hotéis que tenham vista para lagos. Começaremos usando uma pesquisa simples que retorna apenas o `HotelName`, a `Description`, a `Category` e as `Tags`. Em **Cadeia de consulta**, insira esta pesquisa:

    `lake + view&$select=HotelName,Description,Category,Tags&$count=true`

    Analise os resultados e tente localizar os campos que corresponderam aos termos de pesquisa `lake` e `view`. Observe este hotel e a posição dele:

    ```json
    {
      "@search.score": 0.9433406,
      "HotelName": "Lady Of The Lake B & B",
      "Description": "Nature is Home on the beach.  Save up to 30 percent. Valid Now through the end of the year. Restrictions and blackout may apply.",
      "Category": "Luxury",
      "Tags": [
        "laundry service",
        "concierge",
        "view"
      ]
    },
    ```

Esse hotel correspondeu ao termo "lago" no campo `HotelName` e à vista no campo `Tags`. Você deseja aumentar as correspondências de termos no campo `Description` em relação ao nome do hotel. O ideal é que esse hotel seja o último nos resultados.

## Adicionar um perfil de pontuação para aprimorar os resultados da pesquisa

1. Selecione a guia **Perfis de pontuação**.
1. Selecione **+ Adicionar perfil de pontuação**.
1. Em **Nome do perfil**, insira **boost-description-categories**.
1. Adicione os seguintes campos e pesos em **Pesos**:

    ![Uma captura de tela dos pesos que estão sendo adicionados a um perfil de pontuação.](../media/05-media/add-weights-new.png)
1. Em **Nome do campo**, selecione **Descrição**.
1. Em **Peso**, insira **5**.
1. Em **Nome do campo**, selecione **Categoria**.
1. Em **Peso**, insira **3**.
1. Em **Nome do campo**, selecione **Marcas**.
1. Em **Peso**, insira **2**.
1. Selecione **Salvar**.
1. Selecione **Salvar** na parte superior.

### Testar o índice atualizado

1. Retorne à guia **Gerenciador de pesquisa** da página **hotels-sample-index**.
1. Em **Cadeia de consulta**, insira a mesma pesquisa de antes:

    `lake + view&$select=HotelName,Description,Category,Tags&$count=true`

    Verifique os resultados da pesquisa.

    ```json
    {
      "@search.score": 3.5707965,
      "HotelName": "Lady Of The Lake B & B",
      "Description": "Nature is Home on the beach.  Save up to 30 percent. Valid Now through the end of the year. Restrictions and blackout may apply.",
      "Category": "Luxury",
      "Tags": [
        "laundry service",
        "concierge",
        "view"
      ]
    }
    ```

    A pontuação de pesquisa aumentou, de **0,9433406** para **3,5707965**. No entanto, todos os outros hotéis têm pontuações calculadas mais altas. Este hotel é agora o último nos resultados.

## Limpar

Agora que você concluiu o exercício, exclua todos os recursos de que não precisa mais.

1. No portal do Azure, selecione **Grupos de Recursos**.
1. Selecione o grupo de recursos que você não precisa mais, depois selecione **Excluir grupo de recursos**.
