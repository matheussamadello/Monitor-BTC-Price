# Monitor BTC Price

Monitor técnico automatizado de **BTC/USD** que coleta candles da Kraken, calcula indicadores, acompanha estrutura de mercado e publica um relatório estático em HTML, texto e JSON para consulta humana ou consumo por bots, agentes e LLMs.

Este monitor é voltado a **swing trades de médio prazo** e **position trades**, com ênfase em **acumulação, realização parcial e decisões de prazo mais longo**. O gráfico diário é a referência principal de timing e o semanal, o filtro de contexto estrutural. Não é destinado a operações de curto prazo ou day trade.

## Links públicos

- Repositório: https://github.com/matheussamadello/Monitor-BTC-Price
- Página do monitor: https://matheussamadello.github.io/Monitor-BTC-Price/
- Relatório JSON: https://matheussamadello.github.io/Monitor-BTC-Price/relatorio.json
- JSON bruto no repositório: https://raw.githubusercontent.com/matheussamadello/Monitor-BTC-Price/main/docs/relatorio.json

## Referência técnica: BTC/USD

Toda a análise do monitor é feita em **BTC/USD**.

Isso é intencional: uma aplicação que use o relatório pode, por exemplo, comprar BTC usando BRL ou realizar parte de uma posição BTC→BRL, mas os níveis técnicos, suportes, resistências, indicadores e estruturas continuam sendo avaliados em **USD**.

Assim, oscilações de USD/BRL não alteram a tese técnica do BTC/USD. A moeda de execução pode ser BRL; a referência analítica permanece BTC/USD.

## Fonte de dados e timeframes

O monitor consulta o endpoint público OHLC da Kraken para o par `XBTUSD` — nomenclatura usada pela Kraken para BTC/USD — em dois intervalos:

- **Diário:** `1440` minutos.
- **Semanal:** `10080` minutos.

O **diário** é o timeframe principal para timing de pullbacks, rompimentos, retestes, perda/recuperação de níveis, candles e mudanças de momentum.

O **semanal** funciona principalmente como contexto e filtro estrutural: ajuda a identificar se um sinal diário está alinhado, neutro ou em conflito com a estrutura maior. O relatório também calcula os mesmos indicadores principais no semanal e distingue os valores da vela semanal fechada dos valores provisórios da semana em formação.

## Indicadores e leituras calculadas

### RSI

**O período é diferente em cada timeframe**, e o relatório declara qual usou:

| Timeframe | RSI Length | Papel |
| --- | --- | --- |
| Diário | 21 | momentum, perda de força e retomada, na escala de swing e position |
| Semanal | 14 | leitura estrutural de momentum |

Até 2026-09-11 o RSI usava 14 nos dois timeframes, por herdar o mesmo `PERIOD` que o DMI usava. O 14 no diário oscila demais para o horizonte deste monitor.

Uma relação foi preservada de propósito: **em cada timeframe o RSI continua mais responsivo que o DMI/ADX** — 21 contra 28/42 no diário, 14 contra 14/21 no semanal. É do RSI que se espera perceber momentum e retomada antes do DMI confirmar; se ele ficasse mais lento que o ADX, perderia essa função. Há teste fixando isso.

O cálculo usa suavização de Wilder/RMA — igual desde o início, só o período mudou.

O relatório separa:

- `rsi_length`: o período usado **naquele bloco**;
- `rsi_fechado`: calculado apenas com velas fechadas;
- `rsi_provisorio`: inclui a vela atualmente em formação.

Os campos perderam o `14` do nome pelo mesmo motivo dos de DMI: `rsi14_fechado` guardando um RSI de 21 seria mentira.

**Os limiares não mudaram** — 70, 30 e 40 continuam onde estavam. Mas um RSI de 21 é menos extremo que um de 14: `rsi_acima_70` e `rsi_abaixo_30` passam a disparar menos no diário, e `rsi_esfriando` (que exige RSI acima de 40) muda de frequência. As divergências também usam o RSI como oscilador, então mudam de período junto.

O código também detecta divergências RSI confirmadas e provisórias a partir de pivôs de preço.

### DMI/ADX

**Os períodos são diferentes em cada timeframe**, e o relatório declara qual usou:

| Timeframe | DI Length | ADX Smoothing | Papel |
| --- | --- | --- | --- |
| Diário | 28 | 42 | leitura **operacional** de tendência e força, para swing e position |
| Semanal | 14 | 21 | **contexto** da tendência de prazo maior |

Até 2026-09-11 havia um único período, 14, servindo ao mesmo tempo de DI Length e de ADX Smoothing nos dois timeframes. O 14/14 no diário reagia a ruído de curto prazo demais para o horizonte deste monitor; 28/42 alonga a janela e alisa o ADX. No semanal, 14/21 já é lento o bastante nessa escala — alongar mais só atrasaria a leitura sem ganhar filtragem.

São calculados:

- `dmi_di_length` / `dmi_adx_smoothing` — a configuração usada **naquele bloco**;
- `di_plus_fechado` / `di_plus_provisorio`;
- `di_minus_fechado` / `di_minus_provisorio`;
- `adx_fechado` / `adx_provisorio`.

Os campos perderam o `14` do nome na mesma mudança: um campo chamado `adx14_fechado` guardando um ADX alisado em 42 seria mentira. Quem precisa do período lê `dmi_di_length` e `dmi_adx_smoothing`, que saem ao lado dos valores, e o cabeçalho do relatório lista os dois timeframes.

O **semanal não é gatilho de entrada isoladamente** — serve de confirmação e contexto. E o ADX não é o gatilho principal em nenhum dos dois: preço, estrutura, rompimentos/retestes e candles continuam com prioridade.

O cálculo usa a suavização de Wilder/RMA em todas as etapas — só os períodos mudaram. O ADX mede força direcional e deve ser interpretado junto de DI+ e DI−; o monitor não trata ADX isoladamente como direção de mercado.

### EMA89

O monitor calcula uma **EMA exponencial de 89 períodos** e publica, entre outros campos:

Campos que olham a vela **em formação**:

- `ema89`;
- `posicao_vs_ema89`;
- `distancia_ema89_pct`.

`posicao_vs_ema89` e `distancia_ema89_pct` comparam a média com o preço vivo, então mudam durante o dia. Servem para contexto, nunca para confirmação.

Campos calculados **só com velas fechadas**:

- `ema89_fechada_atual`;
- `ema89_fechada_anterior`;
- `ema89_cruzamento_fechado`;
- `distancia_ema89_fechada_atr`.

`ema89_fechada_anterior` é a média no fechamento anterior. Sem ela, quem lê o relatório não conseguia saber se houve travessia sem depender da própria memória de execuções passadas — o monitor publica de hora em hora e não garante essa memória. `ema89_cruzamento_fechado` traz o veredito pronto (`acima`, `abaixo` ou `nenhum`) e `distancia_ema89_fechada_atr` mede a distância do fechamento até a média em ATR do próprio timeframe, que é a margem usada para separar travessia real de simples encostada.

Nada disso exige cálculo novo nem estado guardado entre execuções: são dois pontos de uma série que o monitor já tinha em memória e não publicava.

No uso do relatório por agentes externos, a EMA89 diária pode funcionar como suporte/resistência dinâmica para timing, enquanto a EMA89 semanal é especialmente útil como filtro de contexto maior.

### Dois horizontes

O mesmo relatório atende dois horizontes de swing, sem nenhum campo novo no JSON:

- **tático**, de 1 a 6 semanas, em que o diário pesa mais;
- **estratégico**, de vários meses a mais de um ano, em que o semanal pesa mais.

