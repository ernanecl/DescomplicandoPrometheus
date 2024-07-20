### 4- Criar um diretório com o nome tipos_de_dados com um artigo explicando os tipos de dados que vocês aprenderam.

#### material original:

    https://prometheus.io/docs/tutorials/understanding_metric_types/

#### Tipos de métricas
O Prometheus suporta 4 tipos de métricas, elas são Counter - Gauge - Histogram - Summary



#### Counter
Counter é um tipo de métrica que pode apenas incrementar ou resetar, o valor não pode voltar para o valor anterior.

Ele pode ser usado por metricas como o número de requisições, quantidade de erros, etc.

##### Um exemplo de query do tipo count

    go_gc_duration_seconds_count

Uma função interessante que podemos usar com o tipo de métrica é o rate() no PromQL,
ele pega o histórico de métricas em um período de tempo e calcula o quão rápido o valor está aumentando por segundo.

Rate é aplicável somente em valores de counter.

##### Outro exemplo de query do tipo count com a função rate()

    rate(go_gc_duration_seconds_count[5m])



#### Gauge
Gauge is um número que pode aumentar ou diminuir. Ele pode ser usado por metricas como números de pods em um cluster, o número de eventos em uma fila, etc.


No PromQL, as funções como max_over_time, min_over_time e avg_over_time, podem ser usadas nas métricas gauge.



#### Histogram
Histogram é a métrica mais complexa quando comparada com as duas anteriores. Histogram pode ser usada para calcular qualquer tipo de valor que é contada com base nos valores de intervalo.