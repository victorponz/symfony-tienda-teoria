---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
title: Primeros pasos
layout: post
categories: parte1
conToc: true
permalink: primeros-pasos

---

## ¿Qué aprenderemos?

1. A crear un panel de control para nuestras entidades
2. A usar controladores embebidos
3. A usar servicios

## 2.1 Panel de control

Vamos a crear un panel de control para nuestra aplicación.

Para ello usaremos el bundle EasyAdmin de Symfony.

```
composer require easycorp/easyadmin-bundle
```

Una vez instalado vamos a crear nuestro Dashboard

```
php bin/console make:admin:dashboard
```

Que creará una nueva ruta `admin`  y un nuevo controlador `DashboardController`

Si visitamos la ruta [http://127.0.0.1:8080/admin](http://127.0.0.1:8080/admin) verás la página de inicio

![image-20221027181654194](/symfony-tienda-teoria/assets/img/image-20221027181654194.png)

## 2.2 Team

Primero configuramos la conexión en el archivo `.env`

```
DATABASE_URL=mysql://root:sa@127.0.0.1:3306/tienda
```

Y creamos la base de datos:

```
php bin/console doctrine:database:create
```

Vamos a empezar con los miembros del equipo

![image-20221027182231943](/symfony-tienda-teoria/assets/img/image-20221027182231943.png)

El primer paso es crear la entidad `Team`.

<script id="asciicast-0OjF1EFPSSHI6dqUiGz6QFtmA" src="https://asciinema.org/a/0OjF1EFPSSHI6dqUiGz6QFtmA.js" async></script>

Y generar la migración:

```
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

Y luego generar el controlador CRUD

> -info-**CRUD** son las iniciales de **C**reate, **R**ead, **U**pdate y **D**elete que son las operaciones básicas que se realizan con una entidad

<script id="asciicast-JmpDyeqVjzCQaNV6QFvdNaACv" src="https://asciinema.org/a/JmpDyeqVjzCQaNV6QFvdNaACv.js" async></script>

Y por último, modificar `DashboardController` para que cargue por defecto el panel de control de la entidad `Team`

```php
#[Route('/admin', name: 'admin')]
public function index(): Response
{
$adminUrlGenerator = $this->container->get(AdminUrlGenerator::class);

return $this->redirect($adminUrlGenerator->setController(TeamCrudController::class)->generateUrl());
```

![image-20221027184941326](/symfony-tienda-teoria/assets/img/image-20221027184941326.png)

Ahora ya podemos añadir miembros al equipo.

![image-20221027185039315](/symfony-tienda-teoria/assets/img/image-20221027185039315.png)

Pero claro, EasyAdmin no sabe que el campo `Photo` debe ser de tipo `File`

Así que configuramos `TeamCrudController` para modificar los campos:

```php
<?php

namespace App\Controller\Admin;

use App\Entity\Team;
use EasyCorp\Bundle\EasyAdminBundle\Controller\AbstractCrudController;
use EasyCorp\Bundle\EasyAdminBundle\Field\Field;
use EasyCorp\Bundle\EasyAdminBundle\Field\ImageField;

class TeamCrudController extends AbstractCrudController
{
    public static function getEntityFqcn(): string
    {
        return Team::class;
    }

    public function configureFields(string $pageName): iterable
    {
        return [
            Field::new('name'),
            ImageField::new('photo')->setUploadDir('/public/img')->setBasePath('/img/'),
            Field::new('designation')
        ];
    }
}
```

Lo estamos configurando para que los campos `name` y `designation` sean por defecto y el campo `photo` sea de tipo `ImageField` y además  configuramos el directorio de subida y cuál es la ruta de la carpeta.

Ahora ya podemos subir una foto:

![image-20221027192221148](/symfony-tienda-teoria/assets/img/image-20221027192221148.png)

Ahora vamos a subir todas las imágenes del equipo a la base de datos mediante el siguiente sql:

```sql
INSERT INTO `team` (`id`, `name`, `photo`, `designation`) VALUES
(1, 'Team 1', '/team-1.jpg', 'Designation'),
(2, 'Team 2', '/team-2.jpg', 'Designation'),
(3, 'Team 3', '/team-3.jpg', 'Designation'),
(4, 'Team 4', '/team-4.jpg', 'Designation'),
(5, 'Team 5', '/team-5.jpg', 'Designation');
```

![image-20221027192959150](/symfony-tienda-teoria/assets/img/image-20221027192959150.png)

Ya sólo nos queda hacer el partial para mostrar a los miembros del equipo. Pasamos todo el código html a una plantilla llamada `partials/_team.html.twig` y en la vista `index.html.twig` la incluimos.

Ya solo queda conectarla con la base de datos. 

Primero modificamos el controlador de la ruta `index`

```php
#[Route('/', name: 'index')]
public function index(ManagerRegistry $doctrine): Response
{
    $repository = $doctrine->getRepository(Team::class);
    $team = $repository->findAll();
    return $this->render('page/index.html.twig', compact('team'));
}
```

Y ahora modificamos `_team.html.twig` para recorrer los miembros del equipo:

{% raw %}

```php
    <!-- Team Start -->
    <div class="container-fluid py-5">
        <div class="container">
            <div class="border-start border-5 border-primary ps-5 mb-5" style="max-width: 600px;">
                <h6 class="text-primary text-uppercase">Team Members</h6>
                <h1 class="display-5 text-uppercase mb-0">Qualified Pets Care Professionals</h1>
            </div>
            <div class="owl-carousel team-carousel position-relative" style="padding-right: 25px;">
             {% for teamMember in team %}
                <div class="team-item">
                    <div class="position-relative overflow-hidden">
                        <img class="img-fluid w-100" src="{{ asset('img/' ~ teamMember.photo) }}" alt="">
                        <div class="team-overlay">
                            <div class="d-flex align-items-center justify-content-start">
                                <a class="btn btn-light btn-square mx-1" href="#"><i class="bi bi-twitter"></i></a>
                                <a class="btn btn-light btn-square mx-1" href="#"><i class="bi bi-facebook"></i></a>
                                <a class="btn btn-light btn-square mx-1" href="#"><i class="bi bi-linkedin"></i></a>
                            </div>
                        </div>
                    </div>
                    <div class="bg-light text-center p-4">
                        <h5 class="text-uppercase">{{ teamMember.name }}</h5>
                        <p class="m-0">{{ teamMember.designation }}</p>
                    </div>
                </div>
            {% endfor %}
            </div>
        </div>
    </div>
    <!-- Team End -->
```

{% endraw %}

Resulta que las fotos del equipo también aparecen en la ruta `about` por lo que hay que proceder de la misma forma: 1.- modificar el controlador, 2.- modificar la plantilla `about.html.twig`

```php
#[Route('/about', name: 'about')]
public function about(ManagerRegistry $doctrine): Response
{
    $repository = $doctrine->getRepository(Team::class);
    $team = $repository->findAll();
    return $this->render('page/about.html.twig', compact('team'));
}
```

Pero como veis es el mismo código que en el controlador `index` y hemos de intentar siempre no repetirnos (**DRY** Don't Repeat Yourself). Así que vamos a usar [Controladores Embebidos](https://symfony.com/doc/current/templates.html#embedding-controllers). Creamos un método que obtenga los miembros del equipo y que además renderiza la plantilla:

```php
public function teamTemplate(ManagerRegistry $doctrine): Response
{
    $repository = $doctrine->getRepository(Team::class);
    $team = $repository->findAll();
    return $this->render('partials/_team.html.twig',compact('team'));
}
```

Modificamos las plantillas `index.html.twig` y `about.html.twig`, cambiando

{% raw %}

```twig
{{ include ('partials/_team.html.twig')}}
```

{% endraw %}

por

{% raw %}

```twig
{{ render(controller('App\\Controller\\PageController::teamTemplate')) }}
```

{% endraw %}

Y ya podemos eliminar el código duplicado

```php
#[Route('/', name: 'index')]
public function index(ManagerRegistry $doctrine): Response
{
    return $this->render('page/index.html.twig');
}

#[Route('/about', name: 'about')]
public function about(ManagerRegistry $doctrine): Response
{
    return $this->render('page/about.html.twig');
}
```

Procedemos de la misma manera para el controlador `team`

## 2.3 Productos

> -reto-**Reto**
>
> Crea el panel de control para administrar los productos. La entidad `Product` debe tener los atributos `name`, `photo` y un `price`, este último de tipo **float**

Una vez creada la entidad, carga el siguiente SQL con los productos:

```sql
INSERT INTO `product` (`id`, `name`, `photo`, `price`) VALUES
(1, 'Producto 1', 'product-1.png', 12.45),
(2, 'Producto 2', 'product-2.png', 100),
(3, 'Producto 3', 'product-3.png', 20),
(4, 'Producto 4', 'product-4.png', 50);
(5, 'Producto 5', 'product-3.png', 34);
```

Y ahora crea el partial para los productos que incluirás en la ruta `product` e `index`

![image-20221101195632637](/symfony-tienda-teoria/assets/img/image-20221101195632637.png)

En este caso vemos que también hemos de cargar los productos en dos rutas. Pero esta vez vamos a crear un [servicio](https://symfony.com/doc/current/service_container.html):

> Your application is *full* of useful objects: a "Mailer" object might help you send emails while another object might help you save things to the database. Almost *everything* that your app "does" is actually done by one of these objects. And each time you install a new bundle, you get access to even more!
>
> In Symfony, these useful objects are called **services** and each service lives inside a very special object called the **service container**. The container allows you to centralize the way objects are constructed. It makes your life easier, promotes a strong architecture and is super fast!

Un servicio es una forma de compartir código entre varias partes de la aplicación. Para usarlo sólo hay que hacer type-hinting. Vamos a verlo:

1. Creamos el servicio en `App\Services\ProductsService`

   ```php
   <?php
   namespace App\Service;
   
   use App\Entity\Product;
   use Doctrine\Persistence\ManagerRegistry;
   
   class ProductsService{
       private $doctrine;
   
       public function __construct(ManagerRegistry $doctrine)
       {
           //Como hace falta acceder a ManagerRegistry lo inyectamos en el controlador
           $this->doctrine = $doctrine;
       }
       public function getProducts(): ?array{
           $repository = $this->doctrine->getRepository(Product::class);
           return $repository->findAll();
       }
   }
   ```
   Aquí estamos usando una característica llamada *inyección de dependencias* ya que Symfony se encarga automáticamente de pasarnos una referencia a la clase ManagerRegistry
   
2. Y ahora lo *inyectamos* en todos aquellos métodos en que queramos usarlo. 

   ```php
   #[Route('/product', name: 'product')]
   public function index(ProductsService $productsService): Response
   {
       $products = $productsService->getProducts();
       return $this->render('product/product.html.twig', compact('products'));
   }
   ```

> -reto-**Reto**
>
> Haz lo mismo para la ruta `/`

