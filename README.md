# foro-hub
1. Configuración del Proyecto
Pasos:
Crear el proyecto:

Se utiliza Spring Initializr para generar un proyecto con las siguientes dependencias:
Spring Web (para construir la API REST).
Spring Data JPA (para la interacción con la base de datos).
Spring Security (para autenticación y autorización).
MySQL Driver (para conectarse a MySQL).
Validation (para validar datos de entrada).
Configurar el proyecto:

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
Se crea las interfaces para acceder a la base de datos con Spring Data JPA:

UsuarioRepository:

public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    Usuario findByEmail(String email);
}
TopicoRepository:

public interface TopicoRepository extends JpaRepository<Topico, Long> {
}
RespuestaRepository:

public interface RespuestaRepository extends JpaRepository<Respuesta, Long> {
}

5. Implementación del CRUD para Tópicos
Controlador:
Se crea un controlador REST para manejar las operaciones CRUD de los tópicos.

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
Se usa la anotación @Valid en los controladores y valida los datos con las anotaciones de Bean Validation, como @NotNull, @Size, y @Email.

7. Implementación de Seguridad con JWT
Configura Spring Security para manejar la autenticación y autorización mediante JWT.
Genera y valida tokens JWT para restringir el acceso a las rutas.
Resultado Final
Una API REST funcional con CRUD para tópicos.
Validaciones y manejo de errores implementados.
Seguridad mediante autenticación JWT.
Persistencia de datos usando MySQL.


-- Se implementa Seguridad con JWT y se añade validación de datos: 

1. Seguridad con JWT
A. Dependencias
Se agrega las siguientes dependencias en tu pom.xml:

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>

B. Configuración JWT
1.-JwtUtils.java

package com.foro.config;

import io.jsonwebtoken.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
public class JwtUtils {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private long expiration;

    public String generateToken(String email) {
        return Jwts.builder()
                .setSubject(email)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + expiration))
                .signWith(SignatureAlgorithm.HS256, secret)
                .compact();
    }

    public String getEmailFromToken(String token) {
        return Jwts.parser()
                .setSigningKey(secret)
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(secret).parseClaimsJws(token);
            return true;
        } catch (JwtException e) {
            return false;
        }
    }
}


2.- Se Configura las propiedades JWT en application.properties:

jwt.secret=clave_secreta_muy_segura
jwt.expiration=86400000 # 1 día en milisegundos


C. Configuración de Seguridad
1.-JwtFilter.java

package com.foro.config;

import com.foro.repository.UsuarioRepository;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class JwtFilter extends OncePerRequestFilter {

    private final JwtUtils jwtUtils;
    private final UserDetailsService userDetailsService;

    public JwtFilter(JwtUtils jwtUtils, UserDetailsService userDetailsService) {
        this.jwtUtils = jwtUtils;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            String email = jwtUtils.getEmailFromToken(token);

            if (email != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(email);

                if (jwtUtils.validateToken(token)) {
                    var authToken = new org.springframework.security.authentication.UsernamePasswordAuthenticationToken(
                            userDetails, null, userDetails.getAuthorities());
                    authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        }
        chain.doFilter(request, response);
    }
}

2.- SecurityConfig.java

package com.foro.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .antMatchers("/auth/**").permitAll()
            .anyRequest().authenticated()
            .and()
            .httpBasic();
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}


D. Controlador de Autenticación

package com.foro.controller;

import com.foro.config.JwtUtils;
import com.foro.model.Usuario;
import com.foro.repository.UsuarioRepository;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("/auth")
public class AuthController {

    private final UsuarioRepository usuarioRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtUtils jwtUtils;

    public AuthController(UsuarioRepository usuarioRepository, PasswordEncoder passwordEncoder, JwtUtils jwtUtils) {
        this.usuarioRepository = usuarioRepository;
        this.passwordEncoder = passwordEncoder;
        this.jwtUtils = jwtUtils;
    }

    @PostMapping("/register")
    public Usuario register(@RequestBody Usuario usuario) {
        usuario.setPassword(passwordEncoder.encode(usuario.getPassword()));
        return usuarioRepository.save(usuario);
    }

    @PostMapping("/login")
    public Map<String, String> login(@RequestBody Map<String, String> credentials) {
        Usuario usuario = usuarioRepository.findByEmail(credentials.get("email"));

        if (usuario != null && passwordEncoder.matches(credentials.get("password"), usuario.getPassword())) {
            String token = jwtUtils.generateToken(usuario.getEmail());
            return Map.of("token", token);
        } else {
            throw new RuntimeException("Credenciales inválidas");
        }
    }
}


2. Validaciones de Datos
Usaremos @Valid y validaciones de Bean en el modelo:

Añade validaciones en las entidades:

En Usuario y Topico ya tenemos anotaciones como @NotBlank y @Email.
Usa @Valid en los controladores:

@PostMapping
public ResponseEntity<Topico> crearTopico(@Valid @RequestBody Topico topico) {
    return ResponseEntity.ok(topicoRepository.save(topico));
}


FIN PROYECTO

