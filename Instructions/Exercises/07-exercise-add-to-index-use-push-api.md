---
lab:
  title: Adicionar a um índice usando a API de push
---

# Adicionar a um índice usando a API de push

Você deseja explorar como criar um índice de Pesquisa de IA do Azure e carregar documentos para esse índice usando código C#.

Neste exercício, você clonará uma solução em C# existente e a executará para descobrir o tamanho ideal do lote para carregar documentos. Em seguida, você usará esse tamanho de lote e carregará documentos efetivamente usando uma abordagem com threading.

> **Observação** para concluir este exercício, você precisará de uma assinatura do Microsoft Azure. Caso ainda não tenha uma, inscreva-se em uma avaliação gratuita em [https://azure.com/free](https://azure.com/free?azure-portal=true) .

## Configurar os recursos do Azure

Para economizar tempo, selecione este modelo do Azure Resource Manager para criar os recursos que serão necessários posteriormente no exercício:

1. [Implantar recursos no Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%20lab-files%2Fazuredeploy.json) – selecione este link para criar os seus recursos de IA do Azure.
    ![Uma captura de tela das opções mostradas quando os recursos são implantados no Azure.](../media/07-media/deploy-azure-resources.png)
1. Em **Grupo de recursos**, selecione **Criar** e dê a ele o nome **cog-search-language-exe**.
1. Em **Região**, selecione uma [região com suporte](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability) próxima a você.
1. O **Prefixo do recurso** precisa ser globalmente exclusivo. Insira um prefixo com caracteres numéricos aleatórios e em letras minúsculas, por exemplo, **acs118245**.
1. Em **Local**, selecione a mesma região escolhida acima.
1. Selecione **Examinar + criar**.
1. Selecione **Criar**.
1. Quando a implantação for concluída, selecione **Ir para o grupo de recursos** para ver todos os recursos que você criou.

    ![Uma captura de tela que mostra todos os recursos implantados do Azure.](../media/07-media/azure-resources-created.png)

## Copiar informações da API REST do serviço Pesquisa de IA do Azure

1. Na lista de recursos, selecione o serviço de pesquisa que você criou. No exemplo acima, **acs118245-search-service**.
1. Copie o nome do serviço de pesquisa para um arquivo de texto.

    ![Uma captura de tela da seção de chaves de um serviço de pesquisa.](../media/07-media/search-api-keys-exercise-version.png)
1. À esquerda, selecione **Chaves** e copie a **Chave de administração primária** no mesmo arquivo de texto.

## Baixar código de exemplo para usar no Visual Studio Code

Você executará um código de exemplo do Azure usando o Visual Studio Code. Os arquivos de código foram fornecidos em um repositório do GitHub.

1. Inicie o Visual Studio Code.
1. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` em uma pasta local (não importa qual pasta).
1. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
1. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**.

1. Na navegação à esquerda, expanda a pasta **optimize-data-indexing/v11/OptimizeDataIndexing** e selecione o arquivo **appsettings.json**.

    ![Uma captura de tela que mostra o conteúdo do arquivo appsettings.json.](../media/07-media/update-app-settings.png)
1. Cole o nome do serviço de pesquisa e a chave de administração primária.

    ```json
    {
      "SearchServiceUri": "https://acs118245-search-service.search.windows.net",
      "SearchServiceAdminApiKey": "YOUR_SEARCH_SERVICE_KEY",
      "SearchIndexName": "optimize-indexing"
    }
    ```

    O arquivo de configurações deve ser semelhante ao acima.
1. Salve sua alteração pressionando **Ctrl + S**.
1. Clique com o botão direito do mouse na pasta **OptimizeDataIndexing** e selecione **Abrir no Terminal Integrado**.
1. No terminal, insira `dotnet run` e pressione **Enter**.

    ![Uma captura de tela que mostra o aplicativo em execução no VS Code com uma exceção.](../media/07-media/debug-application.png)
A saída mostra que, nesse caso, o tamanho do lote com o melhor desempenho é de 900 documentos. Ele atinge 6,071 MB por segundo.

## Editar o código para implementar o threading e uma estratégia de retirada e repetição

Há um código comentado que está pronto para alterar o aplicativo para usar threads para carregar documentos no índice de pesquisa.

1. Verifique se você selecionou **Program.cs**.

    ![Uma captura de tela do VS Code que mostra o arquivo Program.cs.](../media/07-media/edit-program-code.png)
1. Comente as linhas 37 e 38 desta forma:

    ```csharp
    //Console.WriteLine("{0}", "Finding optimal batch size...\n");
    //await TestBatchSizesAsync(searchClient, numTries: 3);
    ```

1. Remova a marca de comentário das linhas 44 a 48.

    ```csharp
    Console.WriteLine("{0}", "Uploading using exponential backoff...\n");
    await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8);

    Console.WriteLine("{0}", "Validating all data was indexed...\n");
    await ValidateIndexAsync(indexClient, indexName, numDocuments);
    ```

    O código que controla o tamanho do lote e o número de threads é `await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8)`. O tamanho do lote é 1000 e os threads são oito.

    ![Uma captura de tela que mostra todo o código editado.](../media/07-media/thread-code-ready.png)
    O código será parecido com o acima.

1. Para salvar suas alterações, pressione **CTRL**+**S**.
1. Selecione seu terminal e depois pressione qualquer tecla para encerrar o processo em execução, se ainda não o tiver feito.
1. Execute `dotnet run` no terminal.

    O aplicativo iniciará oito threads e, à medida que cada thread terminar de gravar uma nova mensagem no console:

    ```powershell
    Finished a thread, kicking off another...
    Sending a batch of 1000 docs starting with doc 57000...
    ```

    Depois que 100.000 documentos forem carregados, o aplicativo gravará um resumo (isso pode demorar um pouco para completar):

    ```powershell
    Ended at: 9/1/2023 3:25:36 PM
    
    Upload time total: 00:01:18:0220862
    Upload time per batch: 780.2209 ms
    Upload time per document: 0.7802 ms
    
    Validating all data was indexed...
    
    Waiting for service statistics to update...
    
    Document Count is 100000
    
    Waiting for service statistics to update...
    
    Index Statistics: Document Count is 100000
    Index Statistics: Storage Size is 71453102
    
    ``````

Explore o código no procedimento `TestBatchSizesAsync` para ver como o código testa o desempenho do tamanho do lote.

Explore o código no procedimento `IndexDataAsync` para ver como o código gerencia o threading.

Explore o código no `ExponentialBackoffAsync` para ver como o código implementa uma estratégia de retirada e repetição exponencial.

Você pode pesquisar e verificar se os documentos foram adicionados ao índice no portal do Azure.

![Uma captura de tela que mostra o índice de pesquisa com 100 mil documentos.](../media/07-media/check-search-service-index.png)

## Limpar

Agora que você concluiu o exercício, exclua todos os recursos de que não precisa mais. Comece com o código clonado em seu computador. Em seguida, exclua os recursos do Azure.

1. No **portal do Azure**, selecione Grupos de recursos.
1. Selecione o grupo de recursos que você criou para este exercício.
1. Selecione **Excluir grupo de recursos**. 
1. Confirme a exclusão e depois clique em **Excluir**.
1. Selecione os recursos de que você não precisa e selecione **Excluir**.
