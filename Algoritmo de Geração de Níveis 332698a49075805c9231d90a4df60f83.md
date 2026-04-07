# Algoritmo de Geração de Níveis

[Ideia de Arquitetura](Ideia%20de%20Arquitetura%20334698a49075808eb0c4da7641bc346c.md)

Este documento descreve a arquitetura matemática para o pré-processamento de níveis. O objetivo é gerar caminhos viáveis em um ambiente "bullet hell", garantindo matematicamente que o jogador consiga desviar dos obstáculos, ao mesmo tempo em que força momentos de alta tensão visual ("near-misses").

## 1. Terminologia e Estruturas de Dados

- **CET (Cápsula Espaçotemporal):** Representa o "cenário ideal perfeito" (perfect case scenario) do caminho do jogador. Entre dois instantes de tempo (Nodes t1 e t2), a área que o jogador ocupa é uma Cápsula geométrica (dois semicírculos conectados por um retângulo).
    - **Cálculo do Raio:** Raio da máscara de colisão do Glitchee + Margem de Erro Variável.
    - **Dinâmica do Raio:** Em determinados trechos, a margem de erro diminui, estreitando a CET. Isso aumenta a densidade de projéteis ao redor da cápsula, forçando os "near-misses" e aumentando a dificuldade de forma procedural.
- **Node:** O centro do semicírculo de uma CET, associado a um milissegundo exato (tempo t).
- **MPO (Máscara de Potencial Ofensivo):** A área máxima (polígono) que uma armadilha pode atingir, unindo todas as suas possibilidades de ataque, limitada pela geometria do cenário (paredes) e pelas leis da física (atrito).
- **DP (Distribuição de Probabilidades):** Mapeamento de todos os valores iniciais possíveis para as variáveis de uma armadilha (ex: ângulo inicial). Inicia com 100% de possibilidades. Valores proibidos são marcados com 0.

## 2. Classificação de Armadilhas

- **Imutáveis:** Agentes que não sofrem influência do ambiente ou de outras armadilhas. Eles apenas influenciam os demais. Exemplo: Canhões fixos ou giratórios.
- **Mutáveis:** Agentes cuja posição ou estado são alterados pela física do jogo e pelas ações das armadilhas Imutáveis. Exemplo: Discos empurrados por tiros.

## 3. O Core do Algoritmo: Regressão Causal

Em vez de posicionar armadilhas e testar se o caminho é possível (força bruta), o algoritmo parte do caminho pronto (CET) e calcula quais configurações das armadilhas causariam a morte do jogador. Essas configurações são então proibidas.

**Passo a Passo Geral:**

1. Verifica-se se a MPO de uma armadilha sobrepõe a CET do segmento t1-t2.
2. Se houver sobreposição espacial, calcula-se a Interseção Genuína.
3. Utiliza-se as extremidades dessa interseção para realizar o raycast regressivo até a armadilha (considerando a espessura do projétil).
4. Calcula-se a propriedade inicial (ângulo de disparo, tempo de spawn) que causaria aquele impacto exato.
5. Aplica-se a proibição cirúrgica na DP da armadilha.

## 4. Resolução da CET (A Lógica das 3 Zonas)

Para evitar proibições temporais amplas (Over-constraining) e resolver o perigo de projéteis rápidos atravessarem os segmentos (Tunneling), a CET não é tratada como um bloco temporal único, mas fatiada geometricamente:

1. **Semicírculo do Node N1:** Se o disparo atinge apenas o semicírculo traseiro da cápsula, assume-se que a colisão ocorreria exatamente no tempo t1. O ângulo limitante da armadilha é associado e regredido a partir deste tempo específico.
2. **Semicírculo do Node N2:** Mesma lógica, mas na extremidade frontal da cápsula, associando o impacto ao tempo futuro t2.
3. **O Retângulo (Segmento Intermediário):** Se o disparo atinge a área retangular que conecta N1 e N2, calcula-se o ponto exato de impacto na aresta do retângulo. Usando interpolação linear entre t1 e t2 (baseada na distância daquele ponto até N1), descobre-se o exato milissegundo tx em que o Glitchee passará por ali. A regressão ocorre a partir de tx.

Esta abordagem garante que o sistema proíba apenas a configuração exata da armadilha que causaria colisão naquele micro-instante específico, preservando ao máximo a DP.

## 5. Interseção Genuína e MPO Recortada

Se o cenário possuir obstáculos (paredes) que cortam a MPO da armadilha, a análise de impacto não parte das bordas brutas da CET:

1. Calcula-se a interseção poligonal entre a MPO (já recortada pelas paredes) e a CET.
2. Em vez de usar as extremidades nominais dos semicírculos/retângulo, o algoritmo utiliza os vértices formados pelas arestas de cruzamento entre a MPO e a CET.
3. A regressão (raycast com espessura do disparo) parte desses vértices geométricos precisos para encontrar os ângulos proibidos na armadilha.

## 6. Cinemática e Interações Complexas (Atrito e Múltiplos Impactos)

