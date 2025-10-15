---
lab:
  title: Desenvolver uma solução de vários agentes com o Microsoft Agent Framework
  description: Saiba como a configurar vários agentes para colaborar usando o SDK do Microsoft Agent Framework
---

# Desenvolva uma solução multiagente

Neste exercício, você irá praticar o uso do padrão de orquestração sequencial no SDK do Microsoft Agent Framework. Você criará um pipeline simples com três agentes que trabalham juntos para processar o feedback do cliente e sugerir as próximas etapas. Você irá criar os seguintes agentes:

- O agente Summarizer vai condensar o feedback bruto em uma frase curta e neutra.
- O agente Classifier vai categorizar o feedback como Positivo, Negativo ou Solicitação de recurso.
- Por fim, o agente Recommended Action vai recomendar a etapa de acompanhamento adequada.

Você irá aprender a usar o SDK do Microsoft Agent Framework para decompor um problema, encaminhá-lo para os agentes certos e gerar resultados acionáveis. Vamos começar!

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
    - **Região**: *selecione qualquer **AI Foundry recomendado***\*

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto, incluindo a implantação do modelo gpt-4 selecionado.

1. Quando o projeto for criado, o playground chat abrirá automaticamente.

1. No painel de navegação à esquerda, clique em **Modelos e pontos de extremidade** e selecione a implantação **gpt-4o**.

1. No painel **Configuração**, anote o nome da implantação do modelo; que será **gpt-4o**. Você pode confirmar isso visualizando a implantação na página **Modelos e pontos de extremidade** (basta abrir essa página no painel de navegação à esquerda).
1. No painel de navegação à esquerda, selecione **Visão geral** para ver a página principal do projeto, que será assim:

    ![Captura de tela dos detalhes de um projeto IA do Azure no Portal da Fábrica de IA do Azure.](./Media/ai-foundry-project.png)

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
   pip install azure-identity agent-framework
    ```

1. Digite o seguinte comando para editar o arquivo de configuração fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_openai_endpoint** pelo ponto de extremidade do projeto (copiado da página **Visão geral** do projeto no portal da Fábrica de IA do Azure). Substitua o espaço reservado **your_model_deployment** pelo nome atribuído à implantação do modelo gpt-4o.

1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Criar agentes de IA

Agora você está pronto para criar os agentes para sua solução multiagente! Vamos começar!

1. Insira o seguinte comando para editar o arquivo **agents.py**:

    ```
   code agents.py
    ```

1. Na parte superior do arquivo, no comentário **Add references**, adicione o seguinte código para referenciar os namespaces nas bibliotecas que você precisará para implementar seu agente:

    ```python
   # Add references
   import asyncio
   from typing import cast
   from agent_framework import ChatMessage, Role, SequentialBuilder, WorkflowOutputEvent
   from agent_framework.azure import AzureAIAgentClient
   from azure.identity import AzureCliCredential
    ```

1. Na função **main**, reserve um momento para examinar as instruções do agente. Essas instruções definem o comportamento de cada agente na orquestração.

1. Adicione o seguinte código sob o comentário **Create the chat client**:

    ```python
   # Create the chat client
   credential = AzureCliCredential()
   async with (
       AzureAIAgentClient(async_credential=credential) as chat_client,
   ):
    ```

    Observe que o objeto **AzureCliCredential** permitirá que o código seja autenticado em sua conta do Azure. O objeto **AzureAIAgentClient** incluirá automaticamente as configurações de projeto da Fábrica de IA do Azure a partir da configuração de .env.

1. Adicione o seguinte código sob o comentário **Create agents**:

    (Certifique-se de manter o nível de recuo)

    ```python
   # Create agents
   summarizer = chat_client.create_agent(
       instructions=summarizer_instructions,
       name="summarizer",
   )

   classifier = chat_client.create_agent(
       instructions=classifier_instructions,
       name="classifier",
   )

   action = chat_client.create_agent(
       instructions=action_instructions,
       name="action",
   )
    ```

## Criar uma orquestração sequencial

1. Na função **main**, localize o comentário **Initialize the current feedback** e adicione o seguinte código:
    
    (Certifique-se de manter o nível de recuo)

    ```python
   # Initialize the current feedback
   feedback="""
   I use the dashboard every day to monitor metrics, and it works well overall. 
   But when I'm working late at night, the bright screen is really harsh on my eyes. 
   If you added a dark mode option, it would make the experience much more comfortable.
   """
    ```

1. No comentário **Build a sequential orchestration**, adicione o seguinte código para definir uma orquestração sequencial com os agentes que você definiu:

    ```python
   # Build sequential orchestration
   workflow = SequentialBuilder().participants([summarizer, classifier, action]).build()
    ```

    Os agentes processarão o feedback na ordem em que forem adicionados à orquestração.

1. Adicione o seguinte código sob o comentário **Run and collect outputs**:

    ```python
   # Run and collect outputs
   outputs: list[list[ChatMessage]] = []
   async for event in workflow.run_stream(f"Customer feedback: {feedback}"):
       if isinstance(event, WorkflowOutputEvent):
           outputs.append(cast(list[ChatMessage], event.data))
    ```

    Esse código executa a orquestração e coleta a saída de cada um dos agentes participantes.

1. Adicione o seguinte código sob o comentário **Display outputs**:

    ```python
   # Display outputs
   if outputs:
       for i, msg in enumerate(outputs[-1], start=1):
           name = msg.author_name or ("assistant" if msg.role == Role.ASSISTANT else "user")
           print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
    ```

    Esse código formata e exibe as mensagens das saídas do fluxo de trabalho coletadas da orquestração.

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
   python agents.py
    ```

    Você verá algo semelhante à seguinte saída:

    ```output
    ------------------------------------------------------------
    01 [user]
    Customer feedback:
        I use the dashboard every day to monitor metrics, and it works well overall.
        But when I'm working late at night, the bright screen is really harsh on my eyes.
        If you added a dark mode option, it would make the experience much more comfortable.

    ------------------------------------------------------------
    02 [summarizer]
    User requests a dark mode for better nighttime usability.
    ------------------------------------------------------------
    03 [classifier]
    Feature request
    ------------------------------------------------------------
    04 [action]
    Log as enhancement request for product backlog.
    ```

1. Opcionalmente, você pode tentar executar o código usando diferentes entradas de feedback, como:

    ```output
    I use the dashboard every day to monitor metrics, and it works well overall. But when I'm working late at night, the bright screen is really harsh on my eyes. If you added a dark mode option, it would make the experience much more comfortable.
    ```
    ```output
    I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
    ```

## Resumo

Neste exercício, você praticou a orquestração sequencial com o SDK do Microsoft Agent Framework, combinando vários agentes em um único fluxo de trabalho simplificado. Ótimo trabalho!

## Limpar

Se tiver terminado de explorar os Serviços de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.

1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.

1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
