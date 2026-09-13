# Desde cero: montar un Dashboard de Grafana y conectarlo a GitHub

> Guía práctica para Linux Mint / Ubuntu basada en una instalación real con Grafana 13.2.1 y el plugin oficial `grafana-github-datasource` 2.9.1.

## Objetivo

Al terminar tendrás un Grafana local capaz de consultar GitHub y mostrar de forma legible:

- commits;
- pull requests;
- issues;
- releases y tags;
- workflows;
- ejecuciones de GitHub Actions;
- enlaces de vuelta a GitHub;
- selectores dinámicos de repositorio, rama y workflow.

La idea no es sólo “ver números”, sino transformar datos de la API de GitHub en una **bitácora humana de actividad técnica**.

---

## 1. Arquitectura

```text
GitHub
  │
  │  API + token de sólo lectura
  ▼
grafana-github-datasource
  │
  ▼
Grafana local
  │
  ├── Stats
  ├── Commits
  ├── Pull Requests
  ├── Issues
  └── GitHub Actions
```

En este montaje Grafana funciona en el mismo equipo y se consulta desde:

```text
http://localhost:3000
```

---

## Capturas del proceso

![Paso 1 · Crear un token fine-grained de GitHub](../assets/step-01-create-github-token.png)

![Paso 2 · Configurar el datasource de GitHub en Grafana](../assets/step-02-configure-github-datasource.png)

![Paso 3 · Verificar la conexión](../assets/step-03-verify-connection.png)

![Paso 4 · Importar el dashboard GitHub Default](../assets/step-04-import-github-default-dashboard.png)

![Paso 5 · Explorar el dashboard GitHub Default](../assets/step-05-explore-default-dashboard.png)

![Paso 6 · RHYNUS GitHub Control Center](../assets/step-06-rhynus-github-control-center.png)

---

## 2. Entorno probado

Esta guía fue comprobada con:

```text
Linux Mint 22.x
Grafana 13.2.1
grafana-github-datasource 2.9.1
GitHub.com
Autenticación mediante Fine-grained Personal Access Token
```

Grafana 13.2 es la rama usada durante todo el proceso. Si utilizas otra versión, algunos menús o nombres pueden variar.

---

## 3. Instalar Grafana en Linux Mint / Ubuntu

Linux Mint deriva de Ubuntu, por lo que podemos usar el repositorio APT oficial de Grafana.

### Dependencias

```bash
sudo apt-get install -y apt-transport-https wget gnupg
```

### Añadir la clave del repositorio

```bash
sudo mkdir -p /etc/apt/keyrings

sudo wget -O /etc/apt/keyrings/grafana.asc \
  https://apt.grafana.com/gpg-full.key

sudo chmod 644 /etc/apt/keyrings/grafana.asc
```

### Añadir el repositorio estable

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" \
| sudo tee /etc/apt/sources.list.d/grafana.list
```

### Instalar Grafana OSS

```bash
sudo apt-get update
sudo apt-get install grafana
```

---

## 4. Activar el servicio

```bash
sudo systemctl enable --now grafana-server
```

Comprobar:

```bash
systemctl is-enabled grafana-server
systemctl is-active grafana-server
```

También puedes consultar la versión:

```bash
grafana server -v
```

En instalaciones que todavía exponen el comando antiguo puede funcionar:

```bash
grafana-server -v
```

aunque las versiones modernas avisan de que `grafana-server` está deprecado.

---

## 5. Comprobar el puerto 3000

```bash
sudo ss -ltnp | grep ':3000'
```

Y:

```bash
curl -I http://127.0.0.1:3000/login
```

Si el servicio aparece como `active` pero todavía no hay socket en `:3000`, no reinstales de inmediato.

En un primer arranque Grafana puede tardar mientras realiza migraciones internas.

Revisa:

```bash
sudo journalctl -u grafana-server -n 120 --no-pager
```

La línea que quieres ver es equivalente a:

```text
HTTP Server Listen address=[::]:3000 protocol=http
```

---

## 6. Primer acceso

Abre:

```text
http://localhost:3000
```

En una instalación nueva, Grafana puede solicitar las credenciales iniciales por defecto y obligarte a cambiar la contraseña.

Utiliza una contraseña propia y no reutilices la contraseña de GitHub.

---

## 7. Instalar el plugin oficial de GitHub

El plugin es:

```text
grafana-github-datasource
```

Con Grafana moderno usa:

```bash
sudo grafana cli \
  --homepath "/usr/share/grafana" \
  plugins install grafana-github-datasource
```

Después:

```bash
sudo systemctl restart grafana-server
```

Listar plugins:

```bash
sudo grafana cli \
  --homepath "/usr/share/grafana" \
  plugins ls
