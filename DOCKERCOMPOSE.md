# Básico sobre Docker-compose
Resumo e lembretes sobre como criar arquivos docker-compose.yml.

## Links Úteis
- Docker-compose vs dockerfile: https://cursos.alura.com.br/forum/topico-diferenca-entre-o-dockerfile-e-o-docker-compose-30250

## Introdução
Docker-compose é uma ferramenta útil para definir e executar uma aplicação que utilize vários container simultaneamente. Sem o docker-compose, caso nossa aplicação utilize múltiplos container (banco, business, frontend etc..), teríamos que executar um por um estabelecendo os links, para preparar nosso ambiente. Com o docker-compose, podemos resumir isso à um único arquivo.

## Arquivo
A configuração do arquivo docker-compose.yml é bem similar aos comandos que utilizamos ao efetuar o `docker run`. Veja por exemplo:

```dockerfile
version: '2'

services:
    db:
        image: mysql:5.7
        volumes:
            - db_data:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: somewordpress
            MYSQL_DATABASE: wordpress
            MYSQL_USER: root
            MYSQL_PASSWORD: root
        
    wordpress:
        depends_on:
            - db
        image: wordpress:latest
        ports:
            - "8000:80"
        restart: always
        environment:
            WORDPRESS_DB_HOST: db:3306
            WORDPRESS_DB_USER: root
            WORDPRESS_DB_PASSWORD: root
volumes:
    db_data:
```

**Observação: ** O Docker-compose trabalha com indentação, é importante que as tags estejam corretamente alinhadas.

- `version`: Versão do arquivo
- `services`: Containers que serão criados
- `db/wordpress`: Nome dos containers
- `image`: Docker image
- `volumes`: Mapeamento do volume
- `restart`: Caso algum problema ocorra, o container seja sempre reiniciado
- `environment`: Variáveis de ambiente que serão criadas no container
- `depends_on`: Indica que o container depende de outro para estar ativo
- `ports`: Mapeamento das portas


## Executar arquivo yml
Para criar os containers, devemos abrir o terminal em uma pasta contendo o arquivo yml, utilizar o utilitário `docker-compose` com os comandos `up` e/ou `down`.
```shell
docker-compose up -d
```