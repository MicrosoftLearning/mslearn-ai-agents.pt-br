---
lab:
  title: Desenvolver uma solução multiagente com a Fábrica de IA do Azure
  description: Saiba como configurar vários agentes para colaborar usando o Serviço de Agente da Fábrica de IA do Azure
---

# Desenvolva uma solução multiagente

Neste exercício, você criará um projeto que orquestra vários agentes de IA usando o Serviço de Agente da Fábrica de IA do Azure. Neste exercício, você criará um agente mestre de missões que é responsável por guiar um grupo através de uma masmorra. O grupo consiste em agentes de IA conectados que representam um guerreiro, um curandeiro e um batedor. O agente mestre da missão recebe um cenário do usuário e delega tarefas aos membros do grupo de acordo. Vamos começar!

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

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

    > **Observação**: se um erro de *permissões insuficientes** for exibido, use o botão **Corrigir** para resolvê-lo.

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
   cd ai-agents/Labfiles/06-build-multi-agent-solution/Python
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

1. Insira o seguinte comando para editar o arquivo **agent_chat.py**:

    ```
   code agent_quest.py
    ```

1. Examine o código no arquivo, observando que ele contém cadeias de caracteres para cada nome de agente e instruções.

1. Localize o comentário **Adicionar referências** e adicione o seguinte código para importar as classes necessárias:

    ```python
    # Add references
    from azure.ai.agents import AgentsClient
    from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder
    from azure.identity import DefaultAzureCredential
    ```

1. Localize o comentário **Criar o agente curandeiro no serviço do agente de IA do Azure** e adicione o seguinte código para criar um Agente de IA do Azure.

    ```python
    # Create the healer agent on the Azure AI agent service
    healer_agent = agents_client.create_agent(
        model=model_deployment,
        name=healer_agent_name,
        instructions=healer_instructions
    )
    ```

    Esse código cria a definição do agente no cliente do Projeto de IA do Azure.

1. Localize o comentário **Create a connected agent tool for the healer agent** e adicione o seguinte código:

    ```python
    # Create a connected agent tool for the healer agent
    healer_agent_tool = ConnectedAgentTool(
        id=healer_agent.id, 
        name=healer_agent_name, 
        description="Responsible for healing party members and addressing injuries."
    )
    ```

    Agora, vamos criar os outros agentes membros do grupo.

1. No comentário **Criar o agente batedor e a ferramenta conectada** e adicione o seguinte código:
    
    ```python
    # Create the scout agent and connected tool
    scout_agent = agents_client.create_agent(
        model=model_deployment,
        name=scout_agent_name,
        instructions=scout_instructions
    )
    scout_agent_tool = ConnectedAgentTool(
        id=scout_agent.id, 
        name=scout_agent_name, 
        description="Goes ahead of the main party to perform reconnaissance."
    )
    ```

1. No comentário **Criar o agente guerreiro e a ferramenta conectada**, adicione o seguinte código:
    
    ```python
    # Create the warrior agent and connected tool
    warrior_agent = agents_client.create_agent(
        model=model_deployment,
        name=warrior_agent_name,
        instructions=warrior_instructions
    )
    warrior_agent_tool = ConnectedAgentTool(
        id=warrior_agent.id, 
        name=warrior_agent_name, 
        description="Responds to combat or physical challenges."
    )
    ```


1. No comentário **Criar um agente principal com as ferramentas do Agente Conectado**, adicione o seguinte código:
    
    ```python
    # Create a main agent with the Connected Agent tools
    agent = agents_client.create_agent(
        model=model_deployment,
        name="quest_master",
        instructions="""
            You are the Questmaster, the intelligent guide of a three-member adventuring party exploring a short dungeon. 
            Based on the scenario, delegate tasks to the appropriate party member. The current party members are: Warrior, Scout, Healer.
            Only include the party member's response, do not provide an analysis or summary.
        """,
        tools=[
            healer_agent_tool.definitions[0],
            scout_agent_tool.definitions[0],
            warrior_agent_tool.definitions[0]
        ]
    )
    ```

1. Localize o comentário **Create thread for the chat session** e adicione o seguinte código:
    
    ```python
    # Create thread for the chat session
    print("Creating agent thread.")
    thread = agents_client.threads.create()
    ```


1. No comentário **Criar o prompt da missão**, adicione o seguinte código:
    
    ```python
    prompt = "We find a locked door with strange symbols, and the warrior is limping."
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
   python agent_quest.py
    ```

    Você verá algo semelhante à seguinte saída:

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    We find a locked door with strange symbols, and the warrior is limping.

    MessageRole.AGENT:
    - **Scout:** Decipher the celestial patterns of the strange symbols and determine the sequence to unlock the door. 
    - **Healer:** The warrior's injury has been addressed; moderate strain is relieved through healing magic and restorative salve.
    - **Warrior:** Recovered and ready to assist physically or guard the party as we proceed.

    Cleaning up agents:
    Deleted quest master agent.
    Deleted healer agent.
    Deleted scout agent.
    Deleted warrior agent.
    ```

    Você pode tentar modificar o prompt usando um cenário diferente, para ver como os agentes colaboram.

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.

1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.

1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
