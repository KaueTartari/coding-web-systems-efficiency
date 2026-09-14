---
name: coding-web-systems-efficiency
description: Aplica regras de economia de tokens e precisão ao programar sites, apps web e sistemas — frontend, backend, APIs, banco de dados (SQL, Postgres, MySQL, SQLite, Supabase, Firebase), servidores, Docker, integrações, automações e scripts. Use sempre que a tarefa envolver criar, editar, refatorar, depurar, revisar ou fazer deploy de código, rodar builds/testes/migrations, consultar banco, ler logs, CSV, planilhas ou JSON, ou construir sistemas que usam IA por dentro (chatbot, gerador de texto, agente). Use também quando o usuário falar em economizar tokens, custo, limite de uso, sessão lenta ou contexto cheio, mesmo que não peça isso diretamente.
---

# Programação com economia de tokens

Tudo o que entra no contexto (arquivos lidos, logs, resultados de consulta, screenshots, respostas) é pago de novo a cada mensagem seguinte, e o texto que eu escrevo custa cerca de 5x mais que o que eu leio. Por isso: ler só o necessário, escrever só o necessário e manter a sessão enxuta.

Economia nunca justifica pular uma verificação que evitaria retrabalho — um bug que volta custa muito mais que um teste bem escolhido. O objetivo é gastar tokens onde eles mudam o resultado.

Comandos prontos por ferramenta (banco, logs, testes, dados): `references/comandos.md`. Leia a seção da ferramenta em uso quando precisar.
Sistemas que chamam a API do Claude: `references/api-claude.md`. Leia só nesse caso.

## 1. Output enxuto
- **Edições cirúrgicas:** altere só o trecho exato. Reescrever um arquivo inteiro para mudar poucas linhas paga o arquivo todo em tokens de saída. Arquivo novo só quando for realmente novo.
- **Não repita código já aplicado:** o arquivo já está alterado. Aponte `arquivo:linha`.
- **Sem filler:** nada de preâmbulo, repetir o pedido, explicar o óbvio ou comentários redundantes no código.
- **Diagnósticos, reviews e relatórios:** listas objetivas ou tabelas curtas. Diga o que mudou, o que foi verificado e o que falta.
- **Sem variações não pedidas:** uma boa solução; ofereça alternativas em uma linha se fizer sentido.
- **Reaproveite** componentes, funções, classes, utilitários e variáveis que já existem em vez de gerar blocos parecidos.

## 2. Leitura seletiva do repositório
- **Busque antes de abrir:** procure pelo nome da função, componente, rota, classe CSS, tabela ou mensagem de erro (`rg`, busca por regex). Abra só os arquivos que aparecerem, e só a faixa de linhas relevante.
- **Mapa antes de explorar:** se existir `CLAUDE.md`, `README` ou `package.json`/`pyproject.toml`/`composer.json`, eles dizem stack, scripts e pastas — leia antes de sair listando diretórios.
- **Histórico com mira:** `git log`/`git blame` limitados ao arquivo ou trecho; `git diff --stat` antes do diff completo.
- **Nunca carregue:** `node_modules/`, `vendor/`, `dist/`, `build/`, `.next/`, `.nuxt/`, `__pycache__/`, `.venv/`, `coverage/`, lockfiles, arquivos minificados, `*.map`, dumps `.sql`, bancos `.sqlite`/`.db`, imagens/fontes/vídeos do projeto.
- **Não releia** um arquivo que acabou de ser editado.
- **Leituras e buscas independentes em paralelo**, numa rodada só.

## 3. Banco de dados
Uma tabela inteira ou um backup no chat pode custar dezenas de milhares de tokens e não ajuda a raciocinar.
- **Estrutura primeiro:** para entender o banco, leia o schema (tabelas, colunas, tipos, relações) ou os arquivos de migration/model do projeto — não os dados.
- **Consultas com mira:** só as colunas necessárias (nunca `SELECT *` exploratório), sempre com `LIMIT` (5–20 linhas bastam para ver o formato), e `WHERE` focado no caso do problema.
- **Perguntas de quantidade viram agregação:** `COUNT`, `GROUP BY`, `MIN/MAX` em vez de trazer linhas para contar no olho.
- **Consulta lenta:** `EXPLAIN` resolve melhor que ler dados.
- **Nunca abra** dumps/backups (`.sql`, `.bak`, `.dump`) no contexto; se precisar achar algo neles, use busca por texto e traga só as linhas encontradas.
- **Dados sensíveis:** evite trazer e-mails, senhas, tokens e dados pessoais de clientes para o chat; selecione só o que o problema exige.
- **Escrita no banco** (UPDATE, DELETE, migration em produção): confirme com o usuário antes. Economia não vale um dado apagado.

