# AWS Lambda and MS SQL

This document is a guide for connecting AWS lambda to Microsoft SQL Server. Lambda will run Python 3.6+ and will need the pyodbc module and ODBC driver to communicate with the database. However, these are not included in AWS lambda. An API gateway can be connected to Lambda for clients to run CRUD operations to the database. 

# Requirements

```
RDS MSSQL
Lambda function (Python 3.6)
```

Pyodbc and ODBC is not included in AWS lambda so they must be compiled on a platform, zipped and uploaded. We will use a free-tier EC2 AMI instance (this will mock the Lambda execution environment) to compile the program:

1. Launch an EC2 AMI instance and install and start docker.

   `sudo yum install docker`

   `sudo service docker start`

2. Pull images of the lambda environment and start using bash inside the container.

   `sudo docker run -it --rm --entrypoint bash  -e ODBCINI=/var/task -e 
   ODBCSYSINI=/var/task -v "$PWD":/var/task  lambci/lambda:build-python3.6`

3. Change directory to `/var/task` 

4. Download ODBC source code, compile and take the output

   ```curl ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.5.tar.gz -O
   tar xvzf unixODBC-2.3.5.tar.gz
   cd unixODBC-2.3.5
   ./configure  --sysconfdir=/var/task  --disable-gui --disable-drivers 
   --enable-iconv --with-iconv-char-enc=UTF8 --with-iconv-ucode-enc=UTF16LE
   --prefix=/home
   make install
   cd ..
   mv /home/* .
   mv unixODBC-2.3.5 unixODBC-2.3.5.tar.gz /tmp/```
   ```

5. Install the MS SQL ODBC Driver

   ```curl https://packages.microsoft.com/config/rhel/6/prod.repo > /etc/yum.repos.d/mssql-release.repo
   curl https://packages.microsoft.com/config/rhel/6/prod.repo > /etc/yum.repos.d/mssql-release.repo
   ACCEPT_EULA=Y yum -y install msodbcsql
   export CFLAGS="-I/var/task/include"
   export LDFLAGS="-L/var/task/lib"```
   ```

6. Install pyodbc

   `pip install pyodbc -t .`

7. Copy the driver to current working directory `cp -r /opt/microsoft/msodbcsql .`

8. Run `cat <<EOF > odbcinst.ini` and add the following:

   ```
   [ODBC Driver 13 for SQL Server]
   > Description=Microsoft ODBC Driver 13 for SQL Server
   > Driver=/var/task/msodbcsql/lib64/libmsodbcsql-13.1.so.9.2
   > UsageCount=1
   > EOF
   ```

9. Run `cat <<EOF > odbc.ini` and add the following:

   ```
   [ODBC Driver 13 for SQL Server]
   Driver      = ODBC Driver 13 for SQL Server
   Description = My ODBC Driver 13 for SQL Server
   Trace       = No
   EOF
   ```

10. Test your driver:

    `python -c "import pyodbc; print(pyodbc.drivers());"`

    It should return `['ODBC Driver 13 for SQL Server']`

11. Package is ready zip and store it on your local machine. Exit the docker container.

    ```
    cd ..
    tar -zcvf pyodbcLambdaPackage.tar.gz /task
    ```

12. Copy the zip file from the docker container to your EC2 instance

    ```
    sudo docker ps -a
    sudo docker cp {containerId}:/var/pyodbcLambdaPackage.tar.gz .
    ```

13. Copy the zip file from the EC2 instance to your localhost machine.

    ```
    scp -i {path to your keyfile} ec2-user@{EC2-ip-address}:/home/ec2-user/pyodbcLambdaPackage.tar.gz .\Downloads\
    ```

14. Unpack the zip file and delete:

    ```
    .ssh
    /pyodbc-4.0.26.dist-info
    /share
    .bash_logout
    .bash_profile
    .bashrc
    ```

15. Add a new python file name 'test.py' (can name it to anything you want) containing the following:

    ```
    import pyodbc
    import json
    
    def connect():
        connection = pyodbc.connect('Driver={ODBC Driver 13 for SQL Server};'
                                    'Server=xxxxxxx,1433;'
                                    'Database="";'
                                    'uid=xxxxxxxx;'
                                    'pwd=xxxxxxxx;'
                                    'Trusted_Connection=no;')
        return connection
    
    def lambda_handler(event, context):
    	query = 'SELECT * FROM dbo.tb_Names'
        connection = connect()
        cursor = connection.cursor();
        cursor.execute(query)
    
        for row in cursor:
            print(row)
    
        return {
                'statusCode': 200,
                'body': json.dumps('Hello from Lambda!')
        }
    ```

    You can add your own function instead. The code above is used to test the connection to the database. 

    ​	`Server` - the server name with default port of 1433

    ​	`Database` - the name of the database

    ​	`uid` - user id

    ​	`pwd` - password

    ​	`query` - SQL query

16. Zip all the files and upload it to AWS Lambda. Set the handler to `test.lambda_handler` (`test` for the file name test.py, `lambda_handler` for the definition). Save and test. If test fails, check your imported modules and database connection string.

# References

https://github.com/Miserlou/lambda-packages/tree/master/lambda_packages/pyodbc

https://medium.com/faun/aws-lambda-microsoft-sql-server-how-to-66c5f9d275ed