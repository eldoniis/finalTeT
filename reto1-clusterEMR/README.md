# ReTo 1: Map/Reduce en Python con MRJOB

## Información de la asignatura
---

 -  **Nombre de la asignatura:** Tópicos especiales en telemática.
-   **Código de la asignatura:**  C2361-ST0263-4528
-   **Departamento:** Departamento de Informática y Sistemas (Universidad EAFIT).
-   **Nombre del profesor:** Juan Carlos Montoya Mendoza.
-  **Correo electrónico del docente:** __[jcmontoy@eafit.edu.co](mailto:jcmontoy@eafit.edu.co)__.

## Datos del estudiante
---
-   **Nombre del estudiante:** Donovan Castrillon Franco.
-  **Correo electrónico del estudiante:** __[dcastrillf@eafit.edu.co](mailto:dcastrillf@eafit.edu.co)__.

## Despliegue del cluster EMR via AWS CLI
---
> Se esta siguiendo la guia que se puede encontrar en el siguiente enlace: *https://thecodinginterface.com/blog/create-aws-emr-with-aws-cli/*

Primeramente lo que debemos realizar es crear un bucket s3 para esto seguimos lo siguiente:

1. El siguiente comando nos permite crear el bucket s3

```
aws s3 mb s3://emr-bucket-donovan
```

En este caso el nombre del bucket s3 fue "emr-bucket-donovan"

![](https://raw.githubusercontent.com/eldoniis/finalTeT/main/reto1-clusterEMR/img/create-s3-1.png)

2. Ahora debemos agregar objetos que seran los directorios que tendra nuestro bucket:

```
aws s3api put-object --bucket emr-bucket-donovan --key logs/
aws s3api put-object --bucket emr-bucket-donovan --key inputs/
aws s3api put-object --bucket emr-bucket-donovan --key outputs/
aws s3api put-object --bucket emr-bucket-donovan --key scripts/
```

![](https://raw.githubusercontent.com/eldoniis/finalTeT/main/reto1-clusterEMR/img/create-s3-2.png)

1. Ahora crearemos un security group para nuestro cluster al cual le asociaremos una vpc ya existente:

```
aws ec2 create-security-group --group-name ssh-my-ip \
    --description "For SSHing from my IP" --vpc-id vpc-0ecd9aef75a7f7cba
```

* Agregamos una regla de entrada para permitir la conexion ssh por el puerto 22

```
aws ec2 authorize-security-group-ingress --group-id sg-0f47bb000fcde9de3 \
    --protocol tcp --port 22 --cidr 0.0.0.0/0
```

![](https://raw.githubusercontent.com/eldoniis/finalTeT/main/reto1-clusterEMR/img/create-cluster-4.png)

4. Por ultimo crearemos el cluster emr en el que le instalaremos 4 applicaciones: Hadoop, Hive, Spark, Hue

```
aws emr create-cluster --name donovan-cluster9 --applications Name=Hadoop Name=Hive Name=Spark Name=Hue  \
  --release-label emr-5.33.0 --use-default-roles \
  --instance-count 3 --instance-type m4.large \
  --ebs-root-volume-size 12 \
  --log-uri s3://emr-bucket-donovan/logs \
  --ec2-attributes KeyName=llaveMaestra,AdditionalMasterSecurityGroups=sg-0f47bb000fcde9de3,SubnetId=subnet-098fa85fac8de9d16 \
  --no-auto-terminate
```

Parametros a tener en la cuenta:

- **log-uri**: Sera la URI de nuestro bucket s3 dedidaca a los logs para tener una trazabilidad
- **ec2-attributes**:
  - KeyName: El nombre de la llave que usaremos para conectarnos
  - AdditionalMasterSecurityGroups: id del security group recien creado
  - SubnetId: id de la subred de la VPC

![](https://raw.githubusercontent.com/eldoniis/finalTeT/main/reto1-clusterEMR/img/create-cluster-5.png)

5. Ahora ya tenemos creado nuestro cluster, pero para conectarnos debemos obtener el nombre del DNS publico:

```
aws emr describe-cluster --cluster-id j-3M495CZO2I4GZ | jq -r ".Cluster.MasterPublicDnsName"
```

![](https://raw.githubusercontent.com/eldoniis/finalTeT/main/reto1-clusterEMR/img/create-cluster-6.png)

6. Y ahora nos podemos conectar por ssh como normalmente se hace:

![](https://raw.githubusercontent.com/eldoniis/finalTeT/main/reto1-clusterEMR/img/create-cluster-7.png)