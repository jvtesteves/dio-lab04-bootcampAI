# Explorando o Índice de Pesquisa da Azure AI

## Introdução

Este projeto documenta o processo de criação e exploração de um índice de pesquisa da Azure AI para analisar experiências de clientes da cadeia nacional de cafeterias Fourth Coffee. A solução de mineração de conhecimento desenvolvida facilita a busca por insights a partir de avaliações de clientes, utilizando para isso os recursos da Azure.

## Objetivos

- Criar recursos Azure necessários.
- Extrair dados de uma fonte de dados.
- Enriquecer os dados com habilidades de IA.
- Utilizar o indexador da Azure no portal da Azure.
- Consultar o índice de pesquisa.
- Revisar os resultados salvos no Knowledge Store.

## Passo a Passo

### 1. Criação de Recursos Azure

Para a solução destinada à Fourth Coffee, são necessários os seguintes recursos na sua assinatura Azure:

- **Recurso Azure AI Search**: Gerencia a indexação e a consulta.
- **Recurso Azure AI services**: Fornece serviços de IA para habilidades que sua solução de pesquisa pode usar para enriquecer os dados na fonte de dados com insights gerados por IA.
- **Conta de Armazenamento com contêineres blob**: Armazena documentos brutos e outras coleções de tabelas, objetos ou arquivos.

#### Criar um Recurso Azure AI Search

1. Acesse o portal Azure.
2. Clique em '+ Criar um recurso', procure por Azure AI Search e crie um recurso com as seguintes configurações:
   - Assinatura: Sua assinatura Azure.
   - Grupo de recursos: Selecione ou crie um grupo de recursos com um nome único.
   - Nome do serviço: Um nome único.
   - Localização: Escolha qualquer região disponível.
   - Camada de preços: Básico
3. Selecione 'Revisar + criar' e, após a validação bem-sucedida, selecione 'Criar'.
4. Após a conclusão da implantação, selecione 'Ir para o recurso'.

#### Criar um Recurso Azure AI services

1. No portal Azure, clique em '+ Criar um recurso' e procure por Azure AI services.
2. Configure o recurso com as seguintes configurações:
   - Assinatura: Sua assinatura Azure.
   - Grupo de recursos: O mesmo grupo de recursos do seu recurso Azure AI Search.
   - Região: A mesma localização do seu recurso Azure AI Search.
   - Nome: Um nome único.
   - Camada de preços: Standard S0
3. Selecione 'Revisar + criar' e, após a validação, selecione 'Criar'.

#### Criar uma Conta de Armazenamento

1. No portal Azure, selecione '+ Criar um recurso'.
2. Procure por conta de armazenamento e crie um recurso com as seguintes configurações:
   - Assinatura: Sua assinatura Azure.
   - Grupo de recursos: O mesmo grupo de recursos dos seus recursos Azure AI Search e Azure AI services.
   - Nome da conta de armazenamento: Um nome único.
   - Localização: Escolha qualquer localização disponível.
   - Desempenho: Padrão
   - Redundância: Armazenamento localmente redundante (LRS)
3. Clique em 'Revisar' e depois em 'Criar'.

Nota: Seus recursos Azure AI Search e Azure AI services devem estar na mesma localização!

### 2. Extração de Dados

Para carregar os documentos de avaliações de café para o Azure Storage, siga estes passos:

1. No painel lateral esquerdo do Azure Storage, selecione 'Contêineres'.
2. Clique em '+ Contêiner'. Uma nova aba será aberta ao lado direito.
3. Insira as configurações a seguir e clique em 'Criar':
   - Nome: `coffee-reviews`
   - Nível de acesso público: `Contêiner` (acesso de leitura anônima para contêineres e blobs)
   - Avançado: sem alterações.
