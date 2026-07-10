# Desafio Técnico - Cientista de Dados Junior
## Time de IA - Casa Civil / IplanRio

## Pré-Requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado
  - [Download para Windows](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)
  - [Download para Linux](https://docs.docker.com/desktop/install/linux-install/)

- **Windows**: é necessário ter o [WSL 2](https://learn.microsoft.com/pt-br/windows/wsl/install) habilitado (backend padrão do Docker Desktop).


## Como Reproduzir a Análise

### 1. Build e subida do container

```bash
docker compose up --build
```

### 2. Acessar o Jupyter

Abra `http://localhost:8888` no navegador.

O token de acesso é exibido no terminal na primeira execução (procure por `?token=` na saída do container). Copie o token e cole na página do Jupyter.

### 3. Executar os notebooks

Os notebooks estão na pasta `notebooks/`. Execute na seguinte ordem:

| Ordem | Notebook | Conteúdo |
|-------|----------|----------|
| 1 | `01_analise_exploratoria.ipynb` | Análise exploratória dos chamados |
| 2 | `02_auditoria_modelo_a.ipynb` | Auditoria do modelo em produção |
| 3 | `03_comparacao_e_recomendacao.ipynb` | Comparação modelo A vs B e recomendação |

### 4. Parar o container

Pressione `Ctrl+C` no terminal onde o container está rodando para interrompê-lo. Depois, para limpar os recursos (remover container e rede):

```bash
docker compose down
```

---

## Contexto

A **Central 1746** recebe milhares de chamados de cidadãos todos os dias. Para agilizar o encaminhamento, o time de IA da Casa Civil desenvolveu um classificador automático que lê o texto do chamado e prevê a categoria do serviço (o **modelo A**, hoje em produção). Recentemente, uma nova versão foi desenvolvida (o **modelo B**) e precisamos decidir se vale a pena substituir o modelo atual.

Como Cientista de Dados no time de IA, grande parte do seu trabalho será **testar e avaliar sistemas de IA em contextos reais de aplicação** — e é exatamente isso que este desafio simula: auditar o modelo em produção e recomendar, com base em evidências, se devemos ou não trocá-lo pelo modelo B.

Este desafio avalia suas habilidades em análise exploratória, estatística aplicada à avaliação de modelos e geração de recomendações acionáveis para gestão pública.

> Os dados deste desafio são totalmente sintéticos: os chamados, rótulos e predições foram gerados artificialmente para simular o comportamento estatístico de um sistema real de classificação — nenhum dado de cidadão foi utilizado.

---

## Abordagem

A análise foi dividida em três etapas, acompanhando os notebooks:

1. **Análise exploratória (Parte 1)**: Comecei investigando a distribuição dos chamados, a qualidade dos textos, padrões por canal/bairro/tempo, e a relação entre termos textuais e categorias. O objetivo foi identificar quais variáveis realmente importam para a classificação — a conclusão foi que há termos relevantes contidos nos textos que podem ser o principal preditor.

2. **Auditoria do Modelo A (Parte 2)**: Calculei acurácia, F1-score com intervalos de confiança via bootstrap. Analisei a matriz de confusão e subgrupos de falha. Identifiquei que a categoria `esgoto_vazamento` concentra os piores erros.

3. **Comparação A vs B (Parte 3)**: Usei teste de McNemar, teste pareado porque as predições são sobre os mesmos chamados. Comparei as métricas globais e por categoria, discutindo trade-offs. A recomendação final é substituir o Modelo A pelo B, com ressalvas e monitoramento para a categoria `poda_arvore`.

---

## Sumário
Foram analisados 5.000 chamados sintéticos da Central 1746 com o objetivo de auditar o Modelo A que está atualmente em produção, e verificar se o Modelo B possui desempenho suficiente para substituí-lo. Na análise exploratória inicial, observou-se que o texto dos chamados parece ser a principal fonte de informação para a diferenciação das categorias, uma vez que não foram identificados padrões relevantes associados ao tempo, bairro ou canal de atendimento. Em contrapartida, verificou-se a presença de termos específicos fortemente associados a determinadas categorias, sugerindo que o conteúdo textual é altamente discriminativo para a tarefa de classificação. Essa mesma análise também indicou potenciais desafios na distinção da categoria `poda_arvore`, além de possíveis confusões entre as categorias `buraco_via` e `esgoto_vazamento`, em função da sobreposição de termos relevantes nas descrições dos chamados.


A auditoria mostrou que o Modelo A tem desempenho geral razoável, com acurácia próxima de 77%. Os erros em geral são homogêneos nas categorias, mas possui a principal fragilidade na categoria Vazamento de Esgoto, com taxa de erro de aproximadamente 42%, o que pode gerar impacto operacional relevante por envolver chamados com possível urgência sanitária e necessidade de encaminhamento especializado.

Recomenda-se substituir o Modelo A pelo Modelo B, pois o novo modelo reduz de forma relevante a quantidade de chamados encaminhados incorretamente. Na base avaliada, o Modelo B aumentou a taxa de acerto de cerca de 77% para 87% e corrigiu mais erros do modelo atual do que criou novos erros. A principal vantagem aparece em categorias importantes como Vazamento de Esgoto, que tinha desempenho fraco no modelo atual. O principal risco da troca está em Poda de Árvore, categoria na qual o Modelo B piora a identificação dos chamados. Por isso, a troca deve ser feita com monitoramento específico dessa categoria, revisão humana temporária para casos suspeitos e acompanhamento das métricas nas primeiras semanas de uso.

---

## Instruções

1. Crie um **fork público desse repositório** com suas respostas
2. Use **Jupyter Notebooks** (.ipynb) bem documentados — os notebooks devem rodar do início ao fim sem erros
3. Inclua **README.md** explicando abordagem, como reproduzir e um **sumário executivo de no máximo 1 página** com seus principais achados e a recomendação final
4. Faça commits ao longo do trabalho — o histórico deve refletir a evolução da análise (não faça um único commit no final)

---

## Dados

Arquivo: **`dados/chamados_com_predicoes.csv`** (5.000 chamados rotulados)

| Coluna | Descrição |
|---|---|
| `id_chamado` | Identificador único |
| `data_abertura` | Data de abertura do chamado |
| `bairro` | Bairro informado |
| `canal` | Canal de entrada (app, telefone ou portal) |
| `texto` | Texto do chamado escrito pelo cidadão |
| `categoria_real` | Categoria correta, atribuída por atendente humano |
| `pred_modelo_a` | Categoria prevista pelo modelo A (produção) |
| `conf_modelo_a` | Confiança declarada pelo modelo A (0 a 1) |
| `pred_modelo_b` | Categoria prevista pelo modelo B (candidato) |
| `conf_modelo_b` | Confiança declarada pelo modelo B (0 a 1) |

---

## Parte 1: Análise Exploratória

### 1. Panorama dos Chamados

Explore o corpus e apresente o que um gestor precisaria saber sobre esses chamados: distribuição de categorias, características dos textos, padrões por canal, bairro ou tempo. Monte uma análise limpa, focando em tabelas e visualizações que **importam para o problema de classificação**.

**Entregue**: Análise exploratória com visualizações e uma síntese dos 3-5 achados mais relevantes, explicitando por que cada um importa para avaliar os classificadores.

---

## Parte 2: Auditoria do Modelo em Produção

### 2. Desempenho com Incerteza

Avalie o desempenho global e por categoria do modelo A. Reporte as métricas que julgar adequadas, **com intervalos de confiança**, justificando a escolha das métricas considerando o desbalanceamento das classes.

**Entregue**: Tabela de métricas com incerteza quantificada, método de cálculo dos intervalos explicitado e justificativa das escolhas.

### 3. Onde o Modelo Falha?

O desempenho é homogêneo? Investigue se existem **subgrupos de chamados em que o modelo falha mais** (por categoria, características do texto, canal etc.). Analise também a matriz de confusão: os erros têm padrão?

**Entregue**: Identificação e quantificação dos principais modos de falha, hipóteses sobre suas causas e discussão do **impacto prático** de cada um para o encaminhamento dos chamados.

---

## Parte 3: Modelo A vs. Modelo B

### 4. Devemos Trocar de Modelo?

Compare o desempenho dos dois modelos e recomende: devemos substituir o modelo A pelo B? Atenção a dois pontos: (a) as predições são sobre os **mesmos chamados** — escolha um teste estatístico adequado a esse desenho; (b) verifique se a conclusão da métrica global se sustenta quando você olha **por categoria**.

**Entregue**: Teste de hipótese com justificativa da escolha e interpretação correta do p-valor; comparação por categoria com discussão dos trade-offs encontrados; e um **parágrafo final de recomendação escrito para um gestor não técnico**, com os riscos da troca (se houver) explícitos e, se aplicável, medidas de mitigação. Este parágrafo deve constar também no sumário executivo do README.

---

## Bônus (opcional) - Classificação com LLM

> Desenvolva o bônus se sobrar tempo. Ele não compensa questões obrigatórias incompletas.

Use um LLM de sua escolha para classificar uma amostra dos chamados e compare com os modelos A e B.

**Entregue**: Prompt utilizado, resultados, custo aproximado e limitações da comparação.

---

## Avaliação

Você será avaliado em cada uma das categorias abaixo, com seus respectivos pesos:

- **Rigor estatístico** (métricas adequadas, incerteza, testes corretos, cuidado com conclusões): peso 2
- **Investigação e análise exploratória** (padrões não óbvios, hipóteses, conexão entre EDA e erros dos modelos): peso 1
- **Comunicação** (sumário executivo, visualizações, tradução de resultados em recomendação): peso 1
- **Boas práticas** (reprodutibilidade, organização do repositório, commits, documentação): peso 1

Uma média ponderada será calculada e os melhores candidatos serão chamados para a etapa de entrevistas.

**Dica**: não existe uma única resposta certa. Preferimos uma análise honesta sobre limitações a uma análise que finge certeza — e profundidade importa mais que completude.

---

## Estrutura Sugerida do Repositório

```
desafio-ds-junior/
├── README.md
├── notebooks/
│   ├── 01_analise_exploratoria.ipynb
│   ├── 02_auditoria_modelo_a.ipynb
│   └── 03_comparacao_e_recomendacao.ipynb
├── dados/
│   └── chamados_com_predicoes.csv
├── results/
│   └── figures/
└── requirements.txt
```

---

## FAQ

**1. Posso usar bibliotecas específicas?**
Sim! Sugestões: pandas, numpy, scipy, statsmodels, scikit-learn, matplotlib, seaborn, plotly.

**2. Posso usar assistentes de IA (ChatGPT, Claude, Copilot)?**
Sim, mas você deve ser capaz de explicar e defender cada decisão na entrevista técnica. Conclusões sem código que as produza, ou código que você não entende, contam contra.

**3. Preciso fazer todas as questões?**
As questões 1 a 4 sim, mas profundidade importa mais que completude. O bônus é opcional de verdade.

**4. Preciso treinar um modelo?**
Não. O desafio é de **avaliação** de modelos, não de treinamento — resista à tentação.

---

Boa sorte! 🚀
