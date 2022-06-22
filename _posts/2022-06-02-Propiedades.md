---
typora-copy-images-to: ../assets/img/intro/
typora-root-url: ../../
layout: post
categories: tema1 ejemplo
permalink: propiedades
conToc: true
title : Listado de propiedades
---

Vamos a continuar creando la entidad `PropertyType`. Para ello hemos de crear la entidad POJO que la representa:

```java
package com.ieselcaminas.realestate.entities;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity(name="property_types")
public class PropertyType {
    
    @Id @GeneratedValue
    private long id;

    private String name;

    public PropertyType(){

    }

    public PropertyType(String name) {
        this.name = name;
    }
	//Setters y getters
```

La clase se anota con `@Entity` y a continuación el nombre de la tabla si esta no coincide con el nombre de la clase.

Creamos los getters y los setters y ya podemos crear el repositorio para consultar los datos.

```java
package com.ieselcaminas.realestate.repositories;

import com.ieselcaminas.realestate.entities.PropertyType;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;


public interface PropertyTypeRepository extends JpaRepository<PropertyType, Long> {
    public PropertyType findOneById(int id);
    public List<PropertyType> findAll();

}
```

El repositorio debe extender `JpaRepository` y como tipo debe tener el de la entidad POJO y el tipo de dato del campo que hace de clave primaria

A continuación se escribe la signatura de los métodos que van a acceder a los datos y debe devolver una entidad POJO o una lista de entidades POJO. No hace falta escribir el cuerpo de las consultas SQL ya que lo hace Spring Boot automáticamente por nosotros a partir del nombre del método y los parámetros.

Por ejemplo, `findOneById` nos devolverá aquel con Id igual al pasado como parámetro. `findAll` nos devolverá todos. Si creamos el método `findByName(string name)` devolvería aquellos con dicho `name`

