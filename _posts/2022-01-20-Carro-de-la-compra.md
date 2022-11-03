---
typora-copy-images-to: ../assets/img/
typora-root-url: ../../
title: Carro de la compra
layout: post
categories: parte1
conToc: true
permalink: carro-de-la-compra
---

## ¿Qué aprenderás?

1. A crear prefijos globales en un controlador para todas las rutas del mismo

2. A especificar [métodos de petición](https://developer.mozilla.org/es/docs/Web/HTTP/Methods) en las rutas

3. A trabajar con formato JSON

4. A crear servicios REST

5. A usar ventanas modales de Bootstrap.

6. A usar sesiones para persistir datos

   

## 3.1 Información sobre el producto

Vamos a mostrar una ventana modal con la información del producto al pulsar sobre el icono *Ver*

![image-20221103184949453](/symfony-tienda-teoria/assets/img/image-20221103184949453.png)

Como siempre vamos a crear una nueva ruta en un nuevo controlador llamado `ApiController`

```php
<?php

namespace App\Controller;
use App\Entity\Product;
use Doctrine\Persistence\ManagerRegistry;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Annotation\Route;

#[Route(path:'/api')]
class ApiController extends AbstractController
{
    #[Route('/get/{id}', name: 'api-get',  methods: ['GET'])]
    public function get(ManagerRegistry $doctrine, $id): JsonResponse
    {
        $repository = $doctrine->getRepository(Product::class);
        $product = $repository->find($id);
        if (!$product)
            return new JsonResponse("[]", Response::HTTP_NOT_FOUND);
        
        $data = [
            "id"=> $product->getId(),
            "name" => $product->getName(),
            "price" => $product->getPrice(),
            "photo" => $product->getPhoto()
         ];
        return new JsonResponse($data, Response::HTTP_OK);
    }
}
```

Fíjate que en la línea <span style='color:red'>11</span> aparece `#[Route(path:'/api')]`. Esto hace que todos las rutas que creemos en este controlador estén prefijadas con `/api`. Es decir, el controlador `api-get` responde a la ruta `/api/get/{id}`

Además, fijamos que esta ruta sólo debe responder a peticiones `GET` mediante `methods: ['GET']`.

Como estamos en una [API REST](https://nachoiborraies.github.io/laravel/md/es/06a) (**Re**presentational **S**tate **T**ransfer), hemos de devolver los datos en formato JSON. Si no encontramos el producto devolvemos un array vacío con el `status code` 400; en otro caso, creamos un objeto JSON y lo devolvemos. También fíjate que ahora devolvemos un objeto de la clase `JsonResponse`

Más adelante trabajaremos más a fondo las API Rest.

Por ejemplo, esta es la respuesta a una petición [http://127.0.0.1:8080/api/get/1](http://127.0.0.1:8080/api/get/1):

```json
{"id":2,"name":"Producto 1","price":12.45,"photo":"product-1.png"}
```

Que en el navegador Firefox luce así:

![image-20221103203448424](/symfony-tienda-teoria/assets/img/image-20221103203448424.png)

### 3.1.1 Ventana modal

Vamos a usar el componente `Modal` de Bootstrap para mostrar los datos del producto.

