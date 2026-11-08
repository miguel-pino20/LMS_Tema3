# Guía paso a paso de la práctica de agentes en Fedora 44

Basada en `18.pdf` y en la copia local de `Agent-Framework-Samples`. Revisada el 7 de noviembre de 2026.

## 1. Qué vas a hacer y qué problema tiene la guía original

La tarea consiste en ejecutar cuatro notebooks de **C#/.NET**, analizar sus grafos y agregar un agente traductor al primer caso. No se permite sustituirlos por Python. No necesitas Docker.

Hay dos cambios que afectan al PDF:

1. **GitHub Models fue retirado el 30 de julio de 2026.** Su API ya no está disponible. Crear un PAT nuevo o cambiar sus permisos no soluciona esto. Fuente: [documentación oficial de GitHub](https://docs.github.com/en/github-models).
2. La copia descargada actualmente contiene proyectos de consola en lugar de los notebooks exigidos. Además, sus `.csproj` tienen referencias de ejemplo como `Your_Microsoft.Agents.AI_Project_Path`: no están listos para ejecutar sin ajustes.

Esta guía permite preparar Fedora y recuperar los notebooks. También explica una adaptación para los casos 1–3 con Azure OpenAI, **condicionada a que el profesor acepte sustituir GitHub Models y te proporcione un despliegue compatible**. El caso 4 depende de servicios de agentes y herramientas adicionales en Azure: no basta con copiar el endpoint de un modelo.

No se han ejecutado los cuatro casos contra Azure ni validado tus permisos o recursos remotos. Los pasos con valores de ejemplo requieren tus datos reales.

## 2. Entender las tres variables del paso 2.5.4

| Variable original | Significado | De dónde se obtenía |
|---|---|---|
| `GITHUB_TOKEN` | Credencial que autentica la solicitud. | Un PAT de GitHub con acceso a modelos. |
| `GITHUB_ENDPOINT` | Dirección base de la API que recibe las solicitudes. | Documentación o ejemplo de conexión del servicio. |
| `GITHUB_MODEL_ID` | Identificador exacto del modelo que debe responder. | Catálogo o ejemplo de conexión del modelo. |

Una configuración **histórica, que ya no funciona**, era:

```dotenv
GITHUB_TOKEN=TU_PAT_DE_GITHUB
GITHUB_ENDPOINT=https://models.github.ai/inference
GITHUB_MODEL_ID=openai/gpt-4o-mini
```

El programa interpreta esos valores así: «Con esta credencial, conecta a esta API y utiliza este modelo».

Los notebooks antiguos usan también `https://models.inference.ai.azure.com` y nombres como `gpt-4o-mini`. Esos ejemplos pertenecen a otra etapa del servicio. No combines un endpoint antiguo con un identificador moderno pensando que eso garantiza compatibilidad.

Un token de GitHub no autentica contra Azure OpenAI. Cada servicio tiene sus propias credenciales. Si utilizas Azure, el valor del modelo suele ser **el nombre del deployment**, que puede diferir del nombre comercial del modelo.

## 3. Verificar las herramientas en Fedora

### 3.1. Abrir la carpeta de trabajo

En una terminal ejecuta:

```bash
cd "/home/mangel/Documentos/LMS/tema_3/TAREA PRACTICA"
pwd
```

`cd` cambia de carpeta. Las comillas son necesarias porque la ruta contiene espacios. `pwd` muestra dónde estás para evitar ejecutar los siguientes comandos en otra ubicación.

### 3.2. Comprobar .NET y Git

```bash
dotnet --version
dotnet --list-sdks
dotnet --list-runtimes
git --version
```

El SDK permite desarrollar y restaurar paquetes; el runtime ejecuta aplicaciones. Durante la revisión se encontró el SDK `10.0.113` y Git `2.56.0`.

No necesitas reinstalarlos si estos comandos funcionan. Si faltara el SDK de .NET 10:

```bash
sudo dnf install dotnet-sdk-10.0
```

`dnf` instala paquetes en Fedora. `sudo` solicita privilegios de administración. Este comando instala el SDK, pero no garantiza que un notebook antiguo o una versión de Polyglot funcione con cualquier versión del runtime.

Consulta oficial: [instalar .NET en Fedora](https://learn.microsoft.com/en-us/dotnet/core/install/linux-fedora).

### 3.3. Comprobar VS Code y sus extensiones

```bash
code --version
code --list-extensions
```

En VS Code también puedes abrir Extensiones con `Ctrl+Shift+X` y buscar:

- C# de Microsoft.
- C# Dev Kit.
- Polyglot Notebooks.

Durante la revisión esas extensiones ya estaban instaladas. Polyglot proporciona el kernel que interpreta las celdas C# de un notebook.

### 3.4. Comprobar Azure CLI e iniciar sesión

```bash
az --version
az login
az account show
```

`az login` abre el proceso de autenticación. `az account show` muestra la suscripción seleccionada. Iniciar sesión no crea un proyecto ni concede permisos por sí solo.

Si falla el inicio de sesión mediante navegador:

```bash
az login --use-device-code
```

Sigue la dirección y el código que indique Azure CLI. Si tienes varias suscripciones, selecciona la del curso:

```bash
az account set --subscription "ID_O_NOMBRE_DE_LA_SUSCRIPCION"
az account show
```

Sustituye el marcador por el valor real. Guarda una captura como evidencia del entorno, ocultando datos personales que no necesites entregar.

## 4. Recuperar los notebooks exigidos

Se localizaron en el historial local del repositorio, en el commit:

```text
b6a7fc707daf6487688af10cb94b8566177e0090
```

### 4.1. Verificar que el commit existe

Desde la carpeta de trabajo:

```bash
git -C Agent-Framework-Samples cat-file -t b6a7fc707daf6487688af10cb94b8566177e0090
```

Debe responder `commit`. `-C` indica a Git en qué repositorio trabajar sin cambiar tu carpeta actual.

### 4.2. Extraer una copia separada

Si todavía no existe la carpeta `practica-notebooks`, ejecuta:

```bash
mkdir practica-notebooks

git -C Agent-Framework-Samples archive \
  b6a7fc707daf6487688af10cb94b8566177e0090 \
  07.Workflow |
  tar -x -C practica-notebooks
```

Qué hace cada parte:

- `mkdir`: crea la carpeta de destino.
- `git archive`: obtiene los archivos de `07.Workflow` tal como estaban en ese commit.
- `|`: envía el archivo generado directamente a `tar`.
- `tar -x -C`: extrae el contenido en la carpeta indicada.

Esto conserva tu proyecto descargado. **Si ya extrajiste los archivos y los modificaste, no repitas la extracción encima de tu trabajo.**

### 4.3. Abrir la copia de la práctica

```bash
code practica-notebooks/07.Workflow
```

En el explorador de VS Code abre `code_samples/dotNET`. Debes encontrar:

```text
01.dotnet-agent-framework-workflow-ghmodel-basic.ipynb
02.dotnet-agent-framework-workflow-ghmodel-sequential.ipynb
03.dotnet-agent-framework-workflow-ghmodel-concurrent.ipynb
04.dotnet-agent-framework-workflow-aifoundry-condition.ipynb
```

La copia extraída incluye la imagen `code_samples/imgs/home.png`, necesaria para el caso 2.

## 5. Resolver el reemplazo de GitHub Models

Como la tarea exige GitHub Models para los casos 1–3, solicita al profesor una actualización antes de presentar una adaptación como cumplimiento literal del PDF.

Puedes usar este texto:

> GitHub Models fue retirado el 30 de julio de 2026. Recuperé los notebooks C# desde el historial del repositorio porque la versión actual contiene proyectos de consola. ¿Podemos ejecutar los casos 1–3 con Azure OpenAI en Foundry, conservando sus grafos? Necesito el deployment autorizado, el endpoint de inferencia y el método de autenticación. Para el caso 4 también necesito confirmar la compatibilidad con Persistent Agents, Bing Grounding y Code Interpreter del notebook antiguo.

Para la adaptación descrita en el apartado 8 necesitas:

- Un deployment de Azure OpenAI que admita Chat Completions; para el caso 2 también debe aceptar imágenes.
- El endpoint **Azure OpenAI v1** del recurso.
- Una clave de ese recurso, si el curso permite autenticación por clave y el recurso la tiene habilitada.
- El nombre exacto del deployment.

Si el curso exige identidad de Azure también para los casos 1–3, utiliza el ejemplo de autenticación que te proporcione. La adaptación con clave del apartado 8 no sirve para un recurso que tenga deshabilitadas las claves.

## 6. Crear y proteger el archivo `.env`

### 6.1. Ubicación elegida

Usa un archivo nuevo en:

```text
/home/mangel/Documentos/LMS/tema_3/TAREA PRACTICA/practica-notebooks/.env
```

Este archivo pertenece a la copia de la práctica. No es el `.env` que pueda existir en la carpeta del repositorio original.

Ábrelo con VS Code:

```bash
code "/home/mangel/Documentos/LMS/tema_3/TAREA PRACTICA/practica-notebooks/.env"
```

Si no existe, VS Code permitirá crearlo al guardar.

### 6.2. Contenido para la adaptación con Azure OpenAI

Solo cuando tengas los datos reales, utiliza:

```dotenv
AZURE_OPENAI_ENDPOINT=https://TU_RECURSO.openai.azure.com/openai/v1/
AZURE_OPENAI_API_KEY=TU_CLAVE_DEL_RECURSO
AZURE_OPENAI_DEPLOYMENT=NOMBRE_EXACTO_DEL_DEPLOYMENT

# Configuración separada para el notebook 04 antiguo:
AZURE_AI_PROJECT_ENDPOINT=ENDPOINT_DEL_PROYECTO_COMPATIBLE_CON_PERSISTENT_AGENTS
BING_CONNECTION_ID=ID_DE_LA_CONEXION_BING_DEL_PROYECTO
```

Los marcadores no son valores utilizables. Copia los datos del entorno del curso. El endpoint de inferencia del modelo y el endpoint de un proyecto de agentes tienen funciones diferentes; no los intercambies.

En este esquema el caso 4 usa tu sesión de Azure CLI, no la clave del modelo de los casos 1–3. La variable del deployment del caso 4 se ajusta por separado, como se explica más adelante.

### 6.3. Evitar que Git incluya el archivo

La carpeta extraída no tiene historial Git propio. Si decides convertirla en tu repositorio de entrega, ejecuta:

```bash
cd "/home/mangel/Documentos/LMS/tema_3/TAREA PRACTICA/practica-notebooks"
git init
```

Crea un archivo `.gitignore` en esa carpeta con:

```gitignore
.env
*.env
bin/
obj/
.ipynb_checkpoints/
```

Comprueba la exclusión:

```bash
git check-ignore -v .env
git status --short
```

El primer comando debe mostrar la regla que ignora `.env`. El segundo no debe listar `.env` como un archivo nuevo para añadir. Si ya estaba versionado, ignorarlo no lo elimina del historial; no publiques esa copia con secretos.

Opcionalmente restringe su lectura a tu usuario:

```bash
chmod 600 .env
```

No ejecutes `cat .env` al grabar una demostración ni incluyas sus valores en capturas.

## 7. Comprobar que los notebooks ejecutan C#

1. Abre el notebook 01.
2. Pulsa **Seleccionar kernel** en la parte superior.
3. Selecciona el kernel C# de **.NET Interactive / Polyglot Notebooks**. El texto puede variar según la versión.
4. Si VS Code solicita confiar en la carpeta, revisa el código y habilita la confianza para esta copia que has inspeccionado.
5. Agrega una celda temporal C#:

```csharp
Console.WriteLine("C# funciona en Fedora");
Console.WriteLine(System.IO.Directory.GetCurrentDirectory());
```

La primera línea verifica el intérprete. La segunda muestra el directorio del kernel, del que dependen las rutas relativas.

Después ejecuta las celdas originales desde arriba. Las instrucciones como:

```csharp
#r "nuget: DotNetEnv, 3.1.1"
```

descargan y cargan paquetes en el kernel. La primera ejecución puede tardar. Mantén inicialmente las versiones originales: los notebooks usan API preliminares antiguas y cambiar versiones puede romper los ejemplos.

Si el kernel requiere una versión adicional del runtime, sigue el diagnóstico concreto de Polyglot. Tener el SDK 10 instalado no demuestra por sí solo que todos los kernels y paquetes antiguos sean compatibles.

### Cargar `.env` sin depender del directorio del kernel

En los notebooks recuperados aparece:

```csharp
Env.Load("../../../.env");
```

Sustituye esa llamada por la ubicación absoluta de tu archivo:

```csharp
Env.Load("/home/mangel/Documentos/LMS/tema_3/TAREA PRACTICA/practica-notebooks/.env");
```

Ejecuta esa celda antes de leer las variables. Es un ajuste de configuración local; documéntalo junto con la adaptación aceptada por el profesor.

Comprueba una variable sin imprimir su contenido:

```csharp
Console.WriteLine(
    string.IsNullOrWhiteSpace(Environment.GetEnvironmentVariable("AZURE_OPENAI_API_KEY"))
        ? "Falta configurar la clave"
        : "Clave configurada");
```

## 8. Adaptar la conexión de los casos 1–3 a Azure OpenAI

**Este apartado es una alternativa técnica al servicio retirado. No forma parte literal del PDF y requiere un entorno compatible y aceptación del profesor.** No cambia el orden de los agentes ni las conexiones del grafo.

La API Azure OpenAI v1 admite el cliente `OpenAIClient` con un endpoint propio y una clave del recurso. Consulta: [endpoints y autenticación en C#](https://learn.microsoft.com/en-us/azure/developer/ai/how-to/switching-endpoints).

### 8.1. Sustituir la lectura de las variables

En cada notebook 01, 02 y 03 localiza la celda que obtiene `GITHUB_ENDPOINT`, `GITHUB_MODEL_ID` y `GITHUB_TOKEN`. Sustituye ese bloque por:

```csharp
var github_endpoint = Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")
    ?? throw new InvalidOperationException("Falta AZURE_OPENAI_ENDPOINT");

var github_model_id = Environment.GetEnvironmentVariable("AZURE_OPENAI_DEPLOYMENT")
    ?? throw new InvalidOperationException("Falta AZURE_OPENAI_DEPLOYMENT");

var github_token = Environment.GetEnvironmentVariable("AZURE_OPENAI_API_KEY")
    ?? throw new InvalidOperationException("Falta AZURE_OPENAI_API_KEY");
```

Se conservan los nombres locales `github_*` para que el resto del notebook siga encontrando sus variables. Ahora sus valores proceden de Azure. En particular, `github_token` contiene una clave de Azure, **no un PAT de GitHub**.

Los notebooks 02 y 03 antiguos fijan el modelo en `"gpt-4o"`. Sustituye también esa asignación por la lectura del deployment anterior; no dejes dos declaraciones de la misma variable.

### 8.2. Revisar la construcción del cliente

El bloque del cliente debe usar esas variables:

```csharp
var openAIOptions = new OpenAIClientOptions()
{
    Endpoint = new Uri(github_endpoint)
};

var openAIClient = new OpenAIClient(
    new ApiKeyCredential(github_token), openAIOptions);
```

Si ya existe un bloque equivalente, consérvalo. No agregues una segunda declaración con el mismo nombre. El notebook ya importa los espacios de nombres necesarios, entre ellos `OpenAI` y `System.ClientModel`.

El endpoint debe ser el de Azure OpenAI v1 de tu recurso, por ejemplo con el sufijo `/openai/v1/`. Un endpoint de proyecto de agentes no sustituye este valor.

### 8.3. Reiniciar antes de probar

Después de editar la configuración, reinicia el kernel y ejecuta las celdas desde el principio. Esto evita conservar variables de una ejecución anterior.

La compatibilidad del SDK y el servicio debe confirmarse con la primera ejecución real. Si falla por una API antigua, registra el error y utiliza la versión acordada con el profesor; no actualices paquetes al azar.

## 9. Caso 1: FrontDesk → Concierge

Corresponde al paso 5 del apartado 2.5 del PDF.

1. Abre el notebook 01 y selecciona C#.
2. Ejecuta los paquetes, las importaciones, la carga de `.env` y la conexión al modelo.
3. Ejecuta las celdas que definen las instrucciones y crean los agentes.
4. Localiza el grafo:

```csharp
var workflow = new WorkflowBuilder(frontDeskAgent)
    .AddEdge(frontDeskAgent, reviewerAgent)
    .Build();
```

`frontDeskAgent` es el punto inicial. `AddEdge` lleva su resultado a `reviewerAgent`, llamado Concierge. `Build` construye el workflow.

5. Ejecuta la entrada `I would like to go to Paris.` y las celdas que consumen los eventos.
6. Comprueba que aparece una recomendación y su revisión. El texto puede variar entre ejecuciones.
7. Guarda con `Ctrl+S` para conservar las salidas y toma una captura.

## 10. Caso 2: Sales → Price → Quote

Corresponde al paso 6 del PDF.

1. Abre el notebook 02 y configura su conexión como en el caso 1.
2. Utiliza un deployment que acepte imágenes. Un modelo solo de texto no permite reproducir esta entrada.
3. Localiza la variable `imgPath`. El notebook recuperado utiliza `../imgs/home.png`, una ruta relativa al directorio del kernel.
4. Para evitar confusiones, configura:

```csharp
var imgPath = "/home/mangel/Documentos/LMS/tema_3/TAREA PRACTICA/practica-notebooks/07.Workflow/code_samples/imgs/home.png";
```

5. Ejecuta la lectura de la imagen, la creación de agentes y el grafo.
6. Ejecuta la solicitud con texto e imagen.

Sales identifica muebles; Price propone precios; Quote organiza el resultado como una cotización en Markdown. Los precios son estimaciones generadas por el modelo, no una consulta garantizada a tiendas.

Guarda la salida y verifica que la cotización incluye los muebles detectados y los niveles de precio pedidos.

## 11. Caso 3: ejecución concurrente

Corresponde al paso 7 del PDF.

1. Abre el notebook 03, configura el modelo y ejecuta las celdas en orden.
2. Conserva la entrada `Plan a trip to Seattle in December`.
3. Localiza `AddFanOutEdge`, la conexión de reunión de ramas de esa versión y `WithOutputFrom`.
4. Comprueba la salida de Researcher-Agent y Plan-Agent, y la salida del agregador.

Fan-out distribuye trabajo a varias ramas. Fan-in reúne sus resultados. En este patrón los agentes trabajan en paralelo: no debes convertirlo en una cadena donde Plan espera primero la respuesta de Researcher.

La API actual usa nombres que pueden diferir del notebook antiguo, por ejemplo `AddFanInBarrierEdge`. Analiza el archivo que estás ejecutando, conservando su versión y su grafo.

Guarda el notebook con la salida agregada.

## 12. Caso 4: workflow condicional con Foundry

Corresponde a los pasos 8 y 9 del PDF.

### 12.1. Confirmar el entorno antes de ejecutar

El notebook recuperado utiliza `Azure.AI.Agents.Persistent`, `PersistentAgentsClient` y `AzureCliCredential`. También configura Bing Grounding y Code Interpreter.

Necesitas confirmar con el entorno del curso:

1. Un proyecto cuyo endpoint sea compatible con el SDK Persistent Agents usado por el notebook.
2. Un deployment de chat compatible con esos agentes y herramientas.
3. Permisos suficientes para crear y ejecutar agentes y acceder al modelo. El PDF menciona Foundry User; el administrador debe verificar los roles aplicables al recurso real.
4. Una conexión de Bing Grounding válida y su identificador de recurso.
5. Disponibilidad de Code Interpreter y acceso a los archivos generados.

No se puede garantizar que un proyecto recién creado con las API más recientes funcione con este notebook de 2025. Si el curso no ofrece un entorno compatible, necesitas la versión actualizada del caso 4.

### 12.2. Configurar el notebook

1. Ejecuta `az login` y verifica la suscripción con `az account show`.
2. Reinicia el kernel del notebook 04 si estaba abierto antes de iniciar sesión.
3. Carga el `.env` mediante la ruta absoluta del apartado 7.
4. Comprueba estas lecturas en la celda de configuración:

```csharp
var azure_foundry_endpoint = Environment.GetEnvironmentVariable("AZURE_AI_PROJECT_ENDPOINT")
    ?? throw new InvalidOperationException("Falta AZURE_AI_PROJECT_ENDPOINT");

var bing_conn_id = Environment.GetEnvironmentVariable("BING_CONNECTION_ID")
    ?? throw new InvalidOperationException("Falta BING_CONNECTION_ID");
```

El mensaje de error original del endpoint menciona otro nombre de variable, pero la lectura real usa `AZURE_AI_PROJECT_ENDPOINT`.

5. El notebook fija `azure_foundry_model_id` en `"gpt-4o-mini"`. Sustituye ese valor por el nombre real del deployment del curso. Para conservarlo fuera del código puedes agregar al `.env`:

```dotenv
AZURE_AI_MODEL_DEPLOYMENT=NOMBRE_REAL_DEL_DEPLOYMENT_DEL_CASO_4
```

Y cambiar la asignación por:

```csharp
var azure_foundry_model_id = Environment.GetEnvironmentVariable("AZURE_AI_MODEL_DEPLOYMENT")
    ?? throw new InvalidOperationException("Falta AZURE_AI_MODEL_DEPLOYMENT");
```

6. Ejecuta las celdas en orden. `AzureCliCredential` reutiliza tu sesión de `az login`; no requiere el PAT de GitHub.

### 12.3. Ejecutar la rama aprobada

Conserva primero la solicitud original de tutorial. El Evangelist redacta, el reviewer evalúa el contenido y una condición dirige el resultado.

Comprueba que la respuesta del reviewer es `Yes` y que se ejecuta PublishExecutor. El Publisher usa Code Interpreter: el Markdown puede generarse en el entorno remoto de la herramienta. No asumas que aparecerá automáticamente en tu carpeta de Fedora. Guarda o descarga el archivo mediante el mecanismo que exponga el entorno del curso.

Una respuesta que afirma «archivo guardado» no sustituye la evidencia del archivo: comprueba que existe y se puede recuperar.

### 12.4. Ejecutar la rama rechazada

Solicita un borrador de unas 50 palabras en la entrada del workflow, sin cambiar las condiciones ni los enlaces del grafo. El Evangelist tiene instrucciones de superar 200 palabras, por lo que podría ignorar esa petición.

Repite si fuera necesario, como permite el PDF. Si nunca produce un texto corto, documenta el conflicto de instrucciones y solicita el ajuste del ejemplo; no presentes una salida inventada como ejecución real.

Para la evidencia de rechazo debes ver `No`, el motivo y que Publisher no se ejecuta. Guarda evidencia de ambas ejecuciones por separado para que la segunda no sustituya a la primera.

## 13. Agregar Translator al caso 1

Corresponde al paso 10 del PDF. Hazlo después de guardar evidencia del caso 1 original.

En el notebook 01, después de crear FrontDesk y Concierge y antes de construir el grafo, agrega:

```csharp
AIAgent translatorAgent = openAIClient.GetChatClient(github_model_id).CreateAIAgent(
    name: "Translator",
    instructions: "Traduce al español el texto recibido conservando el tono.");
```

Ese código corresponde a la API del notebook histórico recuperado. Los proyectos actuales usan otros métodos, como `AsAIAgent`; no mezcles ambos ejemplos.

Sustituye la construcción del workflow por:

```csharp
var workflow = new WorkflowBuilder(frontDeskAgent)
    .AddEdge(frontDeskAgent, reviewerAgent)
    .AddEdge(reviewerAgent, translatorAgent)
    .Build();
```

La nueva conexión envía la revisión de Concierge al traductor. Reinicia el kernel, ejecuta desde el principio y conserva la entrada `I would like to go to Paris.`.

Comprueba la respuesta emitida por **Translator**, que debe estar en español. La salida general puede mostrar también mensajes anteriores en inglés: identifica el resultado del último agente.

Guarda el notebook modificado y una captura de la traducción.

## 14. Preparar la entrega

Una organización posible es:

```text
practica-notebooks/
├── .env                    # Solo local; no se entrega
├── .gitignore
├── README.md
├── evidencias/
└── 07.Workflow/
    └── code_samples/
        ├── dotNET/          # Notebooks con sus salidas
        └── imgs/
```

Crea `evidencias` desde la carpeta `practica-notebooks`:

```bash
mkdir -p evidencias
```

Guarda evidencias de las versiones, el kernel C#, los tres primeros casos, el deployment, ambas ramas del caso 4 y Translator. No incluyas claves, tokens ni pantallas de credenciales. Revisa también las salidas guardadas dentro de cada notebook.

En tu `README.md` explica:

1. Fedora y versiones utilizadas.
2. El commit de origen de los notebooks.
3. Los comandos y ajustes de rutas ejecutados.
4. La sustitución de GitHub Models y la aceptación del profesor, si corresponde.
5. Los nombres de los recursos Azure utilizados, sin secretos.
6. El resultado observado en cada caso y el archivo de evidencia asociado.
7. Las respuestas conceptuales:

| Concepto | Explicación |
|---|---|
| Executor | Unidad de procesamiento que recibe mensajes y realiza una tarea. Un agente puede actuar como executor. |
| Edge simple | Conexión que dirige mensajes al siguiente executor sin una condición adicional de enrutamiento. |
| Edge condicional | Conexión que solo se recorre cuando se cumple su condición. |
| Fan-out | Distribución del trabajo hacia varias ramas. |
| Fan-in | Reunión de los resultados de varias ramas. |
| Workflow | Grafo que organiza los executors y sus conexiones y coordina su ejecución. |

## 15. Resolver errores frecuentes

| Error o síntoma | Qué revisar |
|---|---|
| No aparece el kernel C# | Polyglot instalado y habilitado, confianza de la carpeta y diagnóstico del runtime requerido. |
| Fallan las celdas `#r "nuget: ..."` | Conexión con NuGet y versión exacta del paquete. No son comandos de terminal. |
| Variable no configurada | Ruta de `.env`, nombre exacto de la variable y orden de ejecución. |
| Sigue utilizando valores anteriores | Reinicia el kernel y ejecuta todas las celdas en orden. |
| HTTP 401 | Credencial incorrecta, caducada o perteneciente a otro servicio; revisa también el método de autenticación. |
| HTTP 403 | Permisos, política del recurso, acceso de red o autenticación por clave deshabilitada. |
| HTTP 404 o deployment no encontrado | Endpoint de la API, nombre exacto del deployment y recurso donde está desplegado. |
| HTTP 429 | Límite de solicitudes, tokens o cuota. Evita relanzar continuamente. |
| No se encuentra `home.png` | Ruta y mayúsculas/minúsculas. Fedora distingue `dotNET` de `dotnet`. |
| El modelo rechaza la imagen | Deployment sin soporte de visión o entrada incompatible. |
| `Your_Microsoft...Project_Path` no existe | Estás ejecutando el proyecto actual de consola con referencias pendientes, no el notebook recuperado. |
| Bing o Code Interpreter fallan | Herramienta, conexión, permisos, región y compatibilidad del proyecto del caso 4. |
| Una clase o un método no existe | Mezcla de versiones del SDK o de ejemplos. Conserva la versión acordada para la práctica. |
| GitHub Models no responde | Servicio retirado; cambiar el PAT no lo restablece. |

Al pedir ayuda, comparte la celda que falla y el error completo, ocultando credenciales. No publiques `.env`.

## 16. Limpieza al terminar

El PDF exige limpiar los recursos utilizados. Hazlo después de guardar y comprobar tu entrega:

1. Revoca el PAT de GitHub si lo creaste solo para esta práctica.
2. Elimina el `.env` local cuando ya no lo necesites. Si adaptaste los casos con una clave compartida de Azure, no la regeneres sin coordinación con el administrador.
3. Borra únicamente los agentes creados por ti para el caso 4. Repetir celdas que crean agentes puede generar varios recursos.
4. Elimina el deployment si se creó exclusivamente para el reto y ya no lo utiliza nadie.
5. Elimina el grupo de recursos solo si es exclusivo de tu práctica.

No borres recursos compartidos del curso. La autenticación con `az login` por sí sola no crea recursos, y el cierre de VS Code no elimina los recursos remotos.

## 17. Lista de comprobación

- [ ] Herramientas verificadas y kernel C# funcionando.
- [ ] Notebooks recuperados sin sobrescribir el proyecto original.
- [ ] Reemplazo de GitHub Models acordado con el profesor.
- [ ] `.env` cargado correctamente, ignorado por Git y fuera de la entrega.
- [ ] Caso 1 original ejecutado.
- [ ] Caso 2 ejecutado con imagen y cotización.
- [ ] Caso 3 ejecutado con resultados agregados.
- [ ] Entorno y herramientas del caso 4 compatibles con el notebook.
- [ ] Rama `Yes` ejecutada y Markdown recuperado.
- [ ] Rama `No` ejecutada sin Publisher.
- [ ] Translator ejecutado y respuesta final en español.
- [ ] Notebooks guardados con salidas y evidencias revisadas.
- [ ] README completado y recursos exclusivos limpiados al finalizar.

## Fuentes

- `18.pdf`, archivo local con la especificación de la tarea.
- Repositorio local `Agent-Framework-Samples`, copia actual e historial del commit `b6a7fc707daf6487688af10cb94b8566177e0090`.
- [Retirada de GitHub Models](https://docs.github.com/en/github-models).
- [Instalación de .NET en Fedora](https://learn.microsoft.com/en-us/dotnet/core/install/linux-fedora).
- [Conexión a endpoints de Azure OpenAI con C#](https://learn.microsoft.com/en-us/azure/developer/ai/how-to/switching-endpoints).
