---
lab:
  title: Desenvolva agentes de IA usando o Azure OpenAI e o SDK do Kernel Semântico
  description: Saiba como usar o SDK do Kernel Semântico para criar e usar um agente de Serviço do Agente de IA do Azure.
---

# Desenvolver agentes de IA do Azure com o SDK do Kernel Semântico

Neste exercício, você usará o Serviço de Agente de IA do Azure e o Kernel Semântico para criar um agente de IA que processa solicitações de despesas.

> **Dica**: O código usado neste exercício baseia-se no SDK do Kernel Semântico para Python. Você pode desenvolver soluções semelhantes usando os SDKs para Microsoft .NET e Java. Consulte [Linguagens com suporte do Kernel Semântico](https://learn.microsoft.com/semantic-kernel/get-started/supported-languages) para mais detalhes.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar algum comportamento, avisos ou erros inesperados.

## Implantar um modelo em um projeto da Fábrica de IA do Azure

Vamos começar implantando um modelo em um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Na home page, na seção **Explorar modelos e funcionalidades**, pesquise pelo modelo `gpt-4o`, que usaremos em nosso projeto.
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
1. No painel **Configuração**, anote o nome da implantação do modelo; que será **gpt-4o**. Você pode confirmar isso visualizando a implantação na página **Modelos e pontos de extremidade** (basta abrir essa página no painel de navegação à esquerda).
1. No painel de navegação à esquerda, selecione **Visão geral** para ver a página principal do projeto, que será assim:

    ![Captura de tela dos detalhes de um projeto IA do Azure no Portal da Fábrica de IA do Azure.](./Media/ai-foundry-project.png)

## Criar um aplicativo cliente do agente

Agora você está pronto para criar um aplicativo cliente que define um agente e uma função personalizada. Alguns códigos foram fornecidos para você em um repositório do GitHub.

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

    > **Dica**: ao inserir comandos no Cloud Shell, a saída poderá ocupar uma grande quantidade do buffer da tela e o cursor na linha atual pode ficar obscurecido. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Quando o repositório tiver sido clonado, insira o comando a seguir para alterar o diretório de trabalho para a pasta que contém os arquivos de código e listar todos eles.

    ```
   cd ai-agents/Labfiles/04-semantic-kernel/python
   ls -a -l
    ```

    Os arquivos fornecidos incluem código do aplicativo, um arquivo para definições de configuração e um arquivo que contém dados de despesas.

### Definir as configurações do aplicativo

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel --upgrade 
    ```

    > **Observação**: A instalação do *semantic-kernel* instala automaticamente uma versão compatível com o Kernel Semântico do *azure-ai-projects*.

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do projeto (copiado da página **Visão Geral** do projeto no portal da Fábrica de IA do Azure) e o espaço reservado **your_model_deployment** pelo nome que você atribuiu à implantação do modelo gpt-4.
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Escrever código para um aplicativo de agente

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto. Use os comentários existentes como guia, inserindo o novo código no mesmo nível de recuo.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code semantic-kernel.py
    ```

1. Examine o código no arquivo. Ele contém:
    - Algumas instruções **import** para adicionar referências a namespaces comumente usados
    - Uma função *main* que carrega um arquivo que contém dados de despesas, pede instruções ao usuário e, em seguida, chama...
    - Uma função **process_expenses_data** na qual o código para criar e usar seu agente deve ser adicionado
    - Uma classe **EmailPlugin** que inclui uma função de kernel chamada **send_email**, que será usada pelo seu agente para simular a funcionalidade usada para enviar um email.

1. Na parte superior do arquivo, após a instrução **importar** existente, localize o comentário **Adicionar referências** e adicione o seguinte código para fazer referência aos namespaces nas bibliotecas necessárias para implementar seu agente:

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity.aio import DefaultAzureCredential
   from semantic_kernel.agents import AzureAIAgent, AzureAIAgentSettings, AzureAIAgentThread
   from semantic_kernel.functions import kernel_function
   from typing import Annotated
    ```

1. Próximo à parte inferior do arquivo, localize o comentário **Criar um plug-in para a funcionalidade** de e-mail e adicione o código a seguir para definir uma classe para um plug-in que contém uma função que seu agente usará para enviar e-mail (os plug-ins são uma maneira de adicionar funcionalidade personalizada aos agentes do Kernel Semântico)

    ```python
   # Create a Plugin for the email functionality
   class EmailPlugin:
       """A Plugin to simulate email functionality."""
    
       @kernel_function(description="Sends an email.")
       def send_email(self,
                      to: Annotated[str, "Who to send the email to"],
                      subject: Annotated[str, "The subject of the email."],
                      body: Annotated[str, "The text body of the email."]):
           print("\nTo:", to)
           print("Subject:", subject)
           print(body, "\n")
    ```

    > **Observação**: a função *simula o* envio de um e-mail imprimindo-o no console. Em um aplicativo real, você usaria um serviço SMTP ou similar para realmente enviar o e-mail!

1. Faça backup acima do novo código de classe **EmailPlugin**, na função **create_expense_claim**, localize o comentário **Get configuration settings** e adicione o código a seguir para carregar o arquivo de configuração e criar um objeto **AzureAIAgentSettings** (que incluirá automaticamente as definições do Agente de IA do Azure da configuração).

    (Certifique-se de manter o nível de recuo)

    ```python
   # Get configuration settings
   load_dotenv()
   ai_agent_settings = AzureAIAgentSettings()
    ```

1. Encontre o comentário **Conectar-se à Fábrica de IA do Azure**, e adicione o seguinte código para se conectar ao seu projeto da Fábrica de IA do Azure usando as credenciais do Azure com as quais você está atualmente conectado.

    (Certifique-se de manter o nível de recuo)

    ```python
   # Connect to the Azure AI Foundry project
   async with (
        DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True) as creds,
        AzureAIAgent.create_client(
            credential=creds
        ) as project_client,
   ):
    ```

1. Localize o comentário **Definir um agente de IA do Azure que envia um email** de relatório de despesas e adicione o código a seguir para criar uma definição de Agente de IA do Azure para seu agente.

    (Certifique-se de manter o nível de recuo)

    ```python
   # Define an Azure AI agent that sends an expense claim email
   expenses_agent_def = await project_client.agents.create_agent(
        model= ai_agent_settings.model_deployment_name,
        name="expenses_agent",
        instructions="""You are an AI assistant for expense claim submission.
                        When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                        Then confirm to the user that you've done so."""
   )
    ```

1. Localize o comentário **Criar um agente de kernel semântico** e adicione o código a seguir para criar um objeto de agente de kernel semântico para o agente de IA do Azure e inclua uma referência ao plug-in **EmailPlugin**.

    (Certifique-se de manter o nível de recuo)

    ```python
   # Create a semantic kernel agent
   expenses_agent = AzureAIAgent(
        client=project_client,
        definition=expenses_agent_def,
        plugins=[EmailPlugin()]
   )
    ```

1. Localize o comentário **Use the agent to process the expenses data** e adicione o código a seguir para criar um thread para o agente executar e, em seguida, invoque-o com uma mensagem de chat.

    (Certifique-se de manter o nível de recuo):

    ```python
   # Use the agent to process the expenses data
   # If no thread is provided, a new thread will be
   # created and returned with the initial response
   thread: AzureAIAgentThread | None = None
   try:
        # Add the input prompt to a list of messages to be submitted
        prompt_messages = [f"{prompt}: {expenses_data}"]
        # Invoke the agent for the specified thread with the messages
        response = await expenses_agent.get_response(prompt_messages, thread=thread)
        # Display the response
        print(f"\n# {response.name}:\n{response}")
   except Exception as e:
        # Something went wrong
        print (e)
   finally:
        # Cleanup: Delete the thread and agent
        await thread.delete() if thread else None
        await project_client.agents.delete_agent(expenses_agent.id)
    ```

1. Revise o código completado pelo agente, usando os comentários para ajudar você a entender o que cada bloco de código faz e, em seguida, salve as alterações de código (**CTRL+S**).
1. Mantenha o editor de código aberto caso precise corrigir algum erro de digitação no código, mas redimensione os painéis para que você possa ver mais do console de linha de comando.

### Entre no Azure e execute o aplicativo.

1. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para entrar no Azure.

    ```
    az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Consulte [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.
    
1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o hub da Fábrica de IA do Azure, se solicitado.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python semantic-kernel.py
    ```
    
    O aplicativo será executado usando as credenciais da sessão autenticada do Azure para se conectar ao seu projeto e criar e executar o agente.

1. Quando perguntado o que fazer com os dados de despesas, insira o seguinte prompt:

    ```
   Submit an expense claim
    ```

1. Quando o aplicativo for concluído, revise a saída. O agente terá redigido um email para uma declaração de despesas com base nos dados fornecidos.

    > **Dica**: se o aplicativo falhar porque o limite de taxa foi excedido, aguarde alguns segundos e tente novamente. Se não houver cota suficiente disponível em sua assinatura, o modelo talvez não consiga responder.

## Resumo

Neste exercício, você usou o SDK do Serviço do Agente de IA do Azure e o Kernel Semântico para criar um agente.

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
