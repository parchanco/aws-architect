# 32 - Operaciones con AWS Systems Manager

Operar bien significa conocer el estado deseado, detectar desviaciones y responder de forma repetible sin depender de accesos manuales a servidores.

## Objetivos

- administrar nodos sin abrir SSH a Internet;
- inventariar software y aplicar parches controladamente;
- automatizar runbooks y tareas recurrentes;
- gestionar configuración sin mezclarla con secretos;
- responder a incidentes con evidencia y límites de seguridad.

## Capacidades principales

| Necesidad | Systems Manager |
|---|---|
| Sesión auditada a un nodo | Session Manager |
| Ejecutar comandos en una flota | Run Command |
| Automatizar procedimientos | Automation runbooks |
| Inventario de nodos | Inventory |
| Ventanas de mantenimiento | Maintenance Windows |
| Aplicación de parches | Patch Manager |
| Configuración jerárquica | Parameter Store |

Secrets Manager está orientado a secretos y rotación. Parameter Store puede almacenar valores cifrados, pero la elección depende del ciclo de vida, rotación y necesidades del secreto.

## Patrón operativo

```text
evento/alarma → clasificar → contener → diagnosticar → recuperar
               → verificar → comunicar → aprender y automatizar
```

Un runbook debe incluir propósito, precondiciones, permisos, pasos, validación, rollback, escalado y propietario. Las acciones destructivas requieren confirmaciones y límites de concurrencia.

## Acceso sin bastion público

Para usar Session Manager, el nodo necesita agente, conectividad con los endpoints necesarios y un rol de instancia limitado. Registra sesiones cuando la política de la organización lo requiera. El acceso humano debe usar federación y credenciales temporales.

## Gestión de flota

- agrupa nodos mediante etiquetas;
- prueba parches en un anillo pequeño antes de producción;
- limita concurrencia y tolerancia a errores;
- programa ventanas alineadas con el negocio;
- mide cumplimiento y excepciones;
- no ejecutes comandos masivos sin filtro y vista previa.

## Laboratorio: parcheado seguro

1. Registra una instancia de laboratorio como managed node.
2. Accede con Session Manager sin exponer el puerto 22.
3. Recoge inventario y crea grupos `patch-ring=canary` y `patch-ring=stable`.
4. Evalúa parches antes de instalarlos.
5. Aplica primero al canary con límites estrictos.
6. Verifica salud, promueve al grupo estable y guarda evidencia.

### Criterio de finalización

- [ ] No existe entrada SSH pública.
- [ ] El rol del nodo y el del operador están separados.
- [ ] La ejecución queda auditada.
- [ ] Hay validación posterior y procedimiento de rollback.
- [ ] El recurso se elimina al terminar.

## Preguntas de repaso

1. ¿Qué ventajas tiene Session Manager frente a un bastion?
2. ¿Cómo evitas que Run Command afecte a toda la flota por error?
3. ¿Qué diferencia una configuración de un secreto?
4. ¿Qué información mínima necesita un runbook de incidente?

**Anterior:** [31 - CI/CD](31-cicd-devops.md)  
**Siguiente:** [33 - Gobierno multi-cuenta](33-gobierno-multicuenta.md)
