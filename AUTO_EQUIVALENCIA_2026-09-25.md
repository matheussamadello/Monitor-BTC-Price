# Zonas automáticas mais próximas das manuais — BTC

Comparação com dados capturados em **2026-09-25T01:36:30.294Z**, mantendo a mesma entrada e o mesmo estado anterior. Base: `f506d92bedefbec75fc888e332f79c8524fad367`.

## Mudança cirúrgica

- Teto estrutural total: diário **0,8 → 0,5 ATR diário**; semanal **1,2 → 0,3 ATR semanal**.
- Folga em cada borda: **0,15 → 0,05 ATR de referência**. A referência continua sendo o menor entre o ATR médio dos pivôs e o ATR fechado atual.
- A redução da folga evita gastar 0,3 ATR só em margem, preservando mais espaço para os pivôs dentro do teto menor. Um pivô isolado passa a gerar cerca de 0,1 ATR estrutural, sem se tornar uma linha.
- Clustering, split com evidência, fusão condicionada ao teto, filtros de qualidade, pesos, penalidades e publicação de até três zonas por lado permanecem iguais. Não se corta uma zona deixando seus próprios membros fora.

A tolerância operacional permanece em **centro ±0,25 ATR**, respeitando o teto percentual do par. Toques e rejeições automáticos continuam medidos nessa janela histórica de ATR, não exclusivamente dentro da faixa estrutural desenhada. Isso é diferente da auditoria estrita das faixas manuais. O score não foi artificialmente conservado: depois de um reagrupamento, cada zona recalcula suas interações. A perda de sobreposição semanal pode reduzir o score mesmo com contagens idênticas.

## Comparação diária/semanal

IDs identificam continuidade da região, não igualdade dos membros. Uma linha pode ter menos pivôs e janela histórica diferente depois do reagrupamento. Zonas novas não herdam contagens da antiga.

### BTC/USD — diario

ATR: **2.514,23404866**. Teto em preço: **1.257,11702433**. Publicadas: **6 → 6**. Contagens de toques/rejeições iguais em **6 de 6** IDs comparáveis.

| ID | Faixa anterior | Faixa atual | Score antes → agora | Toques antes → agora | Rejeições antes → agora |
|---|---|---|---:|---:|---:|
| usd|diario|z26 | 88.372,8648927–89.602,1351073 | 88.624,28829757–89.350,71170243 | 72 → 61 | 8 → 8 | 5 → 5 |
| usd|diario|z40 | 90.358,8648927–92.177,1351073 | 90.610,28829757–91.382,31170243 | 58 → 61 | 15 → 15 | 10 → 10 |
| usd|diario|z50 | 93.011,5648927–94.488,1351073 | 93.262,98829757–94.236,71170243 | 61 → 61 | 18 → 18 | 14 → 14 |
| usd|diario|z25 | 83.493,6648927–84.798,5351073 | 83.745,08829757–84.547,11170243 | 85 → 85 | 4 → 4 | 2 → 2 |
| usd|diario|z39 | 81.937,40129029–83.141,59870971 | 82.171,2004301–82.907,7995699 | 90 → 90 | 5 → 5 | 3 → 3 |
| usd|diario|z49 | 81.092,91462477–81.829,08537523 | 81.338,30487492–81.583,69512508 | 78 → 65 | 3 → 3 | 2 → 2 |

O conjunto completo não dormente, incluindo zonas candidatas/enfraquecidas fora da seleção pública, passou de 26 para 28. Registros antigos dormentes continuam no estado durante a carência.

### BTC/USD — semanal

ATR: **6.474,8748793**. Teto em preço: **1.942,46246379**. Publicadas: **6 → 6**. Contagens de toques/rejeições iguais em **4 de 5** IDs comparáveis.

