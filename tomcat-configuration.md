Tomcat configuration on RHEL 7
==============================

## Tomcat user and group

The tomcat user login shell is set to /sbin/nologin.  This means that su - tomcat will not work.
The account is a service only accont, and the service is managed by systemd.

```
usermod -s /sbin/nologin
```

The tomcat home directory is /opt/tomcat, and the permissions are set to 750

Other system daemons that need access to the tomcat installation directory
should be added to the tomcat group:

```
$ sudo usermod -a -G tomcat new-group
```

## Tomcat installation

Tomcat is installed from the Apache upstream and extracted in /opt.  There is a symlink
to /opt/tomcat so it can be upgraded without having to move the directory.

```
$ cd /opt
$ sudo tar xvsf apache-tomcat-version.tar.gz
$ sudo ln -s /opt/apache-tomcat-version /opt/tomcat
```

Tomcat manager and ROOT removed from /opt/tomcat/webapps:

```
$ cd /opt/tomcat/webapps
$ sudo rm -rf ROOT manager
```

Permissions on /opt/apache-tomcat-version are to be set to 750

```
$ sudo chmod -R 750 /opt/tomcat
```

## Java Version

OpenJDK over Oracle JDK
java-1.8.0-openjdk.x86_64

## Tomcat service

systemd script for service management:

```
# Systemd unit file for Apache Tomcat
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms3072M -Xmx3072M -Xss256k'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

After installing the unit file, reload systemd:

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable tomcat.service
```

To start, stop, and restart the service:

```
$ sudo systemctl start tomcat
$ sudo systemctl restart tomcat
$ sudo systemctl stop tomcat
```

Status for the service:

```
$ sudo systemctl status tomcat
```

## Deployments

Deployments done from Jenkins will use the deploy account.  This account should
be present on all servers that will be deployed to.  The following commands
are used to set up this user:

```
$ sudo adduser deploy
$ sudo passwd -l deploy
$ sudo usermod -a -G tomcat deploy
```

## Splunk

The splunk user will need added to the tomcat group to have access to the logs:

```
$ sudo usermod -a -G tomcat splunk
```
