:orphan:

CKAN uses Solr_ as its search platform, and uses a customized Solr schema file
that takes into account CKAN's specific search needs. Now that we have CKAN
installed, we need to install and configure Solr.

.. _Solr: http://lucene.apache.org/solr/

.. note::

   These instructions explain how to deploy Solr using the Jetty web
   server, but CKAN doesn't require Jetty - you can deploy Solr to another web
   server, such as Tomcat, if that's convenient on your operating system.

#. Edit the Jetty configuration file (``/etc/default/jetty``) and change the
   following variables::

    NO_START=0            # (line 4)
    JETTY_HOST=127.0.0.1  # (line 16)
    JETTY_PORT=8983       # (line 19)

   .. note::

    This ``JETTY_HOST`` setting will only allow connections from the same machine.
    If CKAN is not installed on the same machine as Jetty/Solr you will need to
    change it to the relevant host or to 0.0.0.0 (and probably set up your firewall
    accordingly).

   Start the Jetty server::

    sudo service jetty start

   You should now see a welcome page from Solr if you open
   http://localhost:8983/solr/ in your web browser (replace localhost with
   your server address if needed).

   .. note::

    If you get the message ``Could not start Jetty servlet engine because no
    Java Development Kit (JDK) was found.`` then you will have to edit the
    ``JAVA_HOME`` setting in ``/etc/default/jetty`` to point to your machine's
    JDK install location. For example::

        JAVA_HOME=/usr/lib/jvm/java-6-openjdk-amd64/

    or::

        JAVA_HOME=/usr/lib/jvm/java-6-openjdk-i386/

#. Replace the default ``schema.xml`` file with a symlink to the CKAN schema
   file included in the sources.

   .. parsed-literal::

      sudo mv /etc/solr/conf/schema.xml /etc/solr/conf/schema.xml.bak
      sudo ln -s |virtualenv|/src/ckan/ckan/config/solr/schema.xml /etc/solr/conf/schema.xml

   Now restart Solr:

   .. parsed-literal::

      |restart_solr|

   and check that Solr is running by opening http://localhost:8983/solr/.


#. Finally, change the :ref:`solr_url` setting in your :ref:`config_file` (|production.ini|) to
   point to your Solr server, for example::

       solr_url=http://127.0.0.1:8983/solr

   .. note::

    These instructions below will explain how to deploy Solr using the Tomcat webserver,

    #.  Download solr-1.4.1 from  https://archive.apache.org/dist/lucene/solr/1.4.1/
        Unpack the data then copy and paste the directory into the directory where the packages are
        installed into for easier path description in the configurations of the servers.

        .. parsed-literal::

          (default):cd /path/to/solr-1.4.1/example
          (default):sudo mv solr/conf/schema.xml solr/conf/schema.xml.backup
          (default):sudo ln -s /usr/local/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml solr/conf/schema.xml

    #.  Install Tomcat

        Download Apache Tomcat from http://tomcat.apache.org/download-70.cgi

        a. Unpack the files and copy them to your local directory

        .. parsed-literal::

          tar -zxvf apache-tomcat-7.0.56.tar.gz
          mv apache-tomcat-7.0.56/ /usr/local/apache-tomcat(your path)

        b. Configure Apache Tomcat:

          i. Copy solr.war to tomcat webapps/ directory:

              .. parsed-literal::

                (default):sudo cp /path/to/solr-1.4.1/example/webapps/solr.war /usr/local/apache-tomcat/webapps/solr.war

          ii. To start Apache-Tomcat:

                .. parsed-literal::

                  (default):/usr/local/apache-tomcat/bin/startup.sh

          iii. To stop Apache-Tomcat:

                .. parsed-literal::

                  (default):/usr/local/apache-tomcat/bin/shutdown.sh

          iv. Configure the Tomcat Servlet:

                .. parsed-literal::

                  cd /usr/local/apache-tomcat/webapps/solr/WEB-INF
                  (your preferred text editor) web.xml

          v. Within the file there is a commented out part of the code that mentions
             "People who want to hardcode their "Solr Home" directly into the WAR File can set the JNDI property here..."
             Replace the commented out code with:

                .. parsed-literal::

                  <env-entry>
                  <env-entry-name>solr/home</env-entry-name>
                  <env-entry-value>/usr/local/Cellar/solr14/1.4.1/libexec/example/solr</env-entry-value>
                  <env-entry-type>java.lang.String</env-entry-type>
                  </env-entry>

              **Warning**:the env-entry-value is solr related configuration and index. (e.g. schema.xml)

          vi. Set the Apache-Tomcat Port (default: 8080): --(optional):

                .. parsed-literal::

                  cd /usr/local/apache-tomcat/conf
                  (your preferred text editor) server.xml


                  <Connector port="8983" protocol="HTTP/1.1" -- set:8983
                  connectionTimeout="20000"
                  redirectPort="8443" />

          vii. Running Apache Tomcat:

                .. parsed-literal::
                  (default):/usr/local/apache-tomcat/bin/startup.sh

        c. Open http://127.0.0.1:8983/solr in a web browser to ensure it works.