Para que armadilhas Mutáveis não entrem em loops infinitos e para que o cálculo retrocausal não sofra de explosão combinatória, o sistema implementa atrito constante, colisões elásticas e travamento de estado.

### 6.1 Atrito Limitando a MPO

A MPO de uma armadilha mutável não é infinita. Ela é um volume limitado pela distância de frenagem calculada pela energia cinética máxima possível.

- **Cálculo da MPO:** Distância Máxima (D) = (Velocidade Inicial ao Quadrado) / (2 * Atrito).
- **Regressão Desacelerada:** Se o Mutável atinge a CET a uma distância D, a velocidade original exigida no momento do impacto é calculada invertendo a cinemática: Velocidade Inicial = Raiz(Velocidade Final ao Quadrado + 2 * Atrito * D).

### 6.2 O Efeito Bilhar (Mutável colidindo com Mutável)

Para evitar cálculos vetoriais exponenciais, armadilhas mutáveis interativas possuem a mesma massa e sofrem colisões perfeitamente elásticas.

- Quando o Disco 1 bate no Disco 2, eles trocam os seus vetores de velocidade. O Disco 1 para instantaneamente e o Disco 2 herda a energia para continuar a trajetória. A regressão torna-se uma cadeia linear de transferência de vetores.

### 6.3 State-Lock (Invulnerabilidade Cinética a Múltiplos Impactos)

Para evitar ramificações infinitas na regressão caso um Disco em movimento seja atingido por outro tiro, aplica-se a regra do "State-Lock" (efeito limpa-neves).

- **Estado Estacionário:** Quando parado, o Disco tem massa igual à do projétil, absorve o impacto e entra em movimento.
- **Estado em Movimento:** Ao deslizar, o Disco assume massa infinita em relação aos projéteis. Qualquer tiro que o atinja durante o trajeto é destruído ou refletido sem alterar o vetor do Disco. Ele só volta ao Estado Estacionário após parar totalmente devido ao atrito. Isso garante previsibilidade de gameplay e mantém o algoritmo em complexidade O(N).

## 7. Exemplos Práticos de Resolução

### Exemplo 1: Armadilha Imutável (C45 - Canhão Giratório 45 Graus)

O C45 gira continuamente e atira a cada 45 graus.

1. Definimos os vértices extremos da Interseção Genuína entre a CET e a MPO do C45.
2. Fazemos a regressão do tempo de viagem do projétil a partir desses vértices (aplicando a regra das 3 Zonas para descobrir o tempo de impacto).
3. Isso nos dá dois ângulos de disparo específicos no C45 que limitam a área de colisão.
4. Marcamos todos os ângulos intermediários como proibidos naquele determinado momento.
5. Regredimos a rotação do canhão a partir desses ângulos proibidos até o tempo inicial (t = 0) para proibir o ângulo de spawn (rotação inicial) do C45 na DP.

### Exemplo 2: Armadilha Mutável (DE - Disco de Espinhos)

O DE mata o jogador ao toque e pode ser movido pelos disparos do C45.

1. Primeiro, calculamos a interação entre a CET e o DE. Descobrimos o intervalo de trajetórias do DE que fariam ele colidir com a CET no tempo mapeado pelas 3 Zonas.
2. Entramos em uma análise recursiva: calculamos a interação do DE com as armadilhas Imutáveis ao seu redor.
3. Avaliamos quais disparos do C45 forneceriam a energia cinética necessária para mover o DE nas trajetórias proibidas recém-descobertas (respeitando a regressão com atrito).
4. Esses ângulos de disparo específicos são repassados para a DP do C45 e marcados como 0 (proibidos).

## 8. Resolução de Conflitos e Prevenção de Cascata (Conjunto Vazio)

Para evitar o "Efeito Cascata" e manter a eficiência computacional, o algoritmo utiliza Colapso Sequencial e Lógica Transacional.

### 8.1 A Regra da Independência (Armadilhas Imutáveis)

Projéteis não colidem com projéteis. Logo, armadilhas Imutáveis interagem apenas com a CET.

- **O Processo:** O algoritmo adiciona armadilhas imutáveis uma a uma. Para cada armadilha adicionada, ele itera sobre todo o caminho (CET), recorta a sua DP e "trava" os valores válidos.
- **Isolamento:** Se a armadilha atual atingir o Conjunto Vazio (0 opções), ela é simplesmente descartada daquela posição, sem afetar as armadilhas anteriormente validadas.

### 8.2 Lógica Transacional para Clusters (Armadilhas Mutáveis)

Armadilhas Mutáveis criam dependência vetorial (cadeias causais).

- **O Processo (Buffer de Cálculo):** Quando um Mutável é posicionado, o algoritmo itera sobre a CET e propaga as restrições reversas pela cadeia causal até os canhões. Essas restrições são armazenadas em um estado temporário (buffer).
- **Commit ou Rollback:** Se todas as armadilhas envolvidas mantiverem pelo menos um intervalo válido, o algoritmo aplica as restrições (Commit). Se qualquer armadilha na cadeia atingir o Conjunto Vazio, os cálculos são descartados (Rollback), o Mutável é rejeitado, e as DPs das armadilhas já validadas permanecem intactas.