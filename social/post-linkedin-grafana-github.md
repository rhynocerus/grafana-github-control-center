# Borrador de publicación para LinkedIn

Hoy convertí GitHub en algo más que una lista de commits.

Monté desde cero un entorno local con **Grafana + GitHub**, instalando el datasource oficial y creando un dashboard propio para leer la actividad de mis proyectos de una forma mucho más humana.

El proceso terminó incluyendo:

• instalación y puesta en marcha de Grafana en Linux Mint  
• conexión segura con GitHub mediante un token de sólo lectura  
• commits y pull requests convertidos en bitácoras legibles  
• variables dinámicas por repositorio y workflow  
• monitorización de GitHub Actions y sus ejecuciones  
• edición y versionado del dashboard como JSON  
• diagnóstico de permisos, migraciones y un pequeño detalle de compatibilidad en `Workflow_Runs`

El resultado es **RHYNUS · GitHub Control Center**, una pequeña cabina local desde la que puedo observar el movimiento de mis repositorios sin depender únicamente de la interfaz de GitHub.

Y quizá lo más interesante no fue el dashboard final, sino documentar el camino: errores reales, diagnóstico y correcciones reproducibles.

Estoy preparando el tutorial:

**“Desde cero: montar un Dashboard de Grafana y conectarlo a GitHub”**

Creo que este tipo de proyectos pequeños son una buena forma de unir Linux, observabilidad, APIs, CI/CD y seguridad en algo tangible.

#Grafana #GitHub #Linux #DevOps #Cybersecurity #GitHubActions #OpenSource #Homelab #Observability
