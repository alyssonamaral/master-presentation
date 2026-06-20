# Roteiro de Fala — Defesa de Mestrado
**Learning Node Orderings for Autoregressive Graph Generation**
Alysson Amaral da Silva · UFPR · 2026

---

## Slide 1 — Título
- Apresentar nome, programa, orientador
- Pergunta central: *em que ordem apresentar os nós ao modelo durante o treino?*
- Essa escolha tem impacto real e mensurável na qualidade da geração

---

## Slide 2 — Grafos estão em todo lugar
- Grafos modelam relações: moléculas (átomos + ligações), redes sociais, circuitos
- Motivação prática: aprender a **gerar novos grafos** a partir de dados observados
- Aplicações: descoberta de fármacos, design de materiais

---

## Slide 3 — Objetivo dos modelos autoregressivos
- Geração autoregressiva: construir o grafo passo a passo como sequência de decisões
- Ações: AddNode, AddEdge, ChooseDest
- Treino com **teacher forcing**: modelo recebe a sequência correta
- Pergunta que surge: **qual sequência apresentar ao modelo?** → problema da ordenação

---

## Slide 4 — O problema central: invariância a permutação
- Grafo com *n* nós admite até *n!* ordenações — todas representam o **mesmo grafo**
- Cada ordenação gera uma sequência de ações diferente → problema de aprendizado diferente
- Tensão: grafo é invariante a permutação, mas redes neurais precisam de representação ordenada

---

## Slide 5 — De grafo para sequência de ações
- A ordenação define **quem está disponível** como destino em cada passo
- ChooseDest: se a ordenação for ruim, destino correto se perde entre muitos candidatos
- Mesmo grafo, ordenações diferentes → destinos diferentes, problema diferente

---

## Slide 6 — Geradores autoregressivos de grafos
- Usamos o **DGMG** (Li et al., 2018) como ambiente controlado
- Escolha intencional: o DGMG torna a dependência na ordenação explícita
- Mantemos a arquitetura fixa → qualquer diferença vem da ordenação

---

## Slide 7 — A pergunta da dissertação
- Ordenação é detalhe de implementação ou **variável de modelagem**?
- Tese: é uma variável de modelagem, e pode ser **otimizada**
- A defesa apresenta evidências empíricas dessa posição

---

## Slide 8 — Ordenações determinísticas
- Experimento controlado: arquitetura fixa, só muda a ordenação
- 8 estratégias testadas: BFS, DFS, Degree, Degeneracy, LexBFS, MCS, Spectral, Random
- Cada uma codifica um prior estrutural diferente sobre o grafo

---

## Slide 9 — Famílias de grafos sintéticos
- 5 famílias com restrições estruturais distintas: ciclos, árvores, árvores binárias, estrelas, BA
- Cada família enfatiza uma propriedade diferente
- Grafos BA são os mais complexos: crescimento, cauda pesada, conectividade esparsa

---

## Slide 10 — Resultados: cada topologia tem um favorito
- Resultado principal: **não existe heurística universalmente ótima**
- Ciclos → DFS/LexBFS (grafo vira caminho); Árvores → BFS (pai na fronteira); Estrelas → Degree (hub primeiro)
- Grafos BA: nenhuma ordenação domina → motiva a parte aprendida

---

## Slide 11 — O caso Barabási–Albert em detalhe
- BFS: conectividade perfeita, mas grafos pequenos demais (média 27 nós)
- Degree: tamanho perfeito, mas conectividade cai para 65%
- Nenhuma heurística é boa em tamanho, conectividade e hubs **ao mesmo tempo**
- Esse trade-off é o espaço onde um método aprendido pode atuar

---

## Slide 12 — Ordenação como moldagem de entropia
- ChooseDest: boa ordenação = distribuição concentrada = aprendizado fácil
- Princípio da **localidade**: vizinhos do nó atual ficam próximos na sequência
- A noção de localidade muda com a topologia → explica por que cada família tem um favorito

---

## Slide 13 — Conclusão do bloco / motivação
- Nenhuma heurística fixa é universalmente ótima
- Pergunta natural: **e se a ordenação fosse aprendida?**
- Essa é a contribuição principal da dissertação

---

## Slide 14 — Arquitetura da política (visão geral)
- Política: rede neural que escolhe nós um a um até produzir uma permutação completa
- Pipeline: Encoder de grafo → Pooling global → MLP de score → Softmax mascarado
- Duas variantes: GCN RL Ordering e Attention RL Ordering

---

## Slide 15 — Componentes da política
- **Encoder**: embedding por nó, usando estrutura do grafo e conjunto já selecionado
- **Indicador de seleção**: sinaliza quais nós estão disponíveis
- **Pooling global**: contexto do grafo parcialmente ordenado
- **Softmax mascarado**: garante validade da permutação (nenhum nó repetido)

---

