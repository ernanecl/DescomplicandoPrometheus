### 2- Criar um diretório chamado tutorial com o tutorial de como instalar o Prometheus no Linux.
### Download e extracao do Prometheus
#### acesse o site oficial do Prometheus
    
    https://prometheus.io/download/

#### após o acesso, copie o link correspondente ao sistema e execute o seguinte comando no terminal

    sudo curl -LO https://github.com/prometheus/prometheus/releases/download/v2.51.1/prometheus-2.51.1.linux-amd64.tar.gz


#### se o pacote "curl" não está instalado, atualize o sistema e instale o pacote
    
    sudo apt update && sudo apt upgrade
    sudo apt install curl

apos realizar essa etapa, execute o comando curl novamente


#### extraia o pacote usando o comando abaixo

    tar -xvzf prometheus-2.51.1.linux-amd64.tar.gz


#### com o comando abaixo, podemos verificar a versao do Prometheus

    ./prometheus --version


### Configuracao do Prometheus
#### depois da extracao, iniciamos a configuracao movendo os arquivos "prometheus" and "promtool" para o diretorio "bin"
#### mover os binarios possibilita executar o Prometheus sem a necessidade de usar o comando "./"

    sudo mv prometheus-2.51.1.linux-amd64/prometheus /usr/local/bin
    sudo mv prometheus-2.51.1.linux-amd64/promtool /usr/local/bin


#### crie um diretório de configuração chamado prometheus no seguinte caminho "/etc/"
#### copie o arquivo "prometheus.yml" para o diretório criado (/etc/prometheus/)

    sudo mkdir /etc/prometheus
    sudo cp prometheus-2.51.1.linux-amd64/prometheus.yml /etc/prometheus/


#### o conteúdo do arquivo prometheus.yml

    global:
        scrape_interval: 15s
        evaluation_interval: 15s
        scrape_timeout: 10s

    rule_files:

    scrape_configs:
        - job_name: "prometheus"
          static_configs: 
            - targets: ["localhost:9090"]


#### mover os diretórios "consoles" e "console_libraries" para "/etc/prometheus/"
    
    sudo mv prometheus-2.51.1.linux-amd64/consoles /etc/prometheus/
    sudo mv prometheus-2.51.1.linux-amd64/console_libraries /etc/prometheus/


#### precisamos de um diretorio para armazenar os dados que o Prometheus gera
#### criamos o diretório do prometheus no diretório "lib"

    sudo mkdir /var/lib/prometheus


#### para executar o Prometheus como serviço e não fazer o root ser responsável pela execução do serviço, criaremos um usuário e grupo para o Prometheus
#### para criar um usuário e adicioná-lo ao grupo exclusivo para o Prometheus, execute os comandos abaixo

    sudo addgroup --system prometheus
    sudo adduser --shell /sbin/nologin --system --group prometheus


#### precisamos criar o Prometheus como serviço, no Linux, o "systemd" gerencia o serviço
#### para isto, precisamos criar um "unit service", ele é um arquivo que fala para o "systemd" como deve iniciar o Prometheus
#### crie o arquico com o nome prometheus.service
    
    sudo vim /etc/systemd/system/prometheus.service


#### dentro do arquivo adicionamos o conteúdo abaixo e salvamos

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

##### o material original pode ser acessado pelo link abaixo

    https://github.com/badtuxx/DescomplicandoPrometheus/blob/main/pt/src/day-1/files/prometheus.service


#### essa etapa precisamos alterar o dono e grupo dos diretórios do Prometheus
#### use o comando "chown" para realizar a tarefa

    sudo chown -R prometheus:prometheus /etc/prometheus
    sudo chown -R prometheus:prometheus /var/lib/prometheus
    sudo chown -R prometheus:prometheus /usr/local/bin/prometheus
    sudo chown -R prometheus:prometheus /usr/local/bin/promtool


#### reinicie o "systemctl daemon" para recarregar a configuração do gerenciador "systemd"

    sudo systemctl daemon-reload


#### inicie e verifique o status do Prometheus

    sudo systemctl start prometheus
    sudo systemctl status prometheus


#### configurando o Prometheus para inicialização automática do sistema

    sudo systemctl enable prometheus
    

#### verificar se está tudo certo nos logs do Prometheus

    sudo journalctl -u prometheus


#### acesse a interface gráfica e as métricas do Prometheus através do navegador

    http://localhost:9090
    http://localhost:9090/metrics
