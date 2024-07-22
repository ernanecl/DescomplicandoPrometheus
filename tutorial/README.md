### Criar um diretório chamado tutorial com o tutorial de como instalar o Prometheus no Linux.

#### Download e extracao do Prometheus

Acesse o site oficial do Prometheus `https://prometheus.io/download/`.

Copie o link correspondente ao sistema e execute o seguinte comando no `terminal`.

```BASH
sudo curl -LO https://github.com/prometheus/prometheus/releases/download/v2.51.1/prometheus-2.51.1.linux-amd64.tar.gz
```

&nbsp;

Caso não tenha o pacote `curl` instalado, atualize o sistema e instale o pacote com os comandos abaixo.
    
```BASH
sudo apt update && sudo apt upgrade
sudo apt install curl
```

Após realizar essa etapa, execute o primeiro comando com `curl` novamente.

&nbsp;

Extraia o pacote usando o comando abaixo.

```BASH
tar -xvzf prometheus-2.51.1.linux-amd64.tar.gz
```

&nbsp;

O comando abaixo podemos verificar a versao do Prometheus.

```BASH
./prometheus --version
```

&nbsp;
&nbsp;

#### Configuração do Prometheus

Depois da extracao, iniciamos a configuracao movendo os arquivos `prometheus` e `promtool` para o diretorio `bin`.
Mover os binarios possibilita executar o `Prometheus` sem a necessidade de usar o comando `./`.

```BASH
sudo mv prometheus-2.51.1.linux-amd64/prometheus /usr/local/bin
sudo mv prometheus-2.51.1.linux-amd64/promtool /usr/local/bin
```
&nbsp;

Crie um diretório de configuração chamado `prometheus` no seguinte caminho `/etc/`.
Copie o arquivo `prometheus.yml` para o diretório criado `/etc/prometheus/`.

```BASH
sudo mkdir /etc/prometheus
sudo cp prometheus-2.51.1.linux-amd64/prometheus.yml /etc/prometheus/
```

&nbsp;

**_No arquivo `prometheus.yml` vamos deixar como no exemplo abaixo._**

```YML
global:
    scrape_interval: 15s
    evaluation_interval: 15s
    scrape_timeout: 10s

rule_files:

scrape_configs:
    - job_name: "prometheus"
        static_configs: 
        - targets: ["localhost:9090"]
```

&nbsp;

No diretório `/etc/prometheus` tambem vamos colocar os diretorios `consoles` e `console_libraries`.

```BASH
sudo mv prometheus-2.51.1.linux-amd64/consoles /etc/prometheus/
sudo mv prometheus-2.51.1.linux-amd64/console_libraries /etc/prometheus/
```

&nbsp;

Precisamos de um diretorio para armazenar os dados que o `Prometheus` vai gerar.
Vamos criar o diretório com o nome `prometheus` dentro do diretório `lib`.

```BASH
sudo mkdir /var/lib/prometheus
```

&nbsp;
&nbsp;

#### Prometheus como servico

Vamos executar o `Prometheus` como serviço e para não fazer o `root` ser responsável pela execução do serviço, criaremos um usuário e grupo para o `Prometheus`.
Para realizar essa tarefa, vamos criar o usuario e grupo com o nome `prometheus`, executantodo os comandos abaixo.

```BASH
sudo addgroup --system prometheus
sudo adduser --shell /sbin/nologin --system --group prometheus
```

&nbsp;

Precisamos criar o `Prometheus` como serviço no `Linux`, o `systemd` gerencia o serviço.
Para isto, precisamos criar um `unit service`, ele é um arquivo que fala para o `systemd` como deve iniciar o `Prometheus`.
Crie o arquivo com o nome `prometheus.service`
    
```BASH
sudo vim /etc/systemd/system/prometheus.service
```

&nbsp;

**_Dentro do arquivo vamos adicionar o conteudo do exemplo abaixo._**

```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

O material `original` pode ser acessado pelo link `https://github.com/badtuxx/DescomplicandoPrometheus/blob/main/pt/src/day-1/files/prometheus.service`.

&nbsp;

Agora essa etapa precisamos alterar o dono e grupo dos diretórios do `Prometheus`.
Use o comando `chown` para realizar a tarefa.

```BASH
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chown -R prometheus:prometheus /usr/local/bin/prometheus
sudo chown -R prometheus:prometheus /usr/local/bin/promtool
```

&nbsp;

Reinicie o `systemctl daemon` para recarregar a configuração do gerenciador `systemd`.

```BASH
sudo systemctl daemon-reload
```
&nbsp;

Inicie e verifique o `status` do `Prometheus`.

```BASH
sudo systemctl start prometheus
sudo systemctl status prometheus
```

&nbsp;

Configurando o `Prometheus` para inicialização automática do sistema.

```BASH
sudo systemctl enable prometheus
```    

&nbsp;

Verificar se está tudo certo atraves dos logs do `Prometheus`.

```BASH
sudo journalctl -u prometheus
```

&nbsp;
&nbsp;

#### Acessando Prometheus e suas metricas

Podemos acessar o `Prometheus` e suas `metricas` pela a interface gráfica pelos links `http://localhost:9090` e `http://localhost:9090/metrics`.

Tambem podemos acessar o `Prometheus` pelo `terminal`, com o seguinte comando.

```BASH
curl http://localhost:9090/metrics
```