A separação vive no prompt, não no monitor: `monitor.mjs` publica fatos de mercado e não sabe qual é o horizonte de quem lê. Divergência entre os dois — semanal íntegro e diário cedendo — é o estado normal de um pullback, não erro de dados.

### ATR(14) — volatilidade

O monitor calcula **ATR de 14 períodos** por Wilder sobre velas fechadas e publica:

- `atr14` — em unidade de preço;
- `atr14_pct` — o mesmo em porcentagem do fechamento.

O ATR sempre existiu internamente, dimensionando a largura das zonas automáticas, mas não era publicado. Agora sai no relatório, porque é a leitura que permite dimensionar distância de stop e tamanho de posição sem refazer a conta por fora.

Use `atr14_pct` para comparar **o mesmo timeframe ao longo do tempo**: o valor absoluto não diz nada sozinho. `0,05` é muito ou pouco dependendo do par e da época; `1,18%` é comparável com qualquer coisa.

Já comparar o ATR diário com o semanal não rende conclusão. A amplitude escala com a raiz do número de períodos, então o semanal fica naturalmente em torno de 2 a 2,5 vezes o diário — a diferença é aritmética, não sinal.

Ele também é a unidade em que a obsolescência dos níveis manuais é medida — ver abaixo.

### Candles

O monitor registra a anatomia das velas fechadas e da vela atual, incluindo:

- abertura, máxima, mínima e fechamento;
- corpo;
- sombra superior;
- sombra inferior;
- proporção do corpo e das sombras em relação ao range;
- direção da vela;
- volume.

Também detecta padrões e contextos que realmente existem no código atual, entre eles:

- bullish engulfing;
- bearish engulfing;
- hammer;
- shooting star;
- Três Soldados Brancos;
- Três Corvos Negros;
- versões provisórias dos padrões de três velas;
- `advance_block` e `stalled_pattern` como sinais de enfraquecimento, não como reversão automática.

O código diferencia a geometria do padrão do contexto em que ele ocorre. Isso evita interpretar, por exemplo, uma sequência de três velas de alta em uma tendência já madura como se ela tivesse necessariamente o significado clássico de reversão.

### Volume

O relatório inclui volume da vela atual, volume da última vela fechada, média de 20 períodos, classificação relativa e tendência recente de volume.

Como a vela atual pode estar incompleta, o monitor publica também a fração do período já transcorrida e marca o volume como parcial quando aplicável.

No semanal existe ainda uma comparação equivalente: o volume dos dias já fechados da semana atual pode ser comparado com os mesmos primeiros dias de semanas anteriores, evitando comparar uma semana parcial diretamente com semanas completas.

### A classificação olha a vela fechada, não a em formação

Volume é **acumulado**. Uma vela diária às 6h da manhã tem só as horas já decorridas; compará-la com a média de velas completas dá sempre um número catastrófico, sem que haja nada de anormal acontecendo.

Como este par negocia 24/7, o contador zera toda meia-noite UTC — então o problema não era raro, aparecia **toda madrugada**. Pior que o número feio: um rompimento real nessa janela saía carimbado como `rompimento_com_volume_fraco`.

A correção óbvia seria escalar a média pela fração decorrida, mas isso supõe que o giro se espalha por igual ao longo do período, o que não acontece. A saída sem suposição nenhuma é comparar **período inteiro contra período inteiro**: `volume_vs_media_pct` e `volume_classificacao` olham a última vela **fechada**, e o relatório declara isso em `volume_referencia: ultima_vela_fechada`.

O volume da vela em formação continua publicado, cru, em `volume_atual`, com `volume_parcial: sim` ao lado.

Isso também corrigiu uma incoerência antiga: `rompimento_confirmado` é avaliado sobre a vela **fechada**, mas buscava a confirmação de volume na vela **viva** — duas velas diferentes na mesma frase. O estado `inconclusivo_periodo_inicial` deixou de existir junto: não há mais período inicial a desconfiar.

### Pivôs e estrutura de mercado

O monitor usa pivôs fractais confirmados para classificar estrutura de preço e publica campos como:

- `estrutura_preco`;
- `estrutura_tendencia`;
- `estrutura_ultimo_topo`;
- `estrutura_ultimo_fundo`;
- `estrutura_eventos`;
- `pivos_topos_recentes`;
- `pivos_fundos_recentes`.

Internamente aparecem classificações como HH, HL, LH e LL, que correspondem a:

- HH = topo mais alto;
- HL = fundo mais alto;
- LH = topo mais baixo;
- LL = fundo mais baixo.

**O fractal é por timeframe**, pelo mesmo motivo do RSI e do DMI, e o relatório declara qual usou em `pivos_fractal`:

| Timeframe | Fractal | Confirmação exigida |
| --- | --- | --- |
| Diário | 5/5 | 5 velas à esquerda e 5 à direita; o pivô só é confirmado depois que as 5 velas à direita fecham |
| Semanal | 2/2 | 2 velas à esquerda e 2 à direita; o pivô só é confirmado depois que as 2 velas semanais à direita fecham |

Até 2026-09-11 os dois usavam 2/2, que era o único parâmetro de análise nunca desacelerado quando o monitor assumiu horizonte de swing e position. Na série sintética de 720 velas **sem tendência nenhuma** usada na auditoria, a `estrutura_tendencia` publicada mudou de direção 98 vezes com 2/2 e 43 vezes com 5/5. O ganho do 5/5 diário vem da janela local mais larga e do maior atraso de confirmação, que filtram extremos curtos e reduzem ruído. Isso **não** significa que exista uma distância mínima fixa de 10, 7 ou qualquer outro número de velas entre pivôs consecutivos. O detector de divergências continua exigindo cinco velas entre os pivôs; o 5/5 diário também reduziu leituras recusadas por pivôs próximos. O semanal fica em 2/2 porque cada vela já cobre uma semana: levá-lo a 5/5 faria um pivô candidato esperar cinco velas semanais à direita para ser confirmado, atraso excessivo para a janela útil deste monitor.

`estrutura_tendencia` tem **cinco** valores, não três:

- `alta` — HH + HL;
- `baixa` — LH + LL;
- `lateral_contracao` — LH + HL, o range aperta com o fundo subindo;
- `lateral_expansao` — HH + LL, o range abre pelas duas pontas;
- `indefinida` — não há pivôs suficientes para declarar estrutura.

Antes, os três últimos saíam todos como `lateral_indefinida`. Isso juntava duas situações opostas, contração e expansão, e chamava de lateral um mercado cujo range está **abrindo**. Pior, afirmava um estado de mercado quando o que havia era ausência de dado. A distinção não é acadêmica: o prompt usa a estrutura semanal como uma das condições que promovem um alerta tático a estratégico, e um semanal fazendo fundo mais alto contava como deterioração da tese de prazo longo.

### Divergências

O monitor publica:

- `divergencia_rsi`: divergências confirmadas;
- `divergencia_rsi_provisoria`: divergências que ainda dependem da vela em formação.

A confirmação usa pivôs; por isso uma divergência provisória pode desaparecer antes do fechamento.

## Dados fechados x dados provisórios

Essa distinção é central no projeto.

Campos `*_fechado` usam somente velas concluídas e são a referência principal para confirmação. Campos `*_provisorio` incorporam a vela em formação e podem mudar até o fechamento.

O mesmo princípio vale para padrões, divergências, candle atual e volume parcial.

Em integrações com bots ou LLMs, é recomendável que sinais de maior convicção exijam fechamento quando a regra depender explicitamente de confirmação, enquanto dados provisórios podem ser usados para acompanhamento antecipado sem serem tratados como equivalentes ao fechamento.

