---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
title: Panel de control
layout: post
categories: parte1
conToc: true
permalink: panel-de-control
---

![](/symfony-tienda-teoria/assets/img/panel-de-control-reducido.gif)

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
>-alert- Si aparace un error avisando de que no encuentra el controlador, limpia la caché mediante el comando `php bin/console cache:clear`

## 2.2 Team

Primero configuramos la conexión en el archivo `.env`

```
DATABASE_URL="mysql://root:sa@127.0.0.1:3306/tienda?serverVersion=5.7&charset=utf8mb4"
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
// Otros use
use EasyCorp\Bundle\EasyAdminBundle\Router\AdminUrlGenerator;

#[Route('/admin', name: 'admin')]
public function index(): Response
{
	$adminUrlGenerator = $this->container->get(AdminUrlGenerator::class);

	return $this->redirect($adminUrlGenerator->setController(TeamCrudController::class)->generateUrl());
}

public function configureMenuItems(): iterable
{
    yield MenuItem::linkToDashboard('Dashboard', 'fa fa-home');
    // Añadimos al menú el enlace al crud de Team
    yield MenuItem::linkToCrud('Team', 'fas fa-list', Team::class);        
}
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

Lo estamos configurando para que los campos `name` y `designation` sean de tipo `Field` (campo de texto) y el campo `photo` sea de tipo `ImageField` y además  configuramos el directorio de subida y cuál es la ruta de la carpeta.

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

Ya sólo nos queda hacer el partial para mostrar a los miembros del equipo en la ruta `index`. 

Pasamos todo el código html a una plantilla llamada `partials/_team.html.twig` y en la vista `index.html.twig` la incluimos.

### Servicios

En este  proyecto vamos a usar una capa más de abstracción, creando servicios. En los servicios es donde se crean todas los accesos a la base de datos y Symfony los inyectará automáticamente (`autowiring`) donde haga falta:

> Imagina un controlador que recibe un pedido. Ese controlador valida los datos, calcula el total con IVA, aplica descuentos, guarda en la base de datos y envía un email de confirmación. Al final tiene 200 líneas. Además, si mañana necesitas calcular el total desde otro sitio (un comando, una API...), tienes que copiar ese código.
>
> La solución es separar responsabilidades:
>
> - El **controlador** solo recibe la petición, llama a quien sabe hacer el trabajo y devuelve la respuesta.
> - El **servicio** es una clase que sabe hacer una tarea concreta. Por ejemplo, `PedidoService` sabe calcular totales y `MailerService` sabe enviar correos.

```php
<?php
namespace App\Service;

use App\Repository\TeamRepository;

class TeamService{
    private $repository;

    public function __construct(TeamRepository $repository)
    {
        $this->repository = $repository;
    }
    public function getTeam(): array{
        return $this->repository->findAll();
    }
}
```

Y ahora lo usamos en el código.

```php
use App\service\TeamService;

....
private $productsService;
private $teamService;
public function __construct(ProductsService $productsService, TeamService $teamService)
{
    // Automáticamente, Symfony sirve estos dos servicios
    $this->productsService = $productsService;
    $this->teamService = $teamService;
}
       
#[Route('/', name: 'index')]
public function index(): Response
{
    $team = $this->TeamService->getTeam();
	return $this->render('page/index.html.twig', compact('team'));
}
```

Creamos el partial para el miembro del equipo:

{% raw %}

```html
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
```

{% endraw %}

Y ahora modificamos `_team.html.twig` para recorrer los miembros del equipo:

{% raw %}

```html
    <!-- Team Start -->
<div class="container-fluid py-5">
    <div class="container">
        <div class="border-start border-5 border-primary ps-5 mb-5" style="max-width: 600px;">
            <h6 class="text-primary text-uppercase">Team Members</h6>
            <h1 class="display-5 text-uppercase mb-0">Qualified Pets Care Professionals</h1>
        </div>
        <div class="owl-carousel team-carousel position-relative" style="padding-right: 25px;">
         {% for teamMember in team %}
            {{include('partials/_teamMember.html.twig', {'teamMember': teamMember})}}
         {% endfor %}
        </div>
    </div>
</div>
<!-- Team End -->
```

{% endraw %}

> -reto-**Reto**
>
> Haz lo mismo en la página `about`

## 2.3 Product

> -reto-**Reto**
>
> Crea el panel de control para administrar los productos. La entidad `Product` debe tener los atributos `name`, `photo` y `price`, este último de tipo **float**

Una vez creada la entidad, carga el siguiente SQL con los productos:

```sql
INSERT INTO `product` (`id`, `name`, `photo`, `price`) VALUES
(1, 'Producto 1', 'product-1.png', 12.45),
(2, 'Producto 2', 'product-2.png', 100),
(3, 'Producto 3', 'product-3.png', 20),
(4, 'Producto 4', 'product-4.png', 50),
(5, 'Producto 5', 'product-3.png', 34);
```

Y ahora crea el partial para los productos que incluirás en la ruta `product` e `index`

![image-20221101195632637](/symfony-tienda-teoria/assets/img/image-20221101195632637.png)

> -reto-**Reto**
>
> Siguiendo el modelo de `Team` incluye un listado de productos donde sea necesario

