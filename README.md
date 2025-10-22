SuperMax Cloud – Plataforma de Análisis de Demanda
-----------------------------------
Implementación de infraestructura como código (IaC) con Terraform
Introducción general
 Este proyecto implementa la infraestructura completa de la plataforma SuperMax Cloud, un sistema de análisis de demanda que integra ingesta, procesamiento y consulta de datos sobre servicios AWS.
 El objetivo principal es provisionar todos los recursos de forma declarativa y reproducible, asegurando coherencia entre entornos y facilidad de mantenimiento.


Descripción de los módulos
-----------------------------------
vpc/ 
    → Define toda la red privada: crea la VPC, subredes, security groups y endpoint de S3.
    → Es la base para que Redshift y las Lambdas trabajen aisladas y seguras.
s3/ 
    → Crea el bucket de datos (data lake) con carpetas sinprocesar/, procesados/, error/.
    → Solo accesible desde la VPC mediante el endpoint, ideal para ETL.
redshift/ 
    → Despliega el clúster Redshift dentro de la VPC privada.
    → Guarda y analiza la información que cargan las Lambdas.
lambda_ingesta/ 
    → Función que procesa archivos del S3 y los inserta en Redshift.
    → Corre en VPC privada, usa el rol académico (LabRole) y el secreto del clúster.
lambda_generic/ 
    → Módulo genérico para crear Lambdas fácilmente (callback, consultas, presign).
    → Soporta variables de entorno, layers y configuración de red opcional.
s3_frontend/ 
    → Hospeda el sitio web estático (dashboard).
    → Genera dinámicamente un config.json con las URLs de la API y Cognito.
apigw_base/ 
    → Crea la API principal y el recurso /LambdaCallback.
    → Sirve como base para añadir rutas y etapas de despliegue.
apigw_attach/ 
    → Vincula la Lambda Callback al endpoint /LambdaCallback del API Gateway.
apigw_route/
    → Crea rutas REST (/consultas y /presign) y sus integraciones con las Lambdas.
    → Configura también las respuestas OPTIONS (CORS).
cognito/ 
    → Implementa el login de usuarios: User Pool, App Client y dominio OIDC.
    → Define los callbacks y logout hacia el frontend.

Otros
-----------------------------------
layer/ 
    → Contiene dependencias externas (pg8000) empaquetadas como layer.
services/
    → Contiene los codigos de las lambdas; 'callback' 'consultas' 'presign'

Módulo externo utilizado
-----------------------------------
→ VPC

Funciones - Descripción de algunas de las que usamos
-----------------------------------
→ ⁠Split(): Divide un texto en partes usando un separador.
→ jsonencode(): Convierte estructuras de Terraform (listas, mapas) en formato JSON.
→ ⁠⁠merge(): Combina varios mapas en uno solo. Si hay claves repetidas, el último valor prevalece.
→⁠ ⁠try(): Evalúa varias expresiones y devuelve la primera que no da error.

Meta-argumentos
-----------------------------------
→depends_on: Fuerza un orden de creación o destrucción entre recursos o módulos.
→⁠⁠lifecycle: Controla el comportamiento de creación, actualización y destrucción.
→⁠count: Permite crear múltiples instancias de un recurso según un número o condición.


Requisitos previos para una correcta ejecución
-----------------------------------
→ Rol “LabRole” existente.
→ Terraform instalado.
→ AWS CLI instalado.

Configuración de credenciales AWS (comandos para MACOS)
-----------------------------------
Configurar usuario--> aws configure
Ver el usuario activo--> aws sts get-caller-identity

EJECUCIÓN
-----------------------------------
1- Inicializar el proyecto (descarga proveedores/módulos)-->  terraform init
2- Validar que todo esté bien escrito--> terraform validate
3- Ver el plan de cambios (sin aplicar)--> terraform plan 
4- Aplicar (crear/modificar recursos)--> terraform apply 

Notas:
 Una vez realizado el apply, el output es el link de la pagina.
 No hay que asignarle valor a ninguna variable desde la consola.
 Cuando se inicia en una nueva cuenta (una cuenta donde nunca antes se había hecho el apply del proyecto), puede que el primer terraform apply no funcione. En ese caso, ejecutar un terraform destroy y luego volver a ejecutar.


USO DE LA PAGINA
-----------------------------------
Una vez iniciada la sesión:
Dentro de la pagina hay dos secciones: 
- Carga de CSV: 
 Se debe cargar los CSV que se desea que sean visualizados en el dashboard el formato del esta descripto en esa misma sección.
- Dashboard: Se visualiza todo lo que haya sido subido con el fomato correcto. 
 Con el boton actualizar, se actualizan los datos.
 Con el boton capturar, saca un ScreenShoot del grafico.

Dejamos un archivo CSV, "ejemplo.csv" con datos para realizar un test del funcionamiento.