4. Em uma nova aba do navegador, baixe os comentários de café compactados de [https://aka.ms/mslearn-coffee-reviews](https://aka.ms/mslearn-coffee-reviews) e extraia os arquivos para a pasta `reviews`.
5. No portal Azure, selecione seu contêiner `coffee-reviews`. Dentro do contêiner, selecione 'Upload'.
6. Na aba de Upload de blob, selecione 'Selecionar um arquivo'.
7. Na janela do Explorer, selecione todos os arquivos na pasta `reviews`, clique em 'Abrir' e depois em 'Upload'.

Após a conclusão do upload, você pode fechar a aba de Upload de blob. Seus documentos agora estão no contêiner de armazenamento `coffee-reviews`.

### 3. Indexar os Documentos

Depois de ter os documentos no armazenamento, você pode usar o Azure AI Search para extrair insights dos documentos. O portal Azure fornece um assistente de Importação de Dados. Com este assistente, você pode criar automaticamente um índice e um indexador para fontes de dados suportadas. Você usará o assistente para criar um índice e importar seus documentos de pesquisa do armazenamento para o índice da Azure AI Search.

1. No portal Azure, navegue até o seu recurso Azure AI Search. Na página Visão Geral, selecione 'Importar dados'.
2. Na página 'Conectar aos seus dados', na lista Fonte de Dados, selecione Armazenamento de Blobs Azure. Complete os detalhes da loja de dados com os seguintes valores:
   - Fonte de Dados: Armazenamento de Blobs Azure
   - Nome da fonte de dados: `coffee-customer-data`
   - Dados a extrair: Conteúdo e metadados
   - Modo de análise: Padrão
   - String de conexão: Selecione 'Escolher uma conexão existente'. Selecione sua conta de armazenamento, selecione o contêiner `coffee-reviews` e, em seguida, clique em Selecionar.
   - Autenticação de identidade gerenciada: Nenhuma
   - Nome do contêiner: essa configuração é preenchida automaticamente após escolher uma conexão existente.
   - Pasta Blob: Deixe em branco.
   - Descrição: Avaliações para as lojas Fourth Coffee.
3. Selecione 'Próximo: Adicionar habilidades cognitivas (Opcional)'.

Na seção 'Anexar Serviços Cognitivos', selecione o recurso Azure AI services.

Na seção 'Adicionar enriquecimentos':
   - Altere o nome do conjunto de habilidades para `coffee-skillset`.
   - Selecione a caixa de seleção 'Habilitar OCR e mesclar todo o texto no campo merged_content'.
   - Nota: É importante selecionar 'Habilitar OCR' para ver todas as opções de campo enriquecido.

Certifique-se de que o campo de dados de origem esteja definido para `merged_content`.
   - Altere o nível de granularidade do enriquecimento para Páginas (pedaços de 5000 caracteres).
   - Não selecione 'Habilitar enriquecimento incremental'
   - Selecione os seguintes campos enriquecidos:

| Habilidade Cognitiva      | Parâmetro | Nome do Campo   |
|---------------------------|-----------|-----------------|
| Extrair nomes de locais   |           | `locations`     |
| Extrair frases-chave      |           | `keyphrases`    |
| Detectar sentimento       |           | `sentiment`     |
| Gerar tags de imagens     |           | `imageTags`     |
| Gerar legendas de imagens |           | `imageCaption`  |

Sob 'Salvar enriquecimentos em uma loja de conhecimento', selecione:
   - Projeções de Imagem
   - Documentos
   - Páginas
   - Frases-chave
   - Entidades
   - Detalhes de imagem
   - Referências de imagem

Se um aviso pedindo uma String de Conexão de Conta de Armazenamento aparecer, selecione 'Escolher uma conexão existente'. Escolha a conta de armazenamento que você criou anteriormente.

Clique em '+ Contêiner' para criar um novo contêiner chamado `knowledge-store` com o nível de privacidade definido como Privado, e selecione 'Criar'.

Selecione o contêiner `knowledge-store`, e então clique em 'Selecionar' na parte inferior da tela.

Selecione 'Projeções de blob Azure: Documento'. Uma configuração para Nome do Contêiner com o contêiner `knowledge-store` preenchido automaticamente é exibida. Não altere o


### 4. Consulta ao Índice de Pesquisa

As consultas ao índice de pesquisa foram realizadas utilizando o Explorador de Pesquisa no portal Azure, uma ferramenta poderosa que facilita a experimentação e teste de diferentes consultas em JSON. Foram executadas consultas para recuperar todos os documentos, além de filtros específicos por localização e sentimento, para entender melhor as experiências dos clientes da Fourth Coffee. Os resultados em JSON permitiram uma análise detalhada das avaliações, incluindo a contagem de documentos que correspondem a critérios específicos, o que destacou as áreas de forte satisfação do cliente e pontos que precisam de melhoria.

## Insights e Aprendizados

O projeto proporcionou vários insights valiosos sobre o processamento e análise de dados de avaliações de clientes usando o Azure AI Search. Aprendemos a configurar e utilizar recursos Azure para extrair, enriquecer e indexar dados de uma maneira que facilita a consulta e a análise posterior. Um dos principais aprendizados foi a importância de habilidades cognitivas para enriquecer os dados e obter insights mais profundos, como sentimentos e frases-chave associadas a cada avaliação. Este processo destacou a capacidade de transformar dados brutos em informações acionáveis, permitindo uma melhor compreensão das experiências dos clientes.

## Ferramentas e Possibilidades

Este projeto demonstrou como soluções de mineração de conhecimento, como o Azure AI Search, podem beneficiar uma variedade de ferramentas e aplicações. Ferramentas de análise de sentimentos, sistemas de recomendação, e plataformas de atendimento ao cliente podem se beneficiar imensamente deste tipo de solução, permitindo uma compreensão mais aprofundada das percepções dos clientes. Além disso, a capacidade de enriquecer e indexar grandes volumes de dados abre possibilidades para aplicações em áreas como pesquisa de mercado, desenvolvimento de produtos, e melhoria contínua da experiência do cliente.

## Conclusão

Este projeto sublinha a importância e o impacto potencial da mineração de conhecimento e análise de dados na compreensão das experiências dos clientes. Ao utilizar o Azure AI Search, pudemos não apenas gerenciar e consultar grandes volumes de dados de avaliações de forma eficiente, mas também extrair insights valiosos que podem orientar melhorias no serviço e no produto. A capacidade de analisar detalhadamente as percepções dos clientes representa uma vantagem competitiva significativa, destacando o valor de soluções avançadas de busca e análise de dados no ambiente de negócios atual.

