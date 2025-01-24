# Laravel com Docker - Guia Completo

Este guia descreve como configurar e usar o Docker para desenvolver e implantar uma aplicação Laravel. Seguindo este passo a passo, você terá um ambiente isolado, escalável e pronto para produção.

---

## Requisitos

- [Docker](https://www.docker.com/products/docker-desktop) instalado
- Conhecimento básico de Laravel

Verifique a instalação do Docker:
```bash
docker --version
docker-compose --version
```

---

## Estrutura do Projeto

1. **`Dockerfile`**: Configura o ambiente PHP.
2. **`docker-compose.yml`**: Orquestra os serviços (PHP, MySQL, Nginx).
3. **`nginx.conf`**: Configuração do Nginx.

---

## Passo 1: Criar o Arquivo `Dockerfile`

Crie um arquivo `Dockerfile` na raiz do projeto Laravel.

```dockerfile
# Use a imagem oficial do PHP com suporte para Laravel
FROM php:8.2-fpm

# Instale dependências do sistema
RUN apt-get update && apt-get install -y \
    curl zip unzip git \
    libpq-dev libzip-dev libonig-dev \
    && docker-php-ext-install pdo pdo_mysql zip

# Instale o Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Copie os arquivos do projeto para o container
WORKDIR /var/www
COPY . /var/www

# Defina permissões e instale dependências do Laravel
RUN chown -R www-data:www-data /var/www \
    && composer install --no-dev --optimize-autoloader

# Exponha a porta do PHP-FPM
EXPOSE 9000

CMD ["php-fpm"]
```

---

## Passo 2: Criar o Arquivo `docker-compose.yml`

O arquivo `docker-compose.yml` define os serviços necessários.

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: laravel_app
    container_name: laravel_app
    volumes:
      - .:/var/www
      - ./storage:/var/www/storage
    networks:
      - laravel
    ports:
      - "9000:9000"

  mysql:
    image: mysql:8
    container_name: laravel_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
      MYSQL_USER: laravel
      MYSQL_PASSWORD: secret
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - laravel
    ports:
      - "3306:3306"

  nginx:
    image: nginx:latest
    container_name: laravel_nginx
    volumes:
      - .:/var/www
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    networks:
      - laravel
    ports:
      - "8080:80"

networks:
  laravel:

volumes:
  mysql_data:
```

---

## Passo 3: Configurar o Nginx

Crie o arquivo `nginx.conf` na raiz do projeto:

```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass app:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_index index.php;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

---

## Passo 4: Configurar o Laravel

1. Atualize o arquivo `.env` para configurar o banco de dados:

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret
```

2. Dê permissão para as pastas:

```bash
chmod -R 777 storage bootstrap/cache
```

---

## Passo 5: Subir os Containers

1. **Construir e iniciar os containers:**
   ```bash
   docker-compose up --build -d
   ```

2. **Verificar os containers em execução:**
   ```bash
   docker ps
   ```

3. **Acessar a aplicação Laravel:**
   Abra o navegador em [http://localhost:8080](http://localhost:8080).

---

## Passo 6: Comandos Úteis

1. **Acessar o container do app:**
   ```bash
   docker exec -it laravel_app bash
   ```

2. **Executar comandos do Artisan:**
   ```bash
   docker exec -it laravel_app php artisan migrate
   ```

3. **Reiniciar os containers:**
   ```bash
   docker-compose restart
   ```

4. **Parar os containers:**
   ```bash
   docker-compose down
   ```

---

## Passo 7: Otimizações para Produção

1. Ative cache no Laravel:
   ```bash
   php artisan config:cache
   php artisan route:cache
   php artisan view:cache
   ```

2. Configure HTTPS com certificados SSL.

3. Use Kubernetes para orquestração em produção.

---

## Conclusão

Agora você tem um ambiente Laravel totalmente funcional usando Docker. Esta configuração suporta tanto desenvolvimento quanto produção, garantindo consistência, escalabilidade e facilidade de manutenção. Para dúvidas ou melhorias, contribua com este guia!