## Níveis manuais

Os níveis manuais ficam centralizados em `NIVEIS_USD` dentro de `monitor.mjs`.

Na versão atual do projeto, as faixas publicadas são:

| Faixa | Label |
| --- | --- |
| US$ 79.000–81.000 | `faixa_79k_81k` |
| US$ 76.000–78.000 | `faixa_76k_78k` |
| US$ 74.000–76.000 | `faixa_74k_76k` |
| US$ 72.000–74.000 | `regiao_suporte_72k_74k` |

Além das faixas, o código mantém uma resistência pontual em **US$ 80.000** e um suporte pontual em **US$ 73.000** para a máquina de estados de rompimento/reteste. Os dois ficam dentro de uma faixa, e não soltos: 80.000 na zona mais rejeitada do gráfico, 73.000 na de mais toques abaixo do preço.

**As faixas foram reancoradas em 2026-09-11**, sobre as zonas automáticas observadas. As anteriores tinham envelhecido: a principal estava cerca de mil dólares abaixo da região que o mercado respeita, a do meio cerca de dois mil, e a de suporte não encostava em zona nenhuma. Cada faixa atual cobre integralmente uma zona do diário: 79–81k é a de score 99, com 8 toques e 7 rejeições; 76–78k é a de score 79 com 9 toques, a única que também casa com o semanal; 72–74k é a de score 79 com 10 toques.

A faixa de **74–76k entrou depois, em 2026-09-11**, promovida pelo radar: ele a apontou como zona madura (score 82, 9 toques) e descoberta, caindo no vão entre as duas vizinhas. Com ela o diário foi a 4 de 4 e o radar parou de apontar.

Isso tem um efeito colateral que vale conhecer: com 72–74k, 74–76k e 76–78k encostadas, o intervalo de **72k a 78k fica continuamente coberto**. É o que os dados dizem, já que as três zonas têm de 9 a 10 toques cada, mas significa que "dentro de uma faixa manual" deixa de discriminar nessa janela. O que segura a leitura útil é a frase nomear **qual** faixa, e não só dizer que está em uma.

No semanal o alinhamento fica em 2 de 4, e não há conserto: as zonas semanais ficam em 66–68k e 58–60k, muito abaixo, e um único conjunto de faixas serve aos dois timeframes. O suporte semanal de score 90 em 66–68k existe e não foi marcado, para as faixas não ficarem espalhadas demais.

As faixas são publicadas diretamente no objeto `niveis_manuais` do `relatorio.json`, derivadas da configuração do código. Portanto, consumidores externos devem preferir o JSON como fonte de verdade dos valores atuais em vez de manter cópias eternas desses números.

## Vigilância dos níveis manuais

Os níveis manuais são a espinha da política de alerta: janela agressiva, confirmação conservadora e a própria revisão de níveis partem todos deles. Quando envelhecem, o monitor não passa a errar — ele fica **mudo** justamente na parte que mais importa, e nada avisa.

Foi o que aconteceu no monitor de XMR: o preço rompeu a resistência manual em 17/08 e seguiu até 29% acima da faixa mais alta configurada, republicando de hora em hora um rompimento que havia muito deixara de ser notícia. Detectar isso estava delegado a quem lesse o relatório, e é exatamente o tipo de coisa que ninguém nota, porque nada acontece.

Agora o relatório publica, por par e por timeframe:

| Campo | O que traz |
| --- | --- |
| `niveis_manuais_situacao` | `atual`, `monitorar` ou `obsoleto` |
| `niveis_manuais_distancia_atr` | distância do **último fechamento** até a faixa manual mais próxima, em ATR |
| `niveis_manuais_faixa_mais_proxima` | qual faixa é essa |
| `niveis_manuais_alinhamento` | `alinhado`, `parcial`, `desalinhado` ou `indefinido` |
| `niveis_manuais_faixas_corroboradas` | quantas faixas caem sobre uma zona automática |

Os cortes são **1 ATR** e **3 ATR**: dentro de uma faixa ou a menos de 1 ATR dela é `atual`; entre 1 e 3 é `monitorar`; além de 3 é `obsoleto`.

As duas pontas da conta usam **vela fechada** — o fechamento e o ATR. A primeira versão passava o preço da vela em formação, o que misturava provisório com confirmado num híbrido sem significado limpo, e contrariava a convenção do próprio monitor, em que o que alimenta decisão usa vela fechada. E este campo alimenta uma: a revisão dos níveis manuais. O custo é uma vela de latência, irrelevante para um sinal cujo caso de origem levou 19 dias para ser notado.

A distância é medida em ATR, e não em porcentagem, de propósito. Cinco por cento é muito num par de câmbio e pouco num de cripto, enquanto "três vezes a volatilidade diária" quer dizer a mesma coisa em qualquer um — um limiar só serve para os três monitores, sem recalibragem.

**Situação e alinhamento medem coisas diferentes.** A situação mede a distância do preço; o alinhamento mede se as faixas continuam caindo onde o mercado de fato reage, comparando cada uma com as zonas automáticas pelo mesmo critério de sobreposição usado nas confluências. Os dois podem discordar, e é justamente a discordância que interessa: uma faixa pode estar a 0,66 ATR do preço, portanto `atual`, e mesmo assim estar deslocada da região que o mercado respeita.

Era o caso do monitor de BTC quando este campo foi criado, e foi ele que motivou a reancoragem das faixas. Nenhuma das três atingia o limite de sobreposição no semanal, e no diário a mais importante ficava cerca de mil dólares abaixo da região que o mercado respeitava. Nada no relatório dizia isso, porque o único campo que olhava as faixas media distância até o preço. O sinal existia por zona, em `confluencia_faixa_manual`, mas nunca era somado.

`obsoleto` não é alerta de mercado: é aviso de manutenção. Significa que os níveis descrevem um regime que ficou para trás e precisam de revisão.

### Perda de suporte: forte x fraca

`rompimento_confirmado_X` só sai quando o **corpo inteiro** da vela fechada está acima da resistência; se só o fechamento passou, sai `rompimento_confirmado_fraco_X`. O suporte não tinha essa distinção: qualquer fechamento abaixo, por qualquer margem, virava `perda_suporte_confirmada_X` e entrava em `deterioracao_tendencia`.

O caso que expôs isso foi o XMR/USD em 2026-09-08: abriu 519,23 e fechou 499,77 com suporte em 500. Fechou 0,23 abaixo — menos de um centésimo de ATR — com o corpo inteiro em cima do nível. Saía como perda confirmada enquanto a máquina de estados, que olha o corpo, dizia `sem_registro`.

Agora o suporte espelha a resistência: `perda_suporte_confirmada_X` exige o corpo abaixo; só o fechamento abaixo vira `perda_suporte_confirmada_fraca_X`. A versão fraca continua contando como `suporte_sob_pressao` em `riscos_tecnicos`, mas não entra em `deterioracao_tendencia`. A síntese também passou a ignorar a versão fraca do rompimento em `confluencia_entrada`, que antes escapava por causa do prefixo.

## Máquina de estados de rompimento e reteste

Os níveis pontuais possuem estado persistente, avaliado sobre a **última vela fechada**, para evitar oscilações intradiárias da máquina de estados.

Os principais estados implementados são:

- `rompimento_candidato`;
- `rompido`;
- `em_reteste`;
- `reteste_confirmado`;
- `rompimento_falhou`;
- `recuperado`.

