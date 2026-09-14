# Sistemas que chamam a API do Claude

Aqui o custo não é da sessão de programação: é do sistema em produção, multiplicado por cada usuário e cada mensagem. Decisões pequenas no código viram grandes diferenças na conta.

Nomes de parâmetros, modelos, preços e versões de recursos mudam. Antes de escrever o código, confirme na documentação atual da Anthropic (ou use a skill `claude-api`, se disponível) — não confie em nomes decorados.

## Sumário
1. Escolha de modelo e esforço
2. Prompt caching
3. Tamanho de resposta
4. Histórico de conversa (chatbots)
5. Ferramentas (tool use)
6. Arquivos e documentos
7. Processamento em lote
8. Medir antes de otimizar

---

## 1. Escolha de modelo e esforço
- **Modelo por tarefa, não um para tudo:** classificação, extração de campos, resumo curto e roteamento funcionam bem com o modelo menor e mais barato (família Haiku). Reserve os modelos maiores para raciocínio complexo, código ou textos longos de qualidade.
- **Roteamento:** um modelo pequeno decide o tipo de pedido e só encaminha para o modelo maior quando necessário.
- **Esforço (`effort`, em `output_config`):** `low`/`medium` para respostas rotineiras; `high`/`max` só onde a qualidade do raciocínio muda o resultado. Raciocínio estendido gera tokens de saída pagos.
- **Não troque de modelo no meio de uma conversa** que usa cache: o cache é por modelo e é perdido.

## 2. Prompt caching
Leitura de cache custa uma fração do input normal (desconto de até ~90%); gravar cache custa um pouco mais que o input normal. Compensa quando o mesmo começo de prompt é reutilizado várias vezes.
- **Ordem fixa, do mais estável para o mais variável:** definições de ferramentas → system prompt → documentos/base de conhecimento → histórico → mensagem nova.
- **Prefixo idêntico byte a byte:** nada de data/hora, nome do usuário, id de sessão ou dados variáveis no system prompt ou nas ferramentas. Coloque isso depois, na mensagem do usuário.
- **Histórico só com acréscimos no fim:** não reescreva nem reordene mensagens antigas.
- **Serialização estável:** gere JSON de ferramentas/documentos sempre na mesma ordem de chaves.
- **Marque os pontos de cache** no fim dos blocos estáveis. Há tamanho mínimo de prompt para cachear e um tempo de expiração; confira os valores atuais para o modelo usado.
- **Verifique na resposta** os campos de uso de cache (tokens lidos/gravados do cache). Se a leitura vier zerada em chamadas repetidas, algo no prefixo está mudando.

## 3. Tamanho de resposta
- **`max_tokens` realista** para cada tipo de chamada, não o máximo permitido.
- **Peça o formato exato** (JSON com campos definidos, "responda em até 3 frases"). Saídas estruturadas evitam texto extra e retentativas por formato inválido.
- **Sem pedir para "explicar o raciocínio"** quando o sistema só usa o resultado.
- **Streaming** melhora a percepção de velocidade, mas não reduz custo.

## 4. Histórico de conversa (chatbots)
- **Não reenvie o histórico inteiro para sempre.** Opções: janela das últimas N trocas; resumo das mensagens antigas guardado e enviado no lugar delas; ou compactação automática de contexto oferecida pela API (estratégia de compaction, na época desta skill com nome versionado como `compact_20260112`).
- **Resultados antigos de ferramentas** (buscas, consultas, páginas lidas) são os maiores vilões do histórico; use a edição de contexto da API com a estratégia de limpeza de resultados de ferramentas (`clear_tool_uses`, também com nome versionado) ou remova-os você mesmo depois de usados. Confira os nomes exatos na documentação atual, pois as versões mudam.
- **Guarde fatos do usuário em banco** (nome, plano, preferências) e injete só o que for relevante, em vez de manter tudo no histórico.
- **Base de conhecimento:** busque os trechos relevantes (RAG) e envie só eles, em vez de colar documentos inteiros em toda chamada. Se o documento for pequeno e fixo, cacheie.

## 5. Ferramentas (tool use)
- **Poucas ferramentas por chamada:** cada definição ocupa tokens em toda requisição. Envie só as ferramentas relevantes para o contexto.
- **Catálogo grande:** use o carregamento sob demanda de ferramentas (busca de ferramentas) oferecido pela API.
- **Descrições curtas e precisas.**
- **Resultados de ferramenta enxutos:** a ferramenta devolve só os campos necessários, paginados e com limite, nunca a tabela inteira.

## 6. Arquivos e documentos
- **Files API:** envie o arquivo uma vez e referencie pelo id, em vez de mandar o conteúdo em cada chamada.
- **Planilhas e dados:** use execução de código no servidor para a IA processar com script e devolver só o resultado.
- **PDFs e imagens:** reduza resolução/páginas ao necessário; imagens grandes consomem muitos tokens.

## 7. Processamento em lote
- **Tarefas que não precisam de resposta imediata** (gerar descrições de produtos, classificar milhares de itens, relatórios noturnos): use a Batch API, que custa cerca de metade do preço. Combina com cache.

## 8. Medir antes de otimizar
- **Registre em log** os campos de uso de cada resposta (input, output, cache lido, cache gravado) por tipo de chamada.
- **Use a contagem de tokens** da API para estimar o custo de prompts antes de colocar em produção.
- **Defina limites** por usuário/dia e alertas de gasto para evitar surpresas com loops ou abuso.
- Otimize primeiro a chamada que mais gasta no total, não a mais fácil de mexer.