```

### Error: “Could not find config defaults”

Si intentas:

```bash
sudo grafana-cli plugins install grafana-github-datasource
```

puedes encontrar un error similar a:

```text
Could not find config defaults,
make sure homepath command line parameter is set
```

La solución práctica es usar el comando moderno con:

```text
--homepath "/usr/share/grafana"
```

---

## 8. Corregir permisos del directorio de plugins

Si los logs muestran `permission denied` dentro de:

```text
/var/lib/grafana/plugins
```

comprueba:

```bash
sudo ls -ld /var/lib/grafana/plugins
```

Y corrige el propietario:

```bash
sudo chown -R grafana:grafana /var/lib/grafana/plugins
sudo systemctl restart grafana-server
```

---

## 9. Crear un token de GitHub de sólo lectura

Para un dashboard personal es preferible utilizar un **Fine-grained Personal Access Token**.

En GitHub:

```text
Settings
→ Developer settings
→ Personal access tokens
→ Fine-grained tokens
→ Generate new token
```

Ejemplo:

```text
Token name:
RHYNUS-Grafana

Description:
Read-only token for local Grafana GitHub monitoring
```

### Repository access

Selecciona:

```text
Only select repositories
```

y marca únicamente los repositorios que quieras observar.

### Principio de mínimo privilegio

El dashboard sólo necesita leer.

Evita:

```text
Write
Administration
Secrets
Webhooks
```

Los permisos concretos disponibles pueden variar según GitHub y el tipo de recurso. Para commits, PR, issues, Actions y seguridad, concede únicamente los permisos `Read-only` que realmente aparezcan y necesites.

Ejemplos habituales:

```text
Metadata                       Read-only
Contents                       Read-only
Issues                         Read-only
Pull requests                  Read-only
Actions                        Read-only
Commit statuses                Read-only
Deployments                    Read-only
Code scanning alerts           Read-only
Repository security advisories Read-only
```

> Importante: nunca publiques ni incluyas el token en capturas, commits, tutoriales o ficheros JSON.

---

## 10. Conectar GitHub con Grafana

En Grafana:

```text
Connections
→ Add new connection
→ GitHub
→ Add new data source
```

Configura:

```text
Authentication Type:
Personal Access Token

Personal Access Token:
[tu token]

GitHub License Type:
Free, Pro & Team
```

Pulsa:

```text
Save & test
```

El resultado correcto es:

```text
Data source is working
```

---

## 11. Importar el dashboard inicial

El plugin incluye un dashboard oficial llamado:

```text
GitHub Default
```

Ruta:

```text
Connections
→ Data sources
→ grafana-github-datasource
→ Dashboards
→ GitHub Default
→ Import
```

Después abre el dashboard desde:

```text
Dashboards
```

Configura:

```text
Organization: tu_usuario
Repository:   tu_repositorio
Branch:       main
```

---

## 12. No modifiques directamente el dashboard original

Haz una copia.

Ejemplo:

```text
RHYNUS · GitHub Control Center
```

Así mantienes `GitHub Default` intacto como plantilla de recuperación.

---

## 13. Convertir la tabla de commits en una bitácora humana

La tabla original contiene campos técnicos útiles para la API, pero incómodos para leer.

Una versión más clara puede mostrar:

```text
Fecha | Actividad | Autor | Commit
```

y ocultar:

```text
author_login
author_email
author_company
pushed_at
```

Ejemplo:

```text
2026-09-13 17:26 | build: refresh V0.8.2 universal controls preview | Luis Alvarado | aff014b2
```

Orden recomendado:

```text
más reciente → más antiguo
```

---

## 14. Humanizar Pull Requests

La tabla de PR puede reducirse a:

```text
Fecha
PR
Título
Estado
+ Añadidas
- Eliminadas
Autor
Enlace
```

Esto permite leer el historial casi como un registro de trabajo.

Ejemplo:

```text
2026-09-08 | #13 | Publicar gameplay actualizado | MERGED | +2 | -2
```

---

## 15. GitHub Actions y workflows

El plugin oficial soporta consultas de:

```text
Workflows
Workflow_Runs
Workflow_Usage
Deployments
```

Una estructura útil es añadir una fila nueva:

```text
GitHub Actions
```

con dos paneles.

### Workflows

```text
Workflow | Estado | Archivo | Actualizado | ID | GitHub
```

### Ejecuciones

```text
Fecha | Workflow | Rama | Evento | Estado | Resultado | Run | Enlace
```

Por ejemplo:

```text
2026-09-13 15:27
pages build and deployment
main
dynamic
completed
success
#30
```

---

## 16. Selector dinámico de workflow

Grafana permite usar variables para no fijar un único workflow.

Conceptualmente:

```text
Repository ▼
Workflow   ▼
```

El usuario ve el nombre:

```text
pages build and deployment
```

mientras la consulta puede trabajar internamente con el ID del workflow.

Esto vuelve el dashboard reutilizable entre repositorios.

---

## 17. Caso real: `Workflow_Runs` muestra “No data”

En la combinación concreta probada con:

```text
Grafana 13.2.1
grafana-github-datasource 2.9.1
```

encontramos una peculiaridad al editar el dashboard como JSON.

El frontend del plugin expone una opción llamada:

```text
workflowID
```

pero el backend de esa versión procesa la propiedad:

```text
workflow
```

Por ello, si construyes manualmente el JSON y obtienes:

```text
No data
```

prueba con:

```json
"options": {
  "workflow": "$workflow",
  "branch": ""
}
```

en lugar de:

```json
"options": {
  "workflowID": "$workflow"
}
```

Este detalle es **específico de la versión probada** y puede cambiar en futuras versiones del plugin.

---

## 18. Editar dashboards como código en Grafana 13

En Grafana 13 el antiguo apartado `JSON Model` puede redirigirte a:

```text
Edit as code
```

El dashboard moderno utiliza un recurso similar a:

```json
{
  "apiVersion": "dashboard.grafana.app/v2",
  "kind": "Dashboard"
}
```

Esto permite versionar el dashboard igual que código.

Recomendación:

```text
V1 → commits humanizados
V2 → pull requests humanizados
V3 → GitHub Actions
V3.1 → corrección Workflow_Runs
```

---

## 19. Exportar y versionar el dashboard

Una práctica muy recomendable es guardar el JSON del dashboard dentro de un repositorio, por ejemplo:

```text
grafana/
├── RHYNUS-GitHub-Control-Center-v1.json
├── RHYNUS-GitHub-Control-Center-v2.json
├── RHYNUS-GitHub-Control-Center-v3.json
└── RHYNUS-GitHub-Control-Center-v3.1.json
```

Nunca incluyas el token.

El datasource almacena las credenciales por separado, por lo que el dashboard puede publicarse sin publicar el PAT.

---

## 20. Diagnóstico rápido

### Grafana está activo pero no abre

```bash
systemctl status grafana-server
sudo journalctl -u grafana-server -n 120 --no-pager
sudo ss -ltnp | grep ':3000'
```

### El plugin no aparece

```bash
sudo grafana cli \
  --homepath "/usr/share/grafana" \
  plugins ls
