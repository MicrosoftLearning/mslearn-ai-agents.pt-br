---
lab:
  title: Desenvolva uma solução multiagente
  description: Saiba como configurar vários agentes para colaborar usando o SDK do Semantic Kernel
---

# Desenvolva uma solução multiagente

Neste exercício, você criará um projeto que orquestra dois agentes de IA usando o SDK do Kernel Semântico. Um agente do *Gerente de Incidentes* analisará os arquivos de log de serviço em busca de problemas. Se um problema for encontrado, o Gerente de Incidentes recomendará uma ação de resolução e um agente do *Assistente de DevOps* receberá a recomendação e invocará a função corretiva e executará a resolução. O agente do Gerente de Incidentes revisará os logs atualizados para garantir que a resolução tenha sido bem-sucedida.

Para este exercício, quatro arquivos de log de exemplo são fornecidos. O código do agente do Assistente DevOps atualiza apenas os arquivos de log de exemplo com algumas mensagens de log de exemplo.

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

Agora você está pronto para implantar um modelo de linguagem de IA generativa para dar suporte aos seus agentes.

1. No painel à esquerda do seu projeto, na seção **Meus ativos**, selecione a página **Modelos + pontos de extremidade**.
1. Na página **Modelos + pontos extremidades**, na guia **Implantações de modelo**, no menu **+ Implantar modelo**, selecione **Implantar modelo base**.
1. Procure o modelo **gpt-4o** na lista, selecione-o e confirme-o.
1. Crie uma nova implantação do modelo com as seguintes configurações selecionando **Personalizar** nos detalhes de implantação:
    - **Nome da implantação**: *Um nome válido para a implantação de modelo*
    - **Tipo de implantação**: padrão global
    - **Atualização automática de versão**: Ativado
    - **Versão do modelo**: *selecione a versão mais recente disponivel*
    - **Recurso de IA conectado**: *selecione a sua conexão de recursos do OpenAI do Azure*
    - **Limite de taxa de tokens por minuto (milhares):** 60 mil *(ou o máximo disponível em sua assinatura, se inferior a 60 mil)*
    - **Filtro de conteúdo**: DefaultV2

    > **Observação**: A redução do TPM ajuda a evitar o uso excessivo da cota disponível na assinatura que você está usando. 60.000 TPM são suficientes para os dados usados neste exercício. Se a sua cota disponível for menor que isso, você poderá concluir o exercício, mas talvez seja necessário aguardar e reenviar as solicitações se o limite de taxa for excedido.

1. Aguarde até que a implantação seja concluída.

## Criar um aplicativo cliente do Agente de IA

