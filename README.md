# prometheusegrafanaviaansible
Instalação do prometheus, exporters e grafana via ansible

Foram recebidas 2 maquinas virtuais Servidor e Cliente, ambas localizadas na GCP.
A Cliente possue 4 containers em funcionamento ( mysql, rabbit, phyton e node)
Como solicitado foi intalado no Servidor o Prometheus, Grafana e o node-exporter e no cliente foram instalados os exporters do node, do Mysql, do rabbit e do docker. tudo isso via Ansible. Gerando 5 dashboards no Grafana com as respectivas métricas. em um prazo de 4 dias, começando a contar da sexta-feira (03/05/2024) a tarde.

Como relatado, parte desse desafio já havia sido realizado posteriormente e com isso um conhecimento e documentação previa.
Por se tratar de um "desafio", me propus a automatizar via ansible já que não o tinha realizado dessa forma. 
No sabádo pela manhã já tinha finalizado a parte que tinha conhecimento previo, Prometheus, Grafana e Node-exporter, percebendo que ainda havia tempo hábil mantive o desenvolvimento via ansible.
  
O ansible foi construido, seguindo as boas práticas utilizando as roles de forma hierarquica, com a estrutura a seguir:

|inventory
|playbook
|roler
    |-prometheus 
    |       |-tasks
    |       |     |-main.yml
    |       |-templates
    |       |     |-config.j2
    |       |-vars
    |       |-handlers
    |
    |-grafana
    |       |-...
    |
    |- prometheus_node_exporter
    |                      |-...
    |
    |- prometheus_mysql_exporter  
    |                      |-...
    |
    |- prometheus_rabbit_exporter
    |                      |-...
    |
    |- prometheus_docker_exporter
                       |-...

Invetory possue os hosts, expecifico de cada step, com o processo de acesso via ssh.

Playbook executa as task (funções) de forma seguimentada, através acionamento das roles.

Roles apresenta de forma individualizada cada instalação, contendo as tasks, templates (arquivos de configurações de referencia), vars (variavéis) e handles (tasks que são executadas apenas se forem notificadas).

Servidor
O processo de instalação foi toda realizada via ansible, com instalação do prometheus, Grafana e node-exporter (para monitoramento da propria maquina de servidor).

Cliente
Para a realização das configurações dos exporters foi necessário a auteração do docker-compose adicionando o mapeamento de portas do Mysql e Rabbit. 
Foi necessária a adição das regras de criação de usuário para o Mysql-exporter e a liberação de permissões para o mesmo no script create-database.sql. Essa alteração se fez necessária para que o usuario do exporter fosse criado sempre que o container reiniciasse.

Com exceção do Docker-exporter todas as instalações foram realizadas apartir do dowload do pacote via repositório web, seguindo pela descompactação, ajustes e criação dos arquivos de serviço, pertimindo a administração do serviço utilizando o systemd. Já o docker que permite por padrão a exposição de métricas, foi necessário criar o arquivo daemon.json no /etc/docker. Reiniciando o serviço para a leitura das novas configurações.
  
E alguns dashboard do Grafana o ajuste de quais metricas estavam sendo apresentadas foi realizado, pois nem todos dashboard padrão contemplava as meetricas que o exporter continha.

Foram criados 5 dashboards 
- Containers Stats: Com o básico sobre a o status dos containers
- full server Status: Contem 2 dashboards (sendo necessário selecionar o node-exporter no job e qual host - servidor ou cliente - quer visualizar)
- mysql exporter dashboard : Apresentação uma coleção bem completa de metricas do Mysql
- RabbitMQ metrics : Com as métricas referente as filas e status do mesmo.


  
