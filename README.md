# Monitor BTC Price

Monitor técnico automatizado de **BTC/USD** que coleta candles da Kraken, calcula indicadores, acompanha estrutura de mercado e publica um relatório estático em HTML, texto e JSON para consulta humana ou consumo por bots, agentes e LLMs.

O projeto foi desenhado para acompanhamento de **swing trades e operações de prazo mais longo**, com o gráfico diário como referência principal de timing e o semanal como filtro de contexto estrutural.

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

### RSI(14)

O monitor calcula **RSI de 14 períodos** usando suavização de Wilder/RMA.

O relatório separa:

- `rsi14_fechado`: calculado apenas com velas fechadas;
- `rsi14_provisorio`: inclui a vela atualmente em formação.

O código também detecta divergências RSI confirmadas e provisórias a partir de pivôs de preço.

### DMI/ADX(14)

São calculados:

- `di_plus14_fechado` / `di_plus14_provisorio`;
- `di_minus14_fechado` / `di_minus14_provisorio`;
- `adx14_fechado` / `adx14_provisorio`.

O cálculo usa a suavização de Wilder/RMA. O ADX mede força direcional e deve ser interpretado junto de DI+ e DI−; o monitor não trata ADX isoladamente como direção de mercado.

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

- **tático**, de 2 a 6 semanas, em que o diário pesa mais;
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

Essas classificações são usadas para reconhecer estrutura de alta, estrutura de baixa e combinações transicionais/indefinidas.

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
| US$ 78.000–80.000 | `faixa_78k_80k` |
| US$ 70.000–72.000 | `faixa_70k_72k` |
| US$ 64.000–66.000 | `regiao_suporte_64k_66k` |

Além das faixas, o código mantém atualmente uma resistência pontual em **US$ 80.000** e um suporte pontual em **US$ 65.000** para a máquina de estados de rompimento/reteste.

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

Os cortes são **1 ATR** e **3 ATR**: dentro de uma faixa ou a menos de 1 ATR dela é `atual`; entre 1 e 3 é `monitorar`; além de 3 é `obsoleto`.

As duas pontas da conta usam **vela fechada** — o fechamento e o ATR. A primeira versão passava o preço da vela em formação, o que misturava provisório com confirmado num híbrido sem significado limpo, e contrariava a convenção do próprio monitor, em que o que alimenta decisão usa vela fechada. E este campo alimenta uma: a revisão dos níveis manuais. O custo é uma vela de latência, irrelevante para um sinal cujo caso de origem levou 19 dias para ser notado.

A distância é medida em ATR, e não em porcentagem, de propósito. Cinco por cento é muito num par de câmbio e pouco num de cripto, enquanto "três vezes a volatilidade diária" quer dizer a mesma coisa em qualquer um — um limiar só serve para os três monitores, sem recalibragem.

`obsoleto` não é alerta de mercado: é aviso de manutenção. Significa que os níveis descrevem um regime que ficou para trás e precisam de revisão.

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
│   ├── index.html
│   ├── index.txt
│   └── relatorio.json
├── monitor.mjs
├── teste-fumaca.mjs
├── README.md
└── PROMPT_BTC_TECHNICAL_WATCH.md
```

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
    [78000, 80000, "faixa_78k_80k"],
    [70000, 72000, "faixa_70k_72k"],
    [64000, 66000, "regiao_suporte_64k_66k"],
  ],
  resistencia: 80000,
  resistenciaLabel: "80000",
  suporte: 65000,
  suporteLabel: "65000",
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
