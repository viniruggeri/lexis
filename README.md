# Lexis — Interpretable Regime Discovery in Dynamical Systems

---

## Princípio Central

Dado um sistema dinâmico desconhecido, Lexis descobre as leis aproximadas que o governam, os regimes onde essas leis são válidas, e o grau de confiança em cada descoberta — sem assumir complexidade ou incerteza a priori.

O objetivo não é prever diretamente $f(x)$, mas **ler a linguagem do sistema** — inferir qual estrutura governa $f$ em diferentes estados, e quão confiante é essa inferência.

---

## Definição de Regime

Um regime é um conjunto local de dinâmicas onde uma família de equações descreve o comportamento do sistema de forma estável dentro de uma janela temporal ou região do espaço de estados, com bordas probabilisticamente definidas.

---

## Pipeline

### 1. Aquisição e Pré-processamento
- Limpeza, normalização e alinhamento temporal
- Estimativa empírica da distribuição de ruído dos dados

### 2. Representação do Sistema
- Features físicas quando disponíveis
- Embedding via autoencoder opcional para alta dimensão

### 3. Detecção Adaptativa de Janelas
- Change point detection baseado em **resíduo SINDy** como sinal de mudança
- Três modos disponíveis:
  - `fixed(size)` — baseline
  - `residual_threshold(k, m)` — MVP
  - `bocpd(hazard, prior)` — probabilístico, recomendado
- BOCPD produz $P(\text{change point em } t)$ — incerteza de borda como output nativo

### 4. Descoberta de Dinâmica Local
Por janela detectada:
- SINDy — identificação esparsa da dinâmica
- Symbolic Regression — opcional, para famílias funcionais mais ricas
- Output: conjunto de equações candidatas por janela

### 5. Seleção por Pareto Frontier
Critérios simultâneos:
- Erro de reconstrução local
- Complexidade simbólica (número de termos, sparsidade)
- Soluções dominantes no trade-off erro vs complexidade são mantidas

### 6. Estimativa de Incerteza em Dois Níveis

**Nível 1 — Incerteza interna ao regime:**
- Ensemble SINDy via subsampling
- Perturbação com estratégia escolhida:
  - `gaussian(sigma)` — baseline
  - `student_t(nu, sigma)` — caudas médias
  - `laplace(scale)` — caudas exponenciais
  - `empirical(residuals)` — aprende dos dados
- Métricas: variância dos coeficientes, frequência de seleção de termos

**Nível 2 — Incerteza de borda:**
- Derivada diretamente do BOCPD
- Regimes com bordas incertas herdam incerteza maior independente da estabilidade interna

### 7. Identificação e Caracterização de Regimes
- Clustering de equações por **distância em trajetórias geradas** sob condições iniciais comuns
- Distância no espectro do Jacobiano como métrica secundária — captura estabilidade local
- Validação: estabilidade estrutural sob perturbação — um regime é válido se os mesmos termos são selecionados e coeficientes variam menos que $\delta$ sob perturbação $\epsilon$

---

## Output por Regime

Para cada regime identificado:
- Equações candidatas rankeadas por Pareto (erro vs complexidade)
- Incerteza interna — variância paramétrica e estrutural
- Incerteza de borda — confiança nos limites temporais
- Variáveis dominantes — sensibilidade local
- Intervalo de validade — temporal ou no espaço de estados
- Classificação emergente: o método caracteriza complexidade e incerteza, não as assume

---

## Propriedades

| propriedade | descrição |
|---|---|
| agnóstico ao sistema | não assume tipo de dinâmica a priori |
| interpretável por construção | output são equações, não black boxes |
| incerteza em dois níveis | interna + borda |
| robusto a caudas | estratégia de perturbação configurável |
| validação estrutural | regimes testados sob perturbação, não só por fit |

---

## Extensões Futuras
- Inferência bayesiana sobre estrutura de equações
- Integração com PINN para regimes com física parcialmente conhecida
