---
typora-copy-images-to: ../assets/img/intro/
typora-root-url: ../../
layout: post
categories: tema1 ejemplo
permalink: esqueleto-aplicacion
conToc: true
title : Esqueleto de un aplicación Spring Boot
---

El primer paso es generar el esqueleto de la aplicación mediante el complemento [Spring Boot Extension Pack](https://marketplace.visualstudio.com/items?itemName=pivotal.vscode-boot-dev-pack) para VSCode.

Una vez instalado, mostramos la paleta de comandos y seleccionamos Spring Initializr: create a maven proyect

Seleccionamos la versión 2.7.0, Java, como Group Id `com.ieselcaminas` y como Artifact Id `realestate`; como empaquetado elegimos `jar`, la versión de java 11 (o la que tengas instalada) y elegimos las dependencias Spring Web, H2 Database, Spring Data JPA SQL y ThymeLeaf. Por último elegimos el directorio donde se va a crear al proyecto.

Esta es la estructura del proyecto creado:

![image-20220616185747916](/spring-realestate-teoria/assets/img/intro/image-20220616185747916.png)

La aplicación se encuentra en el archivo `RealestateApplication.java`

```java
package com.ieselcaminas.realestate;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RealestateApplication {

	public static void main(String[] args) {
		SpringApplication.run(RealestateApplication.class, args);
	}

}
```

Simplemente está anotada con `@SpringBootApplication`

Para arrancarla, ejecutamos

```
./mvnw package
java -jar target/*.jar
```

Y visitamos la url [http://127.0.0.1:8080/](http://127.0.0.1:8080/) que nos mostrará una página de error 404 pero esto indica que funciona correctamente.

![image-20220616190414319](/spring-realestate-teoria/assets/img/intro/image-20220616190414319.png)

## Plantilla

Vamos a usar una plantilla html. Los recursos estáticos como las plantillas se suelen poner en el directorio `resources\templates` y los elementos estáticos en `static`. Desde [aquí](/spring-realestate-teoria/assets/plantilla.zip) os podéis bajar la plantilla y la descomprimís dentro de `resources`.

![image-20220617084630128](/spring-realestate-teoria/assets/img/intro/image-20220617084630128.png)

Y vamos a crear nuestra primera ruta:

```java
package com.ieselcaminas.realestate.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class ZonaPublicaController {
    
    @GetMapping("/")
    public String index(){
        return "index";
    }
}
```

Es una clase que está anotada con `@Controller` y cada ruta se anota con  `@GetMapping` El método debe devolver un `String` que se corresponde con el nombre de una plantilla

Volvemos a compilar y ya podemos visitar la página de portada

![image-20220616195723939](/spring-realestate-teoria/assets/img/intro/image-20220616195723939.png)

## Crear la base de datos

Como motor de base de datos vamos a usar H2, una base de datos ligera que reside en memoria ideal para entornos de prueba.

Configuramos H2 desde el fichero `application.properties`

```
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=root
spring.datasource.password=sa
spring.sql.init.schema-locations=classpath*:db/schema.sql
spring.sql.init.data-locations=classpath*:db/data.sql
# JPA
spring.jpa.hibernate.ddl-auto=none
spring.jpa.open-in-view=true
```

Y automáticamente crea una base de datos siguiendo el esquema y los datos que podéis encontrar [aquí](/spring-realestate-teoria/assets/db.zip). Debéis dejarlos en `resources/db`

