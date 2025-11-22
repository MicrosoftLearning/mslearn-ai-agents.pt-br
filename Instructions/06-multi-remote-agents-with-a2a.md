---
lab:
  title: Conectar-se a agentes remotos com o protocolo A2A
  description: Use o protocolo A2A para colaborar com agentes remotos.
---

# Conectar-se a agentes remotos com o protocolo A2A

Neste exercício, você usará o Agente de IA do Azure com o protocolo A2A para criar agentes remotos simples que interagem entre si. Esses agentes ajudarão os escritores técnicos a preparar suas postagens no blog de desenvolvedores. Um agente de título gerará um título e um agente de estrutura de tópicos usará o título para desenvolver uma estrutura de tópicos concisa para o artigo. Vamos começar

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

## Criar um aplicativo do A2A

Agora você está pronto para criar um aplicativo cliente que usa um agente. Alguns códigos foram fornecidos para você em um repositório do GitHub.

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
   cd ai-agents/Labfiles/06-build-remote-agents-with-a2a/python
   ls -a -l
    ```

    Os arquivos fornecidos incluem:
    ```output
    python
    ├── outline_agent/
    │   ├── agent.py
    │   ├── agent_executor.py
    │   └── server.py
    ├── routing_agent/
    │   ├── agent.py
    │   └── server.py
    ├── title_agent/
    │   ├── agent.py
    |   ├── agent_executor.py
    │   └── server.py
    ├── client.py
    └── run_all.py
    ```

    Cada pasta de agente contém o código do agente de IA do Azure e um servidor para hospedar o agente. O **agente de roteamento** é responsável por descobrir e se comunicar com os agentes de **título** e **estrutura de tópicos**. O **cliente** permite que os usuários enviem prompts para o agente de roteamento. `run_all.py` inicia todos os servidores e executa o cliente.

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents a2a-sdk
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do seu projeto (copiado da página **Visão geral** do projeto no portal da Fábrica) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida com o nome da implantação do seu modelo (que deve ser *gpt-4o*).
1. Depois de substituir o espaço reservado, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Criar um agente detectável

Nesta tarefa, você cria o agente de título que ajuda os escritores a criar títulos populares para seus artigos. Você também define as habilidades e o cartão do agente exigidos pelo protocolo A2A para tornar o agente detectável.

1. Navegue até o diretório `title_agent`:

    ```
   cd title_agent
    ```

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto. Use os níveis de recuo de comentário como guia.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code agent.py
    ```

1. Localize o comentário **Criar o cliente de agentes** e adicione o seguinte código para se conectar ao projeto de IA do Azure:

    > **Dica**: tenha cuidado para manter o nível de recuo correto.

    ```python
   # Create the agents client
   self.client = AgentsClient(
       endpoint=os.environ['PROJECT_ENDPOINT'],
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       )
   )
    ```

1. Localize o comentário **Crie o agente de título** e adicione o seguinte código para criar o agente:

    ```python
   # Create the title agent
   self.agent = self.client.create_agent(
       model=os.environ['MODEL_DEPLOYMENT_NAME'],
       name='title-agent',
       instructions="""
       You are a helpful writing assistant.
       Given a topic the user wants to write about, suggest a single clear and catchy blog post title.
       """,
   )
    ```

1. Localize o comentário **Criar um thread para a sessão de chat** e adicione o seguinte código para criar o thread de chat:

    ```python
   # Create a thread for the chat session
   thread = self.client.threads.create()
    ```

1. Localize o comentário **Enviar mensagem de usuário** e adicione este código para enviar o prompt do usuário:

    ```python
   # Send user message
   self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. No comentário **Criar e executar o agente**, adicione o seguinte código para iniciar a geração de resposta do agente:

    ```python
   # Create and run the agent
   run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    O código fornecido no restante do arquivo processará e retornará a resposta do agente. 

1. Salve o arquivo de código (*CTRL + S*). Agora você está pronto para compartilhar as habilidades e o cartão do agente com o protocolo A2A. 

1. Insira o comando a seguir para editar o arquivo `server.py` do agente de título  

    ```
   code server.py
    ```

1. Localize o comentário **Definir habilidades do agente** e adicione o seguinte código para especificar a funcionalidade do agente:

    ```python
   # Define agent skills
   skills = [
       AgentSkill(
           id='generate_blog_title',
           name='Generate Blog Title',
           description='Generates a blog title based on a topic',
           tags=['title'],
           examples=[
               'Can you give me a title for this article?',
           ],
       ),
   ]
    ```

1. Localize o comentário **Criar cartão do agente** e adicione este código para definir os metadados que tornam o agente detectável:

    ```python
   # Create agent card
   agent_card = AgentCard(
       name='AI Foundry Title Agent',
       description='An intelligent title generator agent powered by Foundry. '
       'I can help you generate catchy titles for your articles.',
       url=f'http://{host}:{port}/',
       version='1.0.0',
       default_input_modes=['text'],
       default_output_modes=['text'],
       capabilities=AgentCapabilities(),
       skills=skills,
   )
    ```

1. Localize o comentário **Criar executor do agente** e adicione o seguinte código para inicializar o executor usando o cartão do agente:

    ```python
   # Create agent executor
   agent_executor = create_foundry_agent_executor(agent_card)
    ```

    O executor do agente atuará como um wrapper para o agente de título que você criou.

1. Localize o comentário **Criar manipulador de solicitações** e adicione o seguinte para lidar com solicitações de entrada usando o executor:

    ```python
   # Create request handler
   request_handler = DefaultRequestHandler(
       agent_executor=agent_executor, task_store=InMemoryTaskStore()
   )
    ```

1. No comentário **Criar aplicativo A2A**, adicione este código para criar a instância de aplicativo compatível com o A2A:

    ```python
   # Create A2A application
   a2a_app = A2AStarletteApplication(
       agent_card=agent_card, http_handler=request_handler
   )
    ```
    
    Esse código cria um servidor A2A que compartilhará as informações do agente de título e tratará as solicitações de entrada para esse agente usando o executor do agente de título.

1. Salve o arquivo de código (*CTRL+S*) quando terminar.

### Habilitar mensagens entre os agentes

Nesta tarefa, você usa o protocolo A2A para permitir que o agente de roteamento envie mensagens para os outros agentes. Você também vai permitir que o agente de título receba mensagens implementando a classe de executor do agente.

1. Navegue até o diretório `routing_agent`:

    ```
   cd ../routing_agent
    ```

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code agent.py
    ```

    O agente de roteamento atua como um orquestrador que trata mensagens do usuário e determina qual agente remoto deve processar a solicitação.

    Quando uma mensagem do usuário é recebida, o agente de roteamento:
    - Inicia um thread de conversa.
    - Usa o método `create_and_process` para avaliar o agente que melhor corresponda à mensagem do usuário.
    - A mensagem é roteada para o agente apropriado por HTTP usando a função `send_message`.
    - O agente remoto processa a mensagem e retorna uma resposta.

    O agente de roteamento finalmente captura a resposta e a retorna ao usuário por meio do thread.

    Observe que o método `send_message` é assíncrono e deve ser aguardado para que a execução do agente seja concluída com êxito.

1. Adicione o seguinte código sob o comentário **Recuperar o cliente A2A do agente remoto usando o nome do agente**:

    ```python
   # Retrieve the remote agent's A2A client using the agent name 
   client = self.remote_agent_connections[agent_name]
    ```

1. Localize o comentário **Construir a carga a ser enviada ao agente remoto** e adicione o seguinte código:

    ```python
   # Construct the payload to send to the remote agent
   payload: dict[str, Any] = {
       'message': {
           'role': 'user',
           'parts': [{'kind': 'text', 'text': task}],
           'messageId': message_id,
       },
   }
    ```

1. Localize o comentário **Encapsular a carga em um objeto SendMessageRequest** e adicione o seguinte código:

    ```python
   # Wrap the payload in a SendMessageRequest object
   message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. Adicione o seguinte código sob o comentário **Enviar a mensagem ao cliente do agente remoto e aguardar a resposta**:

    ```python
   # Send the message to the remote agent client and await the response
   send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```


1. Salve o arquivo de código (*CTRL+S*) quando terminar. Agora, o agente de roteamento pode descobrir e enviar mensagens para o agente de título. Vamos criar o código do executor do agente para tratar essas mensagens que chegam para o agente de roteamento.

1. Navegue até o diretório `title_agent`:

    ```
   cd ../title_agent
    ```

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code agent_executor.py
    ```

    A implementação de classe `AgentExecutor` deve conter os métodos `execute` e `cancel`. O método para cancelar foi fornecido para você. O método `execute` inclui um objeto `TaskUpdater` que gerencia eventos e sinais para o chamador quando a tarefa é concluída. Vamos adicionar a lógica para a execução da tarefa.

1. No método `execute`, adicione o seguinte código sob o comentário **Processar a solicitação**:

    ```python
   # Process the request
   await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. No método `_process_request`, adicione o seguinte código sob o comentário **Obter o agente de título**:

    ```python
   # Get the title agent
   agent = await self._get_or_create_agent()
    ```

1. Adicione o seguinte código sob o comentário **Atualizar o status da tarefa**:

    ```python
   # Update the task status
   await task_updater.update_status(
       TaskState.working,
       message=new_agent_text_message('Title Agent is processing your request...', context_id=context_id),
   )
    ```

1. Localize o comentário **Executar a conversa do agente** e adicione o seguinte código:

    ```python
   # Run the agent conversation
   responses = await agent.run_conversation(user_message)
    ```

1. Localize o comentário **Atualizar a tarefa com as respostas** e adicione o seguinte código:

    ```python
   # Update the task with the responses
   for response in responses:
       await task_updater.update_status(
           TaskState.working,
           message=new_agent_text_message(response, context_id=context_id),
       )
    ```

1. Localize o comentário **Marcar a tarefa como concluída** e adicione o seguinte código:

    ```python
   # Mark the task as complete
   final_message = responses[-1] if responses else 'Task completed.'
   await task_updater.complete(
       message=new_agent_text_message(final_message, context_id=context_id)
   )
    ```

    Agora o agente de título foi encapsulado com um executor de agente que o protocolo A2A usará para tratar as mensagens. Ótimo trabalho!

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
    cd ..
    python run_all.py
    ```
    
    O aplicativo é executado usando as credenciais da sessão autenticada do Azure para se conectar ao seu projeto e criar e executar o agente. Você deverá ver alguma saída de cada servidor à medida que ele for iniciado.

1. Aguarde até que o prompt de entrada seja exibido e insira um prompt como:

    ```
   Create a title and outline for an article about React programming.
    ```

    Após alguns instantes, você deverá ver uma resposta do agente com os resultados.

1. Insira `quit` para sair do programa e parar os servidores.
    
## Resumo

Neste exercício, você usou o SDK do Serviço do Agente de IA do Azure e o SDK do Python do A2A para criar uma solução remota de vários agentes. Você criou um agente compatível com A2A detectável e configurou um agente de roteamento para acessar as habilidades do agente. Você também implementou um executor de agente para gerenciar tarefas e processar mensagens A2A que chegavam. Ótimo trabalho!

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
