# Ansible Role: contrast-tomcat

## Purpose
Install Contrast and optionally configure and enable it in the tomcat installation.
The installation is done for an environment. 

The default installation directory is **/opt/contrast**
We create on subdirectory for the environment where the jar file is copied into.
The log directory is **/opt/contrast/<env>/logs**

## Role Parameters
You must specify the environment with on the command line with **--exta-vars 'env=\<env\>'**
The **env** parameter is only used to create distinct directories for an environment. The Tomcat configuration of Contrast will point to this directory.

## Role Default Variables

Available variables are listed below, along with default values (see defaults/main.yml):

```
---

# Installation directory for Contrast
contrast_home: /opt/contrast

# This flag controls whether or not we modify the setenv.sh file
# with the Contrast parameters
configure_tomcat: true

# When this flag is true, then the JAVA_OPTS -Dcontrast.enabled flag is set to true
# Most environments have this flag disabled
contrast_enabled: false

# Tomcat home
tomcat_home:
tomcat_owner:

```



## Tags
###config-tomcat
Use this tag to run only the tomcat configuration or skip it.


## Example Playbook

```

---
# file: site.yml
#
# Consult contrast-tomcat/defaults/main.yml for a list of variables you can change
#
# Usage:
# ansible-playbook -i hosts -e "env=<env>" site.yml
# env is one of: dev | devOnPerf | ausdev1 | qa | perf | opuat | usuat | hdi | rnd | cprod | sdev
# 
# Example:
# ansible-playbook -i hosts -e dev site.yml
#

- name: Install Contrast and configure Tomcat
  hosts: all
  become_user: root
  become_method: sudo
  become: true
  gather_facts: true

  vars:
    tomcat_home: <tomcat_home>
    tomcat_owner: <tomcat_owner>
    configure_tomcat: false

  roles:
    - { role: contrast-tomcat, tags: ['contrast'] }

```

## Use Cases
Install Contrast with the default setting in environment sdev
```
ansible-playbook -i hosts -e 'env=sdev' site.yml
```
This will install the software and configure and existing setenv.sh to include contrast.jar into the JAVA_OPTS but keeps Contrast disabled

Install Contrast and enable it
```
ansible-playbook -i hosts -e 'env=sdev contrast_enabled=true' site.yml
```

Install Contrast only
```
ansible-playbook -i hosts -e 'env=sdev' --skip-tags=config-tomcat site.yml
```

Install Contrast only
```
ansible-playbook -i hosts -e 'env=sdev' --skip-tags=config-tomcat site.yml
```

Configure Tomcat only
```
ansible-playbook -i hosts -e 'env=sdev' --tags=config-tomcat site.yml
```

## Tomcat Configuration
To configure Tomcat the file setenv.sh is edited and a whole block of JAVA_OPTS is added. 
Ansible searches for a line starting
```
# <<contrast-config-position>>
```
and insert the Contrast configuration below.
If this line is not found, the configuration is put at the end of the file

The result might look like this:
```
# Ansible managed
JAVA_HOME=/usr/lib/jvm/jre
JRE_HOME=/usr/lib/jvm/jre
CATALINA_PID="/opt/ndxis/tc/temp/tomcat.pid"
CATALINA_HOME="/opt/ndxis/tc"
CATALINA_BASE="/opt/ndxis/tc"
#JAVA_OPTS="-server -XX:PermSize=128m -XX:MaxPermSize=512m"
JAVA_OPTS="${JAVA_OPTS} -DappEnv=cloud -DappEnvImage=CodingCloud"
#CATALINA_OPTS="-Xms2g -Xmx3g"

# <<contrast-config-position>> DO NOT DELETE THIS LINE
# BEGIN ANSIBLE MANAGED BLOCK FOR Contrast ==========================
# Contrast
export JAVA_OPTS="$JAVA_OPTS -XX:+PrintVMOptions"
export JAVA_OPTS="$JAVA_OPTS -XX:+PrintCommandLineFlags"
export JAVA_OPTS="$JAVA_OPTS -javaagent:/opt/contrast/sdev/contrast.jar"
export JAVA_OPTS="$JAVA_OPTS -Dcontrast.enabled=true"
export JAVA_OPTS="$JAVA_OPTS -Dcontrast.env=sdev"
export JAVA_OPTS="$JAVA_OPTS -Dcontrast.server=SDEV-10.90.24.81"
export JAVA_OPTS="$JAVA_OPTS -Dcontrast.dir=/opt/contrast/sdev/logs"
# END ANSIBLE MANAGED BLOCK FOR Contrast ==========================

export CLASSPATH=/etc/hadoop/conf
export HADOOP_HOME=/etc/hadoop
export HADOOP_CLASSPATH=/etc/hadoop/conf
JAVA_OPTS="$JAVA_OPTS -javaagent:/home/ndxis/tc/newrelic/newrelic.jar"
```


## Dependencies

The Tomcat configuration will be executed if the file {{ tomcat_home }}/bin/setenv.sh exists

