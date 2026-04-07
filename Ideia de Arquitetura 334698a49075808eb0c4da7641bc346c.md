# Ideia de Arquitetura

A solução arquitetural baseia-se no padrão de **Inversão de Controle via Contrato (Trait)**. O Motor Central não processa a física das armadilhas; ele apenas delega cálculos geométricos e temporais genéricos para cada entidade.

## 1. O Contrato da Entidade (A Interface "Retrocausal")

Qualquer armadilha inserida no jogo deve obrigatoriamente implementar quatro métodos matemáticos independentes. O Motor Central chamará esses métodos em loop.

1. `Get_MPO(scenario_geometry)`: A armadilha recebe a geometria das paredes e devolve seu polígono máximo de alcance espacial.
2. `Intersect_CET(cet_segment)`: A armadilha recebe um fragmento da Cápsula (N1, N2 ou Retângulo) e devolve as coordenadas exatas e o tempo `$tx$` do cruzamento, se houver.
3. `Regress_Origin(intersection_data)`: A armadilha recebe o ponto/tempo do impacto e usa sua física interna (secreta para o Motor) para calcular qual parâmetro inicial gerou aquele impacto. Retorna um conjunto de "Parâmetros Proibidos".
4. `Commit_Restriction(forbidden_params)`: A armadilha atualiza sua própria Distribuição de Probabilidade (DP) ou, se for Mutável, repassa a restrição para a armadilha Imutável "pai" através da cadeia de dependência.

## 2. O Loop do Motor Central (Agnóstico)

O Motor não tem `if (trap == "Laser")`. Seu loop linear (O(N)) é o seguinte:

```
Para cada Armadilha no Nível:
  1. motor.poligono_mpo = armadilha.Get_MPO(paredes)
  2. Para cada Segmento da CET:
       Se motor.poligono_mpo cruza Segmento:
         3. dados_impacto = armadilha.Intersect_CET(Segmento)
         4. parametros_proibidos = armadilha.Regress_Origin(dados_impacto)
         5. armadilha.Commit_Restriction(parametros_proibidos)
         Se armadilha.DP == Vazio: Dispara Rollback Transacional.
```

## 3. Prova de Aplicação: 3 Armadilhas Distintas

Abaixo, demonstramos como três armadilhas com comportamentos lógicos, cinemáticos e temporais totalmente diferentes respondem aos mesmos quatro métodos do Contrato, sem exigir alterações no Motor Central.

### Armadilha A: Laser de Varredura (Imutável, Movimento Angular Contínuo)

*O Laser varre o cenário como um farol.*

1. `Get_MPO`: Retorna um setor circular (fatia de pizza) correspondente à visão do canhão, cortado pelas paredes próximas.
2. `Intersect_CET`: Compara o setor circular com os semicírculos da CET. Descobre que o jogador passa pelo ponto (X,Y) no tempo `$tx$`.
3. `Regress_Origin`: Internamente, aplica uma divisão simples: `(Ângulo_Impacto) - (Velocidade_Rotacao * tx)`. Retorna o ângulo de spawn proibido.
4. `Commit_Restriction`: Remove aquele ângulo específico da sua matriz interna de DP.

### Armadilha B: O Sinal Corrompido (Imutável Anômalo, Disparo com Pausa no Ar)

*Um tiro que viaja, congela no ar por 500ms, e volta a voar com dobro da velocidade.*

1. `Get_MPO`: Retorna um segmento de reta do canhão até a parede oposta.
2. `Intersect_CET`: Cruza a reta com a CET. Identifica a distância linear do impacto e o tempo exato `$tx$`.
3. `Regress_Origin`: A matemática interna da armadilha usa uma função linear definida por partes (Piecewise). Se a distância do impacto estiver antes do ponto de pausa, regride com `Velocidade 1`. Se estiver após o ponto de pausa, calcula a regressão subtraindo o tempo congelado (500ms) e usando a `Velocidade 2`. Retorna o `tempo_de_disparo` proibido.
4. `Commit_Restriction`: Remove o milissegundo exato do impacto da sua lista de tempos de spawn válidos (DP).

### Armadilha C: Lente Aceleradora (Mutável, Alteração Dinâmica de Tempo)

*Um bloco que desliza. Se um tiro passar por dentro dele, a velocidade do tiro dobra permanentemente.*

1. `Get_MPO`: Diferente de um projétil, a MPO da Lente é o retângulo que ela forma deslizando do repouso até o ponto de parada (baseado no atrito dinâmico).
2. `Intersect_CET`: A Lente não fere o jogador. Ela retorna *Null* para colisões com a CET do Glitchee. Contudo, ela implementa uma variação da interface para interceptar os `Regress_Origin` de outras armadilhas.
3. `Regress_Origin`: Quando o raycast regressivo de *outra* armadilha cruza a MPO espacial e temporal da Lente, ela intercepta a requisição. A Lente recalcula o tempo do projétil, dobrando a distância virtual daquele ponto para trás, e devolve o raycast modificado para o motor continuar a regressão até o canhão original.
4. `Commit_Restriction`: Não possui DP própria. Sua restrição (o impacto inicial que a fez deslizar) repassa o Rollback para o tiro Imutável original.