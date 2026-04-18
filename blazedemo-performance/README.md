# BlazeDemo — Testes de Performance

Projeto de testes de performance para a aplicação [BlazeDemo](https://www.blazedemo.com), cobrindo o fluxo completo de compra de passagem aérea. Os scripts foram desenvolvidos com **Apache JMeter 5.6.3** e contemplam três tipos de teste: Smoke, Carga e Pico.

---

## Sumário

- [Visão Geral](#visão-geral)
- [Pré-requisitos](#pré-requisitos)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Cenário de Teste](#cenário-de-teste)
- [Estratégia de Testes](#estratégia-de-testes)
- [Instruções de Execução](#instruções-de-execução)
- [Relatório de Execução](#relatório-de-execução)
- [Critério de Aceitação — Análise e Veredicto](#critério-de-aceitação--análise-e-veredicto)
- [Considerações e Recomendações](#considerações-e-recomendações)

---

## Visão Geral

| Item            | Detalhe                          |
|-----------------|----------------------------------|
| Aplicação       | https://www.blazedemo.com        |
| Ferramenta      | Apache JMeter 5.6.3              |
| Plugin          | Blazemeter Concurrency Thread Group |
| Protocolo       | HTTPS                            |
| Tipos de Teste  | Smoke · Carga · Pico             |
| Data de Execução| 18/04/2026                       |

---

## Pré-requisitos

| Requisito | Versão mínima |
|-----------|--------------|
| Java (JDK ou JRE) | 11+ |
| Apache JMeter | 5.6.3 |
| Plugin JMeter Plugins Manager | Qualquer (para Concurrency Thread Group) |
| Plugin bzm - Concurrency Thread Group | 2.10+ |

### Instalação do Plugin

1. Baixe o [JMeter Plugins Manager](https://jmeter-plugins.org/wiki/PluginsManager/) e copie o `.jar` para `$JMETER_HOME/lib/ext/`.
2. Reinicie o JMeter.
3. Em **Options → Plugins Manager**, localize e instale `bzm - Concurrency Thread Group`.

---

## Estrutura do Projeto

```
blazedemo-performance/
├── README.md
├── evidence/                          # Evidências de execução (capturas de tela, logs)
├── reports/
│   ├── carga/
│   │   ├── results.jtl                # Raw results — Teste de Carga
│   │   └── dashboard/                 # Relatório HTML gerado pelo JMeter
│   │       ├── index.html
│   │       └── statistics.json
│   └── pico/
│       ├── results.jtl                # Raw results — Teste de Pico
│       └── dashboard/
│           ├── index.html
│           └── statistics.json
└── tests/
    ├── data/
    │   ├── passengers.csv             # Massa de dados — passageiros
    │   └── routes.csv                 # Massa de dados — rotas
    ├── smoke/
    │   └── smoketest.jmx              # Smoke Test (1 usuário · 1 iteração)
    ├── carga/
    │   └── blazedemo-carga.jmx        # Teste de Carga (100 usuários · 300s)
    └── pico/
        └── blazedemo-pico.jmx         # Teste de Pico (100 usuários · 300s)
```

---

## Cenário de Teste

**Cenário:** Compra de Passagem Aérea — Passagem comprada com sucesso.

O fluxo é modelado como uma única transação (`TX - Compra Completa`) composta por quatro passos sequenciais:

| Passo | Método | Endpoint            | Asserção                                      |
|-------|--------|---------------------|-----------------------------------------------|
| 01 - GET Home          | GET  | `/`                | Resposta contém `"Find Flights"`              |
| 02 - POST Search Flights | POST | `/reserve.php`   | Resposta contém `"Flights from"` e `"Choose This Flight"` |
| 03 - POST Select Flight  | POST | `/purchase.php`  | Resposta contém `"Your flight from"` e `"Purchase Flight"` |
| 04 - POST Purchase Ticket | POST | `/confirmation.php` | HTTP 200 OK                               |

### Massa de Dados

**Rotas (`routes.csv`):**

| fromPort     | toPort       |
|--------------|--------------|
| Boston       | London       |
| Philadelphia | Berlin       |
| San Diego    | New York     |
| Mexico City  | Dublin       |
| Paris        | Buenos Aires |

**Passageiros (`passengers.csv`):** 4 registros com nome, endereço, dados de cartão (visa, amex, dinerclub) — reutilizados ciclicamente durante o teste.

---

## Estratégia de Testes

### 1. Smoke Test (`smoketest.jmx`)

Execução mínima para validar o script e a conectividade antes dos testes de carga.

| Parâmetro    | Valor |
|--------------|-------|
| Usuários     | 1     |
| Iterações    | 1     |
| Ramp-up      | 1s    |
| Objetivo     | Validar fluxo ponta-a-ponta sem erros |

### 2. Teste de Carga (`blazedemo-carga.jmx`)

Simula a carga esperada em regime de operação normal sustentada.

| Parâmetro       | Valor  |
|-----------------|--------|
| Usuários (alvo) | 100    |
| Ramp-up         | 120 s  |
| Steps (degraus) | 6      |
| Hold (sustentação) | 300 s |
| Thread Group    | Concurrency Thread Group |

### 3. Teste de Pico (`blazedemo-pico.jmx`)

Simula o comportamento do sistema sob condição de pico de acessos simultâneos.

| Parâmetro       | Valor  |
|-----------------|--------|
| Usuários (alvo) | 100    |
| Ramp-up         | 120 s  |
| Steps (degraus) | 6      |
| Hold (sustentação) | 300 s |
| Thread Group    | Concurrency Thread Group |

---

## Instruções de Execução

> **Atenção:** Antes de executar, ajuste os caminhos dos arquivos CSV nos scripts `.jmx` para corresponder ao diretório local do projeto (parâmetro `filename` nas entradas `CSVDataSet`).

### Via Interface Gráfica (GUI) — recomendado para depuração

```bash
# Iniciar JMeter
$JMETER_HOME/bin/jmeter.bat            # Windows
$JMETER_HOME/bin/jmeter.sh             # Linux / macOS

# Abrir o script desejado em File → Open
```

> **Importante:** A interface gráfica não deve ser utilizada para testes de carga real, pois consome recursos adicionais da máquina de teste e pode influenciar os resultados.

### Via Linha de Comando (CLI) — recomendado para execução de testes

#### Smoke Test

```bash
jmeter -n \
  -t tests/smoke/smoketest.jmx \
  -l reports/smoke/results.jtl \
  -e -o reports/smoke/dashboard
```

#### Teste de Carga

```bash
jmeter -n \
  -t tests/carga/blazedemo-carga.jmx \
  -l reports/carga/results.jtl \
  -e -o reports/carga/dashboard
```

#### Teste de Pico

```bash
jmeter -n \
  -t tests/pico/blazedemo-pico.jmx \
  -l reports/pico/results.jtl \
  -e -o reports/pico/dashboard
```

| Parâmetro | Descrição |
|-----------|-----------|
| `-n`      | Modo não-GUI (CLI) |
| `-t`      | Caminho do script `.jmx` |
| `-l`      | Arquivo de log de resultados `.jtl` |
| `-e`      | Gera relatório ao final |
| `-o`      | Diretório de saída do dashboard HTML |

### Visualizando o Dashboard

Após a execução, abra no navegador:

```
reports/carga/dashboard/index.html
reports/pico/dashboard/index.html
```

---

## Relatório de Execução

### Resultados — Teste de Carga

**Configuração:** 100 usuários · Ramp-up 120s (6 degraus) · Hold 300s

| Transação / Request       | Amostras | Erros | % Erro | Média (ms) | Mediana (ms) | P90 (ms) | P95 (ms) | P99 (ms)  | Throughput (req/s) |
|---------------------------|----------|-------|--------|-----------|--------------|----------|----------|-----------|--------------------|
| **TX - Compra Completa**  | 17.430   | 0     | 0,00%  | 1.618     | 1.384        | **2.024**| 2.487    | 6.506     | 41,47              |
| 01 - GET Home             | 17.430   | 0     | 0,00%  | 560       | 436          | 959      | 1.046    | 1.652     | 41,57              |
| 02 - POST Search Flights  | 17.430   | 0     | 0,00%  | 352       | 282          | 431      | 526      | 1.534     | 41,62              |
| 03 - POST Select Flight   | 17.430   | 0     | 0,00%  | 352       | 281          | 432      | 521      | 1.557     | 41,61              |
| 04 - POST Purchase Ticket | 17.430   | 0     | 0,00%  | 354       | 281          | 432      | 526      | 1.723     | 41,61              |
| **TOTAL**                 | **69.720**| **0**| **0,00%** | 405  | 311          | **494**  | 558      | 979       | **165,91**         |

---

### Resultados — Teste de Pico

**Configuração:** 100 usuários · Ramp-up 120s (6 degraus) · Hold 300s

| Transação / Request       | Amostras | Erros | % Erro | Média (ms) | Mediana (ms) | P90 (ms) | P95 (ms) | P99 (ms)  | Throughput (req/s) |
|---------------------------|----------|-------|--------|-----------|--------------|----------|----------|-----------|--------------------|
| **TX - Compra Completa**  | 15.949   | 0     | 0,00%  | 2.151     | 1.759        | **3.494**| 4.339    | 6.688     | 37,92              |
| 01 - GET Home             | 15.949   | 0     | 0,00%  | 638       | 481          | 988      | 1.497    | 2.924     | 38,02              |
| 02 - POST Search Flights  | 15.949   | 0     | 0,00%  | 503       | 351          | 831      | 1.347    | 2.728     | 38,09              |
| 03 - POST Select Flight   | 15.949   | 0     | 0,00%  | 505       | 350          | 839      | 1.381    | 2.732     | 38,09              |
| 04 - POST Purchase Ticket | 15.949   | 0     | 0,00%  | 505       | 350          | 834      | 1.389    | 2.779     | 38,08              |
| **TOTAL**                 | **63.796**| **0**| **0,00%** | 538  | 366          | **646**  | 1.015    | 3.897     | **151,73**         |

---

## Critério de Aceitação — Análise e Veredicto

**Cenário avaliado:** Compra de Passagem Aérea — Passagem comprada com sucesso (`TX - Compra Completa`)

**Critério definido:**
> 250 requisições por segundo com tempo de resposta no percentil 90 (P90) inferior a 2.000 ms.

### Avaliação por Métrica

#### Throughput (Requisições por Segundo)

| Teste  | Throughput Total Obtido | Meta      | Diferença  | Status         |
|--------|------------------------|-----------|------------|----------------|
| Carga  | 165,91 req/s           | 250 req/s | -84,09 req/s (-33,6%) | ❌ REPROVADO |
| Pico   | 151,73 req/s           | 250 req/s | -98,27 req/s (-39,3%) | ❌ REPROVADO |

O sistema, sob carga de 100 usuários virtuais concorrentes, atingiu apenas **~66%** (Carga) e **~61%** (Pico) da capacidade de throughput exigida. Para alcançar 250 req/s seria necessário aumentar significativamente o volume de usuários ou otimizar o desempenho do servidor.

#### Tempo de Resposta — P90 (Transação Completa)

| Teste  | P90 TX Compra Completa | Limite    | Diferença       | Status         |
|--------|------------------------|-----------|-----------------|----------------|
| Carga  | 2.024 ms               | < 2.000 ms| +24 ms (+1,2%)  | ❌ REPROVADO |
| Pico   | 3.494 ms               | < 2.000 ms| +1.494 ms (+74,7%) | ❌ REPROVADO |

No teste de Carga, o P90 da transação completa ultrapassou o limite em apenas 24ms — falha marginal, porém estatisticamente relevante. No teste de Pico a degradação é expressiva: o P90 atingiu 3.494ms, **74,7% acima do limite**.

#### Taxa de Erros

| Teste  | Erros | Total de Amostras | Taxa de Erro | Status      |
|--------|-------|-------------------|--------------|-------------|
| Carga  | 0     | 69.720            | 0,00%        | ✅ APROVADO |
| Pico   | 0     | 63.796            | 0,00%        | ✅ APROVADO |

A ausência total de erros demonstra que o sistema processa corretamente todas as requisições, mesmo sem atender aos SLAs de tempo e throughput.

---

### Veredicto Final

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   RESULTADO: ❌ REPROVADO                                   │
│                                                             │
│   O sistema NÃO atende ao critério de aceitação definido.  │
│                                                             │
│   • Throughput insuficiente em ambos os cenários            │
│   • P90 da transação completa acima do limite de 2.000ms   │
│   • Único ponto positivo: taxa de erros = 0%               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Considerações e Recomendações

### Pontos Positivos

- **Zero erros funcionais** em ambas as execuções (69.720 e 63.796 amostras).
- Todas as asserções de conteúdo (presença de textos esperados) passaram, confirmando a **corretude funcional** sob carga.
- O tempo de resposta de cada **requisição individual** permaneceu dentro de limites aceitáveis (P90 total: 494ms na carga; 646ms no pico).

### Pontos de Atenção

1. **Throughput:** O gargalo de throughput sugere limitação no servidor da aplicação (ou na rede), não no script de teste. O servidor processa ~166 req/s com 100 VUs; escalar para 250 req/s exigiria investigar a capacidade do servidor e possível tuning de infraestrutura.

2. **P90 da Transação Completa:** A transação acumula a latência de 4 requisições encadeadas. O `GET Home` é o passo mais lento individualmente (P90 = 959ms na carga) e contribui de forma desproporcional para o tempo total da transação.

3. **Degradação no Pico:** No teste de pico, o tempo médio da transação aumentou de 1.618ms para 2.151ms (+33%) e o P90 saltou de 2.024ms para 3.494ms (+73%), indicando que a aplicação sofre degradação relevante em condições de pico — risco operacional real.


### Recomendações Técnicas

| Prioridade | Ação |
|------------|------|
| Alta | Investigar e otimizar o endpoint `GET /` (maior latência individual) |
| Alta | Analisar capacidade de infraestrutura do servidor para suportar 250 req/s |
| Média | Parametrizar caminhos dos CSVs para execução portável |
| Média | Ampliar massa de dados (rotas e passageiros) para maior realismo |
| Baixa | Adicionar monitoração de métricas do servidor (CPU, memória, conexões) durante os testes |
| Baixa | Incluir Think Time para simulações mais realistas de comportamento de usuário |

---

> **Relatórios detalhados:** Abra `reports/carga/dashboard/index.html` ou `reports/pico/dashboard/index.html` em um navegador para visualizar gráficos de tempo de resposta, throughput ao longo do tempo e distribuição de percentis.
