## NGINX WORDPRESS SUBDIRECTORY EXAMPLE

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;

    index index.html index.htm index.php;

    server_name 127.0.0.1;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location /yogyakarta {
        alias /var/www/yogyakarta/wordpress;
        try_files $uri $uri/ @yogyakarta;

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }
    }

    location /surabaya {
        alias /var/www/surabaya/wordpress;
        try_files $uri $uri/ @surabaya;

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }
    }

    location @yogyakarta {
        rewrite /yogyakarta/(.*)$ /yogyakarta/index.php?/$1 last;
    }

    location @surabaya {
        rewrite /surabaya/(.*)$ /surabaya/index.php?/$1 last;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    }
}