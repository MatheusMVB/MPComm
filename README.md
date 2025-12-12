# Distributed Database Replication - Multicast communication with Lamport Clock

**Evoluções **

**Implementadas no Trabalho de Sistema Distribuído \(Parte 3\)** **Mecanismo de Coordenação Baseado em Ordenação Total** A Parte 3 do trabalho focou na implementação de um mecanismo de coordenação, conforme solicitado, para garantir a consistência entre as réplicas da base de dados, superando a limitação de falta de consistência observada na Parte 1. Este mecanismo baseia-se na **ordenação total de mensagens**, utilizando **relógios lógicos escalares \(Lamport\)** e uma heurística de desempate. 



**1. Implementação do Relógio Lógico de Lamport** Para permitir a ordenação total, foi introduzido o conceito de **tempo lógico**, com a variável global logicalClock inicializada a zero em cada processo. As regras de atualização do relógio foram aplicadas estritamente durante o envio e a recepção de mensagens: **1.1. No Envio de Mensagens **

Antes de enviar uma mensagem, o relógio lógico é incrementado \(**Regra 2**\). 

O formato da mensagem foi adaptado para incluir esse *timestamp* junto com o **ID do processo** **remetente** e os **dados da operação**. 

● **Formato da mensagem enviada: **

\(myself, logicalClock, op\_data\) 



● Os dados da operação \(op\_data\) continuam sendo as operações de banco de dados definidas na Parte 1: 

**insert, delete, update, query **

****



**1.2. Na Recepção de Mensagens **

Ao receber uma mensagem, o processo segue as **Regras 3a e 3b** para manter a consistência do tempo lógico: 

1. **Atualização para o máximo: **

logicalClock = max\(logicalClock, received\_timestamp\) 



2. **Incremento local: **

Após a atualização, o relógio local é incrementado em 1. 





**2. Mecanismo de Entrega Ordenada \(Fila de Prioridade\)** As mensagens recebidas **não são aplicadas imediatamente**; em vez disso, são armazenadas em uma **fila de prioridade \(deliveryQueue\)**. 

● **Estrutura da Fila: **

A fila armazena tuplas no formato: 

\(received\_timestamp, sender\_id, operation\_data\) 



● **Ordenação: **

A fila é ordenada **estritamente** usando: **1ª chave:** timestamp 

**2ª chave \(desempate\):** sender\_id 



● **Heurística de Desempate: **

Necessária porque dois processos podem gerar mensagens com o mesmo timestamp. 



● **Entrega Final: **

As mensagens só são entregues ao log \(logList\) **após o processo receber a** **notificação de parada** \(stop message = -1\) de **todos os outros processos \(N\)**. 

Isso garante que **todas as mensagens candidatas** à ordenação total estejam presentes antes da entrega final. 





**3. Verificação de Eficácia no Comparison Server \(PT3\)** O Comparison Server na Parte 3 \(comparisonServerPT3\) foi ajustado para medir a eficácia da ordenação total e confirmar a consistência das réplicas. 

● **Log Esperado: **

O servidor espera receber um log de cada peer, cujo tamanho deve ser: **N × N\_MSGS **

****

● **Processo de Comparação: **

A lógica compara a *j-ésima* mensagem do **Peer 0** com a *j-ésima* de todos os outros peers. 



● **Resultado da Consistência: **

A variável unordered conta quantas posições diferem. 



○ **Consistência Garantida:** unordered == 0 → *“Todos os Peers têm a mesma ordem total de* *mensagens” *

* *

○ **Inconsistência Detectada: **

unordered > 0 → existe divergência. 





A implementação do relógio lógico e da fila de entrega ordenada busca demonstrar que, ao impor uma **ordem global estrita \(ordenação total\)**, é possível evitar as divergências de log que ocorriam devido à perda e desordem inerentes ao uso de **UDP sem coordenação** na Parte 1. 





# Document Outline

+ Descrição das Modificações e Evoluções Implementadas no Sistema Distribuído \(Parte 3\)   
	+ Mecanismo de Coordenação Baseado em Ordenação Total  
	+ 1. Implementação do Relógio Lógico de Lamport   
		+ 1.1. No Envio de Mensagens  
		+ 1.2. Na Recepção de Mensagens  

	+ 2. Mecanismo de Entrega Ordenada \(Fila de Prioridade\)  
	+ 3. Verificação de Eficácia no Comparison Server \(PT3\)


