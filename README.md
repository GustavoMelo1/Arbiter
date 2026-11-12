# Arbiter

Projeto em desenvolvimento para comparar modelos de previsão de séries temporais de forma reproduzível. A proposta é avaliar os modelos nas mesmas condições e gerar rankings por horizonte, apoiando a decisão de substituir um modelo por outro.

## Estrutura

```text
arbiter/
├── README.md
├── requirements.txt
├── .gitignore
├── src/
│   ├── ingestion/     # Leitura e carga dos dados
│   ├── validation/    # Validação e quarentena
│   ├── features/      # Variáveis para os modelos
│   ├── models/        # Modelos de previsão
│   ├── backtest/      # Avaliação em janelas temporais
│   ├── metrics/       # Métricas de erro
│   ├── reporting/     # Rankings e relatórios
│   └── config.py      # Configurações
├── tests/             # Testes automatizados
├── docs/              # Documentação
└── data/              # Dados locais
```
