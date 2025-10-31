# STEP FUNCTIONS
PROJETO AWS 02

Projeto feito de acordo com modelo disponibilizado pela AWS (STEP FUNCTION), Detalhadamento elaborado de acordo com o projeto em imagem.

conceitos abordados:

- Fluxo dos dados em STep Functions;
- Uso de ferramentas AWS, projetos prontos StepFunctions;

  ## 🚀 Detalhadamento do projeto elaborado: 

- Ponto inicial do fluxo de execução da Step Function.

# CheckBucketContents
 - Função: Verifica o conteúdo inicial de um bucket S3.
 - Ação: Usa a operação ListObjectsV2 para listar os objetos de um bucket S3.
 - Saída: Retorna uma lista de objetos encontrados, incluindo metadados como Key, Size, LastModified, etc.
 - Objetivo: Determinar se há dados disponíveis para processar.

# Choice
 - Função: Estrutura de decisão.
 - Ação: Verifica o resultado do passo anterior.
 - Condição típica:
 - Se o bucket estiver vazio → fluxo pode terminar.
 - Se houver objetos → continua para o próximo passo (ListObjectsInNoaBucket).

# ListObjectsInNoaBucket
 - Função: Lista os objetos no bucket de origem “NOA”.
 - Ação: Recupera os nomes dos objetos que precisam ser copiados para outro bucket.
 - Saída: JSON com a lista de objetos e dados de paginação (caso o bucket tenha muitos arquivos).

# Map
 - Função: Processa cada item (objeto) listado individualmente.
 - Ação: O estado Map percorre cada item da lista JSON.
 - Internamente contém:
 - Um subestado Map → responsável por iterar item a item.
 - Ação CopyObject → responsável por copiar o objeto para outro bucket.

# CopyObject
 - Função: Copia cada arquivo de um bucket para outro.
 - Ação: Usa a operação CopyObject do S3 para duplicar o arquivo de origem para o bucket de destino.
 - Origem: Bucket NOA.
 - Destino: Bucket de processamento ou backup.

# Choice – Next Page?
 - Função: Verifica se há mais páginas de objetos no bucket (paginação).
 - Ação: Baseia-se na chave NextContinuationToken retornada pelo ListObjectsV2.
 - Condição:
 - Se houver próxima página → volta para ListObjectsInNoaBucket with pagination.
 - Se não → segue para ProcessNOAData.

# ListObjectsInNoaBucket with pagination
 - Função: Lista os objetos restantes no bucket (quando há mais páginas).
 - Ação: Usa o token de continuação (ContinuationToken) para buscar os próximos arquivos.
 - Loop: Continua até listar todos os objetos do bucket.

# ProcessNOAData
 - Função: Inicia o processamento dos dados copiados.
 - Origem de dados: Bucket com os dados já copiados.
 - Ação: Pode preparar dados para análise, conversão, agregação etc.
 - Subações:
 - Invoca uma função Lambda (Lambda Invoke) para processar cada item.

# Lambda Invoke
 - Função: Chama uma função Lambda para processar dados individuais.
 - Ação típica:
 - Leitura de arquivo do S3.
 - Transformação/validação dos dados.
 - Escrita de resultados em outro bucket ou banco de dados (por exemplo, DynamoDB ou S3_RESULTS_BUCKET).

# Reducer
 - Função: Consolida os resultados finais do processamento.
 - Ação: Executa agregação ou redução dos dados processados (por exemplo, somatórios, contagens ou consolidação de resultados em um único arquivo).
 - Saída: Resultado final do fluxo, pronto para ser armazenado ou utilizado por outro serviço.

# End
 - Função: Indica o término bem-sucedido da execução da Step Function.
 - Resultado: Dados copiados, processados e consolidados.


  ![image](https://github.com/DrieleMoreira/AWS-DIO-DESAFIO-02/blob/main/stepfunctions_graph.svg)


