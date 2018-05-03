# Datastore Jobserver
We are currently using a forked version of spark job server.
This initially started because the mainline branch did not support our version of spark(2.0).
 
Since then we have made a few changes to fix stability issues that have not yet been merged back into the mainline project.

## Making changes
For this project we follow the normal RMS github workflow,
the main difference is that we work of off rms-develop and rms-master.

That means feature branches are created from and PR'd against rms-develop.
rms-master will then be updated with changes in rms-develop. 


## Running locally

The instructions in [README](README.md) work for running in client mode locally.
So far, we have not been able to run in cluster mode.
This is due mostly to the way in which akka cluster identifies members of the cluster,
and we haven't found a work around yet.

When running locally the following `dev.conf` file has worked

```
spark {

  master = "local[*]"
  submit.deployMode = "client"
  webUrlPort = 8080

  jobserver {
    # Dev debug timeouts
    context-per-jvm = false
    short-timeout = 60s

    context-deletion-timeout = 60000 ms
    context-creation-timeout = 1000000 s
    yarn-context-creation-timeout = 1000000 s
    default-sync-timeout = 1000000 s


    jobdao = spark.jobserver.io.JobSqlDAO

    filedao {
      rootdir = /tmp/spark-jobserver/filedao/data
    }

    datadao {
      # storage directory for files that are uploaded to the server
      # via POST/data commands
      rootdir = /tmp/spark-jobserver/upload
    }

    sqldao {
      # Slick database driver, full classpath
      slick-driver = slick.driver.H2Driver

      # JDBC driver, full classpath
      jdbc-driver = org.h2.Driver

      # Directory where default H2 driver stores its data. Only needed for H2.
      rootdir = /tmp/spark-jobserver/sqldao/data

      # Full JDBC URL / init string, along with username and password.  Sorry, needs to match above.
      # Substitutions may be used to launch job-server, but leave it out here in the default or tests won't pass
      jdbc {
        url = "jdbc:h2:file:/tmp/spark-jobserver/sqldao/data/h2-db;AUTO_SERVER=TRUE"
        user = ""
        password = ""
      }

      # DB connection pool settings
      dbcp {
        enabled = false
        maxactive = 20
        maxidle = 10
        initialsize = 10
      }
    }

    startH2Server = false
  }

  context-settings {
    # Dev debug timeout
    context-init-timeout = 1000000 s

    passthrough {
      #es.nodes = "192.1.1.1"
      kuduMaster = "10.92.2.42:7051,10.92.2.49:7051,10.92.2.45:7051"
      spark.serializer = "org.apache.spark.serializer.KryoSerializer"
      spark.sql.hive.thriftServer.singleSession = "true"
    }
  }
}
spray.can.server {
  # Debug timeouts
  idle-timeout = infinite
  request-timeout = infinite

  parsing.max-content-length = 512m
}

deploy {
  manager-start-cmd = "/Users/jbuszkiewic/Projects/spark-jobserver/bin/manager_start.sh"
}
```   

## Deployment

Currently deployment is a manual process that is done with the help of the platform team.
Rodrick (@rbrown-rms) has been the primary point of contact in regards to this.

In order to deploy we need to create a new branch with our changes
that follows the following naming convention `rms-{version}` (i.e rms-0.8.2).
Once that happens we need to make sure that [recipe.rb](https://github.com/RMS/rms-ansible/blob/develop/roles/job-server/fpm-recipes/recipe.rb) is changed to pick up the new changes.

In the future we will work with the platform and release teams to get spark job server building in a way that matches our other projects.