```

### El plugin no puede escribir

```bash
sudo chown -R grafana:grafana /var/lib/grafana/plugins
sudo systemctl restart grafana-server
```

### `Save & test` devuelve 401

Comprueba:

```text
token correcto
token no expirado
repositorio incluido
permisos Read-only suficientes
sin espacios al copiar el token
```

### Los cambios tardan en aparecer

El datasource de GitHub utiliza caché para respetar los límites de la API. Una modificación reciente puede tardar algunos minutos en reflejarse.

---

## 21. Seguridad

Para una instalación doméstica/local:

- deja Grafana accesible sólo donde realmente lo necesites;
- usa contraseña propia;
- no publiques el PAT;
- limita el token a repositorios concretos;
- usa únicamente permisos de lectura;
- configura una fecha de expiración;
- rota el token periódicamente;
- evita exponer `:3000` directamente a Internet.

Si en algún momento quieres acceso remoto, utiliza una VPN, túnel seguro o reverse proxy con TLS y autenticación adecuada.

---

## 22. Qué hemos conseguido

El dashboard deja de ser un conjunto de números aislados y se convierte en una pequeña consola operativa:

```text
RHYNUS · GitHub Control Center

Repository: packet-runner
Workflow: pages build and deployment

STATS
Commits | Releases | Tags | PR | Issues

ACTIVIDAD
Fecha | Actividad | Autor | Commit

PULL REQUESTS
Fecha | PR | Título | Estado | Cambios

GITHUB ACTIONS
Workflow | Estado | Archivo
Fecha | Rama | Evento | Resultado | Run
```

Es una combinación interesante de:

```text
Linux
Grafana
GitHub API
GitHub Actions
seguridad de tokens
observabilidad
JSON-as-code
```

y puede funcionar muy bien como proyecto demostrable de portfolio.

---

## 23. Ideas para continuar

Una futura versión podría añadir:

- tarjetas `Actions OK` y `Actions Failed`;
- porcentaje de éxito CI/CD;
- tiempo medio de ejecución;
- alertas de code scanning;
- deployments;
- releases;
- comparación entre repositorios;
- histórico de Renovate;
- alertas de workflows fallidos;
- paneles de seguridad;
- provisioning del dashboard y datasource desde archivos;
- Docker Compose para desplegar todo el entorno de una vez.

---

## Referencias oficiales

- Grafana: instalación en Debian/Ubuntu  
  https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/

- Grafana GitHub Data Source  
  https://grafana.com/docs/plugins/grafana-github-datasource/latest/

- Configuración del datasource GitHub  
  https://grafana.com/docs/plugins/grafana-github-datasource/latest/configure/

- Query editor del plugin GitHub  
  https://grafana.com/docs/plugins/grafana-github-datasource/latest/query-editor/

- Variables del plugin GitHub  
  https://grafana.com/docs/plugins/grafana-github-datasource/latest/template-variables/

- GitHub Fine-grained Personal Access Tokens  
  https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

---

## Nota de publicación

Esta guía describe una instalación real y reproducible, pero Grafana y el plugin de GitHub evolucionan con rapidez. Conviene indicar siempre las versiones probadas y revisar la documentación oficial antes de automatizar el procedimiento en producción.
