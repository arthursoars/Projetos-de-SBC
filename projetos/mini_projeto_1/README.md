# O Oráculo da Varanda 🌱

Um Sistema Baseado em Conhecimento (SBC) desenvolvido com a biblioteca Python `experta`. O sistema recebe características de plantas (adultas, sementes ou mudas para cultivo em água) e utiliza encadeamento progressivo (*forward chaining*) orientado a dados (*data-driven*) para deduzir o preparo do substrato ideal, a alocação microclimática no imóvel, o dimensionamento do vaso e o manejo nutricional NPK.

---

## 1. Descrição do Domínio

O sistema atua como um assistente especialista para cultivo residencial em vasos. A base de conhecimento opera sobre um conjunto delimitado de insumos reais (Areia de Construção, Brita, Substrato Comercial, Húmus de Minhoca, Esterco e Substrato da Mata Atlântica) e mapeia os microclimas físicos disponíveis na residência (Varanda Superior com abertura Norte ao sol pleno, Piso da Varanda em meia-sombra e Janela Fumê no quarto).

A arquitetura do motor organiza o raciocínio em **3 níveis de encadeamento progressivo**:

```text
[ FATOS DE ENTRADA ] (Planta: luz, umidade, drenagem, fase, porte, producao)
       │
       ▼
======================================================================
 NÍVEL 1: TRADUÇÃO & FILTROS DE EXCEÇÃO (Regras R1 a R18)
======================================================================
  ├─ Exceções críticas (Água / Estufa) interceptam o fluxo normal.
  └─ Características biológicas derivam Perfis Intermediários:
     (Ex.: umidade="baixa" ➔ PlanoRega="Espaçada")
       │
       ▼
======================================================================
 NÍVEL 2: O ALQUIMISTA DE SUBSTRATOS (Regras R19 a R21)
======================================================================
  └─ Perfis de solo ativam a formulação física com insumos reais:
     (Ex.: PerfilDrenagem="drenante" ➔ Areia + Brita + Substrato da loja)
       │
       ▼
======================================================================
 NÍVEL 3: DECISÃO CONSOLIDADA (Regra R22)
======================================================================
  └─ Integração total: Substrato + Rega + Alocação + Vaso + NPK
       │
       ▼
[ DECISÃO FINAL CONSOLIDADA & TRACE EXPLICATIVO ]
```

### Resolução de Conflitos

O sistema emprega duas estratégias formais de resolução de conflitos na agenda do motor:

1. **Prioridade Explícita (`salience`):** Regras de manejo reprodutivo (fruto `salience=320` e flor `salience=315`) sobrepõem-se à regra de adubação genérica de manutenção (`salience=300`). Da mesma forma, o cultivo aquático possui prioridade máxima (`salience=500`) para interceptar prescrições de solo e prevenir superaquecimento radicular.

2. **Condição de Ausência (`NOT`):** Garante hipóteses mutuamente exclusivas e comportamento de contingência (*fallback*). A regra geral R18 só dispara se nenhuma nutrição específica foi declarada (`NOT(NecessidadeNutricional())`), e a regra R22 utiliza `NOT(DecisaoFinal())` para prevenir disparos cíclicos (*loops* infinitos) na memória de trabalho.

---

## 2. Base de Conhecimento (22 Regras)

### Nível 1: Exceções e Rotas Rápidas (Berçário e Hidropropagação)

- **R1 (Semente para Estufa):** SE `fase="semente"` E `plantio="estufa"`, ENTÃO alocar na estufa improvisada na janela fumê, substrato de papel toalha úmido, rega contida no recipiente e suspender adubação inicial.

- **R2 (Semente para Semeadura Direta):** SE `fase="semente"` E `plantio="direto"`, ENTÃO alocar na sombra com luz indireta, substrato comercial puro, manter solo úmido e suspender adubação.

- **R3 (Cultivo em Água - Guardrail):** SE `cultivo="agua"` (`salience=500`), ENTÃO anular receitas de solo, prescrever recipiente de vidro com troca semanal e fixar alocação na sombra da varanda para proteger as raízes.

### Nível 1: Tradução Biológica (Plantas Envasadas)

- **R4:** SE `drenagem="alta"`, ENTÃO derivar perfil drenante.

- **R5:** SE `drenagem="media"`, ENTÃO derivar perfil equilibrado.

- **R6:** SE `drenagem="baixa"`, ENTÃO derivar perfil retentivo.

- **R7:** SE `umidade="alta"`, ENTÃO definir rega frequente.

- **R8:** SE `umidade="media"`, ENTÃO definir rega moderada.

- **R9:** SE `umidade="baixa"`, ENTÃO definir rega espaçada.

- **R10:** SE `luz="sol_pleno"` E NÃO for cultivo em água, ENTÃO alocar na parte superior da varanda (abertura Norte).

- **R11:** SE `luz="meia_sombra"` E NÃO for cultivo em água, ENTÃO alocar no piso da varanda.

