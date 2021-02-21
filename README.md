# Wordpress setup

Example configuration for setting up wordpress with docker-compose.

Look for all occurrences of these strings and change them to your needs (check folders and file names as well):

```
example-com
example.com
contact@example.com
```

# Development

## SSL
I used `mkcert` to generate local certificates (example-com.development):
```
mkcert -install
mkcert example-com.development
```

added in /etc/hosts:
```
127.0.0.1 example-com.development
```
if something is wrong after hosts edit:
```
sudo killall -HUP mDNSResponder
```

After enabling ssl I had some issues with wordpress that would still redirect me to port 3000. I solved it with a combination of setting port mapping in docker-compose "3000:443" and changing some 2 props in mysql that would still be set to http://localhost:3000. I changed it for now to just https://localhost

**Wordpress version**
5.5.0 - php7.4 - fpm

## DevOps (website)
Place ENV files:
```
cp .env.example-com-mysql.default .env.example-com-mysql
cp .env.example-com-wordpress.default .env.example-com-wordpress
vim .env.example-com-mysql
vim .env.example-com-wordpress
```

You probably want to upload wp-content folder or some of its contents. Sometimes you need to fix file owners with:
```
chown -R www-data:www-data /var/www/html/wp-content
```

Build
```
docker-compose -f ./docker-compose.prod.yml build
```

Run (if it's a first time)
```
./init-letsencrypt.sh
```

Run
```
docker-compose -f ./docker-compose.prod.yml up -d
docker-compose -f ./docker-compose.prod.yml logs -f
```

## mysql backup restore
Copy to server:
```
rsync -a ~/backup-0001.sql root@<IP-ADDRESS-OF-YOUR-SERVER>:/root/app/example-com-mysql/
```

Run on server:
```
docker exec -it example-com-mysql /bin/bash
mysql -u example-com -p
use example-com;
source /var/lib/mysql/backup-0001.sql
```

To migrate mysql from dev to prod:
```
UPDATE example-com-options SET option_value = REPLACE(option_value, 'https://example-com.development', 'https://example.com') WHERE option_name = 'home' OR option_name = 'siteurl';


UPDATE example-com-posts SET post_content = REPLACE (post_content, 'https://example-com.development', 'https://example.com');


UPDATE example-com-posts SET post_excerpt = REPLACE (post_excerpt, 'https://example-com.development', 'https://example.com');


UPDATE example-com-postmeta SET meta_value = REPLACE (meta_value, 'https://example-com.development','https://example.com');


UPDATE example-com-termmeta SET meta_value = REPLACE (meta_value, 'https://example-com.development','https://example.com');


UPDATE example-com-comments SET comment_content = REPLACE (comment_content, 'https://example-com.development', 'https://example.com');


UPDATE example-com-comments SET comment_author_url = REPLACE (comment_author_url, 'https://example-com.development','https://example.com');


UPDATE example-com-posts SET guid = REPLACE (guid, 'https://example-com.development', 'https://example.com') WHERE post_type = 'attachment';
```

## Setting up a Digital Ocean Droplet

Install Docker: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04
Install Docker Compose: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04
Generate ssh key and add it in github
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
git clone your project repo
