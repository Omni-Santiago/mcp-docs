
# Documentación para la Implementación de MCPs

Esta guía proporciona información esencial para implementar y trabajar con MCPs (Model Context Protocol) en Visual Studio Code, incluyendo la activación de opciones, el modo agente, el funcionamiento general, la configuración avanzada y la resolución de errores comunes.

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

MCP (Model Context Protocol) es un protocolo abierto y extensible diseñado para estandarizar la comunicación entre entornos de desarrollo (como VS Code) y modelos de inteligencia artificial, ya sean locales o remotos. Su objetivo es permitir que los modelos reciban contexto relevante del usuario y del proyecto, procesen solicitudes complejas y devuelvan respuestas estructuradas o acciones automatizables.

### Principios clave de MCP

- **Contextualidad:** El protocolo permite enviar información rica sobre el estado del editor, archivos abiertos, selección de código, historial de comandos y más, para que el modelo pueda responder de manera precisa y relevante.
- **Bidireccionalidad:** No solo se envían preguntas o prompts, sino también comandos, acciones y resultados, permitiendo flujos de trabajo interactivos y automatizados.
- **Extensibilidad:** MCP soporta la definición de nuevos tipos de mensajes, acciones y eventos, adaptándose a las necesidades de cada equipo o proyecto.
- **Agnóstico de transporte:** MCP puede funcionar sobre diferentes mecanismos de comunicación: STDIO, sockets, HTTP, Docker, etc.

### Flujo de funcionamiento

1. El usuario realiza una acción en el editor (por ejemplo, selecciona código o ejecuta un comando).
2. El editor recopila el contexto relevante (archivos, entorno, selección, historial, variables de entorno, etc.).
3. El contexto y la solicitud se envían al modelo a través del canal configurado (STDIO, Docker, SSO, etc.).
4. El modelo procesa la información y responde con sugerencias, acciones, documentación, o incluso comandos para ejecutar en el entorno.
5. El editor interpreta la respuesta y la presenta al usuario, permitiendo ejecutar acciones sugeridas o continuar el flujo.

MCP permite así la integración profunda de IA en el ciclo de desarrollo, automatizando tareas, generando código, explicando fragmentos, detectando errores y mucho más.

---


## ¿Qué hace el MCP?

MCP actúa como un "puente" entre el editor y los modelos de IA, permitiendo:

- **Recepción de contexto enriquecido:** El modelo puede recibir información detallada sobre el estado del proyecto, archivos, entorno, configuración, historial de comandos y más.
- **Ejecución de acciones automatizadas:** El modelo puede sugerir y ejecutar acciones como refactorizaciones, generación de código, documentación automática, análisis estático, pruebas, despliegues, etc.
- **Comunicación segura y flexible:** MCP soporta múltiples mecanismos de autenticación y transporte, permitiendo operar en entornos locales, en la nube o híbridos.
- **Personalización avanzada:** Se pueden definir flujos de trabajo, comandos y respuestas personalizadas para cada equipo, proyecto o usuario.
- **Integración con herramientas externas:** MCP puede interactuar con sistemas de CI/CD, gestores de incidencias, bases de datos, y otros servicios, orquestando tareas complejas desde el editor.

---


## Configuración de MCP: mcp.json

La configuración de MCP se realiza mediante un archivo `mcp.json`, que define cómo y dónde se ejecuta el modelo, así como las opciones de autenticación, variables de entorno y parámetros de ejecución. Este archivo puede ubicarse a nivel de proyecto (en la raíz del repositorio) o a nivel global (en la configuración de usuario de VS Code).

### Configuración a nivel de proyecto

Coloca un archivo `mcp.json` en la raíz de tu proyecto para definir un modelo específico o configuración personalizada para ese repositorio. Ejemplo:

```json
{
  "provider": "docker",
  "image": "mi-modelo-mcp:latest",
  "env": {
    "API_KEY": "tu_api_key"
  },
  "options": {
    "autoStart": true,
    "resources": {
      "cpus": 2,
      "memory": "4g"
    }
  }
}
```

**Campos principales:**
- `provider`: Define el mecanismo de ejecución (`docker`, `stdio`, `sso`, etc.).
- `image`/`command`: Imagen de Docker o comando a ejecutar.
- `env`: Variables de entorno para el modelo.
- `options`: Opciones adicionales como autoStart, recursos, puertos, etc.

### Configuración global

Puedes definir una configuración global en la carpeta de usuario de VS Code (`~/.config/Code/User/mcp.json` en Linux, `%APPDATA%/Code/User/mcp.json` en Windows, o `~/Library/Application Support/Code/User/mcp.json` en macOS). Ejemplo:

```json
{
  "provider": "stdio",
  "command": "python3 mcp_server.py",
  "options": {
    "autoRestart": true,
    "logLevel": "debug"
  }
}
```

Esto permite tener un modelo por defecto para todos los proyectos, que puede ser sobrescrito por un `mcp.json` local.

### Ejemplos de configuración: STDIO, Docker, SSO

- **STDIO**: Ejecuta un proceso local que se comunica por entrada/salida estándar. Útil para modelos escritos en cualquier lenguaje que puedan ejecutarse como proceso hijo.
  ```json
  {
    "provider": "stdio",
    "command": "node mcp-server.js",
    "env": {
      "NODE_ENV": "production"
    }
  }
  ```
- **Docker**: Lanza un contenedor Docker con el modelo MCP, aislando dependencias y facilitando la portabilidad.
  ```json
  {
    "provider": "docker",
    "image": "mi-modelo-mcp:latest",
    "options": {
      "autoRemove": true
    }
  }
  ```
- **SSO (Single Sign-On)**: Integra autenticación corporativa, ideal para entornos empresariales.
  ```json
  {
    "provider": "sso",
    "clientId": "tu-client-id",
    "tenant": "tu-tenant",
    "scopes": ["openid", "profile"]
  }
  ```

> Puedes combinar opciones y definir múltiples proveedores según las necesidades de tu entorno. Consulta la documentación de tu extensión MCP para más detalles sobre los campos soportados.

---


## Comparación MCP vs API tradicional

| Característica           | MCP (Model Context Protocol)                | API Tradicional                  |
|-------------------------|---------------------------------------------|----------------------------------|
| Contexto dinámico       | Sí, contexto completo del editor y proyecto | Limitado, generalmente solo datos|
| Integración IDE         | Directa, bidireccional                      | Indirecta, vía HTTP/REST         |
| Acciones automatizadas  | Sí, acciones y comandos ejecutables         | No, solo respuestas de datos     |
| Configuración flexible  | Sí, múltiples proveedores y opciones        | Limitada, depende del backend    |
| Seguridad               | SSO, variables de entorno, aislamiento      | Depende de la API                |
| Ejecución local/remota  | Ambas (STDIO, Docker, remoto, híbrido)      | Generalmente remota              |
| Extensibilidad          | Alta, mensajes y acciones personalizables   | Baja, limitada al contrato REST  |

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

---

> _Actualizado: Julio 2025_
