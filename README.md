# Asumir un Rol IAM para Subir Archivos a S3

## Objetivo

Permitir que un usuario IAM llamado `s3-support` pueda asumir un rol desde la línea de comandos (CLI) y subir archivos a un bucket de S3 usando ese rol.

## Requisitos previos

- Tener instalada y configurada la AWS CLI.
- Tener acceso a una cuenta de AWS con permisos de administrador.
- Conocer tu AWS Account ID (lo podés obtener desde la consola de AWS o ejecutando el siguiente comando):

```bash
aws sts get-caller-identity
```

## Pasos

### 1. Crear un bucket S3

El nombre del bucket debe ser único a nivel global.

```bash
aws s3api create-bucket --bucket mi-bucket-desafio-123 --region us-east-1
```

Reemplazar `mi-bucket-desafio-123` por un nombre único si da error.

### 2. Crear el usuario IAM `s3-support`

```bash
aws iam create-user --user-name s3-support
```

Obtener el ARN del usuario:

```bash
aws iam get-user --user-name s3-support
```

Guardar el valor del campo `Arn`. Ejemplo:  
`arn:aws:iam::462728423396:user/s3-support`

### 3. Crear la política de permisos para el rol

Crear el archivo `s3-write-policy.json` con el siguiente contenido:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::mi-bucket-desafio-123/*"
    }
  ]
}
```

Reemplazar el nombre del bucket por el utilizado en el paso 1.

### 4. Crear la política de confianza del rol

Crear el archivo `trust-policy.json` con el siguiente contenido:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::462728423396:user/s3-support"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Reemplazar el ARN por el obtenido en el paso 2.

### 5. Crear el rol IAM

```bash
aws iam create-role \
  --role-name S3WriteRole \
  --assume-role-policy-document file://trust-policy.json
```

### 6. Asociar la política de permisos al rol

```bash
aws iam put-role-policy \
  --role-name S3WriteRole \
  --policy-name S3WriteAccess \
  --policy-document file://s3-write-policy.json
```

### 7. Crear credenciales para el usuario `s3-support`

```bash
aws iam create-access-key --user-name s3-support
```

Guardar el `AccessKeyId` y el `SecretAccessKey` en un lugar seguro.

### 8. Configurar AWS CLI con el perfil `s3support`

```bash
aws configure --profile s3support
```

Completar con los datos obtenidos:

- Access Key ID
- Secret Access Key
- Region: `us-east-1`
- Output format: `json`

### 9. Asumir el rol desde la CLI

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::462728423396:role/S3WriteRole \
  --role-session-name pruebaSesion \
  --profile s3support
```

La salida incluirá:

- AccessKeyId
- SecretAccessKey
- SessionToken

### 10. Configurar un perfil temporal con las credenciales del rol

```bash
aws configure --profile assumed-role
```

Ingresar los valores obtenidos en el paso anterior:

- Access Key ID
- Secret Access Key
- Session Token
- Region: `us-east-1`

### 11. Subir un archivo a S3 usando el perfil temporal

Crear un archivo de prueba:

```bash
echo "Hola desde el rol!" > archivo.txt
```

Subir el archivo:

```bash
aws s3 cp archivo.txt s3://mi-bucket-desafio-123/ --profile assumed-role
```
