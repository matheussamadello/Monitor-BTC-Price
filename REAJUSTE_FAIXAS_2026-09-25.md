# Reajuste seletivo das faixas manuais — BTC

Dados congelados capturados em **2026-09-25T01:36:30.294Z**. Comparação na mesma janela histórica da auditoria anterior. Revisão de geometria, sem otimizar score ou criar faixas adicionais.

## Critério

O teto anterior de 0,25 ATR diário e 1% podia excluir pivôs e reações próximos. A largura agora segue a evidência de cada região, com teto de segurança de **0,5 ATR diário na calibração**. Esse teto não é uma largura alvo nem um mecanismo de ajuste automático. Todas as faixas continuam menores que as regiões amplas originais. As que já representavam bem o núcleo foram mantidas.

Contato exige interseção OHLC com a faixa exata, sem margem ATR. Episódios, reações de pelo menos 1 ATR, volume, role reversal, pesos e penalidades seguem a auditoria anterior. O score abaixo é diagnóstico, não um novo campo canônico do monitor. Alargar uma borda pode reagrupar episódios e mudar o momento da saída e a classificação de rejeição, portanto mais toques não significam necessariamente mais evidência independente. Não é backtest prospectivo.

## Faixas alteradas — diário

| Par | Antes | Agora | Largura em ATR diário | Score antes → agora | Toques antes → agora | Rejeições antes → agora | Score atual sem bônus semanal |
|---|---|---|---:|---:|---:|---:|---:|
| BTC/USD | 64.900–65.500 | 64.600–65.700 | 0.438 | 73 → 73 | 10 → 15 | 8 → 13 | 61 |

- **BTC/USD 64.600–65.700**: O nucleo estreito excluia os pivos confirmados de 64675 e 65646,2 de uma concentracao continua com outros quatro pivos. Limites ao redor do conjunto, sem unir regioes distantes.

## Retestes após confirmação do núcleo anterior

Mantém o mesmo início da medição anterior (cinco velas após o pivô selecionado naquela revisão). Não reinicia a janela num pivô mais antigo acrescentado agora. Isso impede que a comparação melhore apenas por incluir mais história. Não representa validação prospectiva dos limites escolhidos hoje.

| Par | Faixa nova | Score antes → agora | Toques antes → agora | Rejeições antes → agora | Episódios abertos agora |
|---|---|---:|---:|---:|---:|
| BTC/USD | 64.600–65.700 | 73 → 73 | 9 → 13 | 7 → 11 | 0 |

As outras cinco faixas foram mantidas. A faixa revisada continua sem toque recente desde agosto. Recuperar pivôs históricos não elimina a necessidade de observar novas reações.

## Compatibilidade e alertas

Só foram alterados limites/labels de faixas selecionadas e sua documentação. Pontos de suporte/resistência, macro, EMA89, RSI, DMI/ADX, pivôs, estrutura, divergências, motor automático, pesos e regras de alertas permanecem iguais. Os campos canônicos e o histórico não foram renomeados ou reescritos.

As faixas ampliadas podem reconhecer presença em preços antes excluídos, aumentando o tempo dentro da região. A troca de label pode mudar a assinatura uma vez. Não se promete a mesma frequência futura de alertas, nem se interpreta essa mudança de configuração como novo movimento de mercado.

Na comparação antes/depois com as mesmas respostas reais congeladas, os gatilhos ativos permaneceram idênticos. Também foram verificadas a igualdade dos indicadores, estrutura, ciclos de níveis pontuais e geometria/score/toques/rejeições das zonas automáticas. A lista de campos alterados está no registro de evidência.

## Reprodução

```sh
node teste-fumaca.mjs
node teste-auditoria-faixas.mjs
node teste-reajuste-faixas.mjs
```

Os [dados congelados](auditoria-faixas-dados-2026-09-25.json) são os mesmos da auditoria anterior. O [registro completo](reajuste-faixas-manuais-2026-09-25.json) contém os pivôs e os resultados diário/semanal das faixas alteradas e mantidas. O teste recalcula as medidas e verifica os limites, janelas e evidências, sem rede.
