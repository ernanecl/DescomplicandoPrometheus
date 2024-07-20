## 3- Criar um diretório com o nome conf com todos os arquivos necessários para a configuração
Todos os arquivos para as configurações conseguimos através do download do site

    https://prometheus.io/download/

### Lista e descrição de todos os diretórios e arquivos necessários para o uso do Prometheus

    # arquivos binários movemos para /usr/local/bin/ (não precisamos modificar)
    - promtool
    - prometheus



    # o arquivo prometheus.yml copiamos para /etc/prometheus/
    - prometheus.yml

    # conteúdo no arquivo
    global:
        scrape_interval: 15s
        evaluation_interval: 15s
        scrape_timeout: 10s

    rule_files:

    scrape_configs:
        - job_name: "prometheus"
          static_configs: 
            - targets: ["localhost:9090"]


    
    # também precisamos colocar os diretórios consoles e consoles_libraries para /etc/prometheus/ (também não precisamos alterar)
    - consoles
    - consoles_libraries



    # precisamos criar um arquivo no /etc/systemd/system/prometheus.service
    - prometheus.service

    # conteúdo do arquivo
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
