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

7. A usar servicios en plantillas

   

## 3.1 Información sobre el producto

Vamos a mostrar una ventana modal con la información del producto al pulsar sobre el icono *Ver*

![image-20221103184949453](/symfony-tienda-teoria/assets/img/image-20221103184949453.png)

El primer paso va a ser crear una ruta que devuelva los datos del producto en formato `JSON` para crear nuestra propia api `REST`. 

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
{"id":1,"name":"Producto 1","price":12.45,"photo":"product-1.png"}
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

Fíjate que le hemos añadido un atributo llamado `data-id` y le hemos puesto una clase llamada `open-info-product` para poder seleccionar con jquery todos los enlaces a ver productos y obtener el id del producto a partir de `data-id`

> -info-Cuando añadimos un atributo con datos a un elemento siempre se prefija con `data-`

Ahora vamos a dibujar la ventana creando un partial llamado `_infoProducto.html.twig` y seguimos las instrucciones de Bootstrap para crear [ventanas modales](https://getbootstrap.com/docs/4.0/components/modal/).

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

Esta plantilla la incluimos dentro de `base.html.twig` para que esté disponible en todas las rutas.

Ya por último un poco de `jquery` en `/public/js/app.js` 

```javascript
//Immediately-Invoked Function Expression (IIFE)
(function(){
    const infoProduct = $("#infoProduct");
    $( "a.open-info-product" ).click(function(event) {
      event.preventDefault();
      const id = $( this ).attr('data-id');
      const href = `/api/show/${id}`;
      $.get( href, function(data) {
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
* `a.open-info-product` es el selector `jquery` para seleccionar todos los enlaces para ver el producto
* Hacemos una llamada asíncrona ([ajax](https://es.wikipedia.org/wiki/AJAX)) mediante `$.get`  y cuando se recibe la respuesta ya sólo queda sustituir los datos por los reales y mostrar la ventana modal.
* `closeInfoProduct` es la clase que tiene la ventana modal en el aspa y el botón para cerrar.

Y ya sólo resta incluir este javascript en la plantilla base:

```html
<script src="{{asset('js/main.js')}}"></script>
<script src="{{asset('js/app.js')}}"></script>
```

Hay que tener la precaución de cargar `app.js` después de jquery y bootstrap

Y !*voilà*¡, ya tenemos la ventana modal en funcionamiento:

![](/symfony-tienda-teoria/assets/img/modal-productos.gif)

## 3.2 Carro de la compra

Vamos a crear una ventana modal para el carro de la compra. Al igual que antes nos hace falta:

1. Gestionar el carro de la compra en la sesión
2. Un `endpoint` en la api para añadir un producto al carro
3. Un HTML para mostrar la ventana del carro
4. Un poco de javascript para unirlo todo

### 3.2.1 Gestión de la sesión

Para gestionar la sesión usaremos la clase `Symfony\Component\HttpFoundation\RequestStack` que se la inyectaremos al constructor:

```php
public function __construct(RequestStack $requestStack)
{
	$this->requestStack = $requestStack;
}
```

La forma de obtener la sesión es:

```php
$session = $this->requestStack->getSession();
```

Para guardar los productos del carro vamos a crear un array asociativo que guardaremos en la sesión con el código de producto y la cantidad. Por ejemplo:

```php
[	
	3 => 1 //Cantidad 1 del producto 3
	4 => 1 //Cantidad 1 del producto 4
]
```

El código completo es el siguiente:

```php
<?php
namespace App\Service;
use Symfony\Component\HttpFoundation\RequestStack;

class CartService{
    private const KEY = '_cart';
    private $requestStack;
    public function __construct(RequestStack $requestStack)
    {
        $this->requestStack = $requestStack;
    }
    public function getSession()
    {
        return $this->requestStack->getSession();
    }
    public function getCart(): array {
        return $this->getSession()->get(self::KEY, []);
    }
    public function add(int $id, int $quantity = 1){
        //https://symfony.com/doc/current/session.html
        $cart = $this->getCart();
        //Sólo añadimos si no lo está 
        if (!array_key_exists($id, $cart))
            $cart[$id] = $quantity;
        $this->getSession()->set(self::KEY, $cart);
    }
}
```

Creamos un array que se almacena con la clave `_cart`.

Aparte del esqueleto, la parte interesante es donde almacena el array en la clave:

```php
public function add(int $id, int $quantity = 1){
    //https://symfony.com/doc/current/session.html
    $cart = $this->getCart();
    //Sólo añadimos si no lo está 
    if (!array_key_exists($id, $cart))
        $cart[$id] = $quantity;
    $this->getSession()->set(self::KEY, $cart);  
}
```

### 3.2.2 Ruta

Ahora creamos la ruta  `/cart/add/{id}`:

```php
....
private $cart;

//Le inyectamos CartService como una dependencia
public  function __construct(ManagerRegistry $doctrine, CartService $cart)
{
    $this->doctrine = $doctrine;
    $this->repository = $doctrine->getRepository(Product::class);
    $this->cart = $cart;
}

... 
#[Route('/add/{id}', name: 'cart_add', methods: ['GET', 'POST'], requirements: ['id' => '\d+'])]
public function cart_add(int $id): Response
{
    $product = $this->repository->find($id);
    if (!$product)
        return new JsonResponse("[]", Response::HTTP_NOT_FOUND);

    $this->cart->add($id, 1);
	
    $data = [
        "id"=> $product->getId(),
        "name" => $product->getName(),
        "price" => $product->getPrice(),
        "photo" => $product->getPhoto(),
        "quantity" => $this->cart->getCart()[$product->getId()]
    ];
    return new JsonResponse($data, Response::HTTP_OK);

}
```

Ahora probamos que la ruta funciona añadiendo manualmente un producto al carro [http://127.0.0.1:8080/cart/add/2](http://127.0.0.1:8080/cart/add/2)

![image-20221118172647402](/symfony-tienda-teoria/assets/img/image-20221118172647402.png)

### 3.2.4 Ventana modal

Al igual que hicimos en el apartado 3.3.1, vamos a hacer una plantilla para mostrar el carro como una ventana modal:

```html
<div class="modal fade" id="cart-modal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
  <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
        <div class="modal-header">
        <h5 class="modal-title">Product added to cart</h5>
        <button type="button" class="close closeCart" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <div id='data-container'>
          <div class="row">
            <div class="col-md-3">
                <img class='img-thumbnail img-responsive' style='max-width:128px' src=''>
            </div>
            <div class="col-md-9">
                <h4 class='name'></h4>
                <input type='number' min='1' id='quantity' value=1><button class='update' class='btn'>Update</button>
            </div>
          </div>
          <hr>
          <div class="row">
            <div class="col-md-4" >
              <a href="" class="btn btn-primary">View cart</a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

> -reto-Haz la ventana modal para el carro

![](/symfony-tienda-teoria/assets/img/modal-carro.gif)

## 3.3 Página de contenido del carro

![image-20221118182720880](/symfony-tienda-teoria/assets/img/image-20221118182720880.png)

Ahora ya podemos crear la página con el contenido del carro.

Como siempre empezamos por la ruta `/cart` que ya debe estar creada:

```php
#[Route('/', name: 'app_cart')]
public function index(): Response
{
    return $this->render('cart/index.html.twig', [
        'controller_name' => 'CartController',
    ]);
}
```

Y nos hará falta un método en el repositorio que nos devuelva todos los productos del carro:

```php
public function getFromCart(CartService $cart): array
{
    if (empty($cart->getCart())) {
        return [];
    }
    $ids = implode(',', array_keys($cart->getCart()));

    return $this->createQueryBuilder('p')
        ->andWhere("p.id in ($ids)")
        ->getQuery()
        ->getResult();
}
```

Estamos creando un consulta SQL `SELECT * FROM products WHERE id in (...)` y le pasamos todos los ids almacenados en el carro usando `array_keys` para obtener los id's e `implode` para unirlos en una cadena separados por comas.

Y ahora lo usamos en la ruta:

```php
#[Route('/', name: 'app_cart')]
public function index(): Response
{
    $products = $this->repository->getFromCart($this->cart);
    return $this->render('cart/index.html.twig', [
        'products' => '$products',
    ]);
}
```

> -reto- Crea un enlace en la navegación para el carro
>
> ![image-20221118181107578](/symfony-tienda-teoria/assets/img/image-20221118181107578.png)



Ahora modificamos la plantilla, que en el original no aparecía y que podéis descargar desde [aquí](/tienda-symfony/assets/cart.html)

> -reto-Crea la lógica para que se muestre el contenido del carro y que se actualice el total del mismo
>
> Os dejo todo el controlador
>
> ```php
> #[Route('/', name: 'app_cart')]
> public function index(): Response
> {
>     $products = $this->repository->getFromCart($this->cart);
>     //hay que añadir la cantidad de cada producto
>     $items = [];
>     $totalCart = 0;
>     foreach($products as $product){
>         $item = [
>             "id"=> $product->getId(),
>             "name" => $product->getName(),
>             "price" => $product->getPrice(),
>             "photo" => $product->getPhoto(),
>             "quantity" => $this->cart->getCart()[$product->getId()]
>         ];
>         $totalCart += $item["quantity"] * $item["price"];
>         $items[] = $item;
>     }
> 
>     return $this->render('cart/index.html.twig', ['items' => $items, 'totalCart' => $totalCart]);
> }
> ```

## 3.4 Retoques finales

### 3.4.1 Actualizar el carro

![image-20221118190927675](/symfony-tienda-teoria/assets/img/image-20221118190927675.png)

> -reto
>
> Nos queda por hacer funcionar los botones `update` y `View Cart`. Para el botón `Update` debes crear la ruta `/cart/update/{id}/{quantity}` y crear el método `update` en  `CartService`
>
> ```php
> public function update(int $id, int $quantity = 1){
> 	
> }
> ```

### 3.4.2 Eliminar un producto del carro

Vamos a crear la ruta `cart/delete/{id}` y el método `delete` en `CartService` y el botón **Remove from Cart**

![image-20221119170021148](/symfony-tienda-teoria/assets/img/image-20221119170021148.png)

Haremos una petición `POST` por ajax, y cuando devuelva la petición borraremos por `jquery` el producto y actualizaremos el total del carro:

> -reto- Crea la ruta `cart/delete/{id}` y el método `delete` en `CartService`. Para eliminar un elemento del array usa ` unset($cart[$id]);`
>
> Después con jQuery selecciona todos los botones y realiza una petición `POST` a la ruta `delete`. Esta debe devolver el total del carro y una vez finalizada la petición debes eliminar el contenedor del producto. Debes añadir un `id` al contenedor, por ejemplo, `id='item-{{item.id}}'`
>
> Si quieres darle un efecto de jQuery al eliminar el contenedor usa
>
> ```javascript
> $(`#item-${id}`).hide('slow', function(){ $(`#item-${id}`).remove(); });
> ```
>
> Además debes actualizar el total del carro

### 3.4.3 Total productos

> -reto-Crea un método en `CartService` que devuelva el total de productos comprados. Al añadir, modificar y eliminar, debes actualizar el total de productos que debe aparecer en la barra de navegación.
>
> ![image-20221119174136033](/symfony-tienda-teoria/assets/img/image-20221119174136033.png)
>
> Como queremos acceder a un método de un servicio para acceder al método `totalItems` de `cartService`, en vez de pasarlo como un parámetro en cada uno de los métodos `render` de las rutas, vamos a definir este servicio como global para `twig`. Localiza el archivo `config/twig.yaml` y que quede así:
>
> ```yaml
> twig:
> default_path: '%kernel.project_dir%/templates'
> globals:
>     # the value is the service's id
>     cart: '@App\Service\CartService'
> when@test:
> twig:
>     strict_variables: true
> ```
>
> Le estamos diciendo que cree la variable `cart` como una instancia de `App\Service\CartService`
>
> Además deberás actualizar el carro desde la ventana modal y desde Remove from Cart

## 3.5 Gestión de usuarios

Crea las rutas `register`, `login` y `logout`. Protege la ruta `/admin` para que sólo puedan acceder los usuarios con rol `ADMIN` y modifica la barra de navegación.





Hay un ejemplo más completo en [https://dev.to/qferrer/introduction-building-a-shopping-cart-with-symfony-f7h](https://dev.to/qferrer/introduction-building-a-shopping-cart-with-symfony-f7h)