## Slide 16 — Duas políticas aprendidas
- **GCN RL**: encoder com convoluções → valida que o framework é viável
- **Attention RL**: encoder com atenção + 3 camadas de message-passing + warm start por degree
- Warm start: 1ª iteração usa ordenação por grau; depois o RL refina

---

## Slide 17 — MDP
- **Estado**: grafo fixo + conjunto de nós já selecionados
- **Ação**: escolher próximo nó entre os disponíveis
- **Recompensa atrasada**: só calculada após DGMG ser treinado e avaliado
- Probabilidade da ordenação = produto das probabilidades de cada escolha no episódio

---

## Slide 18 — Formulação bilevel
- **Nível superior**: política escolhe a ordenação
- **Nível inferior**: DGMG treina na sequência induzida por essa ordenação
- Resolver o nível inferior até convergência a cada passo seria inviável → **otimização alternada**

---

## Slide 19 — Recompensa composta
- **R_pred**: NLL de validação do DGMG — NLL baixa = modelo aprendeu bem
- **R_struct**: métricas estruturais dos grafos gerados (valid-size, conectividade, Gini, hubiness)
- NLL baixa sozinha não garante grafos estruturalmente corretos → o termo estrutural é necessário

---

## Slide 20 — Gradiente de política (REINFORCE)
- Ordenação é discreta → não dá pra fazer backprop direto → usamos REINFORCE
- Ideia: aumentar probabilidade de ordenações bem recompensadas, diminuir as ruins
- Baseline *b* reduz variância sem introduzir viés
- Teorema 6.1 da dissertação formaliza o gradiente para o problema de ordenação

---

## Slide 21 — Otimização alternada
- Loop: política propõe ordenação → tradutor gera sequências → DGMG treina → avalia recompensa → atualiza política
- Política e gerador **co-adaptam** a cada iteração externa
- O mesmo procedimento vale para GCN RL e Attention RL

---

## Slide 22 — Política vs heurística fixa
- Heurística: mesma regra em todo grafo, todo passo
- Política: decisão **condicionada** no grafo atual e no prefixo já construído
- Gráfico de entropia: começa alta (exploração), decresce gradualmente, sem colapsar → a política aprende sem perder diversidade

---

## Slide 23 — Protocolo experimental (BA)
- Foco em **Barabási–Albert**: família mais desafiadora, trade-off real entre métricas
- 6.000 grafos, m=2, N ∈ [20,60]
- Métricas: valid-size ratio, connected ratio, degree Gini, hubiness

---

## Slide 24 — GCN RL vs Attention RL
- **GCN RL**: valid-size perfeito ✓, conectividade 63% ✗ — encoder não expressivo o suficiente
- **Attention RL**: valid-size perfeito ✓ + conectividade 99,6% ✓
- Atenção + 3 camadas + warm start resolvem o gargalo de conectividade

---

## Slide 25 — Tabela comparativa final
- Cada heurística lidera em alguma métrica, mas nenhuma domina **todas ao mesmo tempo**
- BFS: conectividade perfeita, valid-size fraco; Degree: valid-size perfeito, conectividade 65%
- **Attention RL**: único a combinar valid-size perfeito + conectividade 99,6% + hubiness competitivo
- Compara com Degree especificamente: mesmo valid-size, conectividade sobe de 65% para 99,6%

---

## Slide 26 — Evolução da loss
- Comportamento **oscilatório** esperado em otimização alternada com RL
- Melhor checkpoint na **iteração 61** — relativamente tarde → política continua refinando, não colapsa
- Justifica monitorar loss e guardar o melhor checkpoint, não só o modelo final

---

## Slide 27 — Interpretação dos resultados
- Política **não copia** a ordenação por grau — vai além do warm start
- RL refina o prior hub-aware para ganhar conectividade → estratégia híbrida genuína
- Método funciona como **controlador estrutural**: controla como o grafo é exposto ao gerador
- Em princípio, pode ser acoplado a outros geradores sequenciais além do DGMG

---

## Slide 28 — Publicação aceita
- Artigo derivado desta pesquisa aceito na **BRACIS 2026** (36° Brazilian Conference on Intelligent Systems)
- Publicado pela **Springer Nature**
- Validação externa da relevância da pesquisa

---

## Slide 29 — Contribuições da dissertação
- **Análise empírica controlada**: cada topologia favorece uma heurística diferente
- **Formulação do MDP**: ordenação como problema de otimização com recompensa composta
- **Attention RL Ordering**: único método a equilibrar valid-size, conectividade e hubs simultaneamente nos grafos BA

---

## Slide 30 — Limitações e trabalhos futuros
- Limitações: só grafos BA sintéticos; custo computacional alto; alta variância do REINFORCE
- Futuros: datasets reais (moléculas, redes biológicas); actor-critic / importance sampling; análise do que a política aprendeu; grafos com atributos e tipos de arestas

---

## Slide 31 — Mensagem final
- Ordenação não é pré-processamento neutro — é parte do modelo
- Tratá-la como alvo de otimização melhora a geração autorregressiva
- Obrigado — à disposição para perguntas
