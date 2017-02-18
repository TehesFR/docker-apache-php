# About this repo

Development **Drupal** optimized images for apache-php.

Available tags are:
- 7.0, latest ([7.0/Dockerfile](https://github.com/TehesFR/docker-apache-php/blob/master/7.0/Dockerfile))
- 5.6 ([5.6/Dockerfile](https://github.com/TehesFR/docker-apache-php/tree/master/5.6/Dockerfile))

The image basically contains:

- All php libraries needed for Drupal (gd, mbstring, mcrypt, zip, soap, pdo_mysql, mysqli, xsl, opcache, calendar, intl)
- Development tools for Drupal (xdebug, codesniffer, compass, less, node.js, grunt, gulp, composer, drush, phing, phpcpd, phpmetrics)
- Much more...
- The 7.0 image uses a dedicated "web" user. Use sudo if you want to be root inside the container.

# Docker-compose
## Use this docker-compose.yml to create a complete development environment using several custom Docker images:

    version: '2'
    services:
      # web with xdebug - tehes images
      web:
        # tehes/docker-apache-php available tags: latest, 7.0, 5.6
        image: tehes/docker-apache-php:7.0
        ports:
          - "80:80"
          - "9000:9000"
        environment:
          - SERVERNAME=example.local
          - SERVERALIAS=example2.local *.example2.local
          - DOCUMENTROOT=htdocs
        volumes:
          - /home/docker/projets/example/:/var/www/html/
          - /home/docker/.ssh/:/var/www/.ssh/
        links:
          - database:mysql
          - mailhog
          - solr
          - redis
          - tika
        tty: true
        # local IP address of your host in order to use xdebug
        extra_hosts:
          - "docker_host:10.10.10.10"

      # database container - tehes images
      database:
        # tehes/docker-mysql available tags: latest, 5.7
        image: tehes/docker-mysql:5.7
        ports:
          - "3306:3306"
        environment:
          - MYSQL_ROOT_PASSWORD=mysqlroot
          - MYSQL_DATABASE=example
          - MYSQL_USER=example_user
          - MYSQL_PASSWORD=mysqlpwd

      # phpmyadmin container - tehes images
      phpmyadmin:
        image: tehes/docker-phpmyadmin
        ports:
          - "8010:80"
        environment:
          - MYSQL_ROOT_PASSWORD=mysqlroot
          - UPLOAD_SIZE=1G
        links:
          - database:mysql

      # mailhog container - official images
      mailhog:
        image: mailhog/mailhog
        ports:
          - "1025:1025"
          - "8025:8025"

      # solr container - tehes images
      solr:
        # tehes/docker-solr available tags: latest, 6.2, 6.1, 6.0, 5.5, 5.4, 5.3, 5.2, 5.1, 5.0, 4.10, 3.6
        image: tehes/docker-solr:6.2
        ports:
          - "8080:8983"

      # redis container - official images
      redis:
        image: redis:latest
        ports:
          - "6379"

      # phpRedisAdmin container - tehes images
      phpredisadmin:
        image: tehes/docker-phpredisadmin
        ports:
          - "9900:80"
        environment:
          - REDIS_1_HOST=redis
        links:
          - redis

      # Tika server container - tehes images
      tika:
        image: tehes/docker-tika-server
        ports:
          - "9998:9998"

    # ##### PROFILING SECTION - EXPERIMENTAL #####
    #   # Uncomment this block to enable 3 containers for profiling.
    #   # xhprof data will be stored in mongodb and available through the xhgui interface.
    #
    #   # web with xhprof - tehes images
    #   web-prof:
    #     # tehes/docker-apache-php-xhprof available tags: latest, 7.0, 5.6
    #     image: tehes/docker-apache-php-xhprof:7.0
    #     ports:
    #       - "8050:80"
    #     environment:
    #       - SERVERNAME=example.local
    #       - SERVERALIAS=example2.local *.example2.local
    #       - DOCUMENTROOT=htdocs
    #     volumes:
    #       - /home/docker/projets/example/:/var/www/html/
    #       - /home/docker/.ssh/:/var/www/.ssh/
    #     links:
    #       - database:mysql
    #       - mailhog
    #       - solr
    #       - redis
    #       - tika
    #       - mongo
    #     tty: true
    #
    #   # mongo container - official images
    #   mongo:
    #     image: mongo
    #     ports:
    #       - "27017:27017"
    #
    #   # xhgui container - tehes image
    #   xhgui:
    #     image: tehes/docker-xhgui
    #     ports:
    #       - "8040:80"
    #     links:
    #       - mongo
    # ##### END OF PROFILING SECTION #####


[Docker Hub page](https://hub.docker.com/r/tehes/docker-apache-php/)
