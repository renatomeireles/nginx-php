
# PHP e NGINX  com MariaDB
### Versões utilizadas
**Sistema**: [CentOS 7.6.1810](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso)

**Kernel**: [3.10.0-957.1.3.e17.x86_64](https://rpmfind.net/linux/RPM/centos/updates/7.6.1810/x86_64/Packages/kernel-3.10.0-957.1.3.el7.x86_64.html)

**Nginx**: [1.12.2](https://nginx.org/en/download.html)

**PHP**:  [5.4.16](http://php.net/releases/5_4_16.php) (cli)

**Banco de dados MariaDB**:  [5.5.60-MariaDB - MariaDB Server](https://mariadb.org/mariadb-5-5-60-now-available/)

---
### Instalando Nginx

Primeiro será necessário instalar o repositório do Nginx.

Basta usar o comando abaixo:

`# yum install epel-release -y`

>*O parametro -y serve para não ter "break" de aceitar opções do tipo "yes" ou "no" e agilizar o processo aceitando todos os questionamentos de instalação.*

Após finalizado a instalação do repositório, é possível instalar o Nginx com o comando:

`# yum install nginx -y`

Iniciando serviço:

`# systemctl start nginx.service`

Conferindo status do serviço:

`# service nginx status`

Configurando Nginx para inicializar junto com o sistema:

`# systemctl enable nginx.service`

Liberando firewall para Nginx usar a porta padrão 80:

`# firewall-cmd --permanent --zone=public --add-service=http`

Reiniciando o firewall para carregar as novas configurações:

`# firewall-cmd --reload`

### Instalando banco de dados MariaDB

Usar comando:

`# yum install mariadb-server mariadb -y`

Iniciando serviço do MariaDB:

`# systemctl start mariadb.service`

Configurando MariaDB para inicializar junto com sistema:

`# systemctl enable mariadb.service`

Configuração de segurança:

`# mysql_secure_installation`

Na primeira opção de questionamento "Enter current password for root (enter for none)", apenas aperte "Enter". Logo após será questionado sobre criar uma nova senha "Set root password?", aperte "Y" e "Enter" e crie uma senha root para o banco de dados.
Nos próximos questionamentos, apenas escolha "y" em todas. Será removido usuários anônimos, acesso root remoto, base de dados de teste, e será recarregado os privilégios setados.

### Instalando PHP

Instalando módulos necessários do PHP:

`# yum install php php-mysql php-fpm php-apc -y`


Configurando arquivo php.ini:

`# vi /etc/php.ini`

Use a "/" para buscar a linha escrito "cgi.fix_pathinfo". Na linha escrito "cgi.fix_pathinfo=1", remova o ";" do início e altere o valor para "0" e ele será forçado a desabilitar a opção. Após esta alteração o PHP só irá executar os arquivos solicitados.

Configurando arquivo "www.conf" para "escutar" o arquivo socket do php-fpm:

`# vi /etc/php-fpm.d/www.conf`

Altere a linha `listen = 127.0.0.1:900` para buscar o path do socket `listen = /var/run/php-fpm/php-fpm.sock`

Iniciando serviço do php-fpm:

`# systemctl start php-fpm`

Configurando php-fpm para inicializar junto com sistema:

`# systemctl enable php-fpm.service`

Conferindo status do php-fpm:

`# service php-fpm status`

>**Caso seja necessário utilizar outros módulos do php, segue uma lista:**
>
>php-bcmath - A module for PHP applications for using the bcmath library

>php-cli - Command-line interface for PHP

>php-common - Common files for PHP

>php-dba - A database abstraction layer module for PHP applications

>php-devel - Files needed for building PHP extensions

>php-embedded - PHP library for embedding in applications

>php-enchant - Enchant spelling extension for PHP applications

>php-fpm - PHP FastCGI Process Manager

>php-gd - A module for PHP applications for using the gd graphics library

>php-imap - A module for PHP applications that use IMAP

>php-intl - Internationalization extension for PHP applications

>php-ldap - A module for PHP applications that use LDAP

>php-mbstring - A module for PHP applications which need multi-byte string handling

>php-mcrypt - Standard PHP module provides mcrypt library support

>php-mysql - A module for PHP applications that use MySQL databases

>php-mysqlnd - A module for PHP applications that use MySQL databases

>php-odbc - A module for PHP applications that use ODBC databases

>php-pdo - A database access abstraction module for PHP applications

>php-pear.noarch - PHP Extension and Application Repository framework

>php-pecl-memcache - Extension to work with the Memcached caching daemon

>php-pgsql - A PostgreSQL database module for PHP

>php-process - Modules for PHP script using system process interfaces

>php-pspell - A module for PHP applications for using pspell interfaces

>php-recode - A module for PHP applications for using the recode library

>php-snmp - A module for PHP applications that query SNMP-managed devices

>php-soap - A module for PHP applications that use the SOAP protocol

>php-xml - A module for PHP applications which use XML

>php-xmlrpc - A module for PHP applications which use the XML-RPC protocol

### Configuração do Nginx:

Comando para verificar quantos núcleos estão disponíveis na maquina servidor:

`# grep -c 'modelname' /proc/cpuinfo`

Configurando a quantidade de núcleos de processamento do Nginx:

`# vi /etc/nginx/nginx.conf`

Altere a linha `worker_Processes auto;` para o número disponível de processador, exemplo "worker_Processes 2;".

No bloco "http" habilite o gzip utilizando o script abaixo:

`gzip on;
gzip_disable "MSIE [1-6]\.(?!.*SV1)";
gzip_http_version 1.1;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript text/x-js;`

>O gzip serve para realizar uma compressão dos arquivos para agilizar a entrega para os navegadores.


Inserir uma nova linha de include:

`include /etc/nginx/sites-available/*.conf;`

Criar diretório caso ele não exista:

`# vi /etc/nginx/sites-available/exemplo.conf`

Inserrir estes dados como conteúdo do arquivo "exemplo.conf":

    server {
        listen 80;
        server_name exemplo.com www.exemplo.com;
    
        access_log /var/log/nginx/exemplo-access.log;
        error_log /var/log/nginx/exemplo-error error;
    
        root /var/www/html/optimusexemplo.com;
        index  index.html index.php;
    
        location / {
            try_files $uri $uri/ /index.php?$args;
        }
    
        error_page 403 =404;
        location ~ \.php$ {
            root           /var/www/html/exemplo.com;
    #       fastcgi_pass   127.0.0.1:9000;
            fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }

Navegar até a pasta host do servidor para criar o diretório padrão do site:

`# cd /var/www/html`

Criar diretório:

`# mkdir exemplo.com`

Acessar diretório:

`# cd exemplo.com`

Criar arquivo index de teste:

`# vi index.php`

Insira o código abaixo para testar:

`<?php php_info(); ?>`

Verificando se serviço do Nginx está ok:

`# nginx -t`

Caso retorne a mensagem "test is successful", reinicie o serviço:

`# systemctl restart nginx.service`

Verificando se serviço continua ok:

`# service nginx status`

>Caso esteja local, configure o endereço de host de sua maquina para responder ao IP do servidor. Exemplo: 192.168.1.100 exemplo.com
Necessário privilégios administrativos para alteração do arquivo host.
Para windows o arquivo "hosts" fica geralmente no caminho "C:/windows/system32/drivers/etc". O disco local pode variar de acordo com a instalação do sistema.
Para linux o arquivo "hosts" fica no caminho "/etc/hosts"
Para MAC OS o arquivo "hosts" no caminho "/private/etc/hosts"

Acessando o endereço "exemplo.com" no navegador, será exibido a página index criada no diretório "/var/www/html/exemplo.com".

Alterando o usuário de acesso a pasta do site de "root" para "nginx".

`# chown -R nginx:nginx exemplo.com`

Verificando usuário atual de acesso a pasta:

`# ls -l`

O usuário de acesso foi alterado para "nginx".

### Instalando PhpMyAdmin


Instalando o pacote do phpmyadmin:

`# yum install phpmyadmin -y`

Navegar até a pasta do site criada anteriormente:

`# cd /var/www/html/exemplo.com`

Criar link para acessar o phpMyAdmin:

`# ln -s /user/share/phpMyAdmin/ phpmyadmin`

Reinicie os serviços do nginx e php-fpm com os seguintes comandos:

`# service nginx restart`

`# service php-fpm restart`

Utilizando o comando `# ls` é possível ver um atalho da pasta criada. Agora é possível acessar o phpmyadmin no endereço "exemplo.com/phpmyadmin" com usuário root e a senha criada na instalação do MariaDB.

A partir deste momento o servidor está configurado e é possível instalar CMS, ou enviar arquivos de um site qualquer em PHP.

>Caso seja necessário desativar o sistema SELINUX no servidor acesse o diretório `# cd /etc/selinux` e acesse o arquivo "config" com o comando `# vi config` e altere a linha `SELINUX=enforcing` para `SELINUX=disabled`, precione "esc", digite ":wq" para salvar.

>**Recomendação:** Ao criar uma nova base de dados no banco, criar também um usuário específico para acesso ao mesmo e evitar utilização do usuário root.
Para isto, basta selecionar a base criada e clicar em "Privilégios" depois "Adicionar utilizador". Preencha os dados de usuário e marque a opção "Grant all privileges on database"
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ4MTI4MjY0MCwtMjE5MTU2OTEwXX0=
-->