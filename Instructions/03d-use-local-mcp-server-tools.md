# Conectar agentes de IA a ferramentas usando o Protocolo MCP (Protocolo de Contexto de Modelo)

Neste exercício, você irá criar um agente que pode se conectar a um servidor MCP e descobrir automaticamente funções chamáveis.

Você irá criar um agente simples de avaliação de estoque para uma loja de cosméticos. Usando o servidor MCP, o agente poderá recuperar informações sobre o estoque e fazer sugestões de reposição ou liquidação.

> **Dica**: O código usado neste exercício é baseado nos SDKs para Python da Fábrica da Microsoft e MCP. Você pode desenvolver soluções semelhantes usando os SDKs para Microsoft .NET. Consulte as [bibliotecas de clientes do SDK da Fábrica da Microsoft](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) e o [SDK do MCP em C#](https://modelcontextprotocol.github.io/csharp-sdk/api/ModelContextProtocol.html) para obter mais detalhes.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar algum comportamento, avisos ou erros inesperados.

## Criar um projeto da Fábrica

Vamos começar criando um projeto da Fábrica.

1. Em um navegador da Web, abra o [portal da Fábrica](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal da Fábrica.](./Media/ai-foundry-home.png)

    > **Importante**: Verifique se a alternância **Nova Fábrica** está *desativada* para este laboratório.

1. Na home page, clique em **Criar um agente**.
1. Quando solicitado a criar um projeto, insira um nome válido para o projeto e expanda **Opções avançadas**.
1. Confirme as seguintes configurações do projeto:
    - **Recurso Fábrica**: *Um nome válido para o recurso Fábrica*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *selecione qualquer **AI Foundry recomendado***\*

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto.
1. Se solicitado, implante um modelo **gpt-4o** usando a opção de implantação *Global Standard* ou *Standard* (dependendo da disponibilidade da sua cota).

    >**Observação**: Se a cota estiver disponível, um modelo base GPT-4o poderá ser implantado automaticamente ao criar seu agente e projeto.

1. Quando o projeto for criado, o playground de agentes será aberto.

1. No painel de navegação à esquerda, selecione **Visão geral** para ver a página principal do projeto, que será assim:

    ![Captura de tela de uma página de visão geral do projeto da Fábrica.](./Media/ai-foundry-project.png)

1. Copie o valor do **ponto de extremidade do projeto da Fábrica** em um bloco de notas, pois você o usará para se conectar ao seu projeto em um aplicativo cliente.

## Desenvolver um agente que use ferramentas de função MCP

Agora que você criou seu projeto na Fábrica de IA, vamos desenvolver um aplicativo que integra um agente de IA a um servidor MCP.

### Clonar o repositório que contém o código do aplicativo

1. Abra uma nova guia do navegador (mantendo o portal da Fábrica aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.

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
   cd ai-agents/Labfiles/03d-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

    Os arquivos fornecidos incluem o código do aplicativo para servidores e clientes. O Protocolo de Contexto de Modelo fornece uma maneira padronizada de conectar modelos de IA a diferentes fontes de dados e ferramentas. Separamos `client.py` e `server.py` para manter a lógica do agente e as definições das ferramentas modulares e simular uma arquitetura real. 
    
    `server.py` define as ferramentas que o agente pode usar, simulando serviços de back-end ou lógica de negócios. 
    `client.py` lida com a configuração do agente de IA, as solicitações do usuário e a chamada das ferramentas quando necessário.

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects mcp
    ```

    >**Observação:** você pode ignorar qualquer mensagem de aviso ou erro exibida durante a instalação da biblioteca.

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do seu projeto (copiado da página **Visão geral** do projeto no portal da Fábrica) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida com o nome da implantação do seu modelo (que deve ser *gpt-4o*).

1. Depois de substituir o espaço reservado, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Implementar um servidor MCP

Um Servidor MCP (Protocolo de Contexto de Modelo) é um componente que hospeda ferramentas acionáveis. Essas ferramentas são funções Python que podem ser expostas para agentes de IA. Quando as ferramentas são anotadas com `@mcp.tool()`, elas se tornam detectáveis pelo cliente, permitindo que um agente de IA as chame dinamicamente durante uma conversa ou tarefa. Nesta tarefa, você irá adicionar algumas ferramentas que permitirão ao agente realizar verificações de inventário.

1. Digite o seguinte comando para editar o arquivo de código fornecido para a sua função:

    ```
   code server.py
    ```

    Neste arquivo de código, você irá definir as ferramentas que o agente pode usar para simular um serviço de back-end para a loja de varejo. Observe o código de configuração do servidor na parte superior do arquivo. Ele usa `FastMCP` para criar rapidamente uma instância de servidor MCP chamada "Inventory". Este servidor hospedará as ferramentas que você definir e as tornará acessíveis ao agente durante o laboratório.

1. Encontre o comentário **Adicionar uma ferramenta de verificação de inventário** e adicione o seguinte código:

    ```python
   # Add an inventory check tool
   @mcp.tool()
   def get_inventory_levels() -> dict:
        """Returns current inventory for all products."""
        return {
            "Moisturizer": 6,
            "Shampoo": 8,
            "Body Spray": 28,
            "Hair Gel": 5, 
            "Lip Balm": 12,
            "Skin Serum": 9,
            "Cleanser": 30,
            "Conditioner": 3,
            "Setting Powder": 17,
            "Dry Shampoo": 45
        }
    ```

    Esse dicionário representa um inventário de exemplo. A anotação `@mcp.tool()` permitirá que o LLM descubra sua função. 

1. Encontre o comentário **Adicionar uma ferramenta de vendas semanal** e adicione o seguinte código:

    ```python
   # Add a weekly sales tool
   @mcp.tool()
   def get_weekly_sales() -> dict:
        """Returns number of units sold last week."""
        return {
            "Moisturizer": 22,
            "Shampoo": 18,
            "Body Spray": 3,
            "Hair Gel": 2,
            "Lip Balm": 14,
            "Skin Serum": 19,
            "Cleanser": 4,
            "Conditioner": 1,
            "Setting Powder": 13,
            "Dry Shampoo": 17
        }
    ```

1. Salve o arquivo (*CTRL+S*).

### Implementar um cliente MCP

Um cliente MCP é o componente que se conecta ao servidor MCP para descobrir e chamar ferramentas. Você pode pensar nele como a ponte entre o agente e as funções hospedadas no servidor, permitindo o uso dinâmico de ferramentas em resposta aos comandos do usuário.

1. Digite o comando a seguir para começar a editar o código do cliente.

    ```
   code client.py
    ```

    > **Dica**: ao adicionar código ao arquivo, certifique-se de manter o recuo correto.

1. Encontre o comentário **Adicionar referências** e adicione o seguinte código para importar as classes:

    ```python
   # Add references
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, MessageRole, ListSortOrder
   from azure.identity import DefaultAzureCredential
    ```

1. Encontre o comentário **Start the MCP server** e adicione o seguinte código:

    ```python
   # Start the MCP server
   stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
   stdio, write = stdio_transport
    ```

    Em uma configuração padrão de produção, o servidor seria executado separadamente do cliente. Mas, para os fins deste laboratório, o cliente é responsável por iniciar o servidor usando transporte padrão de entrada/saída. Isso cria um canal de comunicação leve entre os dois componentes e simplifica a configuração de desenvolvimento local.

1. Encontre o comentário **Create an MCP client session** e adicione o seguinte código:

    ```python
   # Create an MCP client session
   session = await exit_stack.enter_async_context(ClientSession(stdio, write))
   await session.initialize()
    ```

    Isso cria uma nova sessão de cliente usando os fluxos de entrada e saída da etapa anterior. Chamar `session.initialize` prepara a sessão para descobrir e chamar as ferramentas que estão registradas no servidor MCP.

1. Abaixo do comentário **List available tools**, adicione o seguinte código para verificar se o cliente se conectou ao servidor:

    ```python
   # List available tools
   response = await session.list_tools()
   tools = response.tools
   print("\nConnected to server with tools:", [tool.name for tool in tools]) 
    ```

    Agora sua sessão de cliente está pronta para ser usada com seu agente de IA do Azure.

### Conectar as ferramentas MCP ao seu agente

Nesta tarefa, você irá preparar o agente de IA, receberá os prompts dos usuários e invocará as ferramentas de função.

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

1. Abaixo do comentário **List tools available on the server**, adicione o seguinte código:

    ```python
   # List tools available on the server
   response = await session.list_tools()
   tools = response.tools
    ```

1. Abaixo do comentário **Build a function for each tool**, adicione o seguinte código:

    ```python
   # Build a function for each tool
   def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            result = await session.call_tool(tool_name, kwargs)
            return result
        
        tool_func.__name__ = tool_name
        return tool_func

   functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
   mcp_function_tool = FunctionTool(functions=list(functions_dict.values()))
    ```

    Este código encapsula dinamicamente as ferramentas disponíveis no servidor MCP para que possam ser chamadas pelo agente de IA. Cada ferramenta é transformada em uma função assíncrona e, em seguida, agrupada em um `FunctionTool` para o agente usar.

1. Encontre o comentário **Create the agent** e adicione o seguinte código:

    ```python
   # Create the agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="inventory-agent",
        instructions="""
        You are an inventory assistant. Here are some general guidelines:
        - Recommend restock if item inventory < 10  and weekly sales > 15
        - Recommend clearance if item inventory > 20 and weekly sales < 5
        """,
        tools=mcp_function_tool.definitions
   )
    ```

1. Encontre o comentário **Enable auto function calling** e adicione o seguinte código:

    ```python
   # Enable auto function calling
   agents_client.enable_auto_function_calls(tools=mcp_function_tool)
    ```

1. Abaixo do comentário **Create a thread for the chat session**, adicione o seguinte código:

    ```python
   # Create a thread for the chat session
   thread = agents_client.threads.create()
    ```

1. Encontre o comentário **Invoke the prompt** e adicione o seguinte código:

    ```python
   # Invoke the prompt
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=user_input,
   )
   run = agents_client.runs.create(thread_id=thread.id, agent_id=agent.id)
    ```

1. Encontre o comentário **Retrieve the matching function tool** e adicione o seguinte código:

    ```python
   # Retrieve the matching function tool
   function_name = tool_call.function.name
   args_json = tool_call.function.arguments
   kwargs = json.loads(args_json)
   required_function = functions_dict.get(function_name)

   # Invoke the function
   output = await required_function(**kwargs)
    ```

    Este código usa as informações da chamada de ferramenta da thread do agente. O nome da função e os argumentos são recuperados e usados para invocar a função correspondente.

1. Abaixo do comentário **Append the output text**, adicione o seguinte código:

    ```python
   # Append the output text
   tool_outputs.append({
        "tool_call_id": tool_call.id,
        "output": output.content[0].text,
   })
    ```

1. Abaixo do comentário **Submit the tool call output**, adicione o seguinte código:

    ```python
   # Submit the tool call output
   agents_client.runs.submit_tool_outputs(thread_id=thread.id, run_id=run.id, tool_outputs=tool_outputs)
    ```

    Este código sinalizará à thread do agente que a ação necessária foi concluída e atualizará as saídas da chamada da ferramenta.

1. Encontre o comentário **Display the response** e adicione o seguinte código:

    ```python
   # Display the response
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
    ```

1. Salve o arquivo de código (*CTRL+S*) quando terminar. Você também pode fechar o editor de código (*CTRL+Q*); embora você possa querer mantê-lo aberto caso precise fazer alguma edição no código adicionado. Em ambos os casos, mantenha o painel de linha de comando do Cloud Shell aberto.

### Execute o aplicativo e entre no Azure.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para entrar no Azure.

    ```
   az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Consulte [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.
    
1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o hub da Fábrica, se solicitado.

1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python client.py
    ```

1. Quando solicitado, insira uma consulta como:

    ```
   What are the current inventory levels?
    ```

    > **Dica**: se o aplicativo falhar porque o limite de taxa foi excedido, aguarde alguns segundos e tente novamente. Se não houver cota suficiente disponível em sua assinatura, o modelo talvez não consiga responder.

    Você deve ver alguma saída semelhante à seguinte:

    ```
    MessageRole.AGENT:
    Here are the current inventory levels:

    - Moisturizer: 6
    - Shampoo: 8
    - Body Spray: 28
    - Hair Gel: 5
    - Lip Balm: 12
    - Skin Serum: 9
    - Cleanser: 30
    - Conditioner: 3
    - Setting Powder: 17
    - Dry Shampoo: 45
    ```

1. Você pode continuar a conversa, se quiser. A conversa está *com estado*, portanto, retém o histórico de conversas, o que significa que o agente tem o contexto completo para cada resposta. 

    Tente inserir prompts como:

    ```
   Are there any products that should be restocked?
    ```

    ```
   Which products would you recommend for clearance?
    ```

    ```
   What are the best sellers this week?
    ```

    Quando terminar, digite `quit`.

## Limpar

Agora que você concluiu o exercício, exclua os recursos de nuvem criados para evitar o uso desnecessário de recursos.

1. Abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
