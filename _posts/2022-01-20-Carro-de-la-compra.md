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

El primer paso va a ser crear una ruta que devuelva los datos del producto en formato `JSON` para crea nuestra propia api rest. 

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
    #[Route('/show/{id}', name: 'api-show',  methods: ['GET'])]
    public function show(ManagerRegistry $doctrine, $id): JsonResponse
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

Fíjate que en la línea <span style='color:red'>11</span> aparece `#[Route(path:'/api')]`. Esto hace que todos las rutas que creemos en este controlador estén prefijadas con `/api`. Es decir, el controlador `api-show` responde a la ruta `/api/show/{id}`

Además, fijamos que esta ruta sólo debe responder a peticiones `GET` mediante `methods: ['GET']`.

Como estamos en una [API REST](https://nachoiborraies.github.io/laravel/md/es/06a) (**Re**presentational **S**tate **T**ransfer), hemos de devolver los datos en formato JSON. Si no encontramos el producto devolvemos un array vacío con el `status code` 400; en otro caso, creamos un objeto JSON y lo devolvemos. También fíjate que ahora devolvemos un objeto de la clase `JsonResponse`

Más adelante trabajaremos más a fondo las API Rest.

Por ejemplo, esta es la respuesta a una petición [http://127.0.0.1:8080/api/show/1](http://127.0.0.1:8080/api/show/1):

```json
{"id":2,"name":"Producto 1","price":12.45,"photo":"product-1.png"}
```

Que en el navegador Firefox luce así:

![image-20221103203448424](/symfony-tienda-teoria/assets/img/image-20221103203448424.png)

### 3.1.1 Ventana modal

Vamos a usar el componente `Modal` de Bootstrap para mostrar los datos del producto.

![image-20221106111721940](/symfony-tienda-teoria/assets/img/image-20221106111721940.png)

Para que funcione este componente hacen falta varias cosas:

1. Un enlace o botón que dispare la ventana modal
2. Un código HTML que dibuje la ventana
3. Un poco de javascript para unirlo todo

Como enlace usaremos el propio botón de ver producto al que le inyectaremos como datos el id del mismo.

{% raw %}

```twig
<a class="btn btn-primary py-2 px-3 open-info-product" data-id="{{product.id}}"><i class="bi bi-eye"></i></a>
```

{% endraw %}

Fíjate que le hemos añadido un atributo llamado `data-id` y le hemos puesto una clase llamada `open-info-product` para poder seleccionar con jquery todos los enlaces a ver productos.

Ahora vamos a dibujar la ventana creando un partial llamado `_infoProducto.html.twig` y siguiendo las instrucciones de Bootstrap para crear [ventanas modales](https://getbootstrap.com/docs/4.0/components/modal/).

```html
<div id="infoProduct" class="modal" tabindex="-1" role="dialog">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Product</h5>
        <button type="button" class="close closeInfoProduct" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <div class="border-start border-5 border-primary ps-5 mb-5" style="max-width: 600px;">
                <h4 id='productName' class="text-primary text-uppercase">Name</h4>
            </div>
        <img id='productImage' class="img-fluid mb-4" src="img/product-1.png">
         <div class="text-center bg-primary p-4 mb-2">
            <h1 class="display-4 text-white mb-0">
                  <span id='productPrice'>10</span><small class="align-top" style="font-size: 22px; line-height: 45px;">€</small>
            </h1>
        </div>
      </div>
      <div class="modal-footer">
       <button type="button" class="btn btn-secondary closeInfoProduct" data-dismiss="modal">Close</button>
      </div>
    </div>
  </div>
</div>
```

En esta plantilla lo que realmente importa son los elementos con los id's: `productName`, `productImage` y `productPrice` ya que son los elementos que reemplazaremos con los devueltos por la api.

Esta plantilla la incluimos dentro del partial que muestra el listado de productos.

Ya por último un poco de javascritpt en `/public/js/app.js` 

```javascript
//Immediately-Invoked Function Expression (IIFE)
(function(){
    const infoProduct = $("#infoProduct");
    $( "a.open-info-product" ).click(function(event) {
      event.preventDefault();
      let id = $( this ).attr('data-id');
      let href = `/api/show/${id}`;
      var jqxhr = $.get( href, function(data) {
        $( infoProduct ).find( "#productName" ).text(data.name);
        $( infoProduct ).find( "#productPrice" ).text(data.price);
        $( infoProduct ).find( "#productImage" ).attr("src", "/img/" + data.photo);
        infoProduct.modal('show');
      })
    });
    $(".closeInfoProduct").click(function (e) {
      infoProduct.modal('hide');
    });
})();
```

* `infoProduct` es el ID de la ventana modal 
* `a.open-info-product` es el selector jquery para seleccionar todos los enlaces a ver producto
* Hacemos una llamada asíncrona mediante `$.get`  y cuando se recibe la respuesta ya sólo queda sustituir los datos por los reales y mostrar la ventana modal.
* `closeInfoProduct` es la clase que tiene la ventana modal en el aspa y el botón para cerrar.

Y ya sólo resta incluir este javascript en la plantilla base:

```html
<script src="{{asset('js/main.js')}}"></script>
<script src="{{asset('js/app.js')}}"></script>
```

Hay que tener la precaución de cargar `app.js` después de jquery y bootstrap

Y *voilà*, ya tenemos la ventana modal en funcionamiento:

![](/symfony-tienda-teoria/assets/img/modal-productos.gif)