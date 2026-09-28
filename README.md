# Sistema de Otimização de Carteiras de Investimento (GA vs Gurobi)

Este projeto consiste em uma plataforma de otimização multiobjetivo e programação matemática aplicada à seleção e alocação de carteiras de investimento. O sistema compara o desempenho de duas abordagens distintas de otimização: **Algoritmo Genético (GA)** via `pymoo` e **Programação Inteira Quadrática Mixed-Integer (MIQP)** via `Gurobi` (com suporte a *Warm Start* e *Cold Start*).

Uma interface web intuitiva em **Flask** e **HTML/JS** permite parametrizar restrições operacionais e financeiras, executar os otimizadores, visualizar alocações por ativo/setor e construir a **Fronteira Eficiente** em tempo real.

---

## 📁 Estrutura de Áreas e Arquivos do Projeto

```text
Trabalho_OTM/
│
├── config.py                 # Definição do universo de ativos, setores, falhas conhecidas e hiperparâmetros
├── preparar_dados.py         # Coleta de cotações (yfinance) e CDI (BCB), cálculo de retornos, covariância, CVaR e P/VP
├── modelo_AG.py              # Implementação da Otimização via Algoritmo Genético (pymoo)
├── modelo_GUROBI.py          # Implementação do modelo de otimização matemática via Gurobi (gurobipy)
├── plot.py                   # Geração de gráficos de alocação (pizza) e comparações visuais
├── app.py                    # Servidor web Flask e rotas de integração do sistema
│
├── site/                     # Interface Web (Frontend)
│   └── index.html            # Dashboard interativo responsivo com formulários e visualização de resultados
│
└── plots/                    # Diretório onde são salvos os gráficos gerados dinamicamente
```

---

## ⚙️ Principais Funcionalidades e Modelagem

### 1. Coleta e Tratamento de Dados (`preparar_dados.py`)
- **Ativos Suportados**: Ações B3, BDRs, ETFs, FIIs e Criptoativos mapeados por setores em `config.py`.
- **Indicadores Macroeconômicos e Benchmarks**: Download automático do CDI diário (via API do Banco Central do Brasil) e dos índices Ibovespa e S&P 500 (em BRL).
- **Métricas de Risco e Retorno**:
  - Retornos Médios Históricos e Matriz de Covariância Anualizados.
  - **CVaR (Conditional Value at Risk - 95%)**: Medida de risco de cauda.
  - **P/VP (Preço sobre Valor Patrimonial)**: Indicador de valuation.
  - **Filtro de Liquidez**: Limita a alocação a no máximo 10% do volume médio diário negociado do ativo.

### 2. Algoritmo Genético - GA (`modelo_AG.py`)
- Formulada com a biblioteca `pymoo` como um problema de otimização escalarizado multiobjetivo.
- Minimiza a função objetivo: `(Risco * Lambda) - Retorno + Penalidade P/VP + Penalidade CVaR + Penalidade Caixa`.
- Aplica um operador de reparo customizado (`SectorCapRepair`) para garantir que os limites por setor e os aportes mínimos não sejam violados.

### 3. Programação Quadrática Inteira - Gurobi (`modelo_GUROBI.py`)
- Resolução exata de otimização matemática (MIQP) via `gurobipy`.
- Trabalha com **lotes/cotas inteiras** de ativos com base no capital investido e preço unitário atual.
- **Cardinalidade Restrita**: Define limite máximo global de ativos na carteira e limite máximo de ativos por setor.
- **Modos de Execução**:
  - **Warm Start**: Inicializa as variáveis de decisão usando a solução obtida previamente pelo Algoritmo Genético para acelerar a convergência.
  - **Cold Start**: Resolve o problema do zero sem dicas iniciais.

### 4. Interface Web e Fronteira Eficiente (`app.py` / `index.html`)
- Dashboard interativo para inserção de parâmetros de carteira.
- Visualização gráfica da distribuição percentual e financeira por ativo e por setor.
- Geração da **Fronteira Eficiente** em paralelo variando o fator de aversão ao risco ($\lambda$).