| ID | Faixa anterior | Faixa atual | Score antes → agora | Toques antes → agora | Rejeições antes → agora |
|---|---|---|---:|---:|---:|
| usd|semanal|z27 | 83.450,16876811–89.977,03123189 | 84.097,65625604–84.745,14374396 | 67 → 62 | 3 → 3 | 1 → 1 |
| usd|semanal|z28 | 93.612,46876811–98.870,93123189 | 97.575,95625604–98.223,44374396 | 26 → 26 | 1 → 1 | 1 → 1 |
| usd|semanal|z52 | nova na seleção | 106.931,25625604–107.578,74374396 | — → 55 | — → 3 | — → 2 |
| usd|semanal|z37 | 81.819,66876811–83.762,13123189 | 82.467,15625604–83.114,64374396 | 53 → 53 | 2 → 2 | 1 → 1 |
| usd|semanal|z41 | 75.644,46876811–81.566,43123189 | 80.271,45625604–80.918,94374396 | 88 → 88 | 4 → 4 | 2 → 2 |
| usd|semanal|z26 | 70.925,16876811–76.969,33123189 | 71.572,65625604–73.055,24374396 | 70 → 76 | 5 → 6 | 1 → 3 |

O conjunto completo não dormente, incluindo zonas candidatas/enfraquecidas fora da seleção pública, passou de 42 para 52. Registros antigos dormentes continuam no estado durante a carência.

## Limitações e compatibilidade

O semanal pode continuar mais largo em preço porque seu ATR é maior. Os números escolhidos aproximam as escalas, sem transformar zonas em linhas ou prometer identidade com os manuais. Uma zona de pivô único e score baixo não se torna forte por ficar estreita. Não foi aumentado nenhum score para compensar perda de evidência.

Os cálculos de RSI, DMI/ADX, EMA89, estrutura, divergências, níveis manuais e suas máquinas de estado foram comparados antes/depois e permaneceram iguais. Gatilhos ativos também ficaram idênticos nessa entrada. Nenhuma regra de alerta, prompt, campo canônico ou agendamento foi alterado. Os consumidores externos podem observar outra seleção de zonas, menos confluência ou avisos de manutenção, pois esses dados mudam legitimamente com a geometria. Isso não permite garantir frequência idêntica de notificações externas.

## Verificação reproduzível

```sh
node teste-fumaca.mjs
node teste-auditoria-faixas.mjs
```

O teste de fumaça inclui os novos testes de fronteira, folga e preservação das interações, além de `teste-auto-equivalencia.mjs`, que reproduz o pipeline com [respostas reais congeladas](fixture-auto-equivalencia-2026-09-25.json), valida o [resultado registrado](comparacao-auto-equivalencia-2026-09-25.json) e executa três retries para verificar estabilidade de zonas, IDs e score. Os testes anteriores de split, fusão, role reversal e qualidade continuam ativos; o de "ausência de microzonas" foi substituído pelo de reagrupamento dos restos (adendo abaixo).

## Adendo — restos reagrupados (2026-09-25, depois desta comparação)

O split original, quando não achava vão limpo entre duas concentrações, conservava o núcleo compacto e **descartava** o resto. Esses pivôs saíam de toda zona. Nas séries reais desta captura, isso tirava 2 a 42 pivôs por série do sistema de zonas, inclusive pivôs perto do preço e o 79.490,7 que ancora a faixa manual 79.300–79.700 do BTC.

Agora o resto de cada lado do núcleo é reagrupado pelo mesmo processo. Todo pivô confirmado termina em exatamente uma zona, e toda zona continua dentro do teto de largura. Com a mesma entrada congelada:

| Par | TF | Zonas calculadas | Seleção publicada |
|---|---|---:|---|
| BTC/USD | diario | 28 → 47 | 2 entraram, 2 saíram |
| BTC/USD | semanal | 52 → 75 | 3 entraram, 3 saíram |

Gatilhos ativos inalterados. As zonas que entraram na seleção pública têm score mediano igual ou maior que o das que saíram. Em [comparacao-auto-equivalencia-2026-09-25.json](comparacao-auto-equivalencia-2026-09-25.json), `depois` e `calculadas_depois` passaram a refletir o reagrupamento; os valores desta comparação original continuam gravados em `depois_sem_reagrupamento` e `calculadas_depois_sem_reagrupamento`. `antes` não mudou.
