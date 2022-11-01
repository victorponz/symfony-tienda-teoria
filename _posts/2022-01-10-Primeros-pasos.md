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



## 1.4 Navegación

Ahora mismo la navegación apunta a las páginas `.html` y hemos de modificarla para que apunte a las nuevas rutas.

![image-20221027125437620](/symfony-tienda-teoria/assets/img/image-20221027125437620.png)



Primero vamos a crea una plantilla para el menú de navegación de tal forma que las plantillas queden más estructuradas. A esta plantilla la llamamos `partials/_navigation.html.twig`. El prefijo `_` es opcional, pero es una convención utilizada para diferenciar mejor entre plantillas completas y fragmentos de plantilla).

Y la incluimos en `base.html.twig`

{% raw %}

```twig
{{ include ('partials/_navigation.html.twig')}}
```

{% endraw %}

{% raw %}

```twig
<div class="navbar-nav ms-auto py-0">
                <a href="{{ path('index') }}" class="nav-item nav-link active">Home</a>
                <a href="{{ path('about') }}" class="nav-item nav-link">About</a>
                <a href="{{ path('service') }}" class="nav-item nav-link">Service</a>
                <a href="{{ path('product') }}" class="nav-item nav-link">Product</a>
                <div class="nav-item dropdown">
                    <a href="#" class="nav-link dropdown-toggle" data-bs-toggle="dropdown">Pages</a>
                    <div class="dropdown-menu m-0">
                        <a href="{{ path('price') }}" class="dropdown-item">Pricing Plan</a>
                        <a href="{{ path('team') }}" class="dropdown-item">The Team</a>
                        <a href="{{ path('testimonial') }}" class="dropdown-item">Testimonial</a>
                        <a href="{{ path('blog') }}" class="dropdown-item">Blog Grid</a>
                        <a href="{{ path('detail') }}" class="dropdown-item">Blog Detail</a>
                    </div>
                </div>
                <a href="{{ path('contact') }}" class="nav-item nav-link nav-contact bg-primary text-white px-5 ms-lg-5">Contact <i class="bi bi-arrow-right"></i></a>
            </div>
```

{% endraw %}

Pero ahora ocurre que siempre está activada la ruta a `home`

{% raw %}

```twig
<a href="{{ path('index') }}" class="nav-item nav-link active">Home</a>
```

{% endraw %}

Vamos a arreglarlo obteniendo la ruta actual mediante `app.request.attributes.get('_route')` 

{% raw %}

```twig
<a href="{{ path('index') }}" class="nav-item nav-link {{ (app.request.attributes.get('_route') == 'index')  ? 'active': ''}}">Home</a>
<a href="{{ path('about') }}" class="nav-item nav-link {{ (app.request.attributes.get('_route') == 'about')  ? 'active': ''}}">About</a>
<a href="{{ path('service') }}" class="nav-item nav-link {{ (app.request.attributes.get('_route') == 'service')  ? 'active': ''}}">Service</a>
<a href="{{ path('product') }}" class="nav-item nav-link {{ (app.request.attributes.get('_route') == 'product')  ? 'active': ''}}">Product</a>
```

{% endraw %}

