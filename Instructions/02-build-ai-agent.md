---
lab:
  title: Implantar um agente de IA
  description: Use o Serviço de Agente de IA do Azure para desenvolver um agente que usa ferramentas internas.
---

# Implantar um agente de IA

Neste exercício, você usará o Serviço de Agente de IA do Azure para criar um agente simples que analisa dados e cria gráficos. O agente usa a ferramenta *Intérprete de código* para gerar dinamicamente o código necessário para criar gráficos como imagens e, em seguida, salva as imagens de gráfico resultantes.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar algum comportamento, avisos ou erros inesperados.

## Criar um projeto do Azure AI Foundry

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Na home page, clique em **Criar um agente**.
1. Quando solicitado a criar um projeto, insira um nome válido para o projeto e expanda **Opções avançadas**.
1. Confirme as seguintes configurações do projeto:
    - **Recurso da Fábrica de IA do Azure**: *um nome válido para o recurso da Fábrica de IA do Azure*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *Selecione qualquer **Local compatível com os Serviços de IA***\*

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto.
1. Quando o projeto for criado, o playground Agentes abrirá automaticamente para que você possa selecionar um implantar um modelo:

    ![Captura de tela do playground Agentes de um projeto da Fábrica de IA do Azure](./Media/ai-foundry-agents-playground.png)

    >**Observação**: um modelo base GPT-4o é implantado automaticamente ao criar o agente e o projeto.

1. No painel de navegação à esquerda, selecione **Visão geral** para ver a página principal do projeto, que será assim:

    > **Observação**: se um erro de *permissões insuficientes** for exibido, use o botão **Corrigir** para resolvê-lo.

    ![Captura de tela de uma página de visão geral do projeto da Fábrica de IA do Azure.](./Media/ai-foundry-project.png)

1. Copie os valores do **ponto de extremidade do projeto da Fábrica de IA do Azure** para um bloco de notas, pois você os usará para se conectar ao seu projeto em um aplicativo cliente.

## Criar um aplicativo cliente do agente

Agora você está pronto para criar um aplicativo cliente que usa um agente. Alguns códigos foram fornecidos para você em um repositório do GitHub.

### Clonar o repositório que contém o código do aplicativo

1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.

    Feche todas as notificações de boas-vindas para ver a home page do portal do Azure.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Você pode redimensionar ou maximizar esse painel para facilitar o trabalho.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):

    ```
   rm -r ai-agents -f
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents ai-agents
    ```

    > **Dica**: ao inserir comandos no Cloud Shell, a saída poderá ocupar uma grande quantidade do buffer da tela e o cursor na linha atual pode ficar obscurecido. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Digite o seguinte comando para alterar o diretório de trabalho para a pasta que contém os arquivos de código e relacione todos eles.

    ```
   cd ai-agents/Labfiles/02-build-ai-agent/Python
   ls -a -l
    ```

    Os arquivos fornecidos incluem código do aplicativo, definições de configuração e dados.

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do projeto (copiado da página **Visão Geral** no portal da Fábrica de IA do Azure).
1. Depois de substituir o espaço reservado, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Escrever código para um aplicativo de agente

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto. Use os níveis de recuo de comentário como guia.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code agent.py
    ```

1. Revise o código existente, que recupera as definições de configuração do aplicativo e carrega dados de *data.txt* a serem analisados. O restante do arquivo inclui comentários em que você adicionará o código necessário para implantar seu agente de análise de dados.
1. Localize o comentário **Adicionar referências** e adicione o seguinte código para importar as classes necessárias para criar um agente de IA do Azure que usa a ferramenta de intérprete de código interno:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FilePurpose, CodeInterpreterTool, ListSortOrder, MessageRole
    ```

1. Localize o comentário **Connect to the Agent client** e adicione o código a seguir para se conectar ao projeto da IA do Azure.

    > **Dica**: tenha cuidado para manter o nível de recuo correto.

    ```python
   # Connect to the Agent client
   agent_client = AgentsClient(
       endpoint=project_endpoint,
       credential=DefaultAzureCredential
           (exclude_environment_credential=True,
            exclude_managed_identity_credential=True)
   )
   with agent_client:
    ```

    O código se conecta ao projeto da Fábrica de IA do Azure usando as credenciais atuais do Azure. A instrução final *with agent_client* inicia um bloco de código que define o escopo do cliente, garantindo que ele seja limpo quando o código dentro do bloco for concluído.

1. Localize o comentário **Upload the data file and create a CodeInterpreterTool** dentro do bloco *with project_client* e adicione o seguinte código para fazer upload do arquivo de dados no projeto e criar uma CodeInterpreterTool que possa acessar os dados nele:

    ```python
   # Upload the data file and create a CodeInterpreterTool
   file = agent_client.files.upload_and_poll(
        file_path=file_path, purpose=FilePurpose.AGENTS
   )
   print(f"Uploaded {file.filename}")

   code_interpreter = CodeInterpreterTool(file_ids=[file.id])
    ```
    
1. Localize o comentário **Definir um agente que usa o CodeInterpreterTool** e adicione o seguinte código para definir um agente de IA que analisa dados e pode usar a ferramenta de interpretação de código definida anteriormente:

    ```python
   # Define an agent that uses the CodeInterpreterTool
   agent = agent_client.create_agent(
        model=model_deployment,
        name="data-agent",
        instructions="You are an AI agent that analyzes the data in the file that has been uploaded. If the user requests a chart, create it and save it as a .png file.",
        tools=code_interpreter.definitions,
        tool_resources=code_interpreter.resources,
   )
   print(f"Using agent: {agent.name}")
    ```

