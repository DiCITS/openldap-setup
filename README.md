# openldap-setup
OpenLDAP setup for Fedora 32 and Ubuntu 18.04 LTS (Service and Clients)


## Create a test user

Create a new SHA Password if you have not ldap-tools installed (otherwise install open-ldap package)

```
docker run cmd.cat/slappasswd slappasswd -s mypasswordtest
```

Returned password must be set in the ``userPassword`` parameter within ``testuser.ldif`` file.


The ``testuser.ldif`` file will be the next one:

```
dn: uid=test,ou=Users,dc=imuds,dc=es
objectClass: top
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Test
sn: Test 
initials: TST
uid: test
uidNumber: 11005
gidNumber: 11005
loginShell: /bin/bash
homeDirectory: /home/test
mail: test@domain.net
userPassword: XXXXXXXXXXXXXXX
shadowLastChange: 14083
shadowMax: 99999
shadowMin: 0
shadowWarning: 7
gecos: Test account
```


Add the user to the LDAP server:

```
ldapadd -h ldap://<IP/DOMAIN> -x -D cn=admin,dc=imuds,dc=es -W -f testuser.ldif 
```

Use the ``admin`` password to commit the changes.

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

Follow the installation steps in the graphical environment (it will appear during installation): 

```
(1) specify LDAP server's URI
```

Use for example: ``ldap://<IP>`` or ``ldap://<DOMAIN>``. It is preferable to use the domain name for the LDAP server address, for this you must configure it in the list of hosts (``/etc/hosts``). 

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

Use the LDAP service management account key that allows to query/manage users in the ``cn=admin,dc=imuds,dc=es``

After this setup run:


```
pam-auth-update --force
```

Select the following ``[*]`` :


```
  │    [ ] Pwquality password strength checking                                                                  │ 
  │    [*] Unix authentication                                                                                   │ 
  │    [ ] SSS authentication                                                                                    │ 
  │    [*] LDAP Authentication                                                                                   │ 
  │    [*] Register user sessions in the systemd control group hierarchy                                         │ 
  │    [*] Create home directory on login                                                                        │ 
  │    [*] Inheritable Capabilities Management
```

After the installation is completed, the following files must be modified:

#### /etc/nsswitch.conf

```
vi /etc/nsswitch.conf 
```

Add ``ldap`` to the ``passwd`` and ``group`` entities and you must leave exactly the mechanisms that appear for each entity,  as indicated here:
```
passwd:         compat systemd ldap
group:          compat systemd ldap
shadow:         compat
gshadows:       files
....
```

#### /etc/pam.d/common-password 

```
vi /etc/pam.d/common-password 

```


Go to the line that show that info, and  remove ``[use_authtok]``, let the sentence like the following:

```
password        [success=1 user_unknown=ignore default=die]     pam_ldap.so try_first_pass
```

#### /etc/pam.d/common-session 

```
vi /etc/pam.d/common-session 
```

Add to the end of the file the following sentence, in order to add home folder when user is logged in a node where the user previously has no home:

```
session optional        pam_mkhomedir.so skel=/etc/skel umask=077
```

*Optional:* Modify ``/etc/skel`` folder in order to add command aliases, welcome messages, user configurations, etc.