O registro marca `afastado` sempre que o preço estiver além da distância de reset do nível, **em qualquer estado**. Isso já foi diferente: a marcação só valia ao encerrar um ciclo de reteste, então um nível rompido semanas antes e deixado 29% para trás continuava publicando `afastado: nao`, e quem lesse concluía que o preço ainda estava por perto. O encerramento do ciclo — voltar de `reteste_confirmado` ou `recuperado` para `rompido` — continua restrito aos dois estados em que faz sentido. Estados inativos podem ser arquivados sem apagar o histórico do ciclo.

A máquina diferencia um critério sensível, que apenas arma um `rompimento_candidato`, de um critério mais rigoroso usado para classificar `rompido`.

**A tolerância e a distância de reset são medidas em ATR**, como todo o resto do projeto: 0,25 ATR de largura em torno do nível e 1,5 ATR para encerrar o ciclo. Eram percentuais fixos, e um percentual fixo vale coisas diferentes em cada ativo. Com 0,5%, a janela de reteste valia 0,077 ATR no XMR/USD e 0,592 ATR no USDT/BRL, quase oito vezes mais larga: num par ela era dez vezes mais estreita que uma zona automática e no outro quase do tamanho de uma zona inteira, de modo que `reteste_confirmado` queria dizer coisas diferentes em cada lugar. O reset do USDT/BRL chegava a 3,55 ATR, além dos 3 ATR em que os níveis já são declarados obsoletos, então o ciclo praticamente nunca reiniciava. O monitor de câmbio já tinha os números cortados pela metade à mão para contornar isso; medidos em ATR, aqueles valores ajustados davam 0,23 a 0,30 ATR e 1,36 a 1,78 ATR, quase exatamente os valores que agora valem para qualquer par sem ajuste por fonte.

**Um nível abandonado é arquivado, nunca apagado.** Quando passa da janela de inatividade sem contato, o registro vira `arquivado` e fica dormente; se o preço voltar a encostar, ele acorda e o ciclo recomeça em reteste. Antes o registro era apagado quando ainda não tinha histórico, e na vela seguinte o nível nascia do zero em `rompido` — que o relatório anuncia como rompimento novo. Uma resistência rompida com o preço indo embora reanunciava o mesmo rompimento a cada 32 velas diárias, indefinidamente: sete anúncios em duzentos dias, contra o que o prompt promete, que um rompimento vira notícia uma vez só.

## Zonas automáticas de suporte e resistência

Além dos níveis manuais, o monitor calcula **zonas automáticas** a partir dos pivôs confirmados.

No código atual essas zonas são **contexto técnico**. Elas não alteram sozinhas os gatilhos manuais, a máquina de estados dos níveis ou as confluências principais do monitor.

### ATR e agrupamento de pivôs

As zonas usam ATR(14) de Wilder calculado sobre velas fechadas. Cada pivô recebe o ATR correspondente à época em que ocorreu.

Os pivôs são agrupados por distância normalizada pela volatilidade histórica, com uma verificação entre todos os membros do cluster para evitar o efeito de encadeamento em que A≈B e B≈C acabariam unindo A e C mesmo quando estão distantes entre si.

### Limites estruturais

`limites_estruturais` representam a região histórica da zona. São derivados dos pivôs que formam o cluster e da volatilidade da época desses pivôs.

Esses limites são usados principalmente para:

- identidade da zona;
- matching entre execuções;
- merge de regiões;
- confluência histórica.

Eles não são recalculados retroativamente apenas porque a volatilidade atual mudou.

### Limites operacionais

`limites_operacionais` são ajustados ao regime atual de volatilidade. No código atual, são construídos em torno do centro da zona usando o ATR fechado atual.

São usados principalmente para medir:

- interação atual do preço com a zona;
- estado `em_teste`, `acima` ou `abaixo`;
- distância operacional.

Assim, uma mesma zona pode manter sua identidade estrutural enquanto sua faixa operacional se adapta à volatilidade corrente.

### Score e qualidade da zona

Cada zona recebe um `score` normalizado de 0 a 100 com base nos fatores aplicáveis ao timeframe. O código atual considera:

- número de episódios/toques;
- número de rejeições;
- recência;
- força média da reação em ATR;
- confluência semanal para zonas diárias;
- `role_reversal`;
- contexto de volume.

Também existem penalidades multiplicativas para casos como repetidos rompimentos sem reação, episódio único e wick isolado sem rejeição.

A proximidade do preço não entra no score: ela serve para ordenar quais zonas próximas são publicadas, não para medir a força histórica da zona.

### Radar de promoção

As zonas automáticas são **contexto**. Elas não alimentam a máquina de rompimento e reteste, não entram na linha de gatilhos e não geram alerta de entrada em faixa: isso tudo roda só sobre os níveis manuais. Uma região que o mercado passou a respeitar fica sem máquina de estados até alguém promovê-la a faixa manual. E zonas **expiram** depois de semanas sem toque, enquanto faixas manuais não.

`zonas_candidatas_a_faixa` existe para essa promoção não depender de alguém reparar nela. Lista regiões com **score 70 ou mais e pelo menos 5 toques** que nenhuma faixa manual cobre, no máximo três, das de maior score para as menores, dizendo de que lado do preço cada uma está.

**Só no bloco diário.** No semanal a estrutura fica num patamar diferente do diário, e um único conjunto de faixas serve aos dois timeframes: promover uma zona semanal quebraria o alinhamento diário, que é o operacional. Sinalizar lá seria uma lista enorme, permanente e sem ação possível, então o campo sai sempre como `nenhuma` no semanal.

Os dois cortes filtram exatamente o que não serve. Zonas de 1 toque e score baixo aparecem acima do preço em quase todo par e não significam nada ainda; zona de score alto com 2 toques também não, porque o radar exige as duas coisas. Quando o campo trouxer algo, é manutenção de configuração, não alerta de mercado: a região merece virar faixa manual para ganhar máquina de estados.

Na página, a linha só ocupa espaço quando há candidata.

### Toques, rejeições e role reversal

O monitor reconstrói episódios históricos de contato com cada zona e registra `numero_toques`, `numero_rejeicoes` e `forca_reacao_atr`.

Um `role_reversal` não é marcado apenas porque o preço apareceu do outro lado da região. O código exige uma sequência cronológica de interação de um lado, cruzamento confirmado e nova reação/rejeição pelo lado oposto.

### Confluências

Uma zona pode publicar, entre outros campos:

- `timeframes_confirmando`;
- `confluencia_nivel_manual`;
- `confluencia_faixa_manual`;
- `confluencia_manual_qualquer`;
- `cruzamento_confirmado`;
- `volume_contexto`;
- `volume_relativo_mediano`;
- `distancia_preco_atual_pct`.

`confluencia_nivel_manual` cobre apenas os níveis pontuais: é verdadeiro quando o nível cai dentro dos limites estruturais da zona.

`confluencia_faixa_manual` cobre as faixas de `NIVEIS_USD.faixas`. Como faixa é região e não linha, o critério é interseção entre a faixa e os limites estruturais da zona.

`confluencia_manual_qualquer` agrega as duas. Consumidores externos não devem inferir que `confluencia_nivel_manual` representa sozinho toda forma possível de confluência manual.

Existe ainda `confluencia_resistencia_macro`, publicado apenas quando há uma âncora macro configurada em `NIVEIS_USD.resistenciaMacro`. Na configuração atual não há, então o campo não aparece. Sua ausência é esperada, não é erro.

No relatório público são mostradas até **três zonas acima e três abaixo do preço** entre as zonas publicáveis. O `estado.json` mantém todas as zonas vivas necessárias para preservar identidade e histórico, mesmo quando alguma delas não aparece entre as mais próximas no relatório público.

