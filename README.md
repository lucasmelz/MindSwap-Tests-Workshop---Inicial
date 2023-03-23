
1. Pull Postgres Docker Image:<br>
`docker pull postgres`


2. Run the docker container:<br>

`docker run --name myPostgresDb -p 5455:5432 -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -e POSTGRES_DB=postgresDB -d postgres`

3. Run a bash on the container:

`docker exec -it myPostgresDb bash`


4. Execute PostgreSQL with the role "user"<br>
`psql postgresDB -U user`


5. Create database: <br>
`create database test; `<br>
Optional: You can list all databases with `\l` and connect to the test database with the command `\c test`. You can list all the tables with the command `\dt`.


6. Confirm that your application.properties has the following lines:

<pre><code>spring.datasource.url=jdbc:postgresql://localhost:5455/test
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.username=user
spring.datasource.password=password
</code></pre>