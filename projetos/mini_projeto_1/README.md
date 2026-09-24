# O Oráculo da Varanda 🌱

Um Sistema Baseado em Conhecimento (SBC) desenvolvido com a biblioteca Python `experta`. O sistema recebe características de plantas (adultas, sementes ou mudas aquáticas) e utiliza encadeamento progressivo (*forward chaining*) para deduzir o substrato ideal, a alocação na casa, o dimensionamento do vaso e a receita nutricional NPK.

## 1. Descrição do Domínio
O sistema atua como um especialista de jardinagem focado na realidade física de uma varanda residencial. A base de conhecimento utiliza insumos reais restritos (Areia, Brita, Substrato da Loja, Húmus, Esterco, Substrato da Mata Atlântica). 

A arquitetura lógica baseia-se em 3 níveis de abstração:
1. **Nível 1 (Filtro e Tradução):** Identifica exceções (cultivo em água ou estufas) e converte características biológicas em perfis de drenagem, luz e umidade.
2. **Nível 2 (O Alquimista):** Agrupa os perfis soltos para gerar a receita física do vaso usando os insumos da casa.
3. **Nível 3 (Decisão):** Cruza a receita de solo com os perfis de luz e porte para alocar o vaso no microclima correto da varanda.

A resolução de conflitos é aplicada intensamente: regras específicas usam prioridade (`salience`) para sobrepor regras gerais, e o operador de negação (`NOT`) é utilizado para garantir que plantas de solo seco não recebam esterco e que adubos genéricos atuem apenas como contingência (*fallback*).

---

## 2. Base de Conhecimento (22 Regras)

**Nível 1: Exceções e Desvios (Berçário e Hidropropagação)**
* **R1 (Semente Estufa):** SE é semente para estufa ENTÃO alocar na janela fumê, usar papel toalha úmido e não adubar.
* **R2 (Semente Direta):** SE é semente para plantio direto ENTÃO usar substrato puro, vaso definitivo e manter úmido.
* **R3 (Água):** SE o cultivo é em água (`salience=500`) ENTÃO ignorar terra, usar vaso de vidro e alocar na sombra.

**Nível 1: Perfis Biológicos (Plantas Adultas)**
* **R4 a R6 (Drenagem):** SE a drenagem exigida é (alta/média/baixa) ENTÃO o perfil é (drenante/equilibrado/retentivo).
* **R7 a R9 (Rega):** SE a umidade exigida é (alta/média/baixa) ENTÃO a rega será (frequente/moderada/espaçada).
* **R10 a R12 (Luz):** SE a demanda de luz é (sol pleno/meia sombra/sombra) ENTÃO alocar na (varanda superior / varanda piso / janela fumê).
* **R13 a R15 (Vaso):** SE o porte da planta é (grande/médio/pequeno) ENTÃO o vaso será (grande/médio/pequeno).

**Nível 1: Conflito Nutricional**
* **R16 e R17 (Específicas):** SE produz frutos (`salience=320`) ou flores (`salience=315`) ENTÃO prescrever NPK 04-14-08.
* **R18 (Geral/Fallback):** SE NÃO houver necessidade nutricional detectada (`NOT`) ENTÃO prescrever NPK 10-10-10 (`salience=300`).

**Nível 2: O Alquimista (Receitas)**
* **R19:** SE o perfil é drenante ENTÃO receita = Areia, Brita e Substrato da loja.
* **R20:** SE o perfil é equilibrado ENTÃO receita = Substrato da loja, Húmus e Areia.
* **R21:** SE o perfil é retentivo E NÃO é drenante (`NOT`) ENTÃO receita = Mata Atlântica, Húmus e Esterco.

**Nível 3: Decisão Consolidada**
* **R22:** SE existem Receita, Rega, Local, Vaso e Nutrição E a decisão final ainda não foi tomada (`NOT`, para evitar loops) ENTÃO imprimir o diagnóstico.

---

## 3. Casos de Teste Avaliados

### Caso 1: Conflito de Nutrição (Pimenteira em Sol Pleno)
* **Entrada:** `Planta(nome="Pimenteira", luz="sol_pleno", drenagem="alta", umidade="media", tamanho="grande", producao="fruto")`
* **Saída Esperada:** O *salience* da Regra 16 bloqueia a Regra 18. A planta é alocada na varanda superior, com receita drenante (areia/brita), vaso grande e NPK focado em frutificação.

### Caso 2: Ativação de Fallback por Campos Omitidos (Zamioculca)
* **Entrada:** `Planta(nome="Zamioculca", luz="sombra", drenagem="alta", umidade="baixa", tamanho="pequeno")`
* **Saída Esperada:** Como o campo `producao` foi omitido, as regras específicas falham. A Regra 18 detecta a ausência usando `NOT` e dispara sozinha, recomendando NPK 10-10-10.

### Caso 3: Interceptação Aquática (Jiboia)
* **Entrada:** `Planta(nome="Muda de Jiboia", cultivo="agua", luz="sol_pleno")`
* **Saída Esperada:** O sistema detecta o cultivo na água. O `salience=500` sequestra o fluxo, ignorando o sol pleno pedido pelo usuário e forçando a planta para a sombra em um recipiente de vidro apenas com água para evitar que as raízes cozinhem.
