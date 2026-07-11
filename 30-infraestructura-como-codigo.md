# 30 - Infraestructura como código

Infraestructura como código (IaC) convierte la arquitectura en una definición versionada, revisable y repetible. El objetivo no es automatizar clics, sino conseguir despliegues predecibles y cambios auditables.

## Objetivos

- entender recursos, dependencias, parámetros, salidas y estado;
- elegir entre CloudFormation, AWS CDK y herramientas externas;
- previsualizar cambios y evitar drift;
- dividir una plataforma en componentes con límites claros;
- probar, desplegar y retirar infraestructura de forma segura.

## Opciones principales

| Opción | Cómo se define | Cuándo encaja |
|---|---|---|
| CloudFormation | YAML o JSON declarativo | Control nativo y explícito de AWS |
| AWS CDK | Lenguajes de programación que sintetizan plantillas | Reutilización, abstracciones y equipos de desarrollo |
| Terraform/OpenTofu | HCL y estado administrado | Plataformas multi-proveedor o estándar existente |

La herramienta no reemplaza el diseño: una plantilla reproducible también puede reproducir una mala arquitectura.

## Ejemplo mínimo con CloudFormation

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Bucket privado para una práctica
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, test, prod]
Resources:
  AssetsBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Delete
    UpdateReplacePolicy: Delete
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      Tags:
        - Key: Environment
          Value: !Ref Environment
Outputs:
  BucketName:
    Value: !Ref AssetsBucket
```

No uses este ejemplo sin revisar la política de borrado y retención apropiada para producción.

## Flujo seguro de cambios

```text
formatear → validar → análisis de seguridad → pruebas → change set/plan
           → revisión humana → desplegar → verificar → observar
```

Principios:

- repositorio como fuente de verdad;
- cambios pequeños y reversibles;
- parámetros sin secretos;
- roles de despliegue con mínimo privilegio;
- entornos separados y promoción del mismo artefacto;
- detección de drift y política para corregirlo;
- outputs y nombres estables solo donde sean necesarios.

## Diseño de stacks

Evita una única plantilla gigantesca. Separa por ciclo de vida y propietario:

```text
foundation: cuentas, red y registros
data:       buckets y bases de datos
service:    aplicación, colas y funciones
observe:    dashboards y alarmas
```

Una dependencia transversal debe tener un contrato explícito: output, Parameter Store o catálogo de servicios. Evita copiar IDs a mano.

## Laboratorio

1. Modela por código una VPC pequeña o una aplicación serverless.
2. Añade etiquetas, cifrado, logs y salidas útiles.
3. Valida y revisa el plan antes del despliegue.
4. Cambia una propiedad y observa el change set.
5. Provoca drift solo en laboratorio, detecta la diferencia y restáurala por código.
6. Destruye el stack y comprueba que no deja recursos facturables.

### Criterio de finalización

- [ ] Otra persona puede desplegar el entorno siguiendo el README.
- [ ] El repositorio no contiene secretos ni IDs personales innecesarios.
- [ ] Los controles fallan ante almacenamiento público o sin cifrar.
- [ ] Existe una estrategia de rollback y de retención de datos.

## Preguntas de repaso

1. ¿Cuándo una actualización reemplaza un recurso y qué riesgo implica?
2. ¿Por qué los secretos no deben ser parámetros de texto plano?
3. ¿Cómo tratarías un cambio manual urgente realizado en producción?
4. ¿Qué separarías en stacks diferentes y por qué?

**Anterior:** [29 - FinOps](29-finops-economia-cloud.md)  
**Siguiente:** [31 - CI/CD](31-cicd-devops.md)
