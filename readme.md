<!-- Primeiro passo:
    Executar os comandos:
        docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node1 nginx:alpine
        docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node2 nginx:alpine
        docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node3 nginx:alpine
        docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node4 nginx:alpine
        docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node5 nginx:alpine
     depois execute o comando : 
        docker run -d -p 80:80 -v "${PWD}/default.conf:/etc/nginx/conf.d/default.conf" --name loadbalancer --hostname loadbalancer nginx:alpine

Arquivos:
    arquivo nginx.conf foi usado para alterar no log_format main, colocar como "$http_x_real_ip" para mostrar qual seria o ip que seria mostrado no log dos nodes, foi criado um .conf localmente para ser o subistituto do arquivo padrao do nginx

    arquivo default.conf ele configura o upstream, que seria a lista de nodes disponiveis no loadbalancer, ele tambem é configurado para repassar o ip real da requisição para os nodes usando o proxy_set_header X-Real-IP $remote_addr;

    asquico default.conf no diretorio defaultNodes:
        foi adicionado esse arquivo pois no default.conf dos nodes era necessario fazer uma alteração para registrar os logs corretos, usando access_log /var/log/nginx/access.log main;
         -->

# Atividade: Configuração de Load Balancer com Docker e NGINX

## Descrição
Nesta atividade, foi configurado um balanceador de carga utilizando contêineres Docker com NGINX. O objetivo foi criar múltiplos nodes que atuam como servidores web, balanceados por um servidor NGINX configurado como load balancer. Além disso, foi feita uma configuração nos logs para capturar o IP real dos clientes, utilizando o módulo `proxy_set_header`.

## Passo a Passo

### 1. Execução dos Nodes com NGINX
Primeiramente, foi executado o seguinte comando para subir 5 contêineres NGINX, cada um atuando como um node no sistema:

```bash
docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node1 nginx:alpine

docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node2 nginx:alpine

docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node3 nginx:alpine

docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node4 nginx:alpine

docker run -it -d -p 80 -v "${PWD}/build:/usr/share/nginx/html/" -v "${PWD}/defaultNodes/default.conf:/etc/nginx/conf.d/default.conf" -v "${PWD}/nginx.conf:/etc/nginx/nginx.conf" --name node5 nginx:alpine
```

### 2. Configuração do Load Balancer
Após subir os nodes, foi configurado um contêiner NGINX que atuará como o balanceador de carga, utilizando o comando abaixo:

```bash
docker run -d -p 80:80 -v "${PWD}/default.conf:/etc/nginx/conf.d/default.conf" --name loadbalancer --hostname loadbalancer nginx:alpine
```

## 3. Configuração dos Arquivos
### 3.1 Arquivo nginx.conf
O arquivo `nginx.conf` foi alterado para modificar o formato dos logs, adicionando a captura do IP real dos clientes. Isso foi feito modificando a diretiva `log_format` para incluir `"$http_x_real_ip"`, que exibe o IP real nos logs dos nodes.

```bash
log_format main '$http_x_real_ip - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```

### 3.2 Arquivo `default.conf`
O arquivo `default.conf` contém a configuração do `upstream`, que define a lista de nodes disponíveis para o load balancer. Além disso, o arquivo foi configurado para repassar o IP real da requisição para os nodes utilizando a diretiva `proxy_set_header`.

```bash
upstream loadbalancer {
    server 172.17.0.2;
    server 172.17.0.3;
    server 172.17.0.4;
    server 172.17.0.5;
    server 172.17.0.6;
}
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://loadbalancer;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    access_log /var/log/nginx/nginx-access_log main;
}
```

### 3.3 Arquivo `default.conf` no Diretório `defaultNodes`
Nos nodes, o arquivo default.conf foi configurado para registrar os logs corretamente, utilizando a seguinte diretiva de log:

```bash
access_log /var/log/nginx/access.log main;
```