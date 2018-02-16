### Below I'm upgrading 7.0.57 to 8.5.27
1. `sudo /etc/init.d/tomcat stop`

1. `sudo su - tomcat`
1. `wget http://apache.mirror.anlx.net/tomcat/tomcat-8/v8.5.27/bin/apache-tomcat-8.5.27.tar.gz`
1. `tar -xzf apache-tomcat-8.5.27.tar.gz`
1. `cp -r $CATALINA_BASE/conf $CATALINA_BASE/old_conf` back up the old server settings because we will wipe them below
1. `vim ~/.bash_profile` and edit the tomcat version
1. `. ~/.bash_profile` to reload your profile with the new settings
1. `echo $CATALINA_BASE`
1. `echo $CATALINA_HOME` - checking the out is correct for your new version
1. `cp -r $CATALINA_HOME/conf $CATALINA_BASE`
1. `cd $CATALINA_HOME/bin`
1. `rm -r *.bat`
1. `tar -xzf tomcat-native.tar.gz`
1. `cd tomcat-native-*-src/native/`
1. `yum install openssl`
1. `./configure --prefix=/home/tomcat/apache-tomcat-8.5.27 --with-ssl=/usr/include/openssl --with-java-home=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64 --with-apr=/usr/local/apr` -- make sure the java version is correct
1. `make && make install`
1. `vim $CATALINA_BASE/bin/setenv.sh` --ensuring any settings are correct
1. `cd $CATALINA_BASE`
1. `vim conf/server.xml` and add the entries back in from `old_conf/server.xml`
1. `vim conf/Catalina/{site}/context.xml.default` to reflect the other changes the upgrade may require, check ssplus.xml default context in sts.
1. logout tomcat user
1. `sudo cp /etc/init.d/tomcat7 /etc/init.d/tomcat8
1. `sudo vim /etc/init.d/tomcat8` and update the tomcat version
1. `sudo ln -sf /etc/init.d/tomcat8 /etc/init.d/tomcat`
1. `sudo /etc/init.d/tomcat start`
1. `sudo service nginx restart` - to just restart if you needed to add/update any entries.