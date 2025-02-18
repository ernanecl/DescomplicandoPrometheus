### Criar um diretório com o nome tipos_de_dados com um artigo explicando os tipos de dados que vocês aprenderam.

Para acessar o conteudo original desse material, acesse o link `https://prometheus.io/docs/tutorials/understanding_metric_types/`.

#### Tipos de métricas

O `Prometheus` suporta 4 tipos de métricas, elas são `Counter` - `Gauge` - `Histogram` - `Summary`.

&nbsp;

#### Counter

`Counter` é um tipo de métrica que pode apenas incrementar ou resetar, o valor não pode voltar para o valor anterior.

Ele pode ser usado por metricas como o número de requisições, quantidade de erros, etc.

&nbsp;

**_Exemplo de `query` do tipo `counter`_**

```PROMQL
go_gc_duration_seconds_count
```

Uma função interessante que podemos usar com o tipo de métrica `counter`, é o `rate()` no `PromQL`.
A funcao pega o histórico de métricas em um período de tempo e calcula o quão rápido o valor está aumentando por segundo.

`rate` é aplicável somente em valores de counter.

&nbsp;

**_Um exemplo de `query counter` com a função `rate()`_**

```PROMQL
rate(go_gc_duration_seconds_count[5m])
```

&nbsp;
&nbsp;

#### Gauge

`Gauge` é um número que pode aumentar ou diminuir. Ele pode ser usado por `metricas` como números de `pods` em um `cluster`, o número de eventos em uma fila, etc.

No `PromQL`, as funções como `max_over_time`, `min_over_time` e `avg_over_time`, podem ser usadas nas métricas `gauge`.

&nbsp;
&nbsp;

#### Histogram

`Histogram` é a métrica mais complexa quando comparada com as duas anteriores. `Histogram` pode ser usada para calcular qualquer tipo de valor que é contada com base nos valores de intervalo.

Limites de `bucket` podem ser configurados pelo `desenvolvedor`. Um exemplo comum seria o tempo que leva para responder a uma solicitação, chamado `latência`.

&nbsp;

**_Exemplo:_**

Vamos supor que queremos observar o tempo gasto para processar solicitações de `API`.

Em vez de armazenar o tempo de solicitação para cada solicitação, o `histogram` permite armazená-los em `buckets`.

Definimos `buckets` para o tempo gasto, por exemplo, `le` 0,3, `le` 0,5, `le` 0,7, `le` 1 e `le` 1,2.

- `le` = `less or equal`

Então, esses são nossos `buckets` e, uma vez que o tempo gasto para uma solicitação é calculado, ele é adicionado à contagem de todos os `buckets` cujos limites de `bucket` são maiores que o valor medido.

&nbsp;

**_Exemplo de `query Histogram` - `bucket`_**

    prometheus_http_request_duration_seconds_bucket{handler="/graph"}

&nbsp;
&nbsp;

#### Summary

`Summary` também medem `eventos` e são uma alternativa aos `histogram`, eles são mais baratos, mas perdem mais `dados`.

Eles são calculados no nível do `aplicativo`, portanto, a agregação de `métricas` de várias instâncias no mesmo processo não é possível.

Eles são usados ​​quando os `buckets` de uma métrica não são conhecidos de antemão, mas é altamente recomendável usar `histogram` em vez de `summary` sempre que possível.