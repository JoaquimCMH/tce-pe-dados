<h3 align="center">Resturando Base de Dados Aberta do TCE-PE (banco TomeConta)</h3>

<div align="center">

[![Status](https://img.shields.io/badge/status-active-success.svg)]()
[![License](https://img.shields.io/badge/licence-GNU%20Aferro%20V3-blue.svg)](/LICENSE)

</div>

---

<p align="center">
    <br> 
</p>

## 📝 Índice

- [Sobre](#about)
- [Pré-Requisitos](#req)
- [Uso](#usage)


## 🧐 Sobre <a name = "about"></a>

Este repositório fornece meios, através do docker, para facilitar o uso dos dados abertos oriundos do [Tribunal de Contas do estado de Pernambuco](https://www.tce.pe.gov.br/internet/index.php/dados-abertos/bases-de-dados-completas). Com ele é possível restaurar o banco SQL Server com todos os pacotes necessários para o funcionamento da aplicação, como dependências e bibliotecas.

### 🎈 Pré-requisitos <a name="req"></a>

- Garanta que você tenha, no mínimo, **40gb** de espaço livre em seu computador.

- Os serviços utilizam docker para configuração do ambiente e execução do script. Instale o  [docker](https://docs.docker.com/install/) e tenha certeza que você também tem o  [docker-compose](https://docs.docker.com/compose/install/) instalado. 


##  🏁 Uso <a name="usage"></a>

Essas instruções fornecerão acesso ao banco funcionando em sua máquina local para fins de desenvolvimento, execução de pesquisas ou outros meios para promover o controle social.

Os passos para a restauração e utilização do banco de dados são elecandos abaixo:

1. Após ter clonado o repositório, baixe o [arquivo](https://www.tce.pe.gov.br/internet/docs/dadosabertos/TomeConta.rar) com o banco no formato *.rar* e o descompacte na pasta *dados*.

```shell
# clone o repositório
git clone  https://github.com/JoaquimCMH/tomeconta-tce-pe-dados.git
cd tomeconta-tce-pe-dados

# baixe o arquivo compactado com o backup
wget -P dados "https://www.tce.pe.gov.br/internet/docs/dadosabertos/TomeConta.rar"

# descompacte o arquivo na pasta dados
unrar x dados/TomeConta.rar dados/
```
2. É necessário adicionar criar uma cópia do .env.sample para um arquivo .env na raiz do repositório. Além disso é necessário preencher as variáveis com seus respectivos valores.

Aqui você pode optar por executar o script automático de download e atualização do BD fazendo:

```
sudo ./update-data.sh
```

Ou optar por seguir a próxima seção que detalha os passos separadamente:

3. Execute o docker e crie a pasta onde ficará o arquivo *.bak*.

```shell
# execute o container
sudo docker-compose up -d

# entre no container 
sudo docker exec -it mssql-tce-pe bash

# crie a pasta backup dentro do container
mkdir /var/opt/mssql/backup

# SAIA do container utilizando ctrl+d
```
4. Copie o .bak para o container.

```shell
sudo docker cp ../dados/Backup/TomeConta/TomeConta/TomeConta.bak mssql-tce-pe:/var/opt/mssql/backup
```

5. Restaure o banco.
```shell
# entre no container
sudo docker exec -it mssql-tce-pe bash

# execute o sqlcmd
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'S3cr3t-P4ssw0rd!'

# verifique os arquivos do arquivo de backup
# observe na saída se existe TomeConta.mdf e TomeConta_log.ldf para prosseguir com a importação
RESTORE FILELISTONLY
FROM DISK = '/var/opt/mssql/backup/TomeConta.bak'
GO

# restaure o banco (demora alguns minutos)
RESTORE DATABASE TOMECONTA
FROM DISK = '/var/opt/mssql/backup/TomeConta.bak' WITH
MOVE 'TomeConta' TO '/var/opt/mssql/data/TomeConta.mdf',
MOVE 'TomeConta_log' TO '/var/opt/mssql/data/TomeConta_log.ldf'
GO
```
6. Verifique se a restauração funcionou recuperando as tabelas presente no banco.

```shell
USE TOMECONTA
SELECT * FROM information_schema.tables;
GO
```

7. Por fim, para interromper a execução do container execute:
```shell
sudo docker-compose down
```

Informações sobre os dados estão disponíveis na pasta **doc**.

## ⛏️ Ferramentas <a name = "built_using"></a>

- [Docker](https://www.docker.com/) - Containers
- [SQL Server](https://www.microsoft.com/pt-br/sql-server/sql-server-downloads) - Sistema gerenciador de Banco de dados relacional


