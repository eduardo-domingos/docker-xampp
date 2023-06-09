# Descrição
Ambiente com foco em desenvolvimento Web com PHP

## Dockerfile e docker-compose
- Apache   2.4.5
- PHP      8.x (pode ser alterado no arquivo)
- Composer 2.5.7
- MariaDB  10.9

## Recursos
- SSL 
- Reescrita de URL
- .htaccess
- Strict Mode para banco de dados Desabilitado
- Porta 443 com SSL
- Porta 80 sem SSL

## Volumes compartilhados docker-compose

- Todos os arquivos de configurações (php.ini, certificado etc..) ficam dentro da pasta docker-conf

* ./:/var/www/html 
    * tem relação com o WORKDIR do Dockerfile, será a pasta a onde ficarão os arquivos do projeto

    #

* ./conf/php/php.ini:/etc/php/${php}/apache2/php.ini
    * é um arquivo de php.ini modificado

    #

* ./conf/ssl/localhost.key:/etc/ssl/ca/localhost.key
* ./conf/ssl/localhost.crt:/etc/ssl/ca/localhost.crt
    * arquivos para o SSL (certificado autoassinado), esse diretório é referenciado no arquivo de virtualhost.conf

    #

* ./conf/apache/virtualhost.conf:/etc/apache2/sites-enabled/000-default.conf
    * configuração do virtualhost (portas 80 e 443, SSL, reescrita de URL, .htaccess)

    #

* ./conf/db:/var/lib/mysql
   * persistência do dados do banco

   #

## Configurações personalizadas php.ini

- O ideal é copiar php.ini de dentro do container, criar um backup, e cria um novo arquivo com as alterações abaixo. Tanto o backup quanto o novo arquivo devem permanecer na pasta conf/php, com nomes diferentes para não gerar problema, no arquivo docker-compose.yml essa pasta está mapeada como volume compartilhado, assim é "refletido" as alterações dentro do container

~~~
cp /etc/php/version.php/apache2/php.ini ./
~~~

- memory_limit=512M
- error_reporting=E_ALL
- date.timezone = America/Sao_Paulo
- allow_url_fopen = Off
- max_execution_time = 5
- post_max_size = 25M
- max_input_nesting_level = 64
- file_uploads = On
- upload_max_filesize = 8M
- max_file_uploads = 3
- output_buffering = 4096
- realpath_cache_size = 4096k
- realpath_cache_ttl = 120
- track_errors = Off
- allow_url_fopen = On
- allow_url_include = Off

## Executar Ambiente Docker

- Criar as pastas "db" e "php" dentro da pasta "conf", criar a pasta "www" fora da pasta "conf"

- Quando for executar um projeto php, o mesmo precisa estar dentro da pasta www

- É necessário criar um .env dentro da pasta docker-environment (siga o .env-example como exemplo) ali fica todas as variáveis de ambiente do projeto, seja do container de php e/ou do mysql/mariadb

- Se estiver no Windows é necessário importar o certificado "conf/ssl/server.crt" com "autoridade certificadora de raiz confiável"

- Se estiver em alguma distribuição Linux, é necessário importar o certificado "conf/ssl/server.pem" no navegador (chrome, firefox etc...)

- Todas as configurações dos containers estão centralizados no arquivo docker-compose.yml, assim só é necessário executar o comando abaixo:

~~~~
docker compose up -d
~~~~

- Caso a imagem do php precise de algum ajuste, basta editar o arquivo Dockerfile, para gerar uma nova imagem e executar o seguinte comando:

~~~
docker compose up -d --build
~~~

- Caso queira trocar a versão do php do projeto basta acessar o .env e alterar a variável "php=5.6" para "php=8.2" ou a versão desejada, e executar o comando abaxio:

~~~
docker compose up -d --build
~~~

- Caso vá criar um arquivo php.ini personalizado a partir do php.ini de dentro do container, comente a linha que cria um volume compartilhado para o arquivo php.ini (docker-compose.yml), após acessar o container, faça um cópia e altere e depois de descomente do arquivo docker-compose.yml, será necessário fazer o build o projeto novamente