# Arbiter

**Validador independente de modelos de previsão de séries temporais.**

O Arbiter está sendo desenvolvido para responder a uma pergunta:

> Um modelo novo é melhor do que o que já usamos?

A proposta é comparar modelos com o mesmo histórico, as mesmas datas de corte
e os mesmos horizontes de previsão. O resultado será uma avaliação reproduzível,
com previsões registradas, ranking por horizonte e evidências para apoiar a
decisão de substituir um modelo.

Cada conclusão valerá para os dados e o protocolo daquela execução. Um modelo
pode ter o menor erro no próximo mês e perder para outro ao prever um ano à frente.

## Estado do projeto

**Em preparação para a Fase 0.** A estrutura de diretórios e o repositório Git
estão configurados. Os modelos, o backtest, os testes e o comando de execução
ainda serão implementados. As funcionalidades descritas abaixo representam o
escopo planejado.

## Como vai funcionar

1. **Receber uma série:** ler dados no formato padrão, começando por uma série
   sintética mensal e depois avaliando uma série pública.
2. **Preparar os dados:** verificar a qualidade da entrada e registrar os
   problemas encontrados. A validação com quarentena será ampliada na Fase 1.
3. **Simular previsões passadas:** em cada data de corte, treinar os modelos
   apenas com o histórico disponível e prever os períodos seguintes.
4. **Guardar as previsões:** salvar valor observado e previsto para cada modelo,
   série, origem e horizonte, junto da identificação da execução.
5. **Comparar os resultados:** calcular métricas por horizonte e a diferença
   em relação ao baseline, mostrando também a variação entre origens.
6. **Apoiar a decisão:** na Fase 1, gerar um relatório e um veredito de
   **aprovar, reprovar ou inconclusivo**, conforme regras explícitas.

O objetivo final é executar esse processo com um único comando, sem edição
manual do código para cada nova série.

## Protocolo de avaliação

O backtest seguirá o método **walk-forward**: a data de corte avança ao longo
da série, simulando o que seria possível prever em cada momento.

- **Treino expansivo:** o histórico de treino cresce a cada origem.
- **Pelo menos 12 origens:** os modelos serão avaliados em várias datas de corte.
- **Condições iguais:** todos os modelos usarão as mesmas origens e o mesmo
  histórico disponível, respeitando a disponibilidade de variáveis externas.
- **Horizontes iniciais:** 1, 3, 6 e 12 passos; na série mensal, cada passo é um mês.
- **Sem vazamento de dados:** features e transformações só poderão usar
  informações disponíveis na data da previsão.

O horizonte `h = 3` corresponde à previsão do terceiro mês após a origem.
A avaliação desse horizonte compara esse passo entre as origens, sem acumular
os erros dos três primeiros meses.

### Modelos previstos

| Modelo | Papel na comparação | Etapa |
| --- | --- | --- |
| Ingênuo sazonal | Baseline que repete o valor observado no mesmo período do ciclo anterior | Fase 0 |
| ETS | Modelo estatístico de suavização exponencial | Fase 0 |
| Gradient boosting | Modelo de aprendizado de máquina com valores passados e outras features temporais | Fase 0 |
| Média móvel | Segundo baseline simples | Expansão do núcleo |
| SARIMA | Modelo estatístico com componentes sazonais e busca limitada de parâmetros | Fase 1 |

### Métricas e evidências

A métrica principal será o **WAPE**, calculado para cada série, modelo e horizonte:

```text
WAPE = soma dos erros absolutos / soma dos valores observados absolutos
```

As somas serão feitas sobre as origens avaliadas. O ranking mostrará também a
diferença contra o baseline. MAPE e MASE estão previstas como métricas
complementares, com tratamento explícito dos casos em que não forem definidas.

Como as previsões serão armazenadas individualmente, será possível recalcular
métricas sem treinar todos os modelos novamente. A evolução do projeto incluirá
custo do erro, com pesos diferentes para prever acima ou abaixo do observado,
e resultados por segmento.

## Dados e rastreabilidade

O formato padrão terá três colunas obrigatórias:

| Coluna | Conteúdo |
| --- | --- |
| `serie_id` | Identificador da série |
| `data` | Data da observação |
| `valor` | Valor observado |

Exemplo sintético de entrada:

