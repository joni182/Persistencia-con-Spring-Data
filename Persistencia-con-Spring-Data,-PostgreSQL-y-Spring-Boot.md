
## Persistencia con Spring Data, PostgreSQL y Spring Boot.
Hoy veremos que pueden hacer los chicos de Spring para facilitarnos la vida tanto a la hora de gestionar la persistencia de datos de nuestros proyectos como a la de lanzarlos de forma sencilla con String Boot, el cual con solo unas lineas de código nos levantara nuestra aplicacción web y la dejara lista para usar. Spring Data JPA nos ayudará con el ORM, usando hibernate de forma totalmente trasparente para nosotros e implementando gran parte del código necesario para realizar la gran mayoria de nuestras operaciones CRUD. ¡Al lio!
## Configuración
La filosofia de Spring es hacernos el trabajo lo menos engorroso posible, por eso prescindiremos de los XML y las anotaciones para configurar nuestro framework, para ello usaremos un archivo application.yml que ubicaremos en `/miAplicacion/src/main/resources/application.yml`.

``` jaml
spring:
    jpa:
        database: POSTGRESQL
        show-sql: true
        hibernate.ddl-auto: update
    datasource:
        platform: postgres
        url: jdbc:postgresql://localhost:5432/pruebas01
        username: postgres
        password: admin    
        driverClassName: org.postgresql.Driver

server:
    port: 8080
```
Asi habremos configurado jpa, el acceso a nuestra base de datos Postgres y el puerto en el que escuchará nuestra aplicación.

## Entidades
Para el ejemplo usaremos una entidad simple, como vemos, todas las anotaciones pertenecen al paquete `javax.persistence.*` por lo que no son propiamente de String Data, si no que pertenencen a la API de persistencia de Java.
``` java
import java.util.Date;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

@Entity
public class Animals {

	@Id
	@GeneratedValue( strategy= GenerationType.AUTO ) 
	private long id_animals;
	private String name;
	private String chip;

	public Animals(long id_animals, String name, String chip, Date birth, boolean ppp) {
		super();
		this.id_animals = id_animals;
		this.name = name;
		this.chip = chip;

	}

	public Animals() {
		super();
	}

	public long getId_animals() {
		return id_animals;
	}

	public void setId_animals(long id_animals) {
		this.id_animals = id_animals;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getChip() {
		return chip;
	}

	public void setChip(String chip) {
		this.chip = chip;
	}
	
}
```
## Repositorio
Ahora viene el momento de la mágia. Ahora tocaria implementar toda la lógica del DAO para las **operaciones CRUD** pero nosotros lo haremos en 30 segundos,¿Como?, pues con la anotación `@Repository` y extendiendo de `JpaRepository<Animals,Long>`. Como vemos, es una clase parametrizada en la que deberemos colocar en primer lugar el tipo de la clase entidad que vamos a gestionar y en segundo el tipo de su clave primaria. `<Animals,Long>` 
``` java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import com.joni.spring_data.entities.Animals;

@Repository
public interface AnimalRepository extends JpaRepository<Animals,Long> {
}
```
## Controlador
Llega la hora de configurar nuestro controlador. Vamos ha hacer uso de `animalRepository` y de los metodos que ha heredado para hacer las operaciones básicas **CRUD**, lo haremos con las siguientes anotaciones.
`@RestController`: Es la conjunción de dos anotaciones,`@Controller` y `@ResponseBody`. De esa manera indicará que esa clase es un controlador web y que lo que devuelvan los metodos de esa clase se enviaran al cuerpo de la página web. 
`@RequestMapping`: Con esto podemos indicar la url que disparará el código anotado y bajo que tipo de petición, GET, POST, PUT...
`@RequestBody`: Capturará la información que le llegue de la vista y lo convertira al tipo de dato apropiado.
`@PathVariable`: Tomará de la url la parte que le indiquemos para usarla como parametro de entrada en el método que lo necesitemos. 
En este ejemplo, con la url `localhost:8080/animals/254` nos devolveria el **JSON** equivalente al registro con id 254
```java
    @RequestMapping(method = RequestMethod.GET, value = "/{animalsId}")
    public Animals readAnimals(@PathVariable("animalsId") Long animalsId){
        return animalRepository.findOne(animalsId);
    }
```
```java
mport java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PatchMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.joni.spring_data.entities.Animals;
import com.joni.spring_data.repositories.AnimalRepository;

@RestController
@RequestMapping("/animals")
public class AnimalControllers {
	
	@Autowired
	private AnimalRepository animalRepository;
	
	@RequestMapping("")
	public List<Animals> listAnimals() {
		
		return animalRepository.findAll();
	}
	
	@RequestMapping(value="", method=RequestMethod.POST)
	public Animals createAnimals(@RequestBody Animals animal) {
		
		return animalRepository.save(animal);
	}
	
	@RequestMapping(method=RequestMethod.PUT, value = "/{animalsId}")
	public Animals updateAnimals(@PathVariable("animalsId") Long animalsId,@RequestBody    Animals animal){
		
		if (animal != null && animalRepository.exists(animalsId)){
			
			Animals oldAnimal = animalRepository.findOne(animalsId);
			
			oldAnimal.setName(animal.getName());
			oldAnimal.setChip(animal.getChip());
			
			return animalRepository.save(oldAnimal);
		}
		return null;
	}
	
	@RequestMapping(method = RequestMethod.GET, value = "/{animalsId}")
	public Animals readAnimals(@PathVariable("animalsId") Long animalsId){
		return animalRepository.findOne(animalsId);
	}
}
```
## Spring Boot
Por último con Spring Boot vamos ha hacer que todo esto funcione, solo necesitamos indicarle que vamos a usar Repositorios JPA con `@EnableJpaRepositories` y decirle cual es nuestra clase principal con `@SpringBootApplication`.

Una vez hecho esto, arrancaremos la aplicación con una única line de código:
`SpringApplication.run(SpringDataApplication.class, args);`
``` java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@SpringBootApplication
@EnableJpaRepositories
public class SpringDataApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringDataApplication.class, args);
	}
}
```
