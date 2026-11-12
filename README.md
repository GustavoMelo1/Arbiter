# Arbiter

Validador independente de modelos de previsão de séries temporais.

O objetivo é comparar modelos nas mesmas janelas de avaliação e produzir
evidências reproduzíveis para decidir se um modelo pode substituir o atual.

## Status

Projeto em preparação. A estrutura inicial está criada; o backtest e os modelos
ainda serão implementados.

## Primeira entrega — Fase 0

- Uma série sintética mensal.
- Baseline ingênuo sazonal, ETS e gradient boosting com lags.
- Backtest com treino expansivo e 12 origens.
- WAPE nos horizontes de 1, 3, 6 e 12 meses.
- Previsões individuais e identificação do experimento em SQLite local.
- Um comando para executar a avaliação, com testes de vazamento de dados.

## Estrutura inicial

```text
src/
  ingestion/
  validation/
  features/
  models/
  backtest/
  metrics/
  reporting/
  config.py
tests/
docs/
data/
```

Os arquivos `.gitkeep` preservam no Git as pastas que ainda estão vazias.

## Dados

O projeto utilizará somente dados públicos ou sintéticos. Dados de trabalho não
devem entrar neste repositório. A pasta `data/` é ignorada, exceto pelo marcador
de diretório e por futuras amostras sintéticas em `data/synthetic/`.

## Próximo passo

Implementar a função WAPE e seu primeiro teste com resultado calculado à mão.
As dependências e as instruções de execução serão registradas conforme a
implementação avançar.