Agora você está pronto para criar um aplicativo cliente que define um agente e uma função personalizada. Algum código é fornecido para você em um repositório GitHub.

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
   cd ai-agents/Labfiles/05-agent-orchestration/Python
   ls -a -l
    ```

    Os arquivos fornecidos incluem o código do aplicativo e um arquivo para definições de configuração.

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    > **Observação**: a instalação *do semantic-kernel[azure]* instala automaticamente uma versão semântica compatível com kernel do *azure-ai-projects*.

1. Digite o seguinte comando para editar o arquivo de configuração fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_connection_string** pela cadeia de conexão do seu projeto (copiada da página **Visão Geral** do projeto no portal da Fábrica de IA do Azure), e o espaço reservado **your_model_deployment** pelo nome que você atribuiu à implantação do seu modelo gpt-4.

1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Criar agentes de IA

Agora você está pronto para criar os agentes para sua solução multiagente! Vamos começar!

1. Insira o seguinte comando para editar o arquivo ** agent_chat.py**:

    ```
   code agent_chat.py
    ```

1. Revise o código no arquivo, observando que ele contém:
    - Constantes que definem os nomes e as instruções para seus dois agentes.
    - Uma **função** principal em que a maior parte do código para implantar sua solução multiagente será adicionada.
    - Uma classe **SelectionStrategy** , que você usará para implementar a lógica necessária para determinar qual agente deve ser selecionado para cada turno na conversa.
    - Uma classe **ApprovalTerminationStrategy**, que você usará para implementar a lógica necessária para determinar quando a conversa terminará.
    - Uma classe **DevopsPlugin** que contém funções para executar operações de DevOps.
    - Uma classe **LogFilePlugin** que contém funções para ler e gravar arquivos de log.

    Em primeiro lugar, você criará o agente *Gerente de Incidentes*, que analisará os arquivos de registro de serviço, identificará possíveis problemas e recomendará ações de resolução ou escalonará os problemas quando necessário.

1. Observe a cadeia de caracteres **INCIDENT_MANAGER_INSTRUCTIONS**. Estas são as instruções para o seu agente.

1. Na função **principal**, localize o comentário **Criar o agente do gerente de incidentes no serviço** do agente de IA do Azure e adicione o código a seguir para criar um Agente de IA do Azure.

    ```python
   # Create the incident manager agent on the Azure AI agent service
   incident_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=INCIDENT_MANAGER,
        instructions=INCIDENT_MANAGER_INSTRUCTIONS
   )
    ```

    Esse código cria a definição do agente no cliente do Projeto de IA do Azure.

1. Localize o comentário **Criar um agente de Kernel Semântico para o agente** do gerenciador de incidentes de IA do Azure e adicione o código a seguir para criar um agente de Kernel Semântico com base na definição do Agente de IA do Azure.

    ```python
   # Create a Semantic Kernel agent for the Azure AI incident manager agent
   agent_incident = AzureAIAgent(
        client=client,
        definition=incident_agent_definition,
        plugins=[LogFilePlugin()]
   )
    ```

    Esse código cria o agente do Kernel Semântico com acesso ao **LogFilePlugin**. Esse plug-in permite que o agente leia o conteúdo do arquivo de log.

    Agora vamos criar o segundo agente, que responderá aos problemas e executará operações de DevOps para resolvê-los.

1. Na parte superior do arquivo de código, reserve um momento para observar a cadeia de caracteres **DEVOPS_ASSISTANT_INSTRUCTIONS**. Estas são as instruções que você fornecerá ao novo agente assistente de DevOps.

1. Localize o comentário **Criar o agente de DevOps no serviço do agente de IA do Azure** e adicione o seguinte código para criar uma definição do Agente de IA do Azure:
    
    ```python
   # Create the devops agent on the Azure AI agent service
   devops_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=DEVOPS_ASSISTANT,
        instructions=DEVOPS_ASSISTANT_INSTRUCTIONS,
   )
    ```

1. Localize o comentário **Criar um agente de Kernel Semântico para o agente de IA do Azure de DevOps** e adicione o código a seguir para criar um agente de Kernel Semântico com base na definição do Agente de IA do Azure.
    
    ```python
   # Create a Semantic Kernel agent for the devops Azure AI agent
   agent_devops = AzureAIAgent(
        client=client,
        definition=devops_agent_definition,
        plugins=[DevopsPlugin()]
   )
    ```

    O **DevopsPlugin** permite que o agente simule tarefas de DevOps, como reiniciar o serviço ou reverter uma transação.

### Definir estratégias de bate-papo em grupo

Agora você precisa fornecer a lógica usada para determinar qual agente deve ser selecionado para fazer o próximo turno em uma conversa e quando a conversa deve ser encerrada.

Vamos começar com o **SelectionStrategy**, que identifica qual agente deve fazer o próximo turno.

1. Na classe **SelectionStrategy** (abaixo da função **main**), localize o comentário **Select the next agent that should take the next turn in the chat** e adicione o seguinte código para definir uma função de seleção:

    ```python
   # Select the next agent that should take the next turn in the chat
   async def select_agent(self, agents, history):
        """"Check which agent should take the next turn in the chat."""

        # The Incident Manager should go after the User or the Devops Assistant
        if (history[-1].name == DEVOPS_ASSISTANT or history[-1].role == AuthorRole.USER):
            agent_name = INCIDENT_MANAGER
            return next((agent for agent in agents if agent.name == agent_name), None)
        
        # Otherwise it is the Devops Assistant's turn
        return next((agent for agent in agents if agent.name == DEVOPS_ASSISTANT), None)
    ```

    Esse código é executado em cada turno para determinar qual agente deve responder, verificando o histórico de bate-papo para ver quem respondeu por último.

    Agora vamos implementar a classe **ApprovalTerminationStrategy** para ajudar a sinalizar quando a meta for concluída e a conversa puder ser encerrada.

1. Na classe **ApprovalTerminationStrategy**, encontre o comentário **End the chat if the agent has indicated there is no action needed** e adicione o código seguinte para definir a função de terminação:

    ```python
   # End the chat if the agent has indicated there is no action needed
   async def should_agent_terminate(self, agent, history):
        """Check if the agent should terminate."""
        return "no action needed" in history[-1].content.lower()
    ```

    O kernel invoca essa função após a resposta do agente para determinar se os critérios de conclusão foram atendidos. Nesse caso, a meta é atingida quando o gerente de incidentes responde com "Nenhuma ação necessária". Essa frase é definida nas instruções do agente do gerenciador de incidentes.

### Implantar o bate-papo em grupo

Agora que você tem dois agentes e estratégias para ajudá-los a se revezar e encerrar um chat, você pode implantar o chat em grupo.

1. Faça backup na função principal, localize o comentário **Adicionar os agentes a um chat em grupo com uma estratégia** personalizada de encerramento e seleção e adicione o seguinte código para criar o chat em grupo:

    ```python
   # Add the agents to a group chat with a custom termination and selection strategy
   chat = AgentGroupChat(
        agents=[agent_incident, agent_devops],
        termination_strategy=ApprovalTerminationStrategy(
            agents=[agent_incident], 
            maximum_iterations=10, 
            automatic_reset=True
        ),
        selection_strategy=SelectionStrategy(agents=[agent_incident,agent_devops]),      
   )
    ```

    Neste código, você cria um objeto de chat de grupo de agentes com o gerenciador de incidentes e os agentes de DevOps. Você também define as estratégias de encerramento e seleção para o chat. Observe que o **ApprovalTerminationStrategy** está vinculado apenas ao agente do gerenciador de incidentes, e não ao agente de devops. Isso faz com que o agente do gerenciador de incidentes seja responsável por sinalizar o fim do chat. O **SelectionStrategy** inclui todos os agentes que devem se revezar no chat.

    Observe que o sinalizador de reinicialização automática limpará automaticamente o bate-papo quando ele terminar. Dessa forma, o agente pode continuar analisando os arquivos sem que o objeto de histórico de bate-papo use muitos tokens desnecessários. 

1. Localize o comentário **Append the current log file to the chat** e adicione o seguinte código para adicionar o texto do arquivo de log lido mais recentemente ao chat:

    ```python
   # Append the current log file to the chat
   await chat.add_chat_message(logfile_msg)
   print()
    ```

1. Localize o comentário **Chamar uma resposta dos agentes** e adicione o seguinte código para invocar o chat em grupo:

    ```python
   # Invoke a response from the agents
   async for response in chat.invoke():
        if response is None or not response.name:
            continue
        print(f"{response.content}")
    ```

    Este é o código que aciona o chat. Como o texto do arquivo de log foi adicionado como uma mensagem, a estratégia de seleção determinará qual agente deve ler e responder a ele e, em seguida, a conversa continuará entre os agentes até que as condições da estratégia de encerramento sejam atendidas ou o número máximo de iterações seja atingido.

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
   python agent_chat.py
    ```

    Você verá algo semelhante à seguinte saída:

    ```output
    
    INCIDENT_MANAGER > /home/.../logs/log1.log | Restart service ServiceX
    DEVOPS_ASSISTANT > Service ServiceX restarted successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log2.log | Rollback transaction for transaction ID 987654.
    DEVOPS_ASSISTANT > Transaction rolled back successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log3.log | Increase quota.
    DEVOPS_ASSISTANT > Successfully increased quota.
    (continued)
    ```

    > **Observação**: o aplicativo inclui algum código para aguardar entre o processamento de cada arquivo de log para tentar reduzir o risco de um limite de taxa de TPM ser excedido e o tratamento de exceções caso isso aconteça de qualquer maneira. Se não houver cota suficiente disponível em sua assinatura, o modelo talvez não consiga responder.

1. Verifique se os arquivos de log na pasta de **logs** estão atualizados com mensagens de operação de resolução do DevopsAssistant.

    Por exemplo, log1.log deve ter as seguintes mensagens de log anexadas:

    ```log
    [2025-02-27 12:43:38] ALERT  DevopsAssistant: Multiple failures detected in ServiceX. Restarting service.
    [2025-02-27 12:43:38] INFO  ServiceX: Restart initiated.
    [2025-02-27 12:43:38] INFO  ServiceX: Service restarted successfully.
    ```

## Resumo

Neste exercício, você usou o Serviço de Agente de IA do Azure e o SDK do Kernel Semântico para criar agentes de DevOps e incidentes de IA que podem detectar problemas automaticamente e aplicar resoluções. Ótimo trabalho!

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.

1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.

1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.