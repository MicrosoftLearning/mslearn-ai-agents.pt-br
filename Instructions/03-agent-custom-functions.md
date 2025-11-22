---
lab:
  title: Usar uma função personalizada em um agente de IA
  description: Saiba como usar funções para adicionar recursos personalizados aos seus agentes.
---

# Usar uma função personalizada em um agente de IA

Neste exercício, você explorará a criação de um agente que pode usar funções personalizadas como uma ferramenta para concluir tarefas. Você criará um agente de suporte técnico simples que pode coletar detalhes de um problema técnico e gerar um tíquete de suporte.

> **Dica**: O código usado neste exercício é baseado no SDK para Python da Fábrica da Microsoft. Você pode desenvolver soluções semelhantes usando os SDKs para Microsoft .NET, JavaScript e Java. Consulte as [bibliotecas de clientes do SDK da Fábrica da Microsoft](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para obter detalhes.

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

1. Copie os valores do **ponto de extremidade do projeto da Fábrica** para um bloco de notas, pois você os usará para se conectar ao seu projeto em um aplicativo cliente.

## Desenvolver um agente que use ferramentas de função

Agora que você criou seu projeto na Fábrica de IA, vamos desenvolver um aplicativo que implementa um agente usando ferramentas de função personalizadas.

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
   cd ai-agents/Labfiles/03-ai-agent-functions/Python
   ls -a -l
    ```

    Os arquivos fornecidos incluem o código do aplicativo e um arquivo para definições de configuração.

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents
    ```

    >**Observação:** você pode ignorar qualquer mensagem de aviso ou erro exibida durante a instalação da biblioteca.

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do seu projeto (copiado da página **Visão geral** do projeto no portal da Fábrica) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida com o nome da implantação do seu modelo (que deve ser *gpt-4o*).
1. Depois de substituir o espaço reservado, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Definir uma função personalizada

1. Digite o seguinte comando para editar o arquivo de código fornecido para a sua função:

    ```
   code user_functions.py
    ```

1. Localize o comentário **Criar uma função para enviar um tíquete de suporte** e adicione o código a seguir, que gera um número de tíquete e salva um tíquete de suporte como um arquivo de texto.

    ```python
   # Create a function to submit a support ticket
   def submit_support_ticket(email_address: str, description: str) -> str:
        script_dir = Path(__file__).parent  # Get the directory of the script
        ticket_number = str(uuid.uuid4()).replace('-', '')[:6]
        file_name = f"ticket-{ticket_number}.txt"
        file_path = script_dir / file_name
        text = f"Support ticket: {ticket_number}\nSubmitted by: {email_address}\nDescription:\n{description}"
        file_path.write_text(text)
    
        message_json = json.dumps({"message": f"Support ticket {ticket_number} submitted. The ticket file is saved as {file_name}"})
        return message_json
    ```

1. Localize o comentário **Define a set of callable functions** e adicione o código a seguir, que define estaticamente um conjunto de funções que podem ser chamadas neste arquivo de código (nesse caso, há apenas uma, mas em uma solução real, você pode ter várias funções que seu agente pode chamar):

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. Salve o arquivo (*CTRL+S*).

### Escreva código para implantar um agente que possa usar sua função

1. Digite o comando a seugir para começar a editiar o código de agente.

    ```
    code agent.py
    ```

    > **Dica**: ao adicionar código ao arquivo, certifique-se de manter o recuo correto.

1. Revise o código existente, que recupera as definições de configuração do aplicativo e configura um loop no qual o usuário pode inserir prompts para o agente. O restante do arquivo inclui comentários em que você adicionará o código necessário para implantar seu agente de suporte técnico.
1. Localize o comentário **Adicionar referências** e adicione o seguinte código para importar as classes necessárias para criar um agente de IA do Azure que usa seu código de função como uma ferramenta:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, ToolSet, ListSortOrder, MessageRole
   from user_functions import user_functions
    ```

1. Encontre o comentário **Connect to the Agent client** e adicione o seguinte código para se conectar ao projeto da IA do Azure usando as credenciais atuais do Azure.

    > **Dica**: tenha cuidado para manter o nível de recuo correto.

    ```python
   # Connect to the Agent client
   agent_client = AgentsClient(
       endpoint=project_endpoint,
       credential=DefaultAzureCredential
           (exclude_environment_credential=True,
            exclude_managed_identity_credential=True)
   )
    ```
    
1. Localize o comentário **Definir um agente que possa usar a seção de funções** personalizadas e adicione o código a seguir para adicionar seu código de função a um conjunto de ferramentas e, em seguida, crie um agente que possa usar o conjunto de ferramentas e uma conversa na qual executar a sessão de chat.

    ```python
   # Define an agent that can use the custom functions
   with agent_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
        agent_client.enable_auto_function_calls(toolset)
            
        agent = agent_client.create_agent(
            model=model_deployment,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = agent_client.threads.create()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. Localize o comentário **Enviar um prompt ao agente** e adicione o código a seguir para adicionar o prompt do usuário como uma mensagem e executar o thread.

    ```python
   # Send a prompt to the agent
   message = agent_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    ```

    > **Observação**: usar o método **create_and_process_run** para executar a thread permite que o agente encontre automaticamente as suas funções e opte por usá-las com base em seus nomes e parâmetros. Como alternativa, você pode usar o método **create_run**, caso em que você seria responsável por escrever código para sondar o status de execução para determinar quando uma chamada de função é necessária, chamar a função e retornar os resultados ao agente.

1. Encontre o comentário **Verificar o status da execução para ver se há falhas** e adicione o seguinte código para mostrar qualquer erro que ocorra.

    ```python
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

1. Localize o comentário **Get the conversation history** e adicione o seguinte código para imprimir as mensagens da thread da conversa, ordenando-as em sequência cronológica

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = agent_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
           last_msg = message.text_messages[-1]
           print(f"{message.role}: {last_msg.text.value}\n")
    ```

1. Localize o comentário **Limpar** e adicione o código a seguir para excluir o agente e o thread quando não forem mais necessários.

    ```python
   # Clean up
   agent_client.delete_agent(agent.id)
   print("Deleted agent")
    ```

1. Revise o código, usando os comentários para entender como:
    - Adiciona seu conjunto de funções personalizadas a um conjunto de ferramentas
    - Cria um agente que usa o conjunto de ferramentas.
    - Executa uma conversa com uma mensagem de prompt do usuário.
    - Verifica o status da execução caso haja uma falha
    - Recupera as mensagens da conversa concluída e exibe a última enviada pelo agente.
    - Exibe o histórico de conversas
    - Exclui o agente e a conversa quando não são mais necessários.

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
   python agent.py
    ```
    
    O aplicativo é executado usando as credenciais da sessão autenticada do Azure para se conectar ao seu projeto e criar e executar o agente.

1. Quando solicitado, insira um prompt como:

    ```
   I have a technical problem
    ```

    > **Dica**: se o aplicativo falhar porque o limite de taxa foi excedido, aguarde alguns segundos e tente novamente. Se não houver cota suficiente disponível em sua assinatura, o modelo talvez não consiga responder.

1. Exiba a resposta. O agente pode solicitar seu endereço de e-mail e uma descrição do problema. Você pode usar qualquer endereço de email (por exemplo, `alex@contoso.com`) e qualquer descrição de problema (por exemplo `my computer won't start`)

    Quando tiver informações suficientes, o agente deverá optar por usar sua função conforme necessário.

1. Você pode continuar a conversa, se quiser. A conversa está *com estado*, portanto, retém o histórico de conversas, o que significa que o agente tem o contexto completo para cada resposta. Quando terminar, digite `quit`.
1. Revise as mensagens de conversa que foram recuperadas do encadeamento e os tickets que foram gerados.
1. A ferramenta deve ter salvado tíquetes de suporte na pasta do aplicativo. Você pode usar o `ls`comando para verificar e, em seguida, usar o`cat` comando para exibir o conteúdo do arquivo, assim:

    ```
   cat ticket-<ticket_num>.txt
    ```

## Limpar

Agora que você concluiu o exercício, exclua os recursos de nuvem criados para evitar o uso desnecessário de recursos.

1. Abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
