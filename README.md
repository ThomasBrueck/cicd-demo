## Flujo Avanzado de CI/CD Pipeline (Jenkins)

Este proyecto implementa un Pipeline de Integración y Despliegue Continuo (CI/CD) validado a través de Jenkins. El flujo de trabajo está compuesto documentadamente por las siguientes etapas:

1. **Checkout (SCM):** Se obtiene el código fuente actualizado del repositorio Git apuntando a la rama `master`.
2. **Build & Test:** Jenkins utiliza Maven (`mvn clean package -DskipTests`) para compilar el proyecto y producir el archivo ejecutable. Inmediatamente, la ejecución se aísla únicamente para Pruebas Unitarias empleando `mvn test -Dgroups=UnitTest`.
3. **Análisis Estático (SonarQube):** Se ejecuta el análisis de calidad con `sonar-maven-plugin` conectado al Servidor y Plugin Oficiales de Jenkins. La etapa incluye una **Puerta de Calidad (Quality Gate)** estricta usando Webhooks (`waitForQualityGate`), pausando la ejecución y provocando el aborto/fallo automático e instantáneo del Job en caso de que exista un "Security Hotspot".
4. **Container Security Scan (Trivy):** Tras empaquetar una imagen Docker preliminar, el binario nativo de Trivy instalado en Jenkins la escanea en búsqueda de brechas de software en el framework. Cuenta con el **Gatekeeping Estricto activado (`--exit-code 1 --severity CRITICAL`)**, el cual fue exitosamente probado deteniendo todo el contenedor y rebotando el código antes de permitir enviar versiones de librerías en riesgo ('*fail-fast*').
5. **Deploy:** Esta etapa únicamente se despliega si las puertas de calidad anteriores validan la seguridad. Deshace instancias del proyecto anteriores y levanta el contendor aislando el puerto *8081* para proteger al anfitrión.
6. **Limpieza e Infraestructura (`post { always }`):** Como medida integral, Jenkins retorna siempre los contenedores remanentes a un estado consistente limpiando las cachés, redes e instanciaciones atómicas olvidadas (`docker system prune -f`) y eliminando enteramente el directorio clonado post-build (`cleanWs()`). De esta manera se evita la saturación residual de la memoria local en iteraciones a largo plazo.


------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


# CICD-DEMO

This project aims to be the basic skeleton to apply continuous integration and continuous delivery.

## Topology

CICD Demo uses some kubernetes primitives to deploy:

* Deployment
* Services
* Ingress ( with TLS )

```bash
     internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
   --|-----|--
   [   Pods   ]

```

This project includes:

* Spring Boot java app
* Jenkinsfile integration to run pipelines
* Dockerfile containing the base image to run java apps
* Makefile and docker-compose to make the pipeline steps much simpler
* Kubernetes deployment file demonstrating how to deploy this app in a simple Kubernetes cluster

## Pipeline Setup

Pipelines exist at Travis.

Some pipelines are configured by **GitHub/Projects**. If you have created a repository in one of these, your project will be **automatically** built if it has a Jenkinsfile/Travis/Gitlab/CircleCI.

Other pipelines are configured manually under folders. You can create a project manually with the following steps:

How to run the app:

```make
make
```

## Testing

Unit tests and integrations tests are separated using [JUnit Categories][].

[JUnit Categories]: https://maven.apache.org/surefire/maven-surefire-plugin/examples/junit.html

### Unit Tests

```java
mvn test -Dgroups=UnitTest
```

Or using Docker:

```bash
make build
```

### Integration Tests

```java
mvn integration-test -Dgroups=IntegrationTests
```

Or using Docker:

```bash
make integrationTest
```

### System Tests

System tests run with Selenium using docker-compose to run a [Selenium standalone container][] with Chrome.

[Selenium standalone container]: https://github.com/SeleniumHQ/docker-selenium

Using Docker:

* If you are running locally, make sure the `$APP_URL` is populated and points to a valid instance of your application. This variable is populated automatically in Jenkins.

```bash
APP_URL=http://dev-cicd-demo-master.anzcd.internal/ make systemTest
```