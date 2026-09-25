# Revisão de largura das zonas — BTC

Dados capturados em **2026-09-25T01:36:30.294000+00:00**. Comparação dos dois códigos usando os mesmos candles e o mesmo estado inicial. Base: `7f2bf9f9516975bdfa01d454c35fb50076b68f4e`.

## Causa e alteração

O cluster antigo aceitava distância de até 1 ATR histórico entre seus membros e acrescentava 0,15 ATR por borda. A fusão por sobreposição não voltava a verificar todos os pares nem limitava a largura resultante. Além disso, o ATR dos membros era descartado antes da fusão. O centro suavizado de uma ficha antiga podia ficar entre os novos agrupamentos.

A revisão mantém a arquitetura: pivôs, clusters, episódios, score, casamento e ciclo de vida. Adiciona teto de 0,8 ATR diário e 1,2 ATR semanal na admissão e fusão, incluindo as margens. Separa concentrações com evidência ou conserva o núcleo compacto mais povoado quando não há duas concentrações. Não cria duas zonas a partir de dois pivôs isolados. A janela operacional passa a centro ±0,25 ATR, respeitando o teto percentual do ativo.

O split exige pelo menos dois pivôs de datas diferentes por parte, vão ≥0,2 ATR de referência e ≥2 vezes o espaçamento interno médio de cada lado. A divisão é recursiva. Uma filha não herda o score ou os episódios da mãe: esses valores são recalculados, com os mesmos pesos, penalidades e condições de confirmação. Apenas uma filha pode herdar cada ID anterior. Fichas legadas largas permanecem dormentes no estado durante a carência e não são publicadas.

## Faixas manuais

O teto foi aplicado na data da calibração. As faixas continuam fixas, portanto poderão precisar de nova revisão se a volatilidade mudar. Faixas já dentro do teto foram mantidas, inclusive as de XMR/USD e a faixa promovida de 0,00524–0,00544 no XMR/BTC.

### BTC/USD

ATR diário de referência: **2.514,23**.

| Antes | Depois |
| --- | --- |
| 85.847,83–89.595,87 | 88.372,86–89.602,14 |
| 79.000–81.000 | 79.000–81.000 |
| 76.000–78.000 | 76.000–78.000 |
| 74.000–76.000 | 74.000–76.000 |
| 72.000–74.000 | 72.000–74.000 |
| 64.000–67.000 | 64.297,86–66.023,34 |

## Zonas automáticas publicadas

A comparação por ID mostra a continuidade da ficha, não uma garantia de que todos os pivôs antigos pertencem à nova região. IDs novos identificam as demais regiões resultantes.

| Par / período | ID | Antes | Depois | Score / toques / rejeições depois |
| --- | --- | --- | --- | --- |
| BTC/USD / semanal | `usd\|semanal\|z27` | 82.790,9–91.000 | 83.450,16876811–89.977,03123189 | 67 / 3 / 1 |
| BTC/USD / semanal | `usd\|semanal\|z28` | 94.583,7–100.000 | 93.612,46876811–98.870,93123189 | 26 / 1 / 1 |
| BTC/USD / semanal | `usd\|semanal\|z42` | não publicado | 97.305,96876811–100.971,23123189 | 83 / 4 / 3 |
| BTC/USD / semanal | `usd\|semanal\|z37` | não publicado | 81.819,66876811–83.762,13123189 | 53 / 2 / 1 |
| BTC/USD / semanal | `usd\|semanal\|z41` | não publicado | 75.644,46876811–81.566,43123189 | 88 / 4 / 2 |
| BTC/USD / semanal | `usd\|semanal\|z26` | 71.896,4–80.595,2 | 70.925,16876811–76.969,33123189 | 70 / 5 / 1 |
| BTC/USD / diario | `usd\|diario\|z26` | 85.847,82511872–89.595,87488128 | 88.372,8648927–89.602,1351073 | 72 / 8 / 5 |
| BTC/USD / diario | `usd\|diario\|z51` | não publicado | 90.358,8648927–92.177,1351073 | 61 / 14 / 9 |
| BTC/USD / diario | `usd\|diario\|z41` | não publicado | 93.011,5648927–94.488,1351073 | 61 / 18 / 14 |
| BTC/USD / diario | `usd\|diario\|z25` | 83.314,2677253–84.977,9322747 | 83.493,6648927–84.798,5351073 | 85 / 4 / 2 |
| BTC/USD / diario | `usd\|diario\|z39` | 81.937,40129029–83.141,59870971 | 81.937,40129029–83.141,59870971 | 90 / 5 / 3 |
| BTC/USD / diario | `usd\|diario\|z40` | não publicado | 81.092,91462477–81.829,08537523 | 78 / 3 / 2 |

## Alertas e compatibilidade

A alteração isolada do detector preservou os gatilhos, os estados dos níveis pontuais, a EMA89 semanal, RSI, DMI/ADX, divergências e estrutura. As faixas manuais alteram onde as condições de entrada em faixa são verdadeiras.

No snapshot: **1 gatilhos ativos antes e 1 depois**. As listas exatas constam no JSON desta revisão. Labels de faixas alteradas mudaram. Essa manutenção pode gerar uma diferença de assinatura na primeira execução, sem representar por si só movimento novo do mercado.

Não é possível concluir a frequência futura de notificações da automação externa a partir de um snapshot. As regras de sinais, horários e anti-spam não foram alteradas. Faixas menores cobrem menos preços, mas faixas divididas criam fronteiras distintas. `docs/historico.jsonl` não foi reescrito. Os nomes de campos dos relatórios e estados foram preservados, e memórias antigas continuam legíveis.

## Validação

A suíte `teste-zonas.mjs` cobre os casos solicitados e a calibração manual. As suítes completas, os retratos e a paridade passaram nos três projetos. A validação com dados reais confirmou os tetos e três reexecuções estáveis por monitor. A comparação real utiliza as séries da Kraken para BTC/XMR, Binance para USDT/BRL e Yahoo para USD/BRL, sem troca de fonte entre antes e depois.
