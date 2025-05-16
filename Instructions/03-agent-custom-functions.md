---
lab:
  title: Usar uma função personalizada em um agente de IA
  description: Saiba como usar funções para adicionar recursos personalizados aos seus agentes.
---

# Usar uma função personalizada em um agente de IA

Neste exercício, você explorará a criação de um agente que pode usar funções personalizadas como uma ferramenta para concluir tarefas.

Você criará um agente de suporte técnico simples que pode coletar detalhes de um problema técnico e gerar um tíquete de suporte.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar algum comportamento, avisos ou erros inesperados.

## Criar um projeto do Azure AI Foundry

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Na home page, selecione **+Criar projeto**.
1. No assistente **Criar um projeto**, insira um nome de projeto adequado e, se um hub existente for sugerido, escolha a opção de criar um novo. Em seguida, examine os recursos do Azure que serão criados automaticamente para dar suporte ao hub e ao projeto.
1. Selecione **Personalizar** e especifique as seguintes configurações para o hub:
    - **Nome do hub**: *um nome válido para o seu hub*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Localização**: selecione uma das seguintes regiões\*:
        - eastus
        - eastus2
        - swedencentral
        - westus
        - westus3
    - **Conectar os Serviços de IA do Azure ou o OpenAI do Azure**: *Criar um novo recurso de Serviços de IA*
    - **Conectar-se à Pesquisa de IA do Azure**: Ignorar a conexão

    > \* No momento da redação deste artigo, essas regiões dão suporte ao modelo gpt-4o para uso em agentes. A disponibilidade do modelo é limitada por cotas regionais. No caso de um limite de cota ser atingido mais adiante no exercício, há a possibilidade de você precisar criar outro projeto em uma região diferente.

1. Clique em **Avançar** e revise a configuração. Em seguida, selecione **Criar** e aguarde a conclusão do processo.
1. Quando o projeto for criado, feche todas as dicas exibidas e examine a página do projeto no Portal da Fábrica de IA do Azure, que deve ser semelhante à imagem a seguir:

    ![Captura de tela dos detalhes de um projeto IA do Azure no Portal da Fábrica de IA do Azure.](./Media/ai-foundry-project.png)

## Implantar um modelo de IA generativa

Agora você está pronto para implantar um modelo de linguagem de IA generativo para dar suporte ao seu agente.

1. No painel à esquerda do seu projeto, na seção **Meus ativos**, selecione a página **Modelos + pontos de extremidade**.
1. Na página **Modelos + pontos extremidades**, na guia **Implantações de modelo**, no menu **+ Implantar modelo**, selecione **Implantar modelo base**.
1. Procure o modelo **gpt-4o** na lista, selecione-o e confirme-o.
1. Crie uma nova implantação do modelo com as seguintes configurações selecionando **Personalizar** nos detalhes de implantação:
    - **Nome da implantação**: *Um nome válido para a implantação de modelo*
    - **Tipo de implantação**: padrão global
    - **Atualização automática de versão**: Ativado
    - **Versão do modelo**: *selecione a versão mais recente disponivel*
    - **Recurso de IA conectado**: *selecione a sua conexão de recursos do OpenAI do Azure*
    - **Limite de taxa de tokens por minuto (milhares):** 50 mil *(ou o máximo disponível em sua assinatura, se inferior a 50 mil)*
    - **Filtro de conteúdo**: DefaultV2

    > **Observação**: A redução do TPM ajuda a evitar o uso excessivo da cota disponível na assinatura que você está usando. 50.000 TPM são suficientes para os dados usados neste exercício. Se a sua cota disponível for menor que isso, você poderá concluir o exercício, mas talvez seja necessário aguardar e reenviar as solicitações se o limite de taxa for excedido.

1. Aguarde até que a implantação seja concluída.

## Desenvolver um agente que use ferramentas de função

Agora que você criou seu projeto na Fábrica de IA, vamos desenvolver um aplicativo que implementa um agente usando ferramentas de função personalizadas.

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
   cd ai-agents/Labfiles/03-ai-agent-functions/Python
   ls -a -l
    ```

    Os arquivos fornecidos incluem o código do aplicativo e um arquivo para definições de configuração.

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects
    ```

    >**Observação:** você pode ignorar qualquer mensagem de aviso ou erro exibida durante a instalação da biblioteca.

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_connection_string** pela cadeia de conexão do seu projeto (copiada da página **Visão Geral** do projeto no portal da Fábrica de IA do Azure), e o espaço reservado **your_model_deployment** pelo nome que você atribuiu à implantação do seu modelo gpt-4.
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

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
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FunctionTool, ToolSet
   from user_functions import user_functions
    ```

1. Encontre o comentário **Conectar ao projeto Fábrica de IA do Azure** e adicione o seguinte código para se conectar ao projeto do Azure AI usando as credenciais atuais do Azure.

    > **Dica**: tenha cuidado para manter o nível de recuo correto.

    ```python
   # Connect to the Azure AI Foundry project
   project_client = AIProjectClient.from_connection_string(
        credential=DefaultAzureCredential
            (exclude_environment_credential=True,
             exclude_managed_identity_credential=True),
        conn_str=PROJECT_CONNECTION_STRING
   )
    ```
    
1. Localize o comentário **Definir um agente que possa usar a seção de funções** personalizadas e adicione o código a seguir para adicionar seu código de função a um conjunto de ferramentas e, em seguida, crie um agente que possa usar o conjunto de ferramentas e uma conversa na qual executar a sessão de chat.

    ```python
   # Define an agent that can use the custom functions
   with project_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
            
        agent = project_client.agents.create_agent(
            model=MODEL_DEPLOYMENT,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = project_client.agents.create_thread()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. Localize o comentário **Enviar um prompt ao agente** e adicione o código a seguir para adicionar o prompt do usuário como uma mensagem e executar o thread.

    ```python
   # Send a prompt to the agent
   message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)
    ```

    > **Observação**: usar o **método create_and_process_run** para executar a conversa permite que o agente encontre automaticamente suas funções e opte por usá-las com base em seus nomes e parâmetros. Como alternativa, você pode usar o método **create_run**, caso em que você seria responsável por escrever código para sondar o status de execução para determinar quando uma chamada de função é necessária, chamar a função e retornar os resultados ao agente.

1. Encontre o comentário **Verificar o status da execução para ver se há falhas** e adicione o seguinte código para mostrar qualquer erro que ocorra.

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Localize o comentário **Show the latest response from the agent** e adicione o código a seguir para recuperar as mensagens da conversa concluída e exibir a última que foi enviada pelo agente.

    ```python
   # Show the latest response from the agent
   messages = project_client.agents.list_messages(thread_id=thread.id)
   last_msg = messages.get_last_text_message_by_role("assistant")
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. Localize o comentário **Get the conversation history**, que ocorre após o término do loop, e adicione o seguinte código para imprimir as mensagens da thread da conversa, invertendo a ordem para mostrá-las em sequência cronológica

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = project_client.agents.list_messages(thread_id=thread.id)
   for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
    ```

1. Localize o comentário **Limpar** e adicione o código a seguir para excluir o agente e o thread quando não forem mais necessários.

    ```python
   # Clean up
   project_client.agents.delete_agent(agent.id)
   project_client.agents.delete_thread(thread.id)
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
    
1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o hub da Fábrica de IA do Azure, se solicitado.
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