## `relatorio.json`

`docs/relatorio.json` é a principal interface estruturada do projeto para integrações.

Ele contém:

- cabeçalho e timestamp;
- bloco diário de BTC/USD;
- bloco semanal de BTC/USD;
- preço e OHLC atual;
- EMA89;
- RSI e DMI/ADX fechados e provisórios;
- candles recentes;
- volume;
- estrutura e pivôs;
- divergências;
- padrões;
- alertas técnicos internos;
- confluências e deteriorações;
- `niveis_manuais`;
- `zonas_automaticas`;
- `gatilhos_ativos`.

O JSON é construído a partir do mesmo relatório textual usado para a página, e as zonas automáticas são injetadas a partir do objeto canônico calculado pelo monitor. A intenção do código é impedir que a página e o endpoint JSON passem a representar leituras calculadas diferentes.

Para bots, agentes e LLMs, este é o arquivo recomendado para leitura periódica.

## `estado.json`

`docs/estado.json` é a memória persistente entre execuções.

Ele armazena atualmente:

- `ativos`: gatilhos internos ativos;
- `em`: timestamp de atualização do estado;
- `niveis`: estado persistente da máquina de rompimento/reteste;
- `zonas`: coleção completa de zonas vivas por timeframe, inclusive zonas que podem não estar entre as publicadas no relatório;
- `contadoresZona`: contadores usados para preservar a identidade das zonas.

O arquivo não substitui `relatorio.json` como interface de consumo. Sua principal finalidade é impedir que o monitor esqueça ciclos de níveis, IDs de zonas, históricos e contadores entre uma execução e outra.

## Arquivos gerados

Ao executar `node monitor.mjs`, o monitor cria ou atualiza:

```text
docs/
├── .nojekyll
├── estado.json
├── index.html
├── index.txt
└── relatorio.json
```

### A página publicada (`index.html`)

A página serve a dois leitores ao mesmo tempo, com prioridades opostas.

**Tema.** Segue o **`prefers-color-scheme` do sistema**: quem usa tema escuro no SO ou no navegador abre em *night mode* — fundo azul-noite, azul nos títulos e nas etiquetas —, quem usa claro abre no tema claro, com fundo quase branco e texto quase preto. O botão no topo alterna, e a partir daí a escolha fica salva no navegador e passa a mandar sobre o sistema. Quem nunca clicou continua seguindo o SO até enquanto a página está aberta: trocar o tema do sistema muda a página na hora.

Não há regra por **horário**, de propósito. Quem quer tema escuro à noite já liga o agendamento automático do próprio sistema, e o `prefers-color-scheme` entrega isso de graça. Uma regra própria brigaria com quem escolheu claro deliberadamente, e faria a página mudar de cara sozinha conforme a hora de abrir — o que se lê como defeito, não como recurso. O tema claro **só redefine tokens de cor**: nenhuma regra de layout existe duas vezes, então os dois não têm como divergir de estrutura. Há um teste que compara os dois conjuntos de tokens e falha se alguém acrescentar uma cor no escuro e esquecer do claro — senão o tema claro herdaria uma cor de fundo escuro em silêncio.

Para **você**: um cartão por par com o resumo dos dois timeframes — último fechamento, lado e distância da EMA89 **em porcentagem**, RSI e ADX com DI+/DI− (ambos rotulados com os períodos daquele timeframe), estrutura, situação dos níveis manuais —, mais os alertas técnicos como etiquetas.

**O ATR não aparece no cartão**, e a distância da EMA89 sai em porcentagem em vez de em múltiplos de ATR. Isso é só apresentação, e a razão é que ATR é unidade de **cálculo**, não leitura de relance: `0,0418` é muito ou pouco dependendo do par, enquanto `0,56%` se lê na hora.

Internamente nada mudou. O ATR continua dimensionando a largura das zonas automáticas, medindo a obsolescência dos níveis manuais e servindo de unidade para as margens do prompt — 0,25 ATR para a travessia semanal, 1,0 ATR para a corroboração da perda diária. E o relatório continua publicando `atr14`, `atr14_pct` e `distancia_ema89_fechada_atr` para quem dimensiona stop e tamanho de posição.

A porcentagem da EMA89 é calculada direto pela relação entre o **fechamento** e a EMA89, não convertida do múltiplo de ATR. O denominador é a própria EMA, mesma convenção do `distancia_ema89_pct` — só que com o fechamento no lugar do preço vivo, para o cartão não voltar a misturar vela fechada com vela em formação. O que está em `deterioracao_tendencia` sai em vermelho; o resto, em azul. O carimbo de tempo no topo calcula sozinho, no navegador, há quanto tempo o relatório foi gerado, e muda de cor a partir de 90 minutos.

Para o **agente**: o relatório inteiro continua saindo *verbatim* dentro de um único `<pre>`, em texto puro, com o mesmo escape de sempre (`&` e `<`, nada mais). O prompt usa esta página como fallback quando o `relatorio.json` não responde, e quem lê procura linhas `campo: valor` no fonte — uma única `<span>` ali dentro quebraria isso, e quebraria justamente quando a fonte principal já estivesse fora do ar. Por isso o tema é moldura em volta do bloco, nunca dentro dele, e há um teste de fumaça que compara o `<pre>` byte a byte com o relatório e falha se aparecer qualquer tag lá.

Os cartões **não** reparseiam o texto: eles leem o mesmo objeto de `relatorioParaJSON` que vira o `relatorio.json`, gerado uma vez só e passado para os dois. Dois leitores do mesmo objeto não têm como discordar.

O par que tiver `grafico` na configuração ganha também um gráfico embutido — com a **EMA89** e o **RSI** já carregados, e uma barra com **Diário** e **Semanal** logo acima dele. Trocar ali troca o intervalo do desenho **e o período do RSI junto** — 21 no diário, 14 no semanal, os mesmos que o cartão de cada timeframe publica, para o desenho nunca contradizer o número ao lado. Os períodos são **derivados da configuração**: mudar `tf.rsi.length` move o desenho junto, sem ninguém precisar lembrar de vir aqui. A EMA89 é a mesma nos dois intervalos, porque não é um valor por timeframe.

Esses botões são **nossos**, e não os da barra do TradingView. O widget público roda num iframe de outra origem e não avisa quando alguém troca o intervalo por dentro dele: trocar por lá muda as velas e deixa o RSI como estava. Acompanhar a troca feita na barra do próprio widget exigiria a Charting Library licenciada, que é outro produto — por isso o controle fica do lado de fora, onde dá para redesenhar o widget inteiro com o período certo. Um clique em qualquer uma das barras move todos os gráficos da página, que assim nunca ficam mostrando intervalos diferentes. A EMA é a mesma linha que o cartão cita, e o RSI vai sem a média móvel que o TradingView inclui por padrão nele, que só polui. O ADX/DMI fica de fora de propósito: junto com o RSI o embed fica carregado, e o padrão dele já serve quando se adiciona na hora, pelo próprio widget. Ele é desenhado pelo TradingView e redesenhado quando o tema muda, porque mora num iframe e o tema dele é escolhido na criação do widget, não por CSS. A Kraken não publica widget de embed próprio; o TradingView publica, e serve a série da **própria Kraken** sob o símbolo `KRAKEN:…` — é a mesma fonte do relatório, não uma segunda opinião. Se o script não carregar, fica no lugar um link para o gráfico completo e nada mais na página se perde.

