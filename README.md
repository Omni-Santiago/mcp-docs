
# Documentación para la Implementación de MCPs

Esta guía proporciona información esencial para implementar y trabajar con MCPs (Model Context Protocol) en Visual Studio Code nueva funcionalidad implementada dentro de Github Copilot, incluyendo la activación de opciones, el modo agente, el funcionamiento general, la configuración avanzada y la resolución de errores comunes.

---

## Índice

1. [¿Qué es MCP y cómo funciona?](#que-es-mcp-y-como-funciona)
2. [¿Qué hace el MCP?](#que-hace-el-mcp)
3. [Configuración de MCP: mcp.json](#configuracion-de-mcp-mcpjson)
   - [Configuración a nivel de proyecto](#configuracion-a-nivel-de-proyecto)
   - [Configuración global](#configuracion-global)
   - [Ejemplos de configuración: STDIO, Docker, SSO](#ejemplos-de-configuracion-stdio-docker-sso)
4. [Comparación MCP vs API tradicional](#comparacion-mcp-vs-api-tradicional)
5. [Cómo activar las opciones de MCP](#como-activar-las-opciones-de-mcp)
6. [Cómo activar el modo agente en VS Code](#como-activar-el-modo-agente-en-vs-code)
7. [Errores comunes y soluciones](#errores-comunes-y-soluciones)
8. [Recursos adicionales](#recursos-adicionales)
---



## ¿Qué es MCP y cómo funciona?

MCP (Model Context Protocol) es un protocolo abierto y extensible diseñado para estandarizar la comunicación entre entornos de desarrollo (como VS Code) y modelos de inteligencia artificial, ya sean locales o remotos. Su objetivo es permitir que los modelos reciban contexto relevante del usuario y del proyecto, además de eso permite que procesen solicitudes complejas y devuelvan respuestas estructuradas o acciones automatizables. Permite también interactuar con otros sistemas a partir de paramétros que los puede definir la inteligencia artificial mediante contexto que le pasa el usuario.

### Principios clave de MCP

- **Contextualidad:** El protocolo permite enviar información rica sobre el estado del editor, archivos abiertos, selección de código, historial de comandos y otros sistemas con los que puede interactuar, para que el modelo pueda responder de manera precisa y relevante dependiendo de las necesidades del usuario.
- **Bidireccionalidad:** No solo se envían preguntas o prompts, sino también comandos, acciones y resultados, permitiendo flujos de trabajo interactivos y automatizados.
- **Extensibilidad:** MCP soporta la definición de nuevos tipos de mensajes, acciones y eventos, adaptándose a las necesidades de cada equipo o proyecto.
- **Agnóstico de transporte:** MCP puede funcionar sobre diferentes mecanismos de comunicación: STDIO, sockets, HTTP, Docker, etc.

### Flujo de funcionamiento

1. El usuario realiza una solicitud al modelo, puede ser explicito sobre la herramienta que necesita que use el modelo (por ejemplo, selecciona código, ejecuta un comando o trae un resultado de una base de datos).
2. El editor recopila el contexto relevante (archivos, entorno, selección, historial, variables de entorno, etc.).
3. El contexto y la solicitud se envían al modelo a través del canal configurado (STDIO, Docker, SSO, etc.).
4. El modelo procesa la información y responde con sugerencias, acciones, documentación, o incluso comandos para ejecutar en el entorno.
5. El editor interpreta la respuesta y la presenta al usuario, permitiendo ejecutar acciones sugeridas o continuar el flujo. Ayudando a tener contextos más enriquecidos para otorgar mejores respuestas.

---

## ¿Qué hace el MCP?

MCP actúa como un "puente" de comunicación entre herramientas, prompts, implementaciones con otros sistemas e  y los modelos de IA, permitiendo:

- **Recepción de contexto enriquecido:** El modelo puede recibir información detallada sobre el estado del proyecto, archivos, entorno, configuración, resultados de busqueda en bases de datos, interectuar con otros sistemas, historial de comandos y más.
- **Ejecución de acciones automatizadas:** El modelo puede sugerir y ejecutar acciones como refactorizaciones, generación de código, documentación automática, análisis estático, subir código a Github, pruebas, despliegues, etc.
- **Comunicación segura y flexible:** MCP soporta múltiples mecanismos de autenticación y transporte, permitiendo operar en entornos locales, en la nube o híbridos.
- **Personalización avanzada:** Se pueden definir flujos de trabajo, comandos y respuestas personalizadas para cada equipo, proyecto o usuario.
- **Integración con herramientas externas:** MCP puede interactuar con sistemas de CI/CD, gestores de incidencias, bases de datos, y otros servicios, orquestando tareas complejas desde el editor.

---


## Configuración de MCP: mcp.json

La configuración de MCP se realiza mediante un archivo `mcp.json` dentro de la carpeta .vscode, que define cómo se ejecutan las herramientas y que paramétros son necesarios para que funcionen correctamente, así como las opciones de autenticación, variables de entorno y parámetros de ejecución. Este archivo puede ubicarse a nivel de proyecto (en la raíz del repositorio) o a nivel global (en la configuración de usuario de VS Code).

### Configuración a nivel de proyecto

Coloca un archivo `mcp.json` para definir un modelo específico o configuración personalizada para ese repositorio. Ejemplo:

```json
{
  "mcp": {
    "inputs": [],
    "servers": {
      "adobe-commerce-mcp": {
        "type": "stdio", // Tipo de proveedor: puede ser "stdio", "docker", "sso", etc.
        "command": "npx", // Comando principal a ejecutar para levantar el servidor MCP
        "args": [
          "-y",
          "@rafaelcg/adobe-commerce-dev-mcp@latest"
        ],
        "env": {
          "API_KEY": "tu_api_key"
        }
      }
    }
  }
}
```

**Explicación de los campos y su origen:**
- `type`: Define el mecanismo de ejecución del servidor MCP. Puede ser `stdio` (proceso local), `docker` (contenedor), `sso` (autenticación corporativa), entre otros. Este campo determina cómo se inicia y comunica el modelo con el editor.
- `command`: Es el comando principal que se ejecuta para iniciar el servidor MCP. Por ejemplo, puede ser `npx`, `pipx`, `node`, o cualquier ejecutable compatible con el entorno que solo lo ejecute temporalmente mientras se utiliza.
- `args`: Lista de argumentos que se pasan al comando principal. Permite especificar versiones, parámetros de inicialización, etc.
- `env`: Variables de entorno necesarias para la ejecución del modelo, como claves API, tokens de autenticación, configuración de endpoints, etc. Se recomienda no versionar información sensible.

> El archivo `mcp.json` puede ubicarse en la carpeta `.vscode` del proyecto para una configuración específica, o en la configuración global de usuario para aplicar a todos los proyectos. La estructura y los campos pueden variar según la extensión MCP utilizada, pero los conceptos clave son los mismos: definir cómo se ejecuta en el modelo, con qué parámetros y bajo qué condiciones.

### Configuración global

Puedes definir una configuración global en la carpeta de usuario de VS Code (`~/.config/Code/User/mcp.json` en Linux, `%APPDATA%/Code/User/mcp.json` en Windows, o `~/Library/Application Support/Code/User/mcp.json` en macOS). Ejemplo:

Esto permite tener un modelo por defecto para todos los proyectos, que puede ser sobrescrito por un `mcp.json` local.

### Ejemplos de configuración: STDIO, Docker, SSO

- **STDIO**: Ejecuta un proceso local que se comunica por entrada/salida estándar. Útil para modelos escritos en cualquier lenguaje que puedan ejecutarse como proceso hijo.
```json
{
  "mcp": {
    "servers": {
      "mi-mcp-stdio": {
        "type": "stdio",
        "command": "node",
        "args": ["mcp-server.js"],
        "env": {
          "NODE_ENV": "production"
        }
      },
      "mi-mcp-docker": {
        "type": "docker",
        "image": "mi-modelo-mcp:latest",
        "options": {
          "autoRemove": true
        }
      },
      "mi-mcp-sso": {
        "type": "sso",
        "clientId": "tu-client-id",
        "tenant": "tu-tenant",
        "scopes": ["openid", "profile"]
      }
    }
  }
}
```

En la integración con Github Copilot, los MCPs se inicializan a través de la sección `servers` dentro del objeto `mcp` en el archivo de configuración. Cada servidor MCP puede tener un tipo distinto (`stdio`, `docker`, `sso`, etc.) y se define con los parámetros necesarios para su ejecución. Github Copilot detecta y gestiona estos servidores según la configuración, permitiendo que cada herramienta o flujo de trabajo utilice el servidor adecuado según el contexto y la necesidad del usuario.

> Puedes combinar opciones y definir múltiples proveedores según las necesidades de tu entorno. Consulta la documentación de tu extensión MCP para más detalles sobre los campos soportados.

---

## Comparación MCP vs API tradicional

| Característica             | MCP (Model Context Protocol)                                                                                                   | API Tradicional (REST/HTTP)                                              |
|---------------------------|------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------|
| Contexto dinámico         | Sí. Transmisión de contexto completo: archivos abiertos, selección, variables de entorno, historial, configuración del workspace. | Limitado. Solo datos explícitos enviados en la petición, sin awareness del entorno. |
| Integración IDE           | Integración nativa y bidireccional con el IDE (VS Code), permitiendo acciones, comandos y feedback en tiempo real.              | Indirecta. Integración mediante plugins o extensiones que consumen endpoints HTTP.   |
| Acciones automatizadas    | Sí. El modelo puede sugerir y ejecutar comandos, refactorizaciones, generación de código, despliegues, etc., directamente en el editor. | No. Solo devuelve datos; la automatización debe implementarse manualmente en el cliente. |
| Configuración flexible    | Altamente flexible. Soporta múltiples servidores, tipos de transporte (`stdio`, `docker`, `sso`, etc.), y configuración granular por proyecto o global. | Limitada. Depende de los parámetros soportados por la API y su backend.             |
| Seguridad                 | Soporta autenticación avanzada (SSO, OAuth2), aislamiento de procesos (Docker), gestión de secretos por entorno, y control de permisos a nivel de acción. | Depende de la API. Generalmente autenticación por token o API key, sin aislamiento de ejecución. |
| Ejecución local/remota    | Soporta ejecución local (proceso hijo, Docker), remota (HTTP, SSO) o híbrida. Permite trabajar offline o en entornos restringidos. | Generalmente remota. Requiere acceso a internet y depende de la disponibilidad del backend. |
| Extensibilidad            | Muy alta. Permite definir nuevos tipos de mensajes, acciones, flujos personalizados y orquestación de herramientas externas.      | Baja. Limitada al contrato y endpoints definidos por el proveedor de la API.         |

MCP está diseñado para integrarse profundamente con el flujo de trabajo del desarrollador, permitiendo automatización, personalización y seguridad avanzadas. Una API tradicional suele ser más genérica, menos interactiva y no permite acciones directas sobre el entorno de desarrollo.

---

## Cómo activar las opciones de MCP

---

## Cómo activar las opciones de MCP

1. Abre la configuración de VS Code (`Ctrl + ,` o `Cmd + ,`).
2. Busca "MCP" en la barra de búsqueda.
3. Activa las opciones relevantes, como:
   - **Habilitar MCP**: Permite la comunicación con el modelo.
   - **Mostrar sugerencias automáticas**: Activa la aparición de sugerencias contextuales.
   - **Permitir acciones automatizadas**: Habilita la ejecución de acciones sugeridas por el modelo.
4. Guarda los cambios y reinicia VS Code si es necesario.

---

## Cómo activar el modo agente en VS Code

El modo agente permite que el modelo tome un rol más proactivo, sugiriendo y ejecutando acciones automáticamente.

1. Ve a la configuración de VS Code.
2. Busca la opción "Modo agente" o "Agent Mode".
3. Actívala para permitir que el modelo actúe como agente.
4. Puedes ajustar el nivel de autonomía del agente según tus preferencias.

> **Nota:** El modo agente puede requerir permisos adicionales o la instalación de extensiones específicas.

---

## Errores comunes y soluciones

### 1. No se reciben sugerencias del modelo
- **Solución:** Verifica que MCP esté habilitado en la configuración.
- Asegúrate de tener conexión a internet.
- Reinstala la extensión relacionada con MCP si el problema persiste.

### 2. El modo agente no ejecuta acciones
- **Solución:** Confirma que el modo agente esté activado.
- Revisa los permisos de la extensión en la configuración.
- Consulta los logs de VS Code para más detalles (`Ayuda > Ver registros`).

### 3. Errores de comunicación con el modelo
- **Solución:** Comprueba la configuración de red y firewall.
- Asegúrate de que la API Key o credenciales sean correctas.
- Reinicia VS Code y vuelve a intentarlo.

---

## Recursos adicionales

- [Documentación oficial de MCP](https://github.com/modelcontextprotocol)
- [Extensión de VS Code para MCP](https://marketplace.visualstudio.com/)
- [Soporte y comunidad](https://github.com/modelcontextprotocol/discussions)

### Implementaciones y servidores MCP

- [Implementaciones MCP en npm (paquetes oficiales y de la comunidad)](https://www.npmjs.com/search?q=%40modelcontextprotocol)
- [Listado de servidores MCP en GitHub](https://github.com/modelcontextprotocol/servers?tab=readme-ov-file)
- [Smithery.ai: plataforma de servidores MCP y herramientas](https://smithery.ai/)

---

> _Actualizado: Julio 2025_
