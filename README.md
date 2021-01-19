Instalando Docker:
apt-get update && curl https://get.docker.com |bash
mkdir /tilabs/site -p

echo "Teste Proxy Reverso" > /tilabs/site/index.html


mkdir /tilabs/conf && mkdir /tilabs/cert/
scp /mnt/c/Users/dmoraes/Files/blog.tilabs.com.br/* root@192.168.1.141:/tilabs/cert
cd /tilabs/cert && cat certificate.crt ca_bundle.crt >> certificate.crt
cd /tilabs/conf

server {
    listen   443;

    ssl    on;
    ssl_certificate    		/etc/ssl/tilabs/certificates.crt;
    ssl_certificate_key		/etc/ssl/tilabs/private.key;

    server_name blog.tilabs.com.br;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_max_temp_file_size 128m;
        proxy_pass http://192.168.1.142:8080;
    }
}

docker container run -d -p 443:443 --mount type=bind,src=/tilabs/conf,dst=/etc/nginx/conf.d --mount type=bind,src=/tilabs/cert/,dst=/etc/ssl/tilabs/ --name proxy_reverso nginx
