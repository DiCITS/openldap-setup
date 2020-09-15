# openldap-setup
OpenLDAP setup for Fedora 32 and Ubuntu 18.04 LTS (Service and Clients)

## OpenLDAP Clients

In Ubuntu 18.04 LTS run as sudo:

```
apt -y install libnss-ldap libpam-ldap ldap-utils 
```

If the packages have been reinstalled several times and the graphical application that should be displayed does not appear, then reinstall with: 

```
apt-get purge libnss-ldap libpam-ldap ldap-utils
apt -y install --reinstall libnss-ldap libpam-ldap ldap-utils
```

Follow the installation steps: 

```
(1) specify LDAP server's URI
```

Use for example: ``ldap://<IP>`` or ``ldap://<DOMAIN>`` or 

```
(2) specify suffix
```

Use for example: ``dc=imuds,dc=es``


```
(3) specify LDAP version (generally OK to select Version [3])
```

Use version 3 by default

```
(4) select the one you like. ( this example selects [Yes] )
...
Make local root Database admin:  [Yes] [No]
```

Use ``[No]``


```
(5) select the one you like. ( this example selects [No] )
...
 Does the LDAP database require login? [Yes] [No]
```

Use ``[Yes]``


```
(6) specify LDAP admin account's suffix
```
Here you must use the connection chain that your server provides you with where you can consult the users and you have permissions to do so. Use ``cn=admin,dc=imuds,dc=es``


```
(7) specify password for LDAP admin account
```

Use the LDAP service management account key that allows to query/manage users in the ``cn=admin,dc=imuds,dc=en``

After the installation is completed, the following files must be modified:

```
vi /etc/nsswitch.conf 
```

Add ``ldap`` to the ``passwd`` and ``group`` entities and you must leave exactly the mechanisms that appear for each entity,  as indicated here:
```
passwd:         compat systemd ldap
group:          compat systemd ldap
shadow:         compat
```