---

## 📋 Pré-requisitos

- **Python**: versão 3.9 ou superior.
- **Licença do Gurobi**: O solucionador Gurobi exige uma licença ativa (licença acadêmica ou comercial gratuita). Caso não possua a licença configurada, instale o `gurobipy` e ative sua licença seguindo as instruções oficiais da [Gurobi Web License Service](https://www.gurobi.com/).

### Pacotes Python Necessários
- `flask`
- `gurobipy`
- `pymoo`
- `yfinance`
- `pandas`
- `numpy`
- `matplotlib`
- `python-bcb`

---

## 🚀 Passo a Passo de Instalação e Configuração

### 1. Clonar o repositório e navegar até a pasta do projeto
```bash
git clone <URL_DO_REPOSITORIO>
cd Trabalho_OTM
```

### 2. Criar e ativar um ambiente virtual (recomendado)

- **Windows (PowerShell)**:
  ```powershell
  python -m venv venv
  .\venv\Scripts\Activate.ps1
  ```
- **Linux/macOS**:
  ```bash
  python3 -m venv venv
  source venv/bin/activate
  ```

### 3. Instalar as dependências

```bash
pip install flask gurobipy pymoo yfinance pandas numpy matplotlib python-bcb
```

---

## 💻 Como Executar o Projeto

1. Certifique-se de estar com o ambiente virtual ativado e dentro do diretório `Trabalho_OTM`.
2. Execute o servidor web:
   ```bash
   python app.py
   ```
3. Abra o navegador e acesse:
   ```text
   http://127.0.0.1:5000
   ```

---

## 📖 Guia de Uso da Interface Web

Ao acessar a aplicação (`http://127.0.0.1:5000`), você poderá parametrizar a otimização na barra lateral e nos campos principais:

### 1. Parâmetros de Entrada
- **Valor a Investir (R$)**: O capital total disponível para alocação.
- **Aversão ao Risco ($\lambda$)**: Define o peso da volatilidade na função objetivo (valores maiores priorizam menor risco; valores menores priorizam maior retorno).
- **Risco Máximo Tolerado (%)**: Teto superior para a volatilidade anualizada da carteira.
- **Teto Máximo por Ativo (%)**: Percentual máximo do capital que pode ser alocado em um único ativo (ex: 30%).
- **Teto Máximo por Setor (%)**: Percentual máximo acumulado permitido para um setor inteiro.
- **Máximo de Ativos na Carteira**: Limite global de cardinalidade (ex: no máximo 15 ativos).
- **Máximo de Ativos por Setor**: Limite de cardinalidade setorial (ex: no máximo 4 ativos por setor).
- **Setores Proibidos**: Permite selecionar setores a serem explicitamente ignorados na otimização.

### 2. Execução da Otimização
- Clique no botão **"Otimizar Carteira"**.
- O sistema executará o pré-carregamento de dados, o Algoritmo Genético, o Gurobi Warm Start e o Gurobi Cold Start.
- Em poucos segundos, os resultados comparativos serão exibidos em cartões e gráficos.

### 3. Leitura dos Resultados
- **Resumo da Carteira**: Compara Retorno Esperado, Volatilidade, Número de Ativos e Quantidade de Setores entre as três abordagens.
- **Gráficos de Pizza**: Exibe a alocação visual por ativo para cada método.
- **Tabelas Detalhadas**: Detalha cada ativo selecionado, a quantidade de cotas a comprar, o peso percentual e o valor financeiro alocado.
- **Alocação Setorial**: Mostra o percentual e o valor total investido por setor econômico.
- **Fronteira Eficiente**: Ao clicar em **"Calcular Fronteira Eficiente"**, o sistema gera o gráfico de dispersão Risco x Retorno comparando as curvas do GA e do Gurobi.
