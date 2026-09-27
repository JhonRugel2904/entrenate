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

---

## 6. Catálogo Oficial de Endpoints de la API REST

### 6.1. Módulo de Autenticación
* **Ruta:** `POST /api/v1/auth/login`
* **Acceso:** Público
* **Descripción:** Valida el número de CIP y la contraseña cifrada contra la tabla `usuario`[cite: 2].
* **Cuerpo de Solicitud (JSON):**
  ```json
  {
    "cip": "31245678",
    "password": "Password123*"
  }
  ```
* **Respuesta Exitosa (`200 OK`):**
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tipo": "Bearer",
    "idUsuario": 10,
    "nombreCompleto": "Juan Quispe Huanca",
    "rol": "PERITO_OPERATIVO"
  }
  ```

---

### 6.2. Módulo de Evaluación Antropométrica
* **Ruta:** `POST /api/v1/evaluaciones`
* **Acceso:** Requiere Rol `RRHH_SALUD` o `PERITO_OPERATIVO`
* **Descripción:** Registra peso, talla y ejecuta la fórmula de IMC en Java[cite: 2].
* **Cuerpo de Solicitud (JSON):**
  ```json
  {
    "idUsuario": 10,
    "pesoKg": 78.50,
    "tallaM": 1.72,
    "observaciones": "Evaluación física periódica en turno matutino"
  }
  ```
* **Respuesta Exitosa (`201 Created`):**
  ```json
  {
    "idEvaluacion": 104,
    "fechaRegistro": "2026-09-27T08:30:00",
    "pesoKg": 78.50,
    "tallaM": 1.72,
    "imc": 26.54,
    "clasificacionImc": "Sobrepeso",
    "alertaGenerada": false
  }
  ```

---

### 6.3. Módulo de Pausas Activas y Rutinas Ergonómicas
* **Ruta:** `GET /api/v1/pausas/usuario/{idUsuario}/pendiente`
* **Acceso:** Perito Operativo
* **Descripción:** Consulta si el efectivo cuenta con una pausa de 3 minutos programada por tiempo estático continuo[cite: 2].
* **Respuesta Exitosa (`200 OK`):**
  ```json
  {
    "idRegistroPausa": 45,
    "tipo": "Cervical y Muñecas",
    "duracionSegundos": 180,
    "instrucciones": "Rotaciones de cuello sostenidas 15s y flexo-extensión de muñecas",
    "estado": "PENDIENTE"
  }
  ```

* **Ruta:** `PUT /api/v1/pausas/{idRegistroPausa}/completar`
* **Acceso:** Perito Operativo
* **Descripción:** Confirma la realización de la pausa activa y actualiza los indicadores de fatiga[cite: 2].
* **Respuesta Exitosa (`200 OK`):**
  ```json
  {
    "mensaje": "Pausa activa confirmada exitosamente",
    "nuevoNivelFatiga": 25.0
  }
  ```

---

### 6.4. Módulo de Reportería y Dashboard Gerencial
* **Ruta:** `GET /api/v1/dashboard/metricas`
* **Acceso:** Jefatura / RRHH
* **Descripción:** Provee los KPIs agregados para renderizar las tarjetas y mapas de calor analíticos[cite: 2].
* **Respuesta Exitosa (`200 OK`):**
  ```json
  {
    "peritosEvaluados": 48,
    "nivelFatigaPromedio": 42.0,
    "peritajesActivos": 127,
    "alertasCriticas": 3,
    "distribucionSalud": {
      "optimo": 65,
      "moderado": 25,
      "riesgo": 10
    }
  }
  ```

---

## 7. Instrucciones de Compilación y Ejecución

1. **Clonar el proyecto:**
   ```bash
   git clone [https://github.com/pnp-criminalistica/equilibra-t.git](https://github.com/pnp-criminalistica/equilibra-t.git)
   cd equilibra-t
   ```

2. **Compilar y empaquetar el proyecto (evitando tests durante build inicial):**
   ```bash
   mvn clean package -DskipTests
   ```

3. **Ejecutar el archivo JAR empaquetado:**
   ```bash
   java -jar target/equilibrat-1.0.0.jar
   ```

4. **Acceder a la aplicación:**  
   Abra un navegador web e ingrese a `http://localhost:8080/equilibrat`[cite: 2].
