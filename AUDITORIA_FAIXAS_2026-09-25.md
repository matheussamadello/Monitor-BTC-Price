# Auditoria de qualidade das faixas manuais — BTC

> Registro histórico anterior ao [reajuste seletivo das faixas](REAJUSTE_FAIXAS_2026-09-25.md). As tabelas abaixo preservam os limites e resultados daquela auditoria.

Séries capturadas em **2026-09-25T01:36:30.294Z**. As medidas abaixo são uma auditoria retrospectiva específica dos limites manuais, não scores canônicos publicados pelo monitor nem probabilidades de sucesso.

## Método

- Contato exige interseção da máxima/mínima com a faixa exata. Nenhuma margem ATR aumenta suas bordas.
- Várias velas do mesmo episódio contam como um toque. Novo episódio exige saída confirmada de pelo menos 1 ATR histórico.
- Rejeição exige retorno ao lado de origem, com reação de pelo menos 1 ATR. A excursão é medida em até três velas, como no motor atual. Episódios abertos não são tratados como rejeições.
- Pesos, penalidades, volume histórico e role reversal reutilizam as funções do monitor. Volume é não aplicável no USD/BRL.
- Confirmação semanal conserva o critério do motor: sobreposição de pelo menos 35% com zona semanal ativa/candidata não absorvida. O score sem esse bônus também é mostrado para distinguir evidência própria de corroboração contextual.
- Antes/depois usam a mesma janela desde o primeiro pivô da faixa anterior. Isso inclui contatos anteriores à seleção do núcleo. Uma segunda medição começa depois da confirmação do primeiro pivô do núcleo atual (cinco velas à direita), excluindo a formação inicial.
- O contador pode subir ao estreitar: a borda muda a distância de saída de 1 ATR e, portanto, pode separar episódios antes agrupados. Isso não representa mais velas de contato.
- No semanal, entram apenas velas cujo início está dentro da janela; uma semana iniciada antes do corte não é contada parcialmente. Isso pode excluir a vela de formação inicial.
- A auditoria usa velas fechadas OHLC. Não é backtest, não demonstra rentabilidade e não resolve a ordem intradiária dos movimentos.

## Diário — mesma janela histórica para antes/depois

| Par | Faixa atual | Início da janela | Score antes → depois | Depois sem bônus semanal | Toques antes → depois | Rejeições antes → depois | Último toque |
|---|---|---|---|---|---|---|---|
| BTC/USD | 88.700–89.300 | 2025-01-13 | 72 → 72 | 61 | 10 → 10 | 7 → 7 | 2026-01-29 |
| BTC/USD | 79.300–79.700 | 2025-11-21 | 84 → 83 | 72 | 8 → 9 | 6 → 5 | 2026-09-18 |
| BTC/USD | 76.150–76.700 | 2025-03-11 | 77 → 77 | 66 | 7 → 8 | 3 → 4 | 2026-09-18 |
| BTC/USD | 74.700–75.100 | 2025-04-07 | 82 → 82 | 71 | 10 → 9 | 6 → 5 | 2026-09-16 |
| BTC/USD | 73.400–73.800 | 2024-10-29 | 76 → 76 | 65 | 9 → 10 | 4 → 5 | 2026-08-21 |
| BTC/USD | 64.900–65.500 | 2024-10-23 | 76 → 73 | 61 | 10 → 10 | 8 → 8 | 2026-08-19 |

## Diário — testes depois da confirmação do núcleo atual

Esta janela é menor quando o pivô que ancora o núcleo é recente. Ela verifica retestes posteriores sem reaproveitar a reação da própria formação.

| Par | Faixa | Desde | Score | Sem bônus semanal | Toques / rejeições | Episódios abertos |
|---|---|---|---|---|---|---|
| BTC/USD | 88.700–89.300 | 2025-01-18 | 72 | 61 | 9 / 6 | 0 |
| BTC/USD | 79.300–79.700 | 2026-04-27 | 76 | 65 | 8 / 5 | 0 |
| BTC/USD | 76.150–76.700 | 2025-03-16 | 77 | 66 | 7 / 3 | 0 |
| BTC/USD | 74.700–75.100 | 2026-05-04 | 70 | 59 | 4 / 2 | 0 |
| BTC/USD | 73.400–73.800 | 2024-11-03 | 76 | 65 | 9 / 4 | 0 |
| BTC/USD | 64.900–65.500 | 2024-10-28 | 73 | 61 | 9 / 7 | 0 |

## Reação, volume e papel — núcleo diário

| Par | Faixa | Reação média em ATR | Volume relativo mediano nos contatos | Role reversals confirmados | Toques / rejeições nas últimas 90 velas | Velas desde último contato |
|---|---|---|---|---|---|---|
| BTC/USD | 88.700–89.300 | 2.63 | 1.68 | 2 | 0 / 0 | 238 |
| BTC/USD | 79.300–79.700 | 2.22 | 1.61 | 1 | 5 / 4 | 6 |
| BTC/USD | 76.150–76.700 | 2.68 | 1.53 | 0 | 3 / 2 | 6 |
| BTC/USD | 74.700–75.100 | 2.69 | 1.27 | 2 | 2 / 1 | 8 |
| BTC/USD | 73.400–73.800 | 2.58 | 1.85 | 1 | 1 / 0 | 34 |
| BTC/USD | 64.900–65.500 | 2.67 | 1.07 | 1 | 3 / 2 | 36 |

## Semanal — mesmos limites manuais

| Par | Faixa | Score antes → depois | Toques antes → depois | Rejeições antes → depois | Último toque |
|---|---|---|---|---|---|
| BTC/USD | 88.700–89.300 | 45 → 45 | 2 → 2 | 1 → 1 | 2026-01-29 |
| BTC/USD | 79.300–79.700 | 73 → 73 | 3 → 3 | 1 → 1 | 2026-09-17 |
| BTC/USD | 76.150–76.700 | 87 → 88 | 4 → 4 | 2 → 2 | 2026-09-17 |
| BTC/USD | 74.700–75.100 | 45 → 44 | 2 → 2 | 0 → 0 | 2026-09-10 |
| BTC/USD | 73.400–73.800 | 46 → 46 | 3 → 3 | 0 → 0 | 2026-08-20 |
| BTC/USD | 64.900–65.500 | 73 → 62 | 3 → 2 | 3 → 2 | 2026-08-13 |

## Interpretação

Todas as faixas têm pelo menos oito episódios e quatro rejeições na janela histórica comum. Isso não elimina o envelhecimento: **88.700–89.300** não teve toque desde 2026-01-29. As faixas **73.400–73.800** e **64.900–65.500** também ultrapassam 30 velas sem contato na captura. Devem ser lidas como referências históricas, sem inferir reação atual.

O corte de ativação do detector é 45, mas ultrapassá-lo nesta auditoria não ativa uma faixa manual nem garante sua maturidade. A máquina automática também considera evidência, duas velas e envelhecimento. Nenhuma faixa ou regra operacional foi alterada nesta validação.

## Reprodução e testes

```sh
node auditar-faixas-manuais.mjs
node teste-auditoria-faixas.mjs
```

Os [dados congelados](auditoria-faixas-dados-2026-09-25.json) incluem as séries e fontes utilizadas. Os [resultados detalhados](auditoria-faixas-qualidade-2026-09-25.json) registram cada episódio, rejeição, penalidade e role reversal. Os testes verificam equivalência com o motor quando a geometria coincide, exclusão de contatos fora do núcleo, independência de episódios, rejeição versus travessia, episódio aberto, volume, role reversal e ausência de mutação das entradas.
