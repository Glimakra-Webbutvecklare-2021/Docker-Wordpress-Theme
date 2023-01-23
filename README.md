# Docker Wordpress Theme

En guide i hur du kan komma igång med att arbeta med ett Wordpress tema i en Docker container.

Den officiella versionen (jan 2023) av docker-compose.yml finns på `https://hub.docker.com/_/wordpress`:

```yml
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

## Komplettera docker-compose.yml

Under services > wordpress > volumes lägger du till sökvägen där den lokala miljön används:

````code
    volumes:
      - db:/var/www/html
      - ./wp-content:/var/www/html/wp-content
```

---

## Minimalt wordpress tema

Ett minimalt Wordpress tema innehåller filerna `index.php` och `style.css`. 

I din lokala mapp `wp-content` skapar du en mapp med namnet på ett nytt tema, ex `tosimple`.

I mappen `tosimple` skapar du filen `index.php`. Ange följande innehåll:

```php
<!DOCTYPE html>
<html <?php language_attributes(); ?>>

<head>
    <title>
        <?php bloginfo('name'); ?>
    </title>
    <link rel="stylesheet" href="style.css">
    <?php wp_head(); ?>
</head>

<body>

    <?php
    get_header();
    if (have_posts()):
        while (have_posts()):
            the_post();
            the_content();
        endwhile;
    endif;
    get_sidebar();
    get_footer();
    ?>
</body>
```

I mappen `tosimple` skapar du filen `style.css`. Ange följande innehåll:

```css
/*
Theme Name: ToSimple
Theme URI: http://localhost/themes/tosimple
Author: ...
Author URI: http://localhost
Description: ToSimple theme
Version: 1.0
License: GNU General Public License v2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html
Tags: basic, starter, tosimple
Text Domain: ToSimple

This theme, like WordPress, is licensed under the GPL.
Use it to make something cool, have fun, and share what you've learned with others.
*/

html {
    font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
}
```

Om du gjort förändringar efter det att docker-miljön är klar kan du ev behöva bygga om docker images och|eller volymner.

`docker-compose build --no-cache`


## Backup av mysql data

För container innehållande mysql med namnet `programmering-db-1`.
Se till att containern är aktiv. 

Till lokal mapp `backups` och filen `backup.sql` ange docker kommandot i 

`docker exec programmering-db-1 mysqldump -u root --password=db_password exampledb > backups/backup.sql`

Kontrollera filen `backups/backup.sql`. Testa att ändra `blog_name` i tabellen `wp_options`.

För att återläsa data från en fil `backup.sql` ta först reda på container id med kommandot `docker container ls`

Med följande kommando kan data återläsas om container id är `301ac0db6d66`:

`cat backups/backup.sql | docker exec -i 301ac0db6d66  mysql -u root --password=db_password exampledb`

### Verionshantera delar av docker volymer 

Använd `.gitignore` och ange vad som ska versionshanteras, eller utelämnas från versionshanering.

Ignorera vissa mappar in `wp-content`

```code
wp-content/*
!wp-content/themes/tosimple
```
