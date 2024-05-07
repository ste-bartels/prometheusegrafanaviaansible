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

O arquivo Inventory possui os hosts específicos de cada etapa, com o processo de acesso via SSH.

O playbook executa as tasks de forma sequencial, acionando as roles.

As roles apresentam de forma individualizada cada instalação, contendo as tasks, templates (arquivos de configuração de referência), vars (variáveis) e handlers (tasks executadas apenas se notificadas).

## Servidor

A instalação foi realizada inteiramente via Ansible, incluindo Prometheus, Grafana e Node-exporter, para monitoramento da própria máquina do servidor.

## Cliente

Para configurar os exporters, foi necessário modificar o docker-compose adicionando o mapeamento de portas do MySQL e RabbitMQ. Também foram adicionadas regras de criação de usuário para o MySQL-exporter e liberação de permissões no script create-database.sql. Essas alterações foram feitas para garantir que o usuário do exporter fosse criado sempre que o container reiniciasse.

Com exceção do Docker-exporter, todas as instalações foram feitas baixando o pacote via repositório web, seguido pela descompactação, ajustes e criação de arquivos de serviço, permitindo a administração do serviço utilizando o systemd. Para o Docker, que por padrão permite a exposição de métricas, foi necessário criar o arquivo daemon.json em /etc/docker, reiniciando o serviço para ler as novas configurações.

### Grafana

Grafana
Ajustes foram feitos nos dashboards para selecionar as métricas corretas, já que nem todos os dashboards padrão contemplavam as métricas fornecidas pelos exporters.

Foram criados 5 dashboards:

1. **Containers Stats:** Apresenta informações básicas sobre o status dos containers.

2. **Full Server Status:** Contém 2 dashboards (sendo necessário selecionar o node-exporter no job e qual host - servidor ou cliente - visualizar).

3. **MySQL Exporter Dashboard:** Apresenta uma coleção completa de métricas do MySQL.

4. **RabbitMQ Metrics:** Exibe métricas relacionadas às filas e status do RabbitMQ.


### Utilizando o projeto

1. Faça o clone do repositório:

```
https://github.com/ste-bartels/prometheusegrafanaviaansible.git
```

2. Altere o arquivo `inventory.ini`, modifique o endereço IP das máquinas e o caminho da sua chave privada, certifique-se de selecionar a role desejada para cada host.

3. Modifique o arquivo `ansible.cfg`, informando o seu usuário de serviço para execução do projeto. 

4. Execute o playbook com o comando a seguir:
```
ansible-playbook -i inventory.ini playbook.yml
```