Uma diferença que vale conhecer antes de comparar número com desenho: a Kraken alinha a vela semanal pela **época do Unix**, que caiu numa quinta-feira, então a semana dela vai de quinta a quarta. O TradingView desenha a semana de segunda a domingo. Os dois estão certos dentro da própria régua, mas as velas semanais do gráfico não coincidem com o `ultimo_fechamento_data` semanal do relatório. No diário não há divergência.

Também pode ser criado `alerta.txt` na raiz quando surgem novos gatilhos internos. O workflow oficial, entretanto, faz `git add docs`, portanto esse arquivo não é publicado pelo processo automático atual.

## Estrutura simplificada do repositório

Considerando a estrutura atual e os dois arquivos de documentação deste pacote:

```text
Monitor-BTC-Price/
├── .github/
│   └── workflows/
│       └── monitor.yml
├── docs/
│   ├── .nojekyll
│   ├── estado.json
│   ├── historico.jsonl
│   ├── index.html
│   ├── index.txt
│   └── relatorio.json
├── monitor.mjs
├── teste-fumaca.mjs
├── analisar-historico.mjs
├── README.md
└── PROMPT_BTC_TECHNICAL_WATCH.md
```

## Contexto: as duas linhas para quem não é trader

O relatório tem mais de cem campos por bloco. Quase todos são refinamento, e refinamento serve para quem opera com frequência. Quem acumula ao longo do tempo e às vezes gasta precisa responder uma pergunta só: **é um momento melhor para comprar mais, para esperar ou para converter parte?**

A seção **Contexto**, no topo da página antes dos cartões, responde isso em **duas linhas por par**, dentro de uma caixa só: `longo` diz onde o preço está, `curto` diz o que aconteceu. Foram duas seções separadas até 2026-09-12, e as notas explicativas respondiam por **73% do texto do topo** (684 caracteres de nota contra 209 de leitura). Juntas, some um título e uma nota inteira, o topo cai de 933 para 545 caracteres, e as duas leituras passam a ser lidas lado a lado, que é como se completam. A explicação longa mora aqui no README, que é o lugar dela. Ela usa **doze campos, todos do bloco semanal**: o fechamento, a EMA89, a distância entre os dois em ATR, o RSI, a estrutura, DI+, DI− e ADX, e a situação, a distância, a faixa mais próxima e o alinhamento dos níveis manuais.

A ordem da linha segue a **lista de prioridade de leitura do prompt**: níveis primeiro, média longa depois, indicadores por último. RSI e DMI ficam os dois, porque dizem coisas diferentes: o RSI mede o quanto o preço se esticou, o DMI mede se o movimento tem força. Medidos em série sintética, a correlação entre os dois fica em torno de 0,34 em regime de tendência e cai para praticamente zero em lateralização, ou seja, cerca de nove décimos do que cada um diz o outro não diz.

| Rótulo | Quando aparece |
| --- | --- |
| barato ante a média longa | 1 ATR ou mais **abaixo** da EMA89 semanal, sem queda instalada |
| barato, mas ainda caindo | o mesmo, porém com estrutura semanal em `baixa` **ou** DI− dominante com ADX em 25 ou mais |
| esticado, mas a alta ainda tem força | 1 ATR ou mais **acima**, RSI em 70 ou mais, com DI+ dominante e ADX em 25 ou mais |
| esticado e a alta perdendo força | o mesmo, porém sem essa força |
| acima da média, sem esticamento | longe acima, mas o RSI não acompanha |
| na média longa | a menos de 1 ATR da EMA89, para qualquer lado |

Nenhuma regra nova de mercado nasce aí. O corte de 1 ATR é o mesmo que o monitor usa para separar perto de longe na obsolescência dos níveis, 70 é a referência de RSI que o prompt usa em todo lugar, e 25 é o mesmo corte de força de ADX que a linha de eventos já usava.

**O DMI entrou porque o RSI sozinho juntava situações opostas.** RSI alto diz que subiu bastante nas últimas semanas. Não diz se a alta continua viva. Subiu muito com a compra ainda mandando e subiu muito com o movimento morrendo pedem ações contrárias, e até 2026-09-11 as duas saíam com o mesmo rótulo. Quem separa é o ADX com os DIs, não o RSI. Do lado barato vale o espelho, com uma vantagem: o DMI denuncia a queda viva **antes** da estrutura, que depende de pivôs confirmados e por isso é lenta. Qualquer um dos dois basta para marcar `ainda caindo`.

Abaixo do corte de 25 a razão diz `sem tendência firme` e não nomeia direção. ADX fraco significa que não há tendência instalada, e escrever "alta perdendo força" com ADX em 14 seria inventar uma alta que não existe.

Barato e caindo saem com rótulos diferentes de propósito: juntar os dois seria mentira. E uma alta longe da média sem RSI esticado não vira alarme, porque alta saudável é o estado normal de uma tendência.

**A linha mostra os números que a produziram, e nenhum deles em jargão.** Essa é a única parte da página escrita para quem não sabe análise técnica, e um número numa unidade que a pessoa não entende não deixa nada conferível, que era a razão de a linha existir. Então:

- a linha começa pelos **níveis**, que são a prioridade 1 da lista do prompt. A faixa nascia apoiada na média longa (prioridade 3) e nos indicadores (prioridade 5), pulando o item mais importante. Sai como `dentro da faixa manual de 76.000 a 78.000`, `dentro da região de suporte manual de …`, `encostando na`, `perto da` ou `longe das faixas manuais`. **A frase nomeia os limites da faixa**, porque sem isso ela afirmava algo sobre "uma faixa" e obrigava quem lê a procurar o rótulo no cartão, e ainda a lembrar que esta leitura sai do bloco **semanal**, não do diário. Longe de todas, nomear uma não ajudaria, então ali o intervalo não aparece. As casas decimais se ajustam ao par: 76.000 sai inteiro, 0,00656 sai com cinco casas, 5,12 com duas;
- uma faixa que as zonas observadas não corroboram é citada **com a ressalva** `(que o mercado não vem respeitando)`. Dar destaque a um nível desalinhado sem avisar seria pior que omiti-lo;
- a distância sai em **porcentagem**, que dispensa explicação, e não em ATR;
- o critério de 1 ATR vira **palavra**: `bem acima` e `bem abaixo` contra `perto`. A palavra carrega o critério, o número carrega o tamanho;
- o RSI vira `momentum esticado`, `normal` ou `muito fraco`, sem o número;
- o DMI vira `alta ainda forte`, `queda ainda forte` ou `sem tendência firme`;
- a estrutura sai da linha. Quando ela muda o veredito, isso já aparece no próprio rótulo.

O ATR continua sendo o que **decide**, e continua inteiro no relatório abaixo, para o agente e para quem consome os campos. Ele só não aparece aqui.

Um efeito colateral que confunde à primeira vista: `bem abaixo (−2,9%)` no dólar e `bem acima (+54,2%)` no Monero podem aparecer lado a lado. Está certo. Perto e longe são medidos contra o quanto **cada par** costuma oscilar, então 3% já é bastante no câmbio e é pouco numa cripto. A nota ao pé da faixa diz isso em uma linha.

Ela **descreve enquadramento, não recomenda operação e não gera alerta**. É contexto que muda de estado poucas vezes por ano, e é essa lentidão que a torna útil para horizonte longo.

### Um par por vez

Nos monitores com **mais de um par com cartão**, um seletor no topo mostra um par de cada vez, filtrando a caixa de contexto **e** o cartão ao mesmo tempo. A página inteira passa a falar de um par só, e a altura cai pela metade no monitor de XMR, que é o único onde isso aparece: os outros dois têm um par com cartão e o seletor nem é gerado, porque um botão que não faz nada é pior que nenhum botão.