- **R12:** SE `luz="sombra"` E NÃO for cultivo em água, ENTÃO alocar na janela fumê do quarto.

- **R13:** SE `tamanho="grande"`, ENTÃO recomendar vaso grande.

- **R14:** SE `tamanho="medio"`, ENTÃO recomendar vaso médio.

- **R15:** SE `tamanho="pequeno"`, ENTÃO recomendar vaso pequeno.

### Nível 1: Resolução de Conflito Nutricional

- **R16:** SE `producao="fruto"` (`salience=320`), ENTÃO prescrever NPK 04-14-08 para estímulo à frutificação.

- **R17:** SE `producao="flor"` (`salience=315`), ENTÃO prescrever NPK 04-14-08 para floração.

- **R18:** SE existe planta E NÃO há recomendação nutricional (`salience=300`), ENTÃO prescrever NPK 10-10-10 de manutenção e húmus de minhoca.

### Nível 2: O Alquimista (Composição Física do Substrato)

- **R19:** SE o perfil for drenante, ENTÃO formular mistura de Areia de Construção, Brita no fundo e Substrato Comercial.

- **R20:** SE o perfil for equilibrado, ENTÃO formular Substrato Comercial com Húmus de Minhoca e proporção reduzida de Areia.

- **R21:** SE o perfil for retentivo E NÃO for drenante (`NOT`), ENTÃO formular Substrato da Mata Atlântica, Húmus de Minhoca e Esterco.

### Nível 3: Decisão Consolidada

- **R22:** SE existem Receita, Rega, Local, Vaso e Nutrição E a decisão final ainda NÃO foi gerada (`NOT`), ENTÃO consolidar o fato `DecisaoFinal` e registrar a conclusão estruturada.

---

## 3. Casos de Teste Avaliados

### Caso 1: Planta Adulta Frutífera em Sol Pleno (Conflito de Nutrição)

**Entrada:**

```python
Planta(
    nome="Pimenteira",
    luz="sol_pleno",
    drenagem="alta",
    umidade="media",
    tamanho="grande",
    producao="fruto"
)
```

**Comportamento Esperado:**

R16 e R18 casam simultaneamente no ciclo inicial. Por possuir maior prioridade (`salience=320`), R16 dispara antes de R18. O disparo de R16 declara a nutrição, invalidando a condição `NOT` de R18 e removendo-a da agenda.

**Saída Consolidada:**

| Parâmetro | Resultado |
|---|---|
| **Local** | Cima da varanda (Abertura Norte) |
| **Substrato** | Areia, Brita e Substrato Comercial |
| **Rega** | Moderada |
| **Vaso** | Grande |
| **Nutrição** | NPK 04-14-08 (Frutificação) |

---

### Caso 2: Folhagem com Omissão de Produção (Validação do Fallback)

**Entrada:**

```python
Planta(
    nome="Zamioculca",
    luz="sombra",
    drenagem="alta",
    umidade="baixa",
    tamanho="pequeno"
)
```

**Comportamento Esperado:**

As regras R16 e R17 falham na correspondência de padrões por ausência do atributo `producao`. A regra R18 constata a inexistência de nutrição via operador `NOT` e dispara isoladamente.

**Saída Consolidada:**

| Parâmetro | Resultado |
|---|---|
| **Local** | Janela do quarto (Fumê) |
| **Substrato** | Areia, Brita e Substrato Comercial |
| **Rega** | Espaçada |
| **Vaso** | Pequeno |
| **Nutrição** | NPK 10-10-10 (Manutenção) + Húmus |

---

### Caso 3: Propagação Aquática sob Alerta Térmico (Salience Máxima)

**Entrada:**

```python
Planta(
    nome="Muda de Jiboia",
    cultivo="agua",
    luz="sol_pleno"
)
```

**Comportamento Esperado:**

A regra R3 entra em conflito com regras normais de iluminação e substrato, mas vence a agenda imediatamente pelo valor de prioridade (`salience=500`).

Ela força a alocação para sombra e substitui formulações sólidas por água pura, prevenindo injúria térmica às raízes.

**Saída Consolidada:**

| Parâmetro | Resultado |
|---|---|
| **Local** | Sombra da varanda |
| **Substrato** | Apenas água |
| **Rega** | Trocar a água semanalmente |
| **Vaso** | Recipiente de vidro |
| **Nutrição** | Baixa (gotas de NPK líquido opcionais) |

---

## 4. Arquitetura do Sistema

O funcionamento do sistema pode ser resumido pelo seguinte fluxo:

