# carteira-automacao

Automatizo o registro diário de preços de ativos, a geração de um resumo semanal escrito com a API do Gemini e enviado por e-mail, e a visualização dos dados em um dashboard.

## O problema

Eu anotava à mão, todo dia, os preços das ações da minha carteira e dos ativos que acompanho. Essa tarefa tomava menos de 7 minutos por dia, cerca de 30 minutos por semana.

Os ativos monitorados são:

- **Carteira:** ITUB4, TAEE11 e GGBR4.
- **Acompanhando:** PETR4, BOVA11 (representa o mercado, porque acompanha o Ibovespa) e dólar (USD/BRL).

## Como funciona

1. Uma planilha no Google Sheets obtém as cotações atuais com a função `GOOGLEFINANCE`.
2. A função `registrarPrecos`, em Google Apps Script, roda automaticamente nos dias úteis entre 19h e 20h, por um acionador de tempo. Ela grava na aba Histórico a data, o ticker, o preço, o tipo (Carteira ou Acompanhando) e a Base 100 de cada ativo. Cada execução leva cerca de 2,7 segundos.
3. Toda sexta-feira, entre 20h e 21h, a função `enviarResumoSemanal` calcula no código a variação de cada ativo nos últimos 7 dias, a média simples da carteira e a maior e a menor variação. Depois envia esses números para o Gemini 3.5 Flash-Lite, pela API, que escreve o texto do resumo. O resumo chega no meu e-mail.
4. Um dashboard no Looker Studio mostra duas páginas: o preço de cada ativo e a evolução em Base 100 desde 28/09, com a comparação entre a carteira (média) e o mercado (BOVA11).

## Resultados

- O registro diário, que eu fazia à mão em até 7 minutos, agora leva cerca de 2,7 segundos e roda sem mim.
- O primeiro resumo semanal automático foi enviado em 13/11/2026.

## Decisões de projeto

- **As contas ficam no código, e a IA só escreve o texto.** Num teste, a IA errou qual ação teve a maior baixa da semana, com o mesmo prompt que tinha acertado na rodada anterior. O código dá sempre o mesmo resultado.
- **Troquei o Gemini 3.8 Flash pelo 3.5 Flash-Lite.** O 3.8 Flash dava erro 503 (alta demanda). Como os números chegam prontos, o modelo mais leve basta para escrever o texto, e ele respondeu em cerca de 4 segundos, contra uns 17. Ao trocar de modelo, testei o prompt de novo e ajustei o que falhou.
- **Retry no erro 503.** Se a API responder 503, o código tenta de novo até 3 vezes, esperando 30 segundos entre as tentativas. Qualquer outro erro interrompe a execução de propósito, para gerar a notificação de falha por e-mail.
- **A chave da API fica nas propriedades do script**, nunca no código.
- **Defesa contra prompt injection e regras no prompt.** Os dados vão entre tags e são tratados só como dados. A IA é proibida de recomendar compra ou venda, de fazer previsões e de explicar por que um preço subiu ou caiu, porque não tem acesso às notícias.
- **O aviso "não é recomendação de investimento" é escrito pelo código**, não pela IA, porque é um texto fixo.
- **O histórico de 28/09 a 09/11 foi preenchido manualmente, uma única vez.** O Google bloqueia o acesso aos dados históricos do `GOOGLEFINANCE` pelo Apps Script.
- **Base 100** (preço do dia ÷ preço de 28/09 × 100). Permite comparar no mesmo gráfico ativos com preços muito diferentes, como a BOVA11 (perto de R\$ 170) e o dólar (perto de R\$ 5,50).

## Como construí e testei

- Desenvolvi com ajuda de IA, um passo por vez, testando cada parte e documentando as decisões.
- Testei cada correção do código antes de salvar e conferi as contas com a calculadora.
- Testei o prompt no Google AI Studio com um checklist (números, ordem, tamanho, nenhuma recomendação), sempre em chat novo, até passar em duas rodadas seguidas.
- Conferi um salto de cerca de 9% do dólar em 26/10 na cotação PTAX do Banco Central. O salto era real.
- O dashboard revelou um erro nos dados: a coluna Base 100 tinha escorregado uma linha numa colagem. Corrigi e conferi ponto a ponto com a planilha.

## Limitações conhecidas

- O registro não detecta feriados da B3.
- A média da carteira é simples, sem pesos, porque a planilha não guarda os valores investidos, por privacidade.
- A IA não tem memória entre as semanas, e o termo explicado no resumo se repetiu ("Ticker" em quatro execuções seguidas).
- A IA às vezes ignora uma regra de formato do resumo, mas os números sempre saíram certos, porque vêm calculados pelo código.
- O código supõe que o Histórico está em ordem de data.

## Próximos passos

- O código escolher o termo da semana, e a IA só explicar.
- Marcar a BOVA11 e o dólar com tipos próprios nos dados, para tirar exceções do prompt.
- Um resumo diário com um agente de IA no n8n.

## Tecnologias

Google Sheets, `GOOGLEFINANCE`, Google Apps Script (JavaScript), API do Gemini, Looker Studio e Gmail.

## Links

- [Dashboard no Looker Studio](https://datastudio.google.com/reporting/5e43e672-9708-4ccf-b955-203eef177c7b)
- [Código do Apps Script](Codigo.gs)
- [Os prompts que uso, com o que ajustei em cada um](https://github.com/Douglas2903/ia-aplicada/blob/main/prompts.md)
