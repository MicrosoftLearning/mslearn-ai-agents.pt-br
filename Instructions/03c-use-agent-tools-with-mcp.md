---
lab:
  title: Conectar agentes de IA a um servidor MCP remoto
  description: Saiba como integrar ferramentas do Protocolo de Contexto de Modelo a agentes de IA
---

# Conectar agentes de IA a ferramentas usando o Protocolo MCP (Protocolo de Contexto de Modelo)

Neste exercício, você criará um agente que se conecta a um servidor MCP hospedado na nuvem. O agente usará a pesquisa baseada em IA para ajudar os desenvolvedores a encontrar respostas precisas e em tempo real na documentação oficial da Microsoft. Isso é útil para criar assistentes que dão suporte a desenvolvedores com diretrizes atualizadas sobre ferramentas como Azure, .NET e Microsoft 365. O agente usará a ferramenta `microsoft_docs_search` fornecida para consultar a documentação e retornar resultados relevantes.

> **Dica**: O código usado neste exercício baseia-se no exemplo repositório de suporte do MCP de serviço do Agente de IA do Azure. Confira as [demonstrações do Azure OpenAI](https://github.com/retkowsky/Azure-OpenAI-demos/blob/main/Azure%20Agent%20Service/9%20Azure%20AI%20Agent%20service%20-%20MCP%20support.ipynb) ou visite [Conectar-se a servidores do Protocolo de Contexto de Modelo](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools/model-context-protocol) para obter mais detalhes.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar algum comportamento, avisos ou erros inesperados.

## Criar um projeto da Fábrica de IA do Azure

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Na home page, clique em **Criar um agente**.
1. Quando solicitado a criar um projeto, insira um nome válido para o projeto e expanda **Opções avançadas**.
1. Confirme as seguintes configurações do projeto:
    - **Recurso da Fábrica de IA do Azure**: *um nome válido para o recurso da Fábrica de IA do Azure*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *Selecione qualquer um dos seguintes locais com suporte:* \*
      * Oeste dos EUA 2
      * Oeste dos EUA
      * Leste da Noruega
      * Norte da Suíça
      * Norte dos EAU
      * Sul da Índia

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto.
1. Se solicitado, implante um modelo **gpt-4o** usando a opção de implantação *Global Standard* ou *Standard* (dependendo da disponibilidade da sua cota).

    >**Observação**: Se a cota estiver disponível, um modelo base GPT-4o poderá ser implantado automaticamente ao criar seu agente e projeto.

1. Quando o projeto for criado, o playground de agentes será aberto.

1. No painel de navegação à esquerda, selecione **Visão geral** para ver a página principal do projeto, que será assim:

    ![Captura de tela de uma página de visão geral do projeto da Fábrica de IA do Azure.](./Media/ai-foundry-project.png)

1. Copie o valor do **ponto de extremidade do projeto da Fábrica de IA do Azure**. Você usará esse ponto de extremidade para se conectar ao projeto em um aplicativo cliente.

## Desenvolver um agente que use ferramentas de função MCP

Agora que você criou seu projeto na Fábrica de IA, vamos desenvolver um aplicativo que integra um agente de IA a um servidor MCP.

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
   cd ai-agents/Labfiles/03c-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt --pre azure-ai-projects mcp
    ```

    >**Observação:** você pode ignorar qualquer mensagem de aviso ou erro exibida durante a instalação da biblioteca.

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do seu projeto (copiado da página **Visão geral** do projeto no portal da Fábrica de IA do Azure) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida com o nome da implantação do seu modelo (que deve ser *gpt-4o*).

1. Depois de substituir o espaço reservado, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Conectar um Agente de IA do Azure a um servidor MCP remoto

Nesta tarefa, você vai se conectar a um servidor MCP remoto, preparar o agente de IA e executar um prompt de usuário.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code client.py
    ```

    O arquivo é aberto no Editor de Código.

1. Localize o comentário **Adicionar referências** e adicione o seguinte código para importar as classes:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import McpTool, ToolSet, ListSortOrder
    ```

1. Encontre o comentário **Connect to the agents client** e adicione o seguinte código para se conectar ao projeto da IA do Azure usando as credenciais atuais do Azure.

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True
        )
   )
    ```

1. No comentário **Inicializar ferramenta MCP do agente**, adicione o seguinte código:

    ```python
   # Initialize agent MCP tool
   mcp_tool = McpTool(
        server_label=mcp_server_label,
        server_url=mcp_server_url,
   )
    
   mcp_tool.set_approval_mode("never")
    
   toolset = ToolSet()
   toolset.add(mcp_tool)
    ```

    Esse código vai se conectar ao servidor MCP remoto do Microsft Learn Docs. Esse é um serviço hospedado na nuvem que permite que os clientes acessem informações confiáveis e atualizadas diretamente da documentação oficial da Microsoft.

1. No comentário **Create a new agent**, adicione o seguinte código:

    ```python
   # Create a new agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="my-mcp-agent",
        instructions="""
        You have access to an MCP server called `microsoft.docs.mcp` - this tool allows you to 
        search through Microsoft's latest official documentation. Use the available MCP tools 
        to answer questions and perform tasks."""
   )
    ```

    Nesse código, você dá instruções para o agente e fornece as definições da ferramenta MCO.

1. Localize o comentário **Criar thread para comunicação** e adicione o seguinte código:

    ```python
   # Create thread for communication
   thread = agents_client.threads.create()
   print(f"Created thread, ID: {thread.id}")
    ```

1. Localize o comentário **Criar uma mensagem no thread** e adicione o seguinte código:

    ```python
   # Create a message on the thread
   prompt = input("\nHow can I help?: ")
   message = agents_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=prompt,
   )
   print(f"Created message, ID: {message.id}")
    ```

1. Localize o comentário **Create and process agent run in thread with MCP tools** e adicione o seguinte código:

    ```python
   # Create and process agent run in thread with MCP tools
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id, toolset=toolset)
   print(f"Created run, ID: {run.id}")
    ```
    
    O Agente de IA invoca automaticamente as ferramentas MCP conectadas para processar a solicitação de prompt. Para ilustrar esse processo, o código fornecido sob o comentário **Exibir etapas de execução e chamadas de ferramenta** produzirá todas as ferramentas invocadas do servidor MCP.

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
   python client.py
    ```

1. Quando solicitado, insira uma solicitação de informação técnica, como:

    ```
    Give me the Azure CLI commands to create an Azure Container App with a managed identity.
    ```

1. Aguarde o agente processar seu prompt, usando o servidor MCP para encontrar uma ferramenta adequada para recuperar as informações solicitadas. Você verá algo semelhante à seguinte saída:

    ```
    Created agent, ID: <<agent-id>>
    MCP Server: mslearn at https://learn.microsoft.com/api/mcp
    Created thread, ID: <<thread-id>>
    Created message, ID: <<message-id>>
    Created run, ID: <<run-id>>
    Run completed with status: RunStatus.COMPLETED
    Step <<step1-id>> status: completed

    Step <<step2-id>> status: completed
    MCP Tool calls:
        Tool Call ID: <<tool-call-id>>
        Type: mcp
        Type: microsoft_docs_search


    Conversation:
    --------------------------------------------------
    ASSISTANT: You can use Azure CLI to create an Azure Container App with a managed identity (either system-assigned or user-assigned). Below are the relevant commands and workflow:

    ---

    ### **1. Create a Resource Group**
    '''azurecli
    az group create --name myResourceGroup --location eastus
    '''
    

    {{continued...}}

    By following these steps, you can deploy an Azure Container App with either system-assigned or user-assigned managed identities to integrate seamlessly with other Azure services.
    --------------------------------------------------
    USER: Give me the Azure CLI commands to create an Azure Container App with a managed identity.
    --------------------------------------------------
    Deleted agent
    ```

    Observe que o agente foi capaz de invocar a ferramenta MCP `microsoft_docs_search` automaticamente para atender à solicitação.

1. Você pode executar o aplicativo novamente (usando o comando `python client.py`) para solicitar informações diferentes. Em cada caso, o agente tentará localizar a documentação técnica usando a ferramenta MCP.

## Limpar

Agora que você concluiu o exercício, exclua os recursos de nuvem criados para evitar o uso desnecessário de recursos.

1. Abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
