# Technology Readiness Tool Documentation

Technology Readiness Tool (TRT) is a Java web application that stores information about resources used for online testing
in schools.

## What is included
1. Java web application source code
    - User interface application
    - Batch file processing application
2. Database creation script

## Dependencies

- [Java 1.7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)
- [Tomcat 7](http://tomcat.apache.org/download-70.cgi)
- [MySQL](http://dev.mysql.com/downloads/mysql/)

## Compiling Java
TRT uses Maven to compile and package the deployable binaries.

### Install Maven
Download the [Maven binary](http://maven.apache.org/download.cgi) or install with your package manager.

The project is made up of several sub-modules

- core-components - Shared set of classes used for batch processing and the user interface
- interface-components - JSP tag library and supporting Struts 2 extension classes
- readiness-plugin - Struts 2 actions and JSPs for reporting pages
- batch-webapp - Application responsible for executing file based processing
- readiness - Application that handles user facing functions

The standard Maven commands are supported, 'mvn clean package' builds the binaries. They are located in each of the project's
target directory.

## Configuration

Each application looks for a .properties file at startup for environment specific configuration.

- application-batch.properties - batch-webapp
- application-customer.properties - readiness

The default values for the values can be found in:

- batch-webapp/WEB-INF/application-batch-dev.properties
- readiness/WEB-INF/application-customer-dev.properties

### Properties
#### Cache Configuration
- cache.config: Controls the ehcache configuration file that is loaded. 'dev' is the only value that is supported, ehcache-[dev].xml is the file that is loaded.
- cache.second.level: Passthrough property to 'hibernate.cache.use_second_level_cache' when configuring the Hibernate EntityManager. 'true' and 'false' are valid values.

#### Application Deployment Configuration
The app.customer.* properties are used to generate URLs for login, password reset and links in email templates. These properties do not change the URL of the application. This must be done through DNS and Tomcat configuration.

- app.customer.host: The host portion of a URL that the interface application is deployed at.
- app.customer.port: The port portion of a URL that the interface application is deployed at.
- app.customer.contextPath: The base path portion of a URL that the interface application is deployed at.
- app.customer.protocol: The protocol portion of a URL that the interface application is deployed at.

#### E-mail configuration
emailService* properties control how the system generates and sends emails.

- emailServiceHostName: The host name of the SMTP server that should be used to send emails.
- emailServiceReplyAddress: The email address that will populate the FROM field.
- emailServiceReplyName: The name that will populate the FROM field.
- emailServiceSMTPuser: The username that is used to connect to the SMTP server.
- emailServiceSMTPpass: The password that is used to connect to the SMTP server.
- emailServiceSMTPport: The port to use to connect to the SMTP server.
- emailServiceSSLport: The SSL port to use to connect to the SMTP server.
- emailServiceUseTLS: Use TLS to connect to the SMTP server, true or false.
- emailServiceUseSSL: Use SSL to connect to the SMTP server, true or false.

#### User File Upload Configuration
- file.upload.dir: The directory that will be used to hold file uploads for batch processing.
- file.temp.export.dir: The directory that will be used for temporary files during batch processing.

#### Authentication Configuration
- login.authentication: The type of authentication to use. Valid values are 'dev' and 'cas'. 'dev' mode does not require a password.
- cas.service.url: The service that represents the TRT to the CAS server.
- cas.login.url: The URL for the CAS login page
- cas.logout.url: The URL that invalidates CAS tickets
- cas.validator.url: The URL that validates CAS tickets
- cas.auth.key: A unique key that represents the CAS server

## Tomcat Configuration

### Properties files

The properties files described in the Configuration section can be outside of the application packages. This can be configured in Tomcat by setting a shared loader location for adding files to deployed application’s classpath. See Tomcat’s documentation about the shared loader and the ‘shared.loader’ property in the [tomcat_home]/conf/catalina.properties file.

### Database Connection Pool

The applications require a database connection pool in JNDI.

#### JDBC Driver

The connection pool requires a JDBC driver. Download the [MySQL Connection/J](http://dev.mysql.com/downloads/connector/j/) archive. The JDBC driver is packaged as a JAR. Copy the 'mysql-connector-java-[version]-bin.jar' to the [tomcat_home]/lib directory.

#### Defining the Connection Pool

Add a resource element to Tomcat’s [tomcat_home]/conf/context.xml file.

- [db_username]: The username of a user defined in the MySQL server.
- [db_password]: The password of the MySQL user.
- [schema_name]: The schema name that has the tables for the readiness application. This should be 'core'. References to other schemas need to be prefixed in the query.

```xml
    <Resource auth="Container"
      driverClassName="com.mysql.jdbc.Driver"
      name="core_connection"
      username="[db_username]"
      password="[db_password]"
      type="javax.sql.DataSource"
      url="jdbc:mysql://localhost:3306/[schema_name]?autoReconnect=true"
      validationQuery="SELECT 1"
      testOnBorrow="true" />
```

Note: Setup the `autoReconnect` in the url to reconnect timed out connections. Set the `testOnBorrow` to force a test of the connection using the `validationQuery` before using it each time. The `wait_timeout` period can be increased from the default of 8 hours if desired.

## Database
Execute the database script to create the required tables for the application. The script is 'database.sql'. The application requires three schemas to run: core, core_batch and readiness. The core schema contains organization, device and consortia information. The core_batch schema has the tables required for dependencies on Quartz and Spring Batch. The readiness schema has tables for snapshot reporting data.

### MySQL Configuration

Enable case insensitive table names for queries. With this option MySQL will store all tables in lowercase and queries can reference tables in either uppercase or lowercase. Example my.cnf:
```properties
lower_case_table_names=1
```

Increase the wait_timeout value for connections to the database. Default is 8 hours (28800 seconds).
```properties
wait_timeout=28800
```

## Developer Mode

### Logging In

The database script creates a single user with the username 'ready_admin'. The application has two authentication modes, dev and cas. The default is dev. This allows authentication for any valid username without a password. Enabling 'cas' mode will force the application to use a [CAS](http://www.jasig.org/cas) instance for authenticating users. See [authentication properties](#authentication-configuration) section for the required configuration values when using CAS.

### Struts

Struts also supports a developer mode. This is enabled by defining a constant in either a struts-plugin.xml or struts.xml. See the Struts [documentation](http://struts.apache.org/release/2.3.x/docs/devmode.html) for more information.

## Troubleshooting

The following log entry can be safely ignored.

```properties
0    [localhost-startStop-1] ERROR org.hibernate.ejb.metamodel.MetadataContext  - Unable to locate static metamodel field : net.techreadiness.persistence.domain.ScopeDO_#path
```
