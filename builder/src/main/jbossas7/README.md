Deployment onto JBoss
==========================

The <code>jbossas7</code> directory contains all the artifacts to build the distribution for the JBoss AS 7.1.1

Please follow the next steps in order to deploy the application.

Generate a distribution
---------------------------

Open a terminal window, go to the root directory and execute the following Maven command:

    $ mvn clean install -Dfull  -DskipTests

Go to the <code>builder/target</code> directory and get the file called **jbpm-dashbuilder-jboss-as7.0.war**.

Deploy the jBPM Dashboard
----------------------------

The jBPM Dashboard requires the jBPM History log's database tables to exist. So, it's **MANDATORY** to deploy the
jBPM Human Task console before the jBPM Dashboard. Otherwise, the jBPM Dashboard will not be initialized correctly and
will not be possible to display its key performance indicators.

Get the proper WAR file (e.g. jbpm-dashbuilder-jboss-as7.0.war) and execute the following command:

    $ cd <jboss_home>/bin
    $ ./jboss-cli.sh --connect --command="deploy <path_to_war_file>"

    <path_to_war_file>: is the local path to the application war file.
    e.g. $ ./jboss-cli.sh --connect --command="deploy /home/myuser/myfiles/jbpm-dashbuilder-jboss-as7.0.war" )


The application is configured to use a datasource with the following JNDI name: <code>java:jboss/datasources/ExampleDS</code>.
Notice, this datasource is intended for development/demo purposes and it's present by default at any JBoss installation.

If you want to deploy on a database different from H2 like Oracle, MySQL, Postgres or MS SQL Server please follow the next steps:

* Install the database driver on JBoss (read the JBoss documentation)

* Create an empty database and a JBoss data source which connects to it

* Modify the file *jbpm-dashboard/builder/src/main/jbossas7/WEB-INF/jboss-web.xml*:

        <jboss-web>
           <context-root>/dashbuilder</context-root>
           <resource-ref>
               <res-ref-name>jdbc/dashbuilder</res-ref-name>
               <res-type>javax.sql.DataSource</res-type>
               <jndi-name>java:jboss/datasources/myDataSource</jndi-name>
           </resource-ref>
           ...

   Replace the *jndi-name* parameter value by the JNDI path of the JBoss data source you've just created.

* Modify the file *jbpm-dashboard/builder/src/main/jbossas7/WEB-INF/jboss-deployment-structure.xml*.

  Add the following snippet of configuration inside the *deployment* tag, where *jdbcDriverModuleName* is the name of the JBoss JDBC driver module.

        <dependencies>
            <module name="jdbcDriverModuleName" />
        </dependencies>


After that, your are ready to generate a WAR distribution prepared for the target database.

Single Sign On
---------------------------------

In order to enable SSO between the jBPM Dashboard and the jBPM Human Task Console, please, edit the
<code>[jboss-as7]/standalone/configuration/standalone.xml</code> file and add the <code>&lt;sso/&gt;</code> tag under the virtual server configuration.


    ...
    <virtual-server name="default-host" enable-welcome-root="true">
        <alias name="localhost"/>
        <alias name="example.com"/>
         <sso/>
    </virtual-server>
    ...

User Authentication
--------------------------

Once started, open a browser and type the following URL:
<code>http://localhost:8080/dashbuilder</code>. A login screen should be displayed.

However, some extra configuration is needed before you can sign in:

* The application is based on the J2EE container managed authentication  mechanism.
This means that the login itself is delegated to the application server.

* First of all, in order to login as superuser, using the <code>[jboss-as7]/bin/add-user.sh</code> command utility,
you must create a user with login=<code>root</code> and role=<whatever role has been defined in the web.xml file>.
This is just for container authentication purposes, as the root user's application privileges are not role-linked,
but instead is granted with all permissions).

* The application roles are defined at [builder/src/main/jbossas7/WEB-INF/web.xml](https://github.com/droolsjbpm/jbpm-dashboard/blob/master/builder/src/main/jbossas7/WEB-INF/web.xml) file.
Roles can be used to create access profiles and define custom authorization policies.

* The application uses the JBoss' default security domain as you can see [here](https://github.com/droolsjbpm/jbpm-dashboard/blob/master/builder/src/main/jbossas7/WEB-INF/jboss-web.xml).
Alternatively, you can define your own security domain and use, for instance, an LDAP, a database, or whatever mechanism you want to use as your credential storage.
There are plenty of examples in the JBoss AS documentation about.

Feel free to change any settings regarding the application security and, once finished, to generate a distribution war that fits your needs.

jBPM Process Dashboard
-----------------------------

The jBPM Process Dashboard can be accessed in two ways:

* As root, by selecting the jBPM Dashboard workspace at the top administration toolbar, or
* Typing the following URL: <code>http://localhost:8080/dashbuilder/workspace/jbpm-dashboard</code>

You can create process instances and tasks from the jBPM Human Task console and later on refresh the jBPM Dashboard to
see how the different key performance monitoring indicators are updated.

