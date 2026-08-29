# Como transformar esta pasta no repositório Monitor-USD-Price

Esta pasta é o repositório **Monitor-USD-Price** pronto, hospedado
temporariamente dentro de uma branch do Monitor-BTC-Price porque o
GitHub App usado pelo Claude Code não tem permissão para criar
repositórios (`403: Resource not accessible by integration`).

Nada aqui pertence ao Monitor-BTC-Price. **Não faça merge desta branch
na `main`.**

## Passo a passo

1. No GitHub, crie um repositório novo, público, chamado
   `Monitor-USD-Price`, **sem** README, `.gitignore` ou licença.
2. Localmente:

```bash
git clone --branch claude/monitor-usd-price-repo-vv6aao \
  https://github.com/matheussamadello/Monitor-BTC-Price.git tmp-usd
cp -r tmp-usd/monitor-usd-price ./Monitor-USD-Price
rm -rf tmp-usd ./Monitor-USD-Price/COMO-VIRAR-REPOSITORIO.md

cd Monitor-USD-Price
git init -b main
git add .
git commit -m "monitor USD/BRL: primeira versao"
git remote add origin https://github.com/matheussamadello/Monitor-USD-Price.git
git push -u origin main
```

3. Em **Settings → Pages**, publique a partir da branch `main`, pasta `/docs`.
4. Em **Settings → Actions → General**, confirme que o workflow pode gravar
   no repositório (`Read and write permissions`).
5. Na aba **Actions**, rode o workflow `Monitor USD` manualmente uma vez.
6. **Revise `NIVEIS_USD` em `monitor.mjs`.** Os níveis atuais são números
   redondos plausíveis, escolhidos sem acesso à cotação. Depois da primeira
   execução, olhe o `preco_atual` em `docs/index.txt` e ajuste as faixas, a
   resistência e o suporte para a região real do gráfico.

## O que já foi validado

`node teste-fumaca.mjs` roda sem rede e passa em todos os casos: fonte
primária, fallback, as duas fontes fora do ar, JSON, estado entre
execuções e ancoragem de fuso. O que **não** foi validado é a resposta
real do Yahoo Finance e do Stooq — o ambiente onde este código foi
escrito não tem saída para a internet. O passo 5 é o primeiro teste
contra as APIs de verdade.
