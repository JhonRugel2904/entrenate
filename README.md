# EQUILIBRA-T — Sistema Inteligente de Monitoreo y Gestión Estratégica del Bienestar y Desempeño del Personal
**Laboratorio de Criminalística — Región Policial Tacna (Policía Nacional del Perú)**  
**Documento Técnico de Arquitectura, Despliegue y API REST para Desarrolladores**  
**Versión:** 1.0.0 (Avance de Proyecto Final 2 - Ciclo 2026-II)  
**Clasificación:** Confidencial / Uso Institucional  

---

## 1. Descripción del Sistema

EQUILIBRA-T es una plataforma web desarrollada sobre el ecosistema Java (Spring Boot) diseñada para registrar y procesar métricas antropométricas (peso, talla e IMC), gestionar pausas activas programadas y detectar alertas tempranas de sobrecarga pericial (burnout) en menos de 2 minutos por efectivo policial, garantizando tiempos de respuesta por debajo de los 2 segundos dentro de la intranet policial[cite: 2].

---

## 2. Pila Tecnológica (Stack)

* **Lenguaje de Programación:** Java SE 21 (LTS)[cite: 2]
* **Framework Principal:** Spring Boot 3.3.x (Spring Web, Spring Data JPA, Spring Security, Spring Validation)[cite: 2]
* **Motor de Base de Datos:** PostgreSQL 15+[cite: 2]
* **Seguridad y Cifrado:** JSON Web Tokens (JWT) y algoritmo BCrypt para hash de credenciales bajo la Ley N.° 29733[cite: 2]
* **Gestor de Dependencias y Compilación:** Apache Maven 3.9+[cite: 2]
* **Capa Frontend / Vistas:** Interfaz responsiva institucional basada en HTML5, CSS3 (Tailwind CSS) y JavaScript ES6+[cite: 2]

---

## 3. Requisitos Previos del Entorno de Desarrollo

Antes de desplegar el aplicativo localmente, asegúrese de contar con el siguiente software instalado:

1. **Java Development Kit (JDK):** Versión 21 o superior configurada en el path del sistema (`JAVA_HOME`).
2. **PostgreSQL:** Servidor de base de datos activo en el puerto `5432` con una base de datos creada llamada `equilibrat_db`.
3. **Git:** Para el clonado y versionamiento del repositorio.
4. **Apache Maven:** Herramienta de compilación integrada o vía CLI (`mvn`).

---

## 4. Configuración del Entorno (`application.properties`)

Cree o configure el archivo ubicado en `src/main/resources/application.properties` con los parámetros institucionales:

```properties
# ==========================================
# CONFIGURACIÓN DEL SERVIDOR WEB
# ==========================================
server.port=8080
server.servlet.context-path=/equilibrat

# ==========================================
# ACCESO A LA BASE DE DATOS POSTGRESQL
# ==========================================
spring.datasource.url=jdbc:postgresql://localhost:5432/equilibrat_db?useSSL=false&serverTimezone=America/Lima
spring.datasource.username=postgres
spring.datasource.password=postgres_secure_pass
spring.datasource.driver-class-name=org.postgresql.Driver

# ==========================================
# CONFIGURACIÓN HIBERNATE / JPA
# ==========================================
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=false

# ==========================================
# SEGURIDAD Y JWT
# ==========================================
security.jwt.secret-key=404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970
security.jwt.expiration-time=28800000

# ==========================================
# UMBRALES DE SALUD OCUPACIONAL (MOTOR DE REGLAS)
# ==========================================
app.fatiga.umbral-horas-continuas=2.0
app.fatiga.umbral-estres-critico=8
```

---

## 5. Estructura de Paquetes del Software (Clean Architecture)

El código fuente en el backend sigue una arquitectura modular en 4 capas desacopladas conforme al diagrama de clases del proyecto[cite: 2, 5]:

```text
pe.gob.pnp.equilibrat
├── EquilibratApplication.java        # Punto de entrada de la aplicación Spring Boot
├── config/                           # Parámetros de seguridad (SecurityConfig, JWTFilter)
│   ├── JwtService.java
│   └── SecurityConfiguration.java
├── controller/                       # Controladores REST API (Exposición de endpoints)
│   ├── AuthController.java
│   ├── DashboardController.java
│   ├── EvaluacionController.java
│   └── PausaActivaController.java
├── dto/                              # Objetos de Transferencia de Datos (DTO Request/Response)
│   ├── EvaluacionRequestDTO.java
│   ├── LoginRequestDTO.java
│   └── PausaResponseDTO.java
├── model/                            # Entidades mapeadas mediante Jakarta Persistence (JPA)
│   ├── AlertaBurnout.java
│   ├── CatalogoPausa.java
│   ├── Departamento.java
│   ├── EvaluacionAntropometrica.java
│   ├── JornadaLaboral.java
│   ├── RegistroPausaActiva.java
│   ├── Rol.java
│   └── Usuario.java
├── repository/                       # Interfaces de persistencia (Spring Data JPA)
│   ├── AlertaRepository.java
│   ├── EvaluacionRepository.java
│   ├── JornadaRepository.java
│   ├── RegistroPausaRepository.java
│   └── UsuarioRepository.java
└── service/                          # Lógica de Negocio y Algoritmos Analíticos
    ├── CalculoImcService.java        # Cálculo matemático y categorización médica
    ├── MotorFatigaService.java       # Detección inteligente de señales de burnout
    └── PausaActivaService.java       # Programación de rutinas guiadas
```