```csv
serie_id,data,valor
serie_exemplo,2020-01-01,100.0
serie_exemplo,2020-02-01,108.0
serie_exemplo,2020-03-01,104.0
```

O exemplo ilustra apenas o formato; o backtest exigirá histórico suficiente
para o treino, as 12 origens e o maior horizonte escolhido. Colunas de variáveis
externas poderão ser acrescentadas conforme a camada de features evoluir.

Serão usados somente dados públicos ou sintéticos. A fonte pública ainda será
escolhida, após verificar disponibilidade, licença e qualidade. Dados de trabalho
ou informações sensíveis ficam fora do repositório e da infraestrutura pessoal.

Cada experimento deverá registrar hashes dos dados e da configuração, a versão
do código e as informações necessárias para reproduzir seus resultados.

## Arquitetura planejada

```text
Entrada de dados
    |
    v
Ingestão -> Validação -> Features por data de corte -> Backtest
                |                                       |
                v                                       v
            Quarentena                         Previsões e execuções
                                                        |
                                                        v
                                               Ranking e relatório
```

A Fase 0 usará **SQLite local** para armazenar execuções e previsões. As camadas
de ingestão, validação e quarentena serão aprofundadas na Fase 1. A etapa de
cloud levará as camadas de dados ao **BigQuery**, com execução agendada.

## Estrutura do repositório

A estrutura atual está organizada por responsabilidade:

```text
arbiter/
├── README.md              # Apresentação e escopo do projeto
├── requirements.txt       # Dependências Python; ainda a definir
├── .gitignore             # Exclusões de dados locais e arquivos gerados
├── src/
│   ├── ingestion/         # Leitura e carga idempotente dos dados
│   ├── validation/        # Regras de qualidade e quarentena
│   ├── features/          # Lags, médias móveis e calendário
│   ├── models/            # Modelos com uma interface comum
│   ├── backtest/          # Origens, janelas e execução das previsões
│   ├── metrics/           # Métricas de avaliação
│   ├── reporting/         # Ranking, veredito e relatório
│   └── config.py          # Configuração dos experimentos
├── tests/                 # Testes por módulo e de integração
├── docs/                  # Documentação e decisões de projeto
└── data/                  # Dados locais; ignorados por padrão
```

Os diretórios ainda vazios contêm `.gitkeep` para preservar a estrutura no Git.
As descrições indicam a responsabilidade prevista de cada módulo. A exceção
planejada para versionar dados é `data/synthetic/`, destinada a amostras sintéticas.

## Etapas de desenvolvimento

| Fase | Entrega | Critério de conclusão |
| --- | --- | --- |
| **0 — Avaliação local** | Série sintética, três modelos, 12 origens, WAPE por horizonte e SQLite | Um comando executa a avaliação e o teste de vazamento passa |
| **1 — Núcleo** | Validação com quarentena, SARIMA, relatório, veredito e CI | Uma nova série no formato padrão é avaliada sem alterar o código |
| **2 — Cloud** | BigQuery, Cloud Run, agendamento, intervalos de previsão e cobertura | Uma execução agendada grava resultados e publica o relatório |
| **3 — Extensões** | Monitoramento de drift, dbt, testes estatísticos e mais séries | Evolução após a publicação das fases anteriores |

As bibliotecas candidatas são pandas, statsmodels ou statsforecast, scikit-learn
e pytest. A escolha final e as versões serão registradas durante a implementação.

## Verificações previstas

- **Cálculo das métricas:** comparar um exemplo pequeno com o resultado manual.
- **Vazamento:** alterar valores posteriores a uma origem não pode mudar as
  previsões produzidas naquela origem.
- **Idempotência:** ingerir a mesma fonte duas vezes não pode duplicar registros.
- **Validação:** cada regra deve ter exemplos aceitos e rejeitados.
- **Reprodutibilidade:** repetir dados, configuração, código e ambiente deve
  reproduzir os resultados, com sementes controladas quando aplicável.
- **Integração contínua:** executar a suíte de testes no GitHub Actions.

## Fora do escopo

Modelos inéditos, deep learning, tuning pesado, interface web, autenticação,
multiusuário e disponibilização de previsões em produção.

## Próximo passo

Implementar a função WAPE e seu primeiro teste com resultado calculado à mão;
depois, conectar a série sintética ao baseline e ao backtest. As instruções de
instalação e execução serão adicionadas quando houver uma primeira versão
executável.
