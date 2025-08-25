## Publicar JAR de Spring Boot en GitHub Packages (Maven)

### Requisitos
- **Repo** en GitHub: `chapaxD/jenkins-docker`.
- **PAT** (Personal Access Token) con scopes: `write:packages`, `read:packages` y `repo` si el repo es privado.

### 1) Configurar `pom.xml`
Se agregó un perfil `github` con `distributionManagement` (publica releases y SNAPSHOT al mismo repositorio de GitHub Packages).

```40:56:pom.xml
  <profiles>
    <profile>
      <id>github</id>
      <activation>
        <property>
          <name>gpr.publish</name>
        </property>
      </activation>
      <distributionManagement>
        <repository>
          <id>github</id>
          <name>GitHub Packages</name>
          <url>https://maven.pkg.github.com/${gpr.owner}/${gpr.repo}</url>
        </repository>
        <snapshotRepository>
          <id>github</id>
          <name>GitHub Packages</name>
          <url>https://maven.pkg.github.com/${gpr.owner}/${gpr.repo}</url>
        </snapshotRepository>
      </distributionManagement>
    </profile>
  </profiles>
```

Notas:
- Puedes activar el perfil con `-Pgithub` o por propiedad con `"-Dgpr.publish=true"`.
- Para este proyecto usaremos `-Pgithub`.

### 2) Crear `%USERPROFILE%\.m2\settings.xml` (Windows)
Coloca las credenciales usando variables de entorno (no guardes el token en claro).

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>github</id>
      <username>${env.GITHUB_USERNAME}</username>
      <password>${env.GITHUB_TOKEN}</password>
    </server>
  </servers>
  <!-- opcional: mirrors, proxies, profiles globales -->
  
</settings>
```

### 3) Definir variables de entorno (PowerShell)
```powershell
$env:GITHUB_USERNAME = "chapaxD"
$env:GITHUB_TOKEN    = "ghp_XXXXXXXXXXXXXXXXXXXX"   # PAT con write:packages (+ repo si privado)
```

### 4) Publicar el artefacto
PowerShell puede interpretar mal propiedades con punto. Usa comillas o `--%`.

Forma usada (recomendada):
```powershell
mvn -Pgithub "-Dgpr.owner=chapaxD" "-Dgpr.repo=jenkins-docker" -DskipTests deploy
```

Alternativa con stop-parsing:
```powershell
mvn --% -Pgithub -Dgpr.owner=chapaxD -Dgpr.repo=jenkins-docker -DskipTests deploy
```

Si ya existiera la versión en GitHub Packages, incrementa `<version>` en `pom.xml`.

### 5) Verificar publicación
- En el repo: [Packages del repo](https://github.com/chapaxD/jenkins-docker/packages)
- En tu perfil: [Tus Packages filtrados por repo](https://github.com/users/chapaxD/packages?repo_name=jenkins-docker)

También puedes probar descarga:
```powershell
mvn dependency:get -Dartifact=com.example:demo:0.0.1-SNAPSHOT -DrepoUrl=https://maven.pkg.github.com/chapaxD/jenkins-docker -Dtransitive=false
```

### 6) Consumir el paquete desde otro proyecto Maven
En el `pom.xml` del consumidor, añade el repositorio:
```xml
<repositories>
  <repository>
    <id>github</id>
    <name>GitHub Packages</name>
    <url>https://maven.pkg.github.com/chapaxD/jenkins-docker</url>
  </repository>
  <!-- otros repos -->
</repositories>
```

Y en el consumidor asegúrate de tener el mismo `<server id="github">` en su `~/.m2/settings.xml`.

Dependencia de ejemplo:
```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</dependency>
```

### 7) Buenas prácticas y seguridad
- No compartas tu PAT. Si se expone, **revócalo** y genera uno nuevo.
- Usa variables de entorno para credenciales.
- Haz `git push` de los cambios del `pom.xml` y, si corresponde, crea tags para releases.