1. Localize o comentário **Create a thread for the conversation** e adicione o seguinte código para iniciar uma thread no qual a sessão de chat com o agente será executada:

    ```python
   # Create a thread for the conversation
   thread = agent_client.threads.create()
    ```
    
1. Observe que a próxima seção do código configura um loop para um usuário inserir um prompt, terminando quando o usuário digita "sair".

1. Localize o comentário **Enviar um prompt ao agente** e adicione o código a seguir para adicionar uma mensagem do usuário ao prompt (junto com os dados do arquivo que foi carregado anteriormente) e, em seguida, execute o thread com o agente.

    ```python
   # Send a prompt to the agent
   message = agent_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=user_prompt,
    )

   run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
     
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Localize o comentário **Show the latest response from the agent** e adicione o código a seguir para recuperar as mensagens da conversa concluída e exibir a última que foi enviada pelo agente.

    ```python
   # Show the latest response from the agent
   last_msg = agent_client.messages.get_last_message_text_by_role(
       thread_id=thread.id,
       role=MessageRole.AGENT,
   )
   if last_msg:
       print(f"Last Message: {last_msg.text.value}")
    ```

1. Localize o comentário **Get the conversation history**, que ocorre após o término do loop, e adicione o seguinte código para imprimir as mensagens da thread da conversa, invertendo a ordem para mostrá-las em sequência cronológica

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = agent_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
       if message.text_messages:
           last_msg = message.text_messages[-1]
           print(f"{message.role}: {last_msg.text.value}\n")
    ```

1. Localize o comentário **Obter todos os arquivos** gerados e adicione o código a seguir para obter todas as anotações de caminho de arquivo das mensagens (que indicam que o agente salvou um arquivo em seu armazenamento interno) e copie os arquivos para a pasta do aplicativo. _OBSERVAÇÃO:_ no momento, o conteúdo da imagem não está disponível pelo sistema.

    ```python
   # Get any generated files
   for msg in messages:
       # Save every image file in the message
       for img in msg.image_contents:
           file_id = img.image_file.file_id
           file_name = f"{file_id}_image_file.png"
           agent_client.files.save(file_id=file_id, file_name=file_name)
           print(f"Saved image file to: {Path.cwd() / file_name}")
    ```

1. Localize o comentário **Limpar** e adicione o código a seguir para excluir o agente e o thread quando não forem mais necessários.

    ```python
   # Clean up
   agent_client.delete_agent(agent.id)
    ```

1. Revise o código, usando os comentários para entender como:
    - Conecta-se ao projeto da Fábrica de IA
    - Carrega o arquivo de dados e cria uma ferramenta de interpretação de código que pode acessá-lo.
    - Cria um novo agente que usa a ferramenta de interpretação de código e tem instruções explícitas para analisar os dados e criar gráficos como arquivos .png.
    - Executa um thread com uma mensagem de prompt do usuário junto com os dados a serem analisados.
    - Verifica o status da execução caso haja uma falha
    - Recupera as mensagens da conversa concluída e exibe a última enviada pelo agente.
    - Exibe o histórico de conversas
    - Salva cada arquivo gerado.
    - Exclui o agente e a conversa quando não são mais necessários.

1. Salve o arquivo de código (*CTRL+S*) quando terminar. Você também pode fechar o editor de código (*CTRL+Q*); embora você possa querer mantê-lo aberto caso precise fazer alguma edição no código adicionado. Em ambos os casos, mantenha o painel de linha de comando do Cloud Shell aberto.

### Execute o aplicativo e entre no Azure.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para entrar no Azure.

    ```
    az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Consulte [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.
    
1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o hub da Fábrica de IA do Azure, se solicitado.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
    python agent.py
    ```
    
    O aplicativo é executado usando as credenciais da sessão autenticada do Azure para se conectar ao seu projeto e criar e executar o agente.

1. Quando solicitado, exiba os dados que o aplicativo carregou do arquivo de texto *data.txt* . Em seguida, digite um prompt, como por exemplo:

    ```
   What's the category with the highest cost?
    ```

    > **Dica**: se o aplicativo falhar porque o limite de taxa foi excedido, aguarde alguns segundos e tente novamente. Se não houver cota suficiente disponível em sua assinatura, o modelo talvez não consiga responder.

1. Exiba a resposta. Em seguida, insira outro prompt, desta vez solicitando um gráfico:

    ```
   Create a pie chart showing cost by category
    ```

    O agente deve usar seletivamente a ferramenta de interpretação de código conforme necessário, neste caso, para criar um gráfico com base em sua solicitação.

1. Você pode continuar a conversa, se quiser. A conversa está *com estado*, portanto, retém o histórico de conversas, o que significa que o agente tem o contexto completo para cada resposta. Quando terminar, digite `quit`.
1. Revise as mensagens de conversa que foram recuperadas da conversa e os arquivos que foram gerados.

1. Quando o aplicativo tiver sido finalizado, use o comando **download** do Cloud Shell para baixar cada arquivo .png que foi salvo no aplicativo. Por exemplo:

    ```
   download ./<file_name>.png
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo.

## Resumo

Neste exercício, você usou o SDK do Serviço do Agente de IA do Azure para criar um aplicativo cliente que usa um agente de IA. O agente usa a ferramenta intérprete de código integrada para executar código dinâmico que cria imagens.

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