Três cuidados que valem conhecer:

- **o relatório completo não é filtrado.** Ele é o fallback do prompt e sai sempre inteiro, com todos os pares. O seletor mexe só no resumo visual;
- **sem JavaScript nada fica escondido.** A classe que oculta só é aplicada pelo script, então a página servida traz tudo e continua completa se o script não rodar;
- **o gráfico é redesenhado a cada troca.** O widget do TradingView calcula o tamanho na criação, e um container que nasceu escondido sairia com dimensão zero e ficaria quebrado ao aparecer. Recriar com ele já visível resolve.

Dentro de cada caixa, uma linha separa a leitura `longo` da `curto`. São duas leituras de escalas diferentes e a divisória deixa isso óbvio sem precisar de mais texto.

### O `(?)` ao lado de cada rótulo

O rótulo é curto por obrigação de layout: `esticado e a alta perdendo força` já é o limite do que cabe numa linha, e a explicação inteira não caberia em nenhuma. O `(?)` ao lado resolve isso sem custo de espaço — passando o mouse no computador, tocando no celular, ou com o foco do teclado. Quem já sabe o que o rótulo quer dizer nunca precisa abrir.

O texto vem do mapa `EXPLICACOES`, e a chave sai da **própria leitura**, não do texto do rótulo: casar por texto faria a explicação sumir em silêncio no dia em que alguém reescrevesse um rótulo. O teste de fumaça cobre os dois lados — toda chave que as leituras sabem produzir tem explicação, e nenhuma explicação fica órfã — e também que **nenhuma delas usa jargão**, a mesma regra que já vale para a razão de cada linha.

É markup só da parte visual: o relatório completo, que é o que o agente lê, sai idêntico.

### A linha `curto`: o que aconteceu no diário

A linha `longo` diz **onde** o preço está. A linha `curto`, logo abaixo dela na mesma caixa, diz o que está **acontecendo** no timeframe que o projeto usa para timing.

O diário não é day trade. Os períodos dele são todos calibrados para o horizonte tático de 1 a 6 semanas: RSI 21, DMI 28/42, pivôs 5/5 e janela de reteste de 30 velas. Nada aqui usa a vela em formação.

O núcleo é a **máquina de rompimento e reteste**, que roda só sobre os níveis manuais e é a prioridade 2 da lista de leitura do prompt. É a sequência que mais importa para decidir uma entrada: rompeu, voltou, segurou.

| Rótulo | Quando aparece |
| --- | --- |
| reteste confirmado | algum nível manual em `reteste_confirmado` |
| reteste em curso | em `em_reteste` |
| rompimento falhou | em `rompimento_falhou` |
| nível recuperado | em `recuperado` |
| rompido, sem reteste ainda | em `rompido` |
| rompimento em avaliação | em `rompimento_candidato` |
| cruzou a média diária | sem evento de nível, mas a EMA89 diária foi cruzada no fechamento |
| sinais de enfraquecimento | sem os anteriores, mas `deterioracao_tendencia` traz algo |
| sem evento no diário | nada disso |

A ordem da tabela é a ordem de relevância: com dois níveis em estados diferentes, vence o mais decisivo, não o primeiro da configuração. A razão diz de qual nível se trata, se é resistência ou suporte, e onde o fechamento está em relação à média diária, em porcentagem.

**O destaque em amarelo quer dizer "vale olhar", não quer dizer bom nem ruim.** Um reteste confirmado de resistência rompida para cima e um de suporte perdido para baixo têm o mesmo nome e significados opostos. Inventar a direção aqui seria palpite disfarçado de leitura, então a cor sinaliza atenção e o texto nomeia o nível, deixando o julgamento com quem lê.

Sem jargão, pela mesma razão da faixa longa. E quando o fechamento fica a menos de 0,05% da média, a razão diz `em cima da média diária` em vez de arredondar para `0,0% abaixo`, que afirmaria um lado que o número não sustenta.

## Histórico: o substrato para medir

Nenhum parâmetro deste projeto foi validado contra resultado. Os períodos, os limiares, os pesos do score das zonas: tudo foi escolhido por raciocínio, e raciocínio bem argumentado continua sendo palpite até alguém medir. `docs/historico.jsonl` existe para que um dia seja possível medir.

É um arquivo **append-only**, uma linha JSON por entrada, versionado junto com o resto. Ele não altera o relatório, não dispara nada e não é lido por nenhuma decisão do monitor.

**Por que registra condição, e não alerta.** Os alertas não saem daqui. Quem decide alertar é o agente no ChatGPT, que lê o prompt e resolve sozinho, e o monitor não tem como ver essa decisão. O que o monitor vê, e pode registrar com precisão, são as condições que ele publicou e o preço de cada fechamento. Isso basta para a pergunta que importa: cada condição foi seguida de que movimento?

Grava uma linha por par e timeframe sempre que a **vela fechada** muda ou qualquer condição muda. Execuções horárias sobre a mesma vela fechada não repetem linha, porque a assinatura que decide isso ignora o horário. Como a vela entra na assinatura, todo fechamento gera linha mesmo sem condição nenhuma, e é dessa série de preços que saem os retornos futuros. Bloco em falha não vira entrada: registrar uma queda de fonte como se fosse leitura de mercado contaminaria a medição depois.

Cada linha traz a vela, o fechamento, o ATR, RSI, ADX com DI+/DI−, estrutura, lado e cruzamento da EMA89, os alertas técnicos, deterioração, confluências, riscos, mudanças de nível e a situação e o alinhamento das faixas manuais.

### Como medir

```bash
node analisar-historico.mjs 10
```

O argumento é o horizonte em velas fechadas. O script junta cada condição ao que o preço fez depois e imprime, por condição, a quantidade de amostras, o retorno mediano **em ATR** e a fração de vezes em que subiu. Em ATR, e não em porcentagem, pelo mesmo motivo do resto do projeto: 3% é muito num par de câmbio e pouco num de cripto, e uma tabela que mistura os dois não quer dizer nada.

A linha `TODAS AS VELAS (referência)` é o que o par fez em toda vela do período. **É contra ela que se compara, não contra zero.** Uma condição que não bate a referência não está acrescentando informação, por melhor que pareça o número absoluto. Condições com menos de cinco amostras são omitidas.

Vale rodar em mais de um horizonte. Uma condição útil deveria continuar útil em 5 e em 20 velas; uma que só funciona num horizonte específico é ruído que encontrou um número.

**Não espere resposta nos primeiros meses.** Detectar uma vantagem pequena contra a volatilidade diária exige muitas amostras, e o script prefere dizer que não sabe a inventar conclusão com uma dúzia de casos.

### O que o histórico não alcança

Ele não sabe o que o agente escolheu enviar. Se você quiser medir isso também, o prompt pede que cada alerta termine numa linha compacta de registro; basta colar essas linhas em `alertas-enviados.jsonl` na raiz. Sem isso, dá para saber quais condições têm valor, mas não se o agente está encaminhando as certas.

O rodapé carrega o horizonte, e aceita quatro valores: `TÁTICO`, `ESTRATÉGICO`, `AMBOS` e `MANUTENÇÃO`. O último é obrigatório para revisão de níveis e radar de promoção, e existe por dois motivos. Aviso de configuração marcado como tático ocuparia a vaga tática da execução, podendo suprimir um sinal de mercado real no mesmo ciclo. E contaminaria a medição depois, misturando manutenção com chamada de mercado na mesma coluna.

## Teste de fumaça

