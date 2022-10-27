---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
title: Primeros pasos
layout: post
categories: parte1
conToc: true
permalink: primeros-pasos
---

## 1.1 Crear proyecto

Al igual que hicimos en el proyecto [Symfony Blog Paso a Paso](https://victorponz.github.io/symfony-blog-teoria/primeros-pasos), vamos a crear un nuevo proyecto Symfony.

```
composer create-project symfony/website-skeleton symfony-tienda
```

Y también descargamos la plantilla de la [aplicación](https://github.com/victorponz/symfony-tienda/blob/main/pet-shop-website-template.zip) 

Esta plantilla la descomprimimos en raíz de la carpeta `public` de tal forma que esta carpeta quedará de la siguiente manera:

![image-20221027102944170](/symfony-tienda-teoria/assets/img/image-20221027102944170.png)

Ponemos en marcha el servidor `php -S 127.0.0.1:8080 -t public` y ya podemos visitar la página de inicio de la plantilla en [http://127.0.0.1:8080/index.html](http://127.0.0.1:8080/index.html)

![portada](/symfony-tienda-teoria/assets/img/image-20221027103300754.png)

## 1.2 Plantilla

Todas las páginas de la aplicación comparten la cabecera y el pie. Vamos a crear la plantilla `base.html.twig` con dichos elementos. El título de la página y la parte centrar será lo variable.

El primer paso va a ser crear un controlador para las páginas

```
php bin/console make:Controller PageController
```

Copiar el contenido de `index.html` en `base.html.twig` y crear un bloque para el título:

{% raw %}

```twig
<title>{% block title %}PET SHOP - Pet Shop Website Template {% endblock %}</title>
```

{% endraw %}

Y ahora cortar el html comprendido entre 

```html
<!-- Navbar End -->
...contenido original
<!-- Footer Start -->
```

Y pegarlo dentro de la plantilla  `page/index.html.twig` 

Ahora modificamos `base.html.twig` para crear un bloque para el cuerpo central de la página:

{% raw %}

```twig
<!-- Navbar End -->
{% block body %}
{% endblock %}
<!-- Footer Start -->
```

{% endraw %}

Modificamos `page/index.html.twig` para que extienda la plantilla base:

Y finalmente modificamos el controlador `App/Controller/PageController`

```php
class PageController extends AbstractController
{
    #[Route('/', name: 'index')]
    public function index(): Response
    {
        return $this->render('page/index.html.twig', []);
    }
}
```

Si todo ha ido bien, ya podremos visitar la página sin necesidad de `index.html`

![index](/symfony-tienda-teoria/assets/img/image-20221027103300754.png)

Ahora ya podemos eliminar el archivo `public/index.html`

> -reto-
>
> ## 1.3 Resto de páginas
>
> Convierte el resto de páginas de la plantilla. Para la parte **PRODUCT** crea un controlador llamado `ProductController` y para **BLOG** y **DETAIL** un controlador llamado `BlogController`
>
> Ahora ya puedes eliminar las páginas `.html`







