# SESION DE LABORATORIO N° 01: GESTION AUTOMATIZADA DE PRUEBAS CON GITHUB

## OBJETIVOS
  * Desarrollar la automatización de la gestión de pruebas de una aplicación utilizando Github Actions.

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de Bash (powershell).
    - Conocimientos básicos de Contenedores (Docker).
    - Conocimientos básicos de lenguaje YAML.
  * Hardware:
    - Virtualization activada en el BIOS..
    - CPU SLAT-capable feature.
    - Al menos 4GB de RAM.
  * Software:
    - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior)
    - Docker Desktop 
    - Powershell versión 7.x
    - Net 8 o superior
    - Visual Studio Code

## CONSIDERACIONES INICIALES
  * Clonar el repositorio mediante git para tener los recursos necesarios
  * Tener una cuenta de Github valida. 

## DESARROLLO
1. Abrir un navegador de internet e ingrear a la pagina de SonarCloud (https://www.sonarsource.com/products/sonarcloud/), iniciar sesión con su cuenta de Github.
2. En el navegador de internet, en la pagina de SonarCloud, generar un nuevo token con el nombre que desee, luego de generar el token, guarde el resultado en algún archivo o aplicación de notas. Debido a que se utilizará mas adelante.
3. En el navegador de internet, en la pagina de SonarCloud, hacer click en el icono + y luego en la opción *Analyze projects*. En la ventana de Analyze Projects, seleccionar la opción *create a project manually* para crear un proyecto de manera manual.
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/17b92d10-c2ca-4f7f-90d5-919c0b27ca6b)

4. En el navegador de internet, en la pagina de SonarCloud, en la pagina de nuevo proyecto ingresar el nombre *BancaApp*, tomar nota del valor generado en el cuadro Project Key que sera utilizado mas adelante, confirmar la creación del proyecto haciendo click en el boton Next.
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/570d2cb9-a6d4-4629-a981-8408c308dc1e)

5. En el navegador de internet, en la pagina de SonarCloud, en la pagina de *Set up your project or Clean as You Code*, seleccionar la opción *Previuos version*, confirmar la creación del proyecto haciendo click en el boton Create Project.
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/3d7c6776-e79e-4f68-bd40-5a1175c0b150)

5. En el navegador de internet, ingresar a la pagina Github del repositorio de su proyecto. En la sección Code, crear la rama *bddreporte*
   ![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/abbdaa3b-1af8-4d6e-b693-4f83e443d20b)

6. En el navegador de internet, en pagina Github del repositorio de su proyecto. En la sección Settings, ingresar a la opción Pages y en Branch seleccionar la rama recientemente creada, seguidamente hacer click en el botón *Save*.
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/e5c84d72-0b80-4f10-83b8-bed3619d1101)

7. En el navegador de internet, en pagina Github del repositorio de su proyecto. En la sección Settings, en la opción Pages despues de unos minutos aparecerá la url publica del proyecto. Tomar nota de esa dirección que sera utilizada mas adelante.
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/23f7c00f-9709-4442-b84f-9323ecfe744f)

8. En el navegador de internet, en pagina Github del repositorio de su proyecto. En la sección Settings, ingresar a la opción Secrets and variables y luego en la opción Actions, hacer click en el botón *New repository secret*.
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/19bf5a41-1b5f-4664-86cc-c821fcc01551)
  
9. En el navegador de internet, en pagina Github del repositorio de su proyecto. En la pagina de Actions secrets / New Secret, en el nombre ingresar el valor SONAR_TOKEN y en secreto ingresar el valor del token de SonarCloud generado en el paso 2.
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/3320bc5c-32c8-4f4c-bbcb-5852909d522c)

10. Abrir Visual Studio Code, cargar la carpeta del repositorio del proyecto. Seguidamente crear la carpeta `.github` y dentro de esta la carpeta `workflows`. Seguidamente crear el archivo ci.yml con el siguiente contenido
```Yaml
name: Tarea Automatizada de ejecución de pruebas

env:
  DOTNET_VERSION: '8.x'                     # la versión de .NET
  SONAR_ORG: 'p-cuadros'                    # Nombre de la organización de sonar cloud
  SONAR_PROJECT: 'p-cuadros_bancaapp'        # Key ID del proyecto de sonar
on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  coverage:
    name: CoverageReport
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configurando la versión de NET
        uses: actions/setup-dotnet@v4
        with:
          java-version: ${{ env.DOTNET_VERSION }}
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - name: Configurando la versión de NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}
      - name: Restaurar los paquetes
        run: dotnet restore 
      - name: Ejecutar pruebas
        run: dotnet test --collect:"XPlat Code Coverage"
      - name: ReportGenerator
        uses: danielpalme/ReportGenerator-GitHub-Action@5.3.7
        with:
          reports: ./*/*/*/coverage.cobertura.xml
          targetdir: coveragereport
          reporttypes: MarkdownSummary;MarkdownAssembliesSummary;MarkdownSummaryGithub
      - name: Upload coverage report artifact
        uses: actions/upload-artifact@v4
        with:
          name: CoverageReport 
          path: coveragereport 
      - name: Publish coverage in build summary # 
        run: cat coveragereport/SummaryGithub.md >> $GITHUB_STEP_SUMMARY 
        shell: bash
      - name: Instalar Scanner
        run: dotnet tool install -g dotnet-sonarscanner
      - name: Ejecutar escaneo
        run: | 
          dotnet-sonarscanner begin /k:"${{ env.SONAR_PROJECT }}" /o:"${{ env.SONAR_ORG }}" /d:sonar.login="${{ secrets.SONAR_TOKEN }}" /d:sonar.host.url="https://sonarcloud.io"
          dotnet build
          dotnet-sonarscanner end /d:sonar.login="${{ secrets.SONAR_TOKEN }}"
      - name: Install Living Doc
        run: dotnet tool install -g SpecFlow.Plus.LivingDoc.CLI
      - name: Generate living doc
        run: livingdoc test-assembly ./Bank.Domain.Tests/bin/Debug/net8.0/Bank.Domain.Tests.dll -t ./Bank.Domain.Tests/bin/Debug/net8.0/TestExecution.json -o ./report/index.html
      - uses: actions/upload-artifact@v3
        with:
          name: specflow
          path: report
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: bddreporte
          publish_dir: ./report/
```
11. En el Visual Studio Code o en un terminal, subir o confirmar los cambios al repositorio.
12. En el navegador de internet, en pagina Github del repositorio de su proyecto. En la sección Actions, se podra visualizar el siguiente resultado.
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/a1534320-2438-4161-84ca-36143c045c53)

13. En el navegador de internet, en la pagina de SonarCloud, en el nuevo proyecto creado se podra visualizar el resultado de la ejecución
![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/8ea2ac22-b40b-4067-9bf0-4b972a5eb313)

14. En el navegador de internet, ingresar a la direcciòn generada en el paso 7, de Github Pages y se podrá visualizar el siguiente resultado:

![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/a4f9ad6e-6aa1-424b-a496-9a16cad2ac2d)

![image](https://github.com/UPT-FAING-EPIS/lab_ci_pruebas_01/assets/10199939/33266cc0-98e6-4585-a863-8cefd2748dab)