`teste-fumaca.mjs` roda o monitor inteiro contra séries sintéticas no formato OHLC da Kraken, **sem tocar na rede**. Existe porque o monitor publica sozinho de hora em hora: sem ele, um refactor que quebre o parse ou o cálculo só apareceria em produção, com o relatório já no ar.

Verifica:

- que o relatório sai inteiro, sem `NaN` e sem `undefined`;
- que RSI, ADX, EMA89 e a estrutura de pivôs são calculados;
- que as linhas `eventos:` e `eventos_semanal:` continuam nos seus blocos;
- que a fonte fora do ar vira `FALHA:` citando o status, sem interromper o relatório;
- que um erro de aplicação da fonte aparece no relatório;
- que a vela em formação não puxa a classificação de volume, e que uma queda ou um pico reais na vela fechada continuam sendo detectados;
- que o `relatorio.json` continua parseável e tipado, com as faixas manuais de cada par;
- que uma segunda execução lê o estado da anterior sem quebrar.

Rode com:

```bash
node teste-fumaca.mjs
```

O workflow roda esse teste **antes** de gerar o relatório: se algo quebrou, o job para ali em vez de publicar um relatório pela metade.

## GitHub Actions

O workflow oficial está em `.github/workflows/monitor.yml`.

### Frequência

O cron atual é:

```yaml
- cron: "20 * * * *"
```

Ou seja, o GitHub Actions solicita uma execução **uma vez por hora, no minuto 20 UTC**. Os três monitores da família são espaçados em 20 minutos — **XMR no 00, BTC no 20, USD no 40** — para as chamadas às fontes ficarem distribuídas e dar para saber qual execução é qual só pelo horário no log. Como todo cron do GitHub Actions, o início efetivo pode sofrer atraso de fila da própria plataforma.

O workflow também possui `workflow_dispatch`, permitindo execução manual pela aba **Actions**.

### Node.js usado oficialmente

O workflow atual usa:

```yaml
- uses: actions/setup-node@v5
  with:
    node-version: "22"
```

Portanto, **Node.js 22 é a versão usada pelo workflow oficial**.

Isso não significa, por si só, que Node.js 22 seja uma exigência absoluta para execução local. O próprio `monitor.mjs` não possui dependências externas e declara usar o `fetch` nativo disponível em Node 20+. Assim, a configuração oficialmente exercitada em CI é Node 22, enquanto o código atual foi escrito para não depender de pacotes npm e usa recursos compatíveis com Node moderno.

### Persistência e publicação

A cada execução, o workflow:

0. roda `node teste-fumaca.mjs`;
1. faz `git fetch origin main`;
2. faz `git reset --hard origin/main` antes de rodar o monitor;
3. executa `node monitor.mjs`;
4. adiciona a pasta `docs` ao commit;
5. cria um commit se houver mudança;
6. tenta enviar o commit para `main`;
7. em caso de conflito por outro push concorrente, repete o ciclo até cinco vezes.

O `reset` antes da execução é importante porque `docs/estado.json` funciona como memória persistente. Dessa forma, cada tentativa lê o estado mais recente já publicado antes de recalcular o relatório.

O checkout usa `fetch-depth: 0`. Push a partir de clone raso funciona no GitHub, mas o ciclo de `fetch` e `reset --hard` dentro do loop de retry fica mais previsível com o histórico completo.

## Executando localmente

Clone o repositório:

```bash
git clone https://github.com/matheussamadello/Monitor-BTC-Price.git
cd Monitor-BTC-Price
```

Confirme sua versão do Node:

```bash
node --version
```

O workflow usa Node.js 22. O código atual não possui `package.json` nem dependências npm e usa `fetch` nativo, portanto não há etapa de `npm install`.

Execute:

```bash
node monitor.mjs
```

Os arquivos em `docs/` serão atualizados localmente. O monitor consulta a Kraken pela internet durante a execução.

## Fazendo um fork

1. Abra o repositório no GitHub.
2. Clique em **Fork**.
3. Crie o fork na sua conta.
4. Abra a aba **Actions** do fork e habilite os workflows se o GitHub os tiver deixado desativados.
5. Confira em **Settings → Actions → General** se o workflow tem permissão para gravar no repositório. O arquivo `monitor.yml` solicita `contents: write`; políticas da conta ou organização ainda podem restringir essa permissão.
6. Execute manualmente o workflow `Monitor BTC` uma vez com **Run workflow** para validar o fork.

Não há secrets obrigatórios no workflow atual.

## Configurando GitHub Pages

O projeto gera o conteúdo estático dentro de `docs/`.

Para publicar um fork no mesmo modelo:

1. abra **Settings → Pages**;
2. em **Build and deployment**, selecione publicação a partir de uma branch;
3. escolha a branch `main`;
4. escolha a pasta `/docs`;
5. salve e aguarde a publicação.

Com isso, `docs/index.html` passa a ser a página principal e `docs/relatorio.json` fica disponível como endpoint estático do GitHub Pages.

Em um fork com outro nome de usuário/repositório, ajuste os URLs usados por bots ou LLMs para o novo endereço do Pages.

## Personalizando níveis e faixas

A configuração manual fica no objeto `NIVEIS_USD` de `monitor.mjs`.

Exemplo da estrutura atual:

```js
const NIVEIS_USD = {
  faixas: [
    [79000, 81000, "faixa_79k_81k"],
    [76000, 78000, "faixa_76k_78k"],
    [74000, 76000, "faixa_74k_76k"],
    [72000, 74000, "regiao_suporte_72k_74k"],
  ],
  resistencia: 80000,
  resistenciaLabel: "80000",
  suporte: 73000,
  suporteLabel: "73000",
};
```

As faixas são automaticamente refletidas em `niveis_manuais.faixas` no `relatorio.json`.

Ao alterar níveis pontuais, mantenha coerentes o valor e o respectivo label, pois o label participa dos nomes de campos da máquina de estados publicada no relatório.

## Usando o JSON com bots, agentes e LLMs

Uma integração externa pode consultar periodicamente:

```text
https://matheussamadello.github.io/Monitor-BTC-Price/relatorio.json
```

Um consumidor robusto deve, no mínimo:

1. guardar o maior `timestamp` já processado;
2. ignorar snapshots iguais ou mais antigos;
3. diferenciar campos fechados de provisórios;
4. ler `niveis_manuais.faixas` dinamicamente;
5. tratar zonas automáticas como contexto/confluência, e não como gatilho isolado;
6. evitar transformar cada campo de `alertas_tecnicos` em uma notificação independente;
7. fundir sinais relacionados para reduzir spam.

O arquivo [`PROMPT_BTC_TECHNICAL_WATCH.md`](./PROMPT_BTC_TECHNICAL_WATCH.md) contém uma política pronta e mais completa para uma LLM/agente transformar o `relatorio.json` em alertas seletivos de swing trade, incluindo compra de BTC com BRL, realização parcial BTC→BRL, hierarquia de sinais, regras anti-spam, interpretação de RSI/DMI/ADX e tratamento dos níveis e zonas automáticas.

## Relação entre o monitor e o prompt de alerta

São duas camadas separadas:

- **`monitor.mjs`** coleta dados, calcula indicadores, estrutura, níveis, estados e zonas e publica o snapshot técnico.
- **`PROMPT_BTC_TECHNICAL_WATCH.md`** define como uma LLM/agente deve interpretar snapshots sucessivos para decidir se existe uma mudança nova e material que merece uma mensagem.

O prompt não é necessário para gerar `relatorio.json`; ele serve como camada externa de interpretação e notificação.
