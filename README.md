---

git## Principios SOLID aplicados

A continuación se describen algunos principios SOLID implementados en el código:

### Principio de Responsabilidad Única (SRP)
El servicio `VerOfertasService` se encarga exclusivamente de obtener las ofertas activas.
```java
@Service
public class VerOfertasService {
private final OfertaRepository ofertaRepository;

public VerOfertasService(OfertaRepository ofertaRepository) {
this.ofertaRepository = ofertaRepository;
}

public List<OfertaLaboral> ejecutar() {
return ofertaRepository.listarPorEstado(EstadoOferta.ACTIVA);
}
}
```

### Principio de Inversión de Dependencias (DIP)
`RegistrarOfertaService` depende de interfaces (`OfertaRepository` y `EmpresaRepository`) y no de implementaciones concretas.
```java
@Service
public class RegistrarOfertaService {
private final OfertaRepository ofertaRepository;
private final EmpresaRepository empresaRepository;

public RegistrarOfertaService(OfertaRepository ofertaRepository,
EmpresaRepository empresaRepository) {
this.ofertaRepository = ofertaRepository;
this.empresaRepository = empresaRepository;
}
// ...
}
```

### Principio Abierto/Cerrado (OCP)
`InactivarOfertaService` extiende el comportamiento sin modificar `OfertaLaboral` ni el repositorio.
```java
@Service
public class InactivarOfertaService {
public Optional<OfertaLaboral> ejecutar(UUID id) {
return ofertaRepository.buscarPorId(id)
.map(oferta -> {
oferta.setEstado(EstadoOferta.INACTIVA);
return ofertaRepository.guardar(oferta);
});
}
}
```
## Principios Clean Code aplicados

A continuación se presentan algunas prácticas de **Clean Code** aplicadas en el proyecto junto con los fragmentos de código correspondientes.

### Nombres significativos
La entidad `OfertaLaboral` emplea nombres claros para sus atributos, facilitando la comprensión del dominio.
```java
@Entity
public class OfertaLaboral {
private LocalDateTime fechaPublicacion;
private EstadoOferta estado;
// ...
}
```

### Funciones pequeñas
Los servicios mantienen métodos cortos y con una única responsabilidad, por ejemplo `ejecutar()` en `VerOfertasService`.
```java
public List<OfertaLaboral> ejecutar() {
return ofertaRepository.listarPorEstado(EstadoOferta.ACTIVA);
}
```

### Comentarios explicativos
Se añaden comentarios de propósito en clases clave como en `EmpresaNoEncontradaException`.
```java
/** Se lanza cuando no se encuentra una empresa al registrar una oferta. */
public class EmpresaNoEncontradaException extends RuntimeException { ... }
```

### Estructura de código fuente clara
El código se organiza por capas: controladores en `api.controllers`, lógica en `application.services` y persistencia en `infrastructure.repository`.

### Objetos bien definidos
`OfertaLaboral` modela de forma explícita una oferta con sus campos y relaciones.
```java
@ManyToOne(fetch = FetchType.LAZY)
private Empresa empresa;
```

### Tratamiento de errores centralizado
`GlobalExceptionHandler` captura excepciones de la aplicación y devuelve respuestas adecuadas.
```java
@ExceptionHandler(EmpresaNoEncontradaException.class)
public ResponseEntity<String> handleEmpresaNoEncontrada(...){
return ResponseEntity.notFound().build();
}
```

### Clases con una única responsabilidad
`RegistrarOfertaService` se centra únicamente en la creación de ofertas.
```java
public class RegistrarOfertaService {
public OfertaLaboral ejecutar(UUID empresaId, OfertaLaboral oferta) { ... }
}