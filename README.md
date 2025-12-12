# Distributed Database Replication - Multicast communication with Vector Clocks

# Implementadas no Trabalho de Sistema Distribuído (Parte 4)

## Mecanismo de Coordenação Baseado em Ordenação Causal (Relógios

## Vetoriais)

A Parte 4 do trabalho buscou implementar um mecanismo de coordenação mais leve
**[User Query]** que ainda garantisse a consistência entre as réplicas, optando pela
ordenação causal em vez da ordenação total estrita implementada na Parte 3, **[User
Query]**. Este mecanismo utiliza **Relógios Vetoriais (Vector Clocks)** , eliminando a
necessidade da heurística complementar de desempate de timestamp utilizada com os
relógios escalares **[User Query]**.

## 1. Implementação do Relógio Vetorial

Para suportar a ordenação causal, cada processo mantém um vetor global
(vectorClock), inicializado com zeros, onde cada entrada **V[i]** conta o número de
eventos que o processo **Pᵢ** executou.
**1.1. No Envio de Mensagens**
O processo segue a **Regra 2 do Relógio Vetorial** antes de enviar uma mensagem:

1. A própria entrada do vetor ( **vectorClock[myself]** ) é incrementada.
    Esta ação representa o evento de entrega local (implícita) e prepara o vetor para
    ser enviado.
2. A mensagem é carimbada com o vetor completo, juntamente com o **ID do**
    **processo remetente** e os **dados da operação**.
● **Formato da mensagem enviada:**
    (process_id, vector_clock, operation_data)
**1.2. Na Recepção e Entrega de Mensagens (Regras 3a e 3b)**
O relógio vetorial é atualizado **somente após a entrega de uma mensagem** (imediata
ou desbloqueada da fila):


1. **Atualização para o máximo (Regra 3a):**
    O vetor local é atualizado componente a componente:
    V_local[k] = max(V_local[k], V_recebido[k])
2. **Incremento local (Regra 3b):**
    O relógio local na entrada do próprio processo ( **vectorClock[myself]** ) é
    incrementado em 1.

## 2. Mecanismo de Entrega Causal (Fila de Espera)

Mensagens recebidas que ainda não podem ser entregues (por violarem a ordem
causal) são armazenadas em uma **fila de espera (deliveryQueue)**.
● **Estrutura da Fila:**
Armazena tuplas no formato:
(sender_id, received_vector, operation_data)
● **Condição Causal (can_deliver_causally):**
Uma mensagem só pode ser entregue se satisfizer:
○ **Adiantamento correto do remetente:**
received_vector[sender_id] ==
current_vector[sender_id] + 1
○ **Dependências satisfeitas:**
Para todo **k ≠ sender_id** :
received_vector[k] <= current_vector[k]
● **Processamento da Fila:**
A fila é processada repetidamente pela função **process_queue_and_deliver**
para tentar desbloquear mensagens.
Este processamento ocorre:
○ imediatamente após a recepção de uma nova mensagem, ou
○ em caso de _timeout_ durante o loop de recebimento,
evitando, assim, bloqueios.
● **Entrega Final:**
A entrega só é considerada completa quando:


```
○ stopCount atinge o número total de processos ( N ), E
○ a fila de entrega está vazia.
```
## 3. Verificação de Eficácia no Comparison Server (PT4)

O **ComparisonServer na Parte 4** (implementado em comparisonserver-PT4)
mantém a lógica de receber os logs finais de todos os peers para comparação.
● **Log Esperado:**
O tamanho esperado é:
**N × N_MSGS**
Cada log armazena as mensagens entregues na ordem causal.
● **Processo de Comparação:**
Compara-se a **j-ésima mensagem do Peer 0** com a **j-ésima mensagem de
todos os outros peers**.
● **Resultado da Consistência:**
A variável unordered conta discrepâncias:
○ unordered == 0 → _“Todos os Peers entregaram as mensagens na
mesma ordem causal”_

## 4. Desafios Encontrados e Próximos Passos

A implementação da ordenação causal na Parte 4 utilizou o protocolo **UDP** para
comunicação das mensagens de dados, assim como na Parte 3.
● **Problemas de Execução:**
Durante os testes, foram observados erros e a execução do código ficou travada
( **deadlock** ) **[User Query]**.
O log do peer também indicou uma advertência fatal de incompatibilidade de
tamanho de log, sugerindo que o número esperado de mensagens não foi
entregue.
● **Causa Provável:**
A perda inerente de pacotes no protocolo UDP, combinada com a complexidade
da ordem causal


(onde a falta de uma mensagem pode impedir a entrega de mensagens futuras),
pode levar a **deadlocks** ou a **logs incompletos [User Query]**


