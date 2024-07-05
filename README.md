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





### Parte II: Creación de la aplicación
1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o Bank
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd Bank
dotnet new webapi -o Bank.WebApi
dotnet sln add ./Bank.WebApi/Bank.WebApi.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new mstest -o Bank.WebApi.Tests
dotnet sln add ./Bank.WebApi.Tests/Bank.WebApi.Tests.csproj
dotnet add ./Bank.WebApi.Tests/Bank.WebApi.Tests.csproj reference ./Bank.WebApi/Bank.WebApi.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Bank.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. En VS Code, en el proyecto Bank.WebApi proceder la carpeta `Models` y dentro de esta el archivo BankAccount.cs e introducir el siguiente código:
```C#
namespace Bank.WebApi.Models
{
    public class BankAccount
    {
        private readonly string m_customerName;
        private double m_balance;
        private BankAccount() { }
        public BankAccount(string customerName, double balance)
        {
            m_customerName = customerName;
            m_balance = balance;
        }
        public string CustomerName { get { return m_customerName; } }
        public double Balance { get { return m_balance; }  }
        public void Debit(double amount)
        {
            if (amount > m_balance)
                throw new ArgumentOutOfRangeException("amount");
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance -= amount;
        }
        public void Credit(double amount)
        {
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance += amount;
        }
    }
}
```
7. Luego en el proyecto Bank.WepApi.Tests añadir un nuevo archivo BanckAccountTests.cs e introducir el siguiente código:
```C#
using Bank.WebApi.Models;
using NUnit.Framework;

namespace Bank.Domain.Tests
{
    public class BankAccountTests
    {
        [Test]
        public void Debit_WithValidAmount_UpdatesBalance()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 4.55;
            double expected = 7.44;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);
            // Act
            account.Debit(debitAmount);
            // Assert
            double actual = account.Balance;
            Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly");
        }
    }
}
```
8. En el terminal, ejecutar las pruebas de lo nostruiido hasta el momento:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
> Resultado
```Bash
Failed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: < 1 ms
```
9. En el terminal, instalar la herramienta de .Net Sonar Scanner que permitirá conectarse a SonarQube para realizar las pruebas estáticas de la seguridad del código de la aplicación :
```Bash
dotnet tool install -g dotnet-sonarscanner
```
> Resultado
```Bash
Puede invocar la herramienta con el comando siguiente: dotnet-sonarscanner
La herramienta "dotnet-sonarscanner" (versión 'X.X.X') se instaló correctamente
```
10. En el terminal, ejecutar :
```Bash
dotnet sonarscanner begin /k:"apibank" /d:sonar.token="TOKEN" /d:sonar.host.url="https://sonarcloud.io" /o:"ORGANIZATION" /d:sonar.cs.opencover.reportsPaths="*/*/*/coverage.opencover.xml"
```
> Donde:
> - TOKEN: es el token que previamente se genero en la pagina de Sonar Source
> - ORGANIZATION: es el nombre de la organización generada en Sonar Source

12. En el terminal, ejecutar:
```Bash
dotnet build --no-incremental
dotnet test --collect:"XPlat Code Coverage;Format=opencover"
```
13. Ejecutar nuevamente el paso 8 para lo cual se obtendra una respuesta similar a la siguiente:
```
dotnet sonarscanner end /d:sonar.token="TOKEN"
```
17. En la pagina de Sonar Source verificar el resultado del analisis.

![image](https://github.com/UPT-FAING-EPIS/lab_calidad_01/assets/10199939/4e4892d3-71e2-4437-9713-a270ebf61b06)

---
## Actividades Encargadas
1. Adicionar los escenarios, casos de prueba, metodos de prueba y modificaciones para verificar el método de crédito.
2. Adjuntar la captura donde se evidencia el incremento del valor de cobertura en un archivo cobertura.png.