Tienes el listado completo de Querys [aquí](https://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/#jpa.query-methods.query-creation)

Modificamos el controlador `inicio` para realizar la consulta de todos los tipos de propiedad:

```java
@Controller
public class ZonaPublicaController {
    
    @Autowired
    PropertyTypeRepository propertyTypeRepository;

    @GetMapping("/")
    public String inicio(Model model){
        model.addAttribute("propertyTypes", propertyTypeRepository.findAll());
        return "index";
    }
}
```

Autoinyectamos con `@Autowired` el repositorio `PropertyTypeRepository` e informamos el atributo del modelo `propertyTypes` con todos los tipos de propiedad. El atributo `propertyTypes` estará disponible como una variable en la plantilla.

A continuación modificamos la plantilla `index.html` para mostrar un `select` con todos los tipos de propiedad.

```html
<div class="col-md-4">
<select class="form-select border-0 py-3">
<option selected>Select property type</option>
<option th:each="propertyType : ${propertyTypes}" th:value="${propertyType.id}" value="1" th:text="${propertyType.name}">Property Type 1</option>
</select>
</div>
```

Con `th:each` realizamos un `foreach` de cada `propertyType` de `propertyTypes` y le asignamos el `value` 

`th:value="${propertyType.id}" ` y el texto `th:text="${propertyType.name}"` 

Thymeleaf sobreescribe los datos value="1" y "Property Type 1" por los valores adecuados.

## Entidad `Property`

```java
package com.ieselcaminas.realestate.entities;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import com.ieselcaminas.realestate.entities.PropertyType;
import javax.persistence.ManyToOne;

@Entity(name = "properties")
public class Property {
    @Id @GeneratedValue
    private long id;

    @ManyToOne
    private PropertyType propertyType;

    private long price;

    private String title;

    private String address;

    private String image;

    public Property(){
        
    }
    public Property(PropertyType propertyType, long price, String title, String address, String image) {
        this.propertyType = propertyType;
        this.price = price;
        this.title = title;
        this.address = address;
        this.image = image;
    }
    
 	Setters y getters   
}
```

Aquí creamos un clave ajena con la entidad `PropertyType` de tipo `@ManyToOne`

Creamos el repositorio:

```java

package com.ieselcaminas.realestate.repositories;

import com.ieselcaminas.realestate.entities.Property;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;
public interface PropertyRepository extends JpaRepository<Property, Long>{
    
    public Property findById(long id);
    
    public List<Property> findAll();

}
```

Informamos la variable `properties` del modelo:

```java
 	@Autowired
    PropertyRepository propertyRepository;
    @GetMapping("/")
    public String inicio(Model model){
        model.addAttribute("propertyTypes", propertyTypeRepository.findAll());
        model.addAttribute("properties", propertyRepository.findAll());
        return "index";
    }
```

Y, por último, modificamos la plantilla:

```html
<div class="col-lg-4 col-md-6 wow fadeInUp" data-wow-delay="0.1s" th:each="property : ${properties}">
    <div class="property-item rounded overflow-hidden">
        <div class="position-relative overflow-hidden">
            <a href=""><img class="img-fluid" src="../static/img/property-1.jpg" th:src="${property.image}" alt=""></a>
            <div class="bg-white rounded-top text-primary position-absolute start-0 bottom-0 mx-4 pt-1 px-3" th:text="${property.propertyType.name}">Appartment</div>
        </div>
        <div class="p-4 pb-0">
            <h5 class="text-primary mb-3" th:text="${property.price}">$12,345</h5>
            <a class="d-block h5 mb-2" th:text="${property.title}" href="">Golden Urban House For Sell</a>
            <p><i class="fa fa-map-marker-alt text-primary me-2" th:text="${property.address}"></i>123 Street, New York, USA</p>
        </div>
    </div>
</div>
```

## Búsqueda de propiedades

Vamos a crear el  buscador de propiedades. Creamos el formulario:

```html
<form action="/search">
    <div class="col-md-10">
        <div class="row g-2">
            <div class="col-md-4">
                <input type="text" name="q" class="form-control border-0 py-3" placeholder="Search Keyword">
            </div>
            <div class="col-md-4">
                <select name="propertyTypeId" class="form-select border-0 py-3">
                    <option selected value="-1">Select property type</option>
                    <option th:each="propertyType : ${propertyTypes}" th:value="${propertyType.id}" value="1" th:text="${propertyType.name}">Property Type 1</option>
                </select>
            </div>
        </div>
    </div>
    <div class="col-md-2">
        <input class="btn btn-dark border-0 w-100 py-3" type="submit" value="Search">
    </div>
</form>
```

de tal forma que la búsqueda será de la forma `search?q=urban&propertyTypeId=1`

Creamos dos métodos en repositorio:

* uno para buscar por tipo de propiedad y título 

  ```java
  findByPropertyTypeAndTitleContainsIgnoreCase(PropertyType propertyType, String title);
  ```

* y otro para buscar sólo en el título

  ```
  findByTitleContainsIgnoreCase(String title);
  ```

Y ahora creamos el controlador:

```java
@GetMapping("/search")
public String search(Model model, @RequestParam(name="q", required = false) String query, @RequestParam(name="propertyTypeId", required = false) long propertyTypeId){
    PropertyType propertyType = propertyTypeRepository.findOneById(propertyTypeId);
    if (propertyType != null)
        model.addAttribute("properties", propertyRepository.findByPropertyTypeAndTitleContainsIgnoreCase(propertyType, query));
    else
        model.addAttribute("properties", propertyRepository.findByTitleContainsIgnoreCase(query));
    return "search";
}
```

En este caso hemos obtenido los parámetros `q` y `propertyTypeId` del query string usando la anotación `@RequestParam`

Además, como también hemos de obtener el listado de todos los tipos de propiedades vamos a crear un atributo en el modelo que se auto-informe al crear la clase:

```java
@ModelAttribute("propertyTypes")
public List<PropertyType> propertyTypes(){
	return  propertyTypeRepository.findAll();
}
```

Con la anotación `@ModelAttribute` este atributo está disponible para todas las plantillas del controlador.

Y refactorizamos `inicio`

```java
@GetMapping("/")
public String inicio(Model model){
    model.addAttribute("properties", propertyRepository.findAll());
    return "index";
}
```

Ya tememos el listado de la búsqueda:

