### A. Infraestructura como Código

#### 1  Introducción de IaC:

Esta infraestructura funciona de manera que nos permite definir, aproviosionar y también administrar a TI. En vez de tener que hacer configuraciones manuales en los servidores.
Los beneficios que podemos encontrar en la utilización son: 
- Reproducibilidad:        Con el mismo código se puede construir varios entornos.
- Consistencia:            Amenorar las diferencias entre los entornos de desarrollo, pruebas y producción.
- Velocidad de Despliegue: Tomar la automatización como ventaja para ahorrar tiempo.
- Control sobre versiones: El uso de repositorios como GitHub nos permiten hacer cambios sin interferir con otros y a la vez colaborar en equipo.

#### 2 Escritura de IaC:
El uso de Terraform muy usada cuando se habla de IaC, fue desarrollada por HashiCorp.
Nos permite trabajar en plataformas como: AWS, Azure, GCP, etc. con los archivos de extensión `.tf`

Al usar Terraform:
. Los bloques de configuración son la definición de los recursos tales como: redes, máquinas virtuales, bases de datos, etc.
. Se usan las variables, outputs y los archivos modulares para poder separar las responsabilidades. 
. Hay comandos para desplegar y también para destruir como pueden ser: `terraform apply` y `terraform destroy`

#### 3 Patrones para módulos:
Como se mencionó Terraform nos permite organizar la infraestructura en los módulos reutilizables, las cuales tienen tareas específicas.
Un módulo ordinario contiene estos 2 archivos:

`main.tf`-------> Define los recursos principales

`variable.tf`---> Son definidas las varibles de entrada.

`out.tf`--------> Difine las variables de salida, las cuales pueden ser usadas por otros módulos. 


Para 3 módulos(network, database, application)

```yaml
project-root/
|-- main.tf                
|-- variables.tf           
|-- outputs.tf             
│
|-- modules/
│   |-- network/
│   │   |-- main.tf
│   │   |-- variables.tf
│   │   |--outputs.tf
│   │
│   |-- database/
│   │   |-- main.tf
│   │   |-- variables.tf
│   │   |-- outputs.tf
│   │
│   |-- application/
│       |-- main.tf
│       |-- variables.tf
│       |-- outputs.tf
│
|-- terraform.tfvars

```

En cuestión de la jerarquía:
- Para mantener separados los componentes que se reutilizan se usó la carpeta `modules/`.

- Como se ve cada módulo tiene sus propios archivos, con lo cual la edición es de forma independiente.
  
- El archivo `main.tf` dirige a los módulos y también los conecta usando los outputs y variables. 


#### 4 Patrones para dependencias 
La gestión entre las dependencias en los módulos tiene una base en el uso de los outputs y inputs. 
En cuestión de los módulos pueden mostrar algunos valores como IP, nombre de la red, credenciales. Esto se logra a travez del uso del outputs.
Se consiguen de la forma: `module.nombre_del_modulo.outoput_deseado` .

Un ejemplo para entenderlo mejor que muestra la dependencia entre database y application.


```yaml
# main.tf
module "network" {
  source = "./modules/network"
}

module "database" {
  source      = "./modules/database"
  vpc_id      = module.network.vpc_id
}

module "application" {
  source        = "./modules/application"
  db_endpoint   = module.database.db_endpoint
  subnet_ids    = module.network.subnet_ids
}
```


y dentro de `modules/database/outputs.tf`
debería haber algo así:

```yaml
# modules/database/outputs.tf
output "db_endpoint" {
  value = aws_db_instance.main.endpoint
}
```

- Donde dice que el módulo database muestra `db_endpoint`.
  
- Luego el mpodulo de application toma este valor de `db_endpoint` para conectarse con la base de datos.



### B. Contenerización y despliegue de aplicaciones modernas

#### 1 Contenerización de una aplicación con Docker

- ¿Qué son los contenedores?
  
  Tienen la característica de empaquetar una aplicación con todas sus dependencias. Mientras que las máquins virtuales no requieren de un sistema operativo. 
  
- Dockerfile

  El archivo el cual llega a definir como construir una imagen Docker, la cual su estructura básica es:

  
``` yaml
From node:18
WORKDIR /app
COPY
RUN npm install
CMD ["npm", "start"]
```
 
- Imagen vs Contenedor  
  * **imagen:** Una plantilla la cual son solo de lectura, estas tienen instrucciones para la creación de un contenedor. 
  * **contenedor:** 
