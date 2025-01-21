# foro-hub
//1. Configuración del Proyecto
//Pasos:
//Crear el proyecto:

//Se utiliza Spring Initializr para generar un proyecto con las siguientes dependencias:
//Spring Web (para construir la API REST).
//Spring Data JPA (para la interacción con la base de datos).
//Spring Security (para autenticación y autorización).
//MySQL Driver (para conectarse a MySQL).
//Validation (para validar datos de entrada).
//Configurar el proyecto:

Se abre el proyecto en tu IDE.
Se asegura de que las dependencias se descarguen correctamente.
2. Configuración de la Base de Datos
Pasos:
Configurar application.properties:

Se abre el archivo src/main/resources/application.properties y se configura la conexión a MySQL:


spring.datasource.url=jdbc:mysql://localhost:3306/foro_db
spring.datasource.username=tu_usuario
spring.datasource.password=tu_contraseña
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect

Se asegura  de que el usuario y la base de datos (foro_db) existan en tu servidor MySQL.

Se crea la base de datos en MySQL:

Se ejecuta cliente MySQL:

CREATE DATABASE foro_db;

3. Diseño de las Entidades
Vamos a crear las entidades necesarias para manejar tópicos, usuarios y respuestas, estableciendo relaciones entre ellas.

Entidades:
Usuario: Representa a los participantes del foro.

@Entity
public class Usuario {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nombre;
    private String email;
    private String password;

    // Getters y setters
}
Topico: Representa una pregunta o asunto discutido.

java
Copiar
Editar
@Entity
public class Topico {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String titulo;
    private String mensaje;

    @ManyToOne
    private Usuario autor;

    // Getters y setters
}
Respuesta: Representa las respuestas a un tópico.

java
Copiar
Editar
@Entity
public class Respuesta {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String mensaje;

    @ManyToOne
    private Topico topico;

    @ManyToOne
    private Usuario autor;

    // Getters y setters
}
4. Implementación de Repositorios
Crea las interfaces para acceder a la base de datos con Spring Data JPA:

UsuarioRepository:

java
Copiar
Editar
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    Usuario findByEmail(String email);
}
TopicoRepository:

java
Copiar
Editar
public interface TopicoRepository extends JpaRepository<Topico, Long> {
}
RespuestaRepository:

java
Copiar
Editar
public interface RespuestaRepository extends JpaRepository<Respuesta, Long> {
}
5. Implementación del CRUD para Tópicos
Controlador:
Crea un controlador REST para manejar las operaciones CRUD de los tópicos.

java
Copiar
Editar
@RestController
@RequestMapping("/topicos")
public class TopicoController {
    private final TopicoRepository topicoRepository;

    public TopicoController(TopicoRepository topicoRepository) {
        this.topicoRepository = topicoRepository;
    }

    @PostMapping
    public ResponseEntity<Topico> crearTopico(@RequestBody @Valid Topico topico) {
        return ResponseEntity.ok(topicoRepository.save(topico));
    }

    @GetMapping
    public List<Topico> listarTopicos() {
        return topicoRepository.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Topico> obtenerTopico(@PathVariable Long id) {
        return topicoRepository.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Topico> actualizarTopico(@PathVariable Long id, @RequestBody @Valid Topico topico) {
        return topicoRepository.findById(id)
                .map(existing -> {
                    existing.setTitulo(topico.getTitulo());
                    existing.setMensaje(topico.getMensaje());
                    return ResponseEntity.ok(topicoRepository.save(existing));
                }).orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminarTopico(@PathVariable Long id) {
        return topicoRepository.findById(id)
                .map(topico -> {
                    topicoRepository.delete(topico);
                    return ResponseEntity.noContent().build();
                }).orElse(ResponseEntity.notFound().build());
    }
}
6. Validaciones
Usa la anotación @Valid en los controladores y valida los datos con las anotaciones de Bean Validation, como @NotNull, @Size, y @Email.
7. Implementación de Seguridad con JWT
Configura Spring Security para manejar la autenticación y autorización mediante JWT.
Genera y valida tokens JWT para restringir el acceso a las rutas.
Resultado Final
Una API REST funcional con CRUD para tópicos.
Validaciones y manejo de errores implementados.
Seguridad mediante autenticación JWT.
Persistencia de datos usando MySQL.
