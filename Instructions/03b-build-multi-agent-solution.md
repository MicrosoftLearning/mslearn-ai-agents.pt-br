---
lab:
  title: Desenvolver uma solução multiagente com a Fábrica de IA do Azure
  description: Saiba como configurar vários agentes para colaborar usando o Serviço de Agente da Fábrica de IA do Azure
---

# Desenvolva uma solução multiagente

Neste exercício, você criará um projeto que orquestra vários agentes de IA usando o Serviço de Agente da Fábrica de IA do Azure. Você criará uma solução de IA que ajuda na triagem de tíquetes. Os agentes conectados irão avaliar a prioridade do tíquete, sugerir uma equipe para atribuição e determinar o nível de esforço necessário para concluir o tíquete. Vamos começar!

> **Dica**: O código usado neste exercício é baseado no SDK para Python da Fábrica de IA do Azure. Você pode desenvolver soluções semelhantes usando os SDKs para Microsoft .NET, JavaScript e Java. Consulte as [bibliotecas de clientes do SDK da Fábrica de IA do Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para obter mais detalhes.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar algum comportamento, avisos ou erros inesperados.

## Implantar um modelo em um projeto da Fábrica de IA do Azure

Vamos começar implantando um modelo em um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Na home page, na seção **Explorar modelos e recursos**, pesquise pelo modelo `gpt-4o`, que usaremos em nosso projeto.
1. Nos resultados da pesquisa, selecione o modelo **gpt-4o** para ver os detalhes e, na parte superior da página do modelo, clique em **Usar este modelo**.
1. Quando solicitado a criar um projeto, insira um nome válido para o projeto e expanda **Opções avançadas**.
1. Confirme as seguintes configurações do projeto:
    - **Recurso da Fábrica de IA do Azure**: *um nome válido para o recurso da Fábrica de IA do Azure*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *Selecione qualquer **Local compatível com os Serviços de IA***\*

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto, incluindo a implantação do modelo gpt-4 selecionado.
1. Quando o projeto for criado, o playground chat abrirá automaticamente.

    > **Observação**: a configuração padrão do TPM para este modelo pode ser muito baixa para este exercício. Um TPM baixo ajuda a evitar o uso excessivo da cota disponível na assinatura que você está usando. 

1. No painel de navegação à esquerda, clique em **Modelos e pontos de extremidade** e selecione a implantação **gpt-4o**.

1. Clique em **Editar** e aumente o **Limite de taxa de tokens por minuto**

   > **OBSERVAÇÃO**: 40.000 TPM são suficientes para os dados usados neste exercício. Se a sua cota disponível for menor que isso, você poderá concluir o exercício, mas talvez seja necessário aguardar e reenviar as solicitações se o limite de taxa for excedido.

1. No painel de navegação à esquerda, selecione **Visão geral** para ver a página principal do projeto, que será assim:

    ![Captura de tela de uma página de visão geral do projeto da Fábrica de IA do Azure.](./Media/ai-foundry-project.png)

1. Copie o valor do **ponto de extremidade do projeto da Fábrica de IA do Azure** para um bloco de notas, pois você o usará para se conectar ao seu projeto em um aplicativo cliente.

## Criar um aplicativo cliente do Agente de IA

Agora você está pronto para criar um aplicativo cliente que define os agentes e as instruções. Algum código é fornecido para você em um repositório GitHub.

### Preparar o ambiente

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

    > **Dica**: ao digitar comandos no Cloud Shell, a saída pode ocupar uma grande parte do buffer da tela e o cursor da linha atual pode ficar oculto. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Quando o repositório tiver sido clonado, digite o seguinte comando para alterar o diretório de trabalho para a pasta que contém os arquivos de código e relacione todos eles.

    ```
   cd ai-agents/Labfiles/03b-build-multi-agent-solution/Python
   ls -a -l
    ```

    Os arquivos fornecidos incluem o código do aplicativo e um arquivo para definições de configuração.

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects
    ```

1. Digite o seguinte comando para editar o arquivo de configuração fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do projeto (copiado da página **Visão Geral** do projeto no portal da Fábrica de IA do Azure) e o espaço reservado **your_model_deployment** pelo nome que você atribuiu à implantação do modelo gpt-4.

1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Criar agentes de IA

Agora você está pronto para criar os agentes para sua solução multiagente! Vamos começar!

1. Insira o seguinte comando para editar o arquivo **agent_triage.py**:

    ```
   code agent_triage.py
    ```

1. Examine o código no arquivo, observando que ele contém cadeias de caracteres para cada nome de agente e instruções.

1. Localize o comentário **Adicionar referências** e adicione o seguinte código para importar as classes necessárias:

    ```python
    # Add references
    from azure.ai.agents import AgentsClient
    from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder, ToolSet, FunctionTool
    from azure.identity import DefaultAzureCredential
    ```

1. Abaixo do comentário **Instruções para o agente principal**, insira o seguinte código:

    ```python
    # Instructions for the primary agent
    triage_agent_instructions = """
    Triage the given ticket. Use the connected tools to determine the ticket's priority, 
    which team it should be assigned to, and how much effort it may take.
    """
    ```

1. Localize o comentário **Criar o agente de prioridade no serviço de agente de IA do Azure** e adicione o seguinte código para criar um agente de IA do Azure.

    ```python
    # Create the priority agent on the Azure AI agent service
    priority_agent = agents_client.create_agent(
        model=model_deployment,
        name=priority_agent_name,
        instructions=priority_agent_instructions
    )
    ```

    Esse código cria a definição do agente no cliente do Projeto de IA do Azure.

1. Localize o comentário **Criar uma ferramenta de agente conectado para o agente de prioridade** e adicione o seguinte código:

    ```python
    # Create a connected agent tool for the priority agent
    priority_agent_tool = ConnectedAgentTool(
        id=priority_agent.id, 
        name=priority_agent_name, 
        description="Assess the priority of a ticket"
    )
    ```

    Agora vamos criar os outros agentes de triagem.

1. Abaixo do comentário **Criar o agente de equipe e a ferramenta conectada**, adicione o seguinte código:
    
    ```python
    # Create the team agent and connected tool
    team_agent = agents_client.create_agent(
        model=model_deployment,
        name=team_agent_name,
        instructions=team_agent_instructions
    )
    team_agent_tool = ConnectedAgentTool(
        id=team_agent.id, 
        name=team_agent_name, 
        description="Determines which team should take the ticket"
    )
    ```

1. Abaixo do comentário **Criar o agente de esforço e a ferramenta conectada**, adicione o seguinte código:
    
    ```python
    # Create the effort agent and connected tool
    effort_agent = agents_client.create_agent(
        model=model_deployment,
        name=effort_agent_name,
        instructions=effort_agent_instructions
    )
    effort_agent_tool = ConnectedAgentTool(
        id=effort_agent.id, 
        name=effort_agent_name, 
        description="Determines the effort required to complete the ticket"
    )
    ```


1. No comentário **Criar um agente principal com as ferramentas do Agente Conectado**, adicione o seguinte código:
    
    ```python
    # Create a main agent with the Connected Agent tools
    agent = agents_client.create_agent(
        model=model_deployment,
        name="triage-agent",
        instructions=triage_agent_instructions,
        tools=[
            priority_agent_tool.definitions[0],
            team_agent_tool.definitions[0],
            effort_agent_tool.definitions[0]
        ]
    )
    ```

1. Localize o comentário **Create thread for the chat session** e adicione o seguinte código:
    
    ```python
    # Create thread for the chat session
    print("Creating agent thread.")
    thread = agents_client.threads.create()
    ```


1. Abaixo do comentário **Criar o prompt de tíquete**, adicione o seguinte código:
    
    ```python
    # Create the ticket prompt
    prompt = "Users can't reset their password from the mobile app."

    ```

1. No comentário **Enviar um prompt para o agente**, adicione o seguinte código:
    
    ```python
    # Send a prompt to the agent
    message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=prompt,
    )
    ```

1. No comentário **Criar e processar o Agente, execute a thread com ferramentas**, adicione o seguinte código:
    
    ```python
    # Create and process Agent run in thread with tools
    print("Processing agent thread. Please wait.")
    run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    ```


1. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código. Você pode mantê-lo aberto (caso precise editar o código para corrigir erros) ou usar o comando **CTRL+Q** para fechar o editor de código enquanto mantém a linha de comando do cloud shell aberta.

### Execute o aplicativo e entre no Azure.

Agora você está pronto para executar seu código e ver seus agentes de IA colaborarem.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para entrar no Azure.

    ```
   az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Consulte [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o hub da Fábrica de IA do Azure, se solicitado.

1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python agent_triage.py
    ```

    Você verá algo semelhante à seguinte saída:

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    Users can't reset their password from the mobile app.

    MessageRole.AGENT:
    ### Ticket Assessment

    - **Priority:** High — This issue blocks users from resetting their passwords, limiting access to their accounts.
    - **Assigned Team:** Frontend Team — The problem lies in the mobile app's user interface or functionality.
    - **Effort Required:** Medium — Resolving this problem involves identifying the root cause, potentially updating the mobile app functionality, reviewing API/backend integration, and testing to ensure compatibility across Android/iOS platforms.

    Cleaning up agents:
    Deleted triage agent.
    Deleted priority agent.
    Deleted team agent.
    Deleted effort agent.
    ```

    Você pode tentar modificar o prompt de tíquete usando um cenário diferente, para ver como os agentes colaboram. Por exemplo, "Investigar erros 502 ocasionais no ponto de extremidade de pesquisa".

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.

1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.

1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