```text
                    ┌─────────────────────┐
                    │   Dados da Planta   │
                    │                     │
                    │ • Luz               │
                    │ • Umidade           │
                    │ • Drenagem          │
                    │ • Fase              │
                    │ • Tamanho           │
                    │ • Produção          │
                    │ • Tipo de cultivo   │
                    └──────────┬──────────┘
                               │
                               ▼
                  ┌────────────────────────┐
                  │   Motor de Inferência  │
                  │       Experta          │
                  └────────────┬───────────┘
                               │
                ┌──────────────┴──────────────┐
                │                             │
                ▼                             ▼
       ┌─────────────────┐          ┌──────────────────┐
       │ Exceções        │          │ Regras Biológicas│
       │ R1 - R3         │          │ R4 - R18         │
       └────────┬────────┘          └────────┬─────────┘
                │                            │
                └─────────────┬──────────────┘
                              │
                              ▼
                  ┌────────────────────────┐
                  │ Formulação do Substrato│
                  │       R19 - R21        │
                  └────────────┬───────────┘
                               │
                               ▼
                  ┌────────────────────────┐
                  │ Decisão Consolidada    │
                  │          R22           │
                  └────────────┬───────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Recomendação     │
                    │       Final         │
                    └─────────────────────┘
```

---

## 5. Tecnologias Utilizadas

- **Python**
- **Experta** — motor de regras e inferência baseado em conhecimento
- **Forward Chaining** — estratégia de encadeamento progressivo
- **Fatos e Regras** — representação do conhecimento
- **Salience** — mecanismo de prioridade entre regras
- **Operador `NOT`** — controle de condições de ausência e fallback

---

## 6. Conceitos de Inteligência Artificial Aplicados

O projeto demonstra a aplicação prática de conceitos fundamentais de **Inteligência Artificial Simbólica** e **Sistemas Baseados em Conhecimento**.

### Sistema Baseado em Conhecimento

O conhecimento especializado é separado da lógica de execução por meio de fatos e regras.

```text
CONHECIMENTO
     │
     ├── Fatos
     │    ├── luz
     │    ├── umidade
     │    ├── drenagem
     │    ├── tamanho
     │    └── produção
     │
     └── Regras
          ├── R1 - R18
          ├── R19 - R21
          └── R22
```

### Encadeamento Progressivo

O sistema parte dos fatos fornecidos pelo usuário e deriva novos fatos progressivamente até chegar à decisão final.

```text
Fatos iniciais
      │
      ▼
Características biológicas
      │
      ▼
Perfis intermediários
      │
      ▼
Formulação do substrato
      │
      ▼
Decisão consolidada
```

### Resolução de Conflitos

Quando múltiplas regras podem ser ativadas simultaneamente, o motor utiliza o mecanismo de `salience` para determinar a prioridade de execução.

```text
Cultivo em água       → salience 500
        │
        ▼
Produção de fruto     → salience 320
        │
        ▼
Produção de flor      → salience 315
        │
        ▼
Manutenção geral      → salience 300
```

---

## 7. Objetivo do Projeto

O objetivo do **Oráculo da Varanda** é demonstrar como técnicas de Inteligência Artificial Simbólica podem ser utilizadas para transformar conhecimento especializado sobre cultivo de plantas em um sistema automatizado de tomada de decisão.

A partir de um conjunto limitado de características fornecidas pelo usuário, o sistema consegue:

- Identificar necessidades relacionadas à iluminação;
- Determinar a frequência de rega;
- Classificar a necessidade de drenagem;
- Recomendar o tamanho do vaso;
- Formular combinações de substratos;
- Identificar necessidades nutricionais;
- Resolver conflitos entre regras;
- Tratar situações excepcionais, como sementes e cultivo em água;
- Consolidar todas as inferências em uma recomendação final.

---

## 8. Considerações sobre o Raciocínio

Uma das principais características do sistema é que a decisão final não é determinada diretamente pelos dados de entrada.

Em vez disso, o sistema realiza uma sequência de inferências:

```text
Entrada
  │
  ├── luz
  ├── umidade
  ├── drenagem
  ├── tamanho
  └── produção
       │
       ▼
  Inferências intermediárias
       │
       ├── Perfil de drenagem
       ├── Plano de rega
       ├── Localização
       └── Tamanho do vaso
       │
       ▼
  Formulação do substrato
       │
       ▼
  Necessidade nutricional
       │
       ▼
  Decisão Final
```

Esse processo permite que o sistema apresente não apenas uma recomendação, mas também um **rastro explicativo (*trace*)** de como a decisão foi obtida.

---

## 9. Conclusão

O **Oráculo da Varanda** demonstra a construção de um Sistema Baseado em Conhecimento utilizando regras declarativas e encadeamento progressivo.

A organização das regras em diferentes níveis permite separar:

1. **Exceções e situações especiais;**
2. **Interpretação das características biológicas;**
3. **Formulação física do substrato;**
4. **Resolução de necessidades nutricionais;**
5. **Consolidação da decisão final.**

O uso de mecanismos como `salience` e `NOT` permite lidar com conflitos, prioridades e situações de fallback, tornando o processo de inferência mais estruturado e previsível.

O projeto utiliza, portanto, um domínio cotidiano — o cultivo residencial de plantas — para demonstrar conceitos fundamentais de **Inteligência Artificial Simbólica, Sistemas Especialistas, Representação do Conhecimento e Inferência Baseada em Regras**.
