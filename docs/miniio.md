# Mini.IO

![miniio-logo](https://infraops.info/content/images/2020/12/minio-logo.png)

## Introdução

MinIO é um armazenamento de objetos de alta performance compatível com Amazon S3, ideal para armazenamento de grandes volumes de dados. Este guia mostra como configurar e utilizar o MinIO localmente utilizando Docker e enviar arquivos CSV utilizando Python.

## Pré-requisitos

- Docker instalado em sua máquina. Se você ainda não tem o Docker instalado, siga as instruções no [site oficial do Docker](https://docs.docker.com/get-docker/).
- Python instalado em sua máquina. Se você ainda não tem o Python instalado, siga as instruções no [site oficial do Python](https://www.python.org/downloads/).

## Passos para Configuração

### 1. Baixar a Imagem do MinIO

Primeiramente, baixe a imagem do MinIO do Docker Hub utilizando o seguinte comando:

```bash
docker pull minio/minio
```

### 2. Executar o Container do MinIO

Após baixar a imagem, você pode executar um container do MinIO com o comando:

```bash
docker run -d --name minio-server -p 9000:9000 -p 9001:9001 -e "MINIO_ROOT_USER=admin" -e "MINIO_ROOT_PASSWORD=password123" minio/minio server /data --console-address ":9001"
```

Substitua `admin` e `password123` pelos valores desejados para o usuário root e a senha.

### 3. Verificar o Container em Execução

Para verificar se o container está em execução, utilize o comando:

```bash
docker ps
```

Você deve ver algo similar a isso:

```
CONTAINER ID   IMAGE                   COMMAND                  CREATED        STATUS        PORTS                                           NAMES
<container_id> minio/minio             "/usr/bin/docker-ent…"   <some_time>    Up <some_time>   0.0.0.0:9000->9000/tcp, 0.0.0.0:9001->9001/tcp  minio-server
```

### 4. Acessar o Console do MinIO

Abra seu navegador e acesse o console do MinIO através do endereço:

```
http://localhost:9001
```

Entre com o usuário e senha que você configurou (`admin` e `password123` no exemplo acima).

### 5. Instalar o MinIO Client (mc) no PowerShell

Para instalar o `mc` (MinIO Client) no Windows usando PowerShell, siga os seguintes passos:

1. **Baixar o MinIO Client:**

   Abra o PowerShell e execute o seguinte comando para baixar o `mc`:

   ```powershell
   Invoke-WebRequest -Uri https://dl.min.io/client/mc/release/windows-amd64/mc.exe -OutFile mc.exe
   ```

2. **Mover o Executável para um Local no PATH:**

   Para facilitar o uso, mova o executável `mc.exe` para um diretório incluído na variável de ambiente `PATH`. Por exemplo:

   ```powershell
   Move-Item -Path .\mc.exe -Destination "C:\Program Files\mc.exe"
   ```

   Adicione `"C:\Program Files\"` ao `PATH` se ainda não estiver lá. Você pode fazer isso através das Configurações do Sistema -> Variáveis de Ambiente, ou via PowerShell:

   ```powershell
   [System.Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\Program Files\", [System.EnvironmentVariableTarget]::Machine)
   ```

### 6. Configurar o MinIO Client

Após instalar o `mc`, configure-o para conectar ao seu servidor MinIO local:

```powershell
mc alias set local http://localhost:9000 admin password123
```

### 7. Criar um Bucket

Para criar um bucket, utilize o comando:

```powershell
mc mb local/meu-bucket
```

### 8. Enviar Arquivos CSV com Python

Primeiro, instale o pacote `minio` utilizando pip:

```bash
pip install minio
```

### 9. Código Python para Enviar Arquivo CSV

Aqui está um exemplo de código Python para enviar um arquivo CSV para o MinIO:

```python
from minio import Minio
from minio.error import S3Error
import os

# Configurar o cliente MinIO
client = Minio(
    "localhost:9000",
    access_key="admin",
    secret_key="password123",
    secure=False
)

# Nome do bucket e do arquivo
bucket_name = "meu-bucket"
file_path = "exemplo.csv"
object_name = os.path.basename(file_path)

# Criar um arquivo CSV de exemplo
with open(file_path, 'w') as f:
    f.write("id,nome\n")
    f.write("1,Alice\n")
    f.write("2,Bob\n")

# Enviar o arquivo CSV para o MinIO
try:
    client.fput_object(bucket_name, object_name, file_path)
    print(f"{object_name} enviado com sucesso para {bucket_name}.")
except S3Error as e:
    print("Erro ao enviar o arquivo:", e)
```

### 10. Executar o Código Python

Salve o código acima em um arquivo Python, por exemplo, `upload_csv.py`, e execute-o:

```bash
python upload_csv.py
```

Você verá uma mensagem confirmando que o arquivo foi enviado com sucesso para o bucket especificado no MinIO.

### 11. Verificar o Arquivo no MinIO

Acesse o console do MinIO em `http://localhost:9001`, navegue até o bucket `meu-bucket` e verifique se o arquivo `exemplo.csv` está presente.

### Configuração de Volume Persistente (Opcional)

Para garantir que seus dados persistam mesmo após reiniciar o container, você pode configurar volumes do Docker:

```bash
docker run -d --name minio-server -p 9000:9000 -p 9001:9001 -e "MINIO_ROOT_USER=admin" -e "MINIO_ROOT_PASSWORD=password123" -v /path/to/minio/data:/data minio/minio server /data --console-address ":9001"
```

Substitua `/path/to/minio/data` pelo caminho onde você deseja armazenar os dados do MinIO em sua máquina local.

### Conclusão

Com esses passos, você pode facilmente configurar e utilizar o MinIO localmente utilizando Docker e enviar arquivos CSV utilizando Python. Essa configuração facilita o desenvolvimento e a realização de testes com o MinIO sem a necessidade de configuração complexa.

Se você tiver alguma dúvida ou problema, consulte a [documentação oficial do MinIO](https://docs.min.io/) para obter mais detalhes e suporte.