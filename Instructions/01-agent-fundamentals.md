---
lab:
  title: Explore o desenvolvimento do agente de I
  description: Dê os primeiros passos no desenvolvimento de agentes de IA explorando o Serviço de Agente de IA do Azure no Portal da Fábrica de IA do Azure.
---

# Explore o desenvolvimento do Agente de IA

Neste exercício, você usará o Serviço de Agente de IA do Azure no Portal da Fábrica de IA do Azure para criar um agente de IA simples que auxilia os funcionários em solicitações de despesas.

Este exercício levará aproximadamente **30** minutos.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar algum comportamento, avisos ou erros inesperados.

## Criar um projeto e um agente da Fábrica de IA do Azure

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./Media/ai-foundry-home.png)

1. Na home page, clique em **Criar um agente**.
1. Quando solicitado a criar um projeto, insira um nome válido para o projeto.
1. Expanda **Opções avançadas** e especifique as seguintes configurações:
    - **Recurso da Fábrica de IA do Azure**: *um nome válido para o recurso da Fábrica de IA do Azure*
    - **Assinatura**: *sua assinatura do Azure*
    - **Resource group**: *selecione o grupo de recursos ou crie um novo*
    - **Região**: *selecione qualquer **AI Foundry recomendado***\**

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto.
1. Se solicitado, implante um modelo **gpt-4o** usando o tipo de implantação **Global padrão** ou **Padrão** (dependendo da disponibilidade de cota) e personalize os detalhes da implantação para definir um **Limite de tokens por minuto** de 50 mil (ou o máximo disponível, se for inferior a 50 mil).

    > **Observação**: A redução do TPM ajuda a evitar o uso excessivo da cota disponível na assinatura que você está usando. 50.000 TPM são suficientes para os dados usados neste exercício. Se a sua cota disponível for menor do que isso, você poderá concluir o exercício, mas poderá ocorrer erros se o limite de taxa for excedido.

1. Quando o projeto for criado, o playground Agentes abrirá automaticamente para que você possa selecionar um implantar um modelo:

    ![Captura de tela do playground Agentes de um projeto da Fábrica de IA do Azure](./Media/ai-foundry-agents-playground.png)

    >**Observação**: um modelo base GPT-4o é implantado automaticamente ao criar o agente e o projeto.

Você verá que um agente com um nome padrão foi criado para você, juntamente com a implantação do modelo base.

## Criar o agente

Agora que você tem um modelo implantado, está pronto para criar um agente de IA. Neste exercício, você criará um agente simples que responde a perguntas com base em uma política de despesas corporativas. Você fará o download do documento de política de despesas e o usará como *dados de base* para o agente.

1. Abra outra guia do navegador, faça o download de [Expenses_policy.docx](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-agent-fundamentals/Expenses_Policy.docx) de `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-agent-fundamentals/Expenses_Policy.docx` e salve o arquivo localmente. Este documento contém detalhes da política de despesas da corporação fictícia "Contoso".
1. Retorne à guia do navegador que contém o playground Agentes da Fábrica e encontre o painel **Configuração** (pode estar ao lado ou abaixo da janela de chat).
1. Defina o **Nome do agente** como `ExpensesAgent`, certifique-se de que a implantação do modelo gpt-4o que você criou anteriormente esteja selecionada e defina as **Instruções** como:

    ```prompt
   You are an AI assistant for corporate expenses.
   You answer questions about expenses based on the expenses policy data.
   If a user wants to submit an expense claim, you get their email address, a description of the claim, and the amount to be claimed and write the claim details to a text file that the user can download.
    ```

    ![Captura de tela da página de configuração do agente de IA no Portal da Fábrica de IA do Azure.](./Media/ai-agent-setup.png)

1. Mais abaixo no painel **Configuração** , ao lado do cabeçalho **Conhecimento**, selecione **+ Adicionar**. Em seguida, na caixa de diálogo **Adicionar conhecimento** , selecione **Arquivos**.
1. Na caixa de diálogo **Adicionando arquivos**, crie um novo repositório vetorial chamado `Expenses_Vector_Store`, carregando e salvando o **arquivo local Expenses_policy.docx** que você baixou anteriormente.
1. No painel **Configuração**, na seção **Conhecimento**, verifique se **Expenses_Vector_Store** está relacionado e indicado como contendo 1 arquivo.
1. Abaixo da seção **Conhecimento**, ao lado de **Ações**, selecione **+ Adicionar**. Em seguida, na caixa de diálogo **Adicionar ação**, selecione **Interprete de código** e, em seguida, clique em **Salvar** (você não precisa carregar nenhum arquivo para o interprete de código).

    Seu agente usará o documento que você carregou como fonte de conhecimento para *fundamentar* as respostas (em outras palavras, o agente responderá a perguntas com base no conteúdo deste documento). Ele usará a ferramenta intérprete de código conforme necessário para executar ações gerando e executando seu próprio código Python.

## Testar seu agente

Agora que você criou um agente, pode testá-lo no playground Chat.

1. Na entrada do chat do playground, insira o prompt `What's the maximum I can claim for meals?` e revise a resposta do agente, que será baseada nas informações do documento de política de despesas que você adicionou como conhecimento à configuração do agente.

    > **Observação**: se o agente não responder porque o limite de taxa foi excedido. aguarde alguns segundos e tente novamente. Se não houver cota suficiente disponível em sua assinatura, o modelo talvez não consiga responder. Se o problema persistir, tente aumentar a cota do modelo na página **Modelos + pontos de extremidade**.

1. Continue com o seguinte prompt de acompanhamento `I'd like to submit a claim for a meal.` e revise a resposta. O agente solicitará as informações necessárias para enviar uma declaração.
1. Forneça ao agente um endereço de email; por exemplo, `fred@contoso.com`. O agente reconhecerá a resposta e solicitará as informações restantes necessárias para o relatório de despesas (descrição e valor)
1. Envie um prompt que descreva a declaração e o valor; por exemplo, `Breakfast cost me $20`.
1. O agente usará o intérprete de código para preparar o arquivo de texto do relatório de despesas e fornecer um link para que você possa baixá-lo.

    ![Captura de tela do playground no Portal da Fábrica de IA do Azure.](./Media/ai-agent-playground.png)

1. Baixe e abra o documento de texto para ver os detalhes da declaração de despesas.

## Limpar

Agora que você concluiu o exercício, exclua os recursos de nuvem criados para evitar o uso desnecessário de recursos.

1. Abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