## 4. Logs de servidor e aplicação
- **Filtre antes de ler:** por nível (`ERROR`, `FATAL`, `Exception`, `500`), por termo (rota, id do pedido, usuário) e pela janela de horário do problema.
- **Pegue só o final:** as últimas 50–100 linhas costumam conter o erro; amplie só se necessário.
- **Stack trace:** a primeira linha da exceção e as linhas que apontam para o código do projeto importam; o resto (frameworks, bibliotecas) quase nunca.
- **Erros repetidos:** conte e agrupe (`sort | uniq -c`) em vez de ler o mesmo erro 300 vezes.
- **Logs de requisições/rede no navegador:** filtre por erro ou pela rota em questão.

## 5. Planilhas, CSV, JSON e arquivos grandes
- **Não abra o arquivo inteiro.** Primeiro descubra o tamanho e o formato: cabeçalho e 5 linhas, ou as chaves do JSON.
- **Resuma com script:** use um script local (Python/pandas, `jq`, `csvkit`, `awk`) para filtrar, contar, agrupar ou comparar, e traga só o resultado consolidado.
- **Respostas de API volumosas:** extraia só os campos relevantes (`jq '.data[0]'`, chaves de primeiro nível) em vez de imprimir o payload.
- **Planilha `.xlsx`:** leia com script (nomes das abas, cabeçalhos, algumas linhas), não converta tudo para texto.
- Se o mesmo tipo de análise for se repetir, salve o script no projeto e reutilize.

## 6. Testes, build e comandos
- **Rode só o teste da parte alterada** (arquivo, função ou nome do teste) enquanto trabalha. A suíte completa uma vez, no fim, antes de declarar pronto.
- **Filtre a saída:** só falhas e o resumo final. Rode em modo silencioso/sem cores quando a ferramenta permitir.
- **Build, lint, install:** traga só erros e as últimas linhas. Um `npm install` ou build completo despejado no contexto custa milhares de tokens.
- **Não rode o mesmo comando de novo** só para rever a saída; guarde a informação da primeira vez.
- **Comandos longos ou servidores:** rode em segundo plano e consulte a saída filtrada, em vez de ficar lendo tudo.

## 7. Frontend no navegador
- Para checar texto, estrutura ou se um elemento existe, leia a página como texto/árvore de acessibilidade; screenshot custa bem mais.
- Screenshot só quando o que importa é visual (layout, cor, alinhamento), em escala reduzida ou recortada na área alterada.
- Uma checagem por rodada de mudanças. Console e rede filtrados por erros.
- Mockups e imagens de referência: reduza a resolução antes de analisar.

## 8. Subagentes
- Cada subagente começa do zero e relê o projeto. Faça direto o que for rápido.
- Use subagentes (em segundo plano) só para tarefas pesadas e independentes, ou quando o usuário pedir, e passe o contexto necessário no pedido para evitar exploração repetida.

## 9. Sistemas que usam IA por dentro
Quando o projeto chama a API do Claude (chatbot, gerador de texto, classificador, agente, atendimento automático), o custo passa a ser do sistema em produção, a cada usuário. Aí as regras de cache, `effort`, escolha de modelo, lote e compactação viram parte do código. Leia `references/api-claude.md` antes de escrever essa parte.

## 10. Higiene da sessão (dicas para o usuário)
Estas ações são do usuário. Mencione em uma linha quando fizer sentido — ao concluir uma funcionalidade ou quando a sessão estiver longa — sem repetir a cada resposta:
- Começar uma sessão nova (ou `/clear` no terminal) a cada bug ou funcionalidade concluída.
- Editar a mensagem com erro em vez de mandar outra corrigindo.
- Desligar conectores e plugins que o projeto não usa (e-mail, agenda, Slack etc.).
- Esforço/modelo mais leve para ajustes simples e mais forte para arquitetura ou bug difícil, sem trocar no meio da mesma sessão.
- Manter um `CLAUDE.md` curto no projeto (stack, pastas, comandos de build/teste, como acessar o banco e os logs, padrões) para evitar exploração repetida a cada sessão.
