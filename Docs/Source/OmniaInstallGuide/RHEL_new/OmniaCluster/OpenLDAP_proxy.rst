Setting up OpenLDAP as a proxy server
=======================================

Omnia allows the internal OpenLDAP server to be configured as a proxy, where it utilizes the external LDAP servers as a backend database to store user data and acts as an authentication entity to allow/deny them access to the cluster. OpenLDAP client will be configured through the proxy server which means that there won't be any direct communication between OpenLDAP client and the external LDAP server.

.. note:: If the OpenLDAP server is set up as a proxy, the user database is not replicated onto the server.

Perform the following steps to configure OpenLDAP as a proxy server:

1. Before proceeding with the new configuration, first remove the existing LDAP configurations by removing the ``/usr/local/openldap/etc/openldap/slapd.d/`` folder and then create another directory with the same folder hierarchy using the ``mkdir`` command.  Execute the following commands to perform these operations: ::

		rm -rf /usr/local/openldap/etc/openldap/slapd.d/
		mkdir /usr/local/openldap/etc/openldap/slapd.d/

2. Now, locate the ``slapd.conf`` config file present in ``/usr/local/openldap/etc/openldap/`` and modify the file to add the new LDAP configurations. Add the following lines to the config file based on the operating system running on the cluster:

    For RHEL: ::

        include        /usr/local/openldap/etc/openldap/schema/core.schema
        include        /usr/local/openldap/etc/openldap/schema/cosine.schema
        include        /usr/local/openldap/etc/openldap/schema/nis.schema
        include        /usr/local/openldap/etc/openldap/schema/inetorgperson.schema


        pidfile         /usr/local/openldap/var/run/slapd.pid
        argsfile        /usr/local/openldap/var/run/slapd.args

        # Load dynamic backend modules:
        modulepath      /usr/local/openldap/libexec/openldap
        moduleload      back_ldap.la
        moduleload      back_meta.la

        #######################################################################
        # Meta database definitions
        #######################################################################
        database        meta
        suffix          "dc=phantom,dc=test"
        rootdn          cn=admin,dc=phantom,dc=test
        rootpw          Dell1234

        uri             "ldap://10.5.0.104:389/dc=phantom,dc=test"
        suffixmassage   "dc=phantom,dc=test" "dc=perf,dc=test"
        idassert-bind
         bindmethod=simple
         binddn="cn=admin,dc=perf,dc=test"
         credentials="Dell1234"
         flags=override
         mode=none
        TLSCACertificateFile    /etc/openldap/certs/ldapserver.crt
        TLSCertificateFile      /etc/openldap/certs/ldapserver.crt
        TLSCertificateKeyFile   /etc/pki/tls/certs/ldapserver.key

Change the **<parameter>** values in the config file, as described below:

* **database**: Database used in the ``slapd.conf`` file, that captures the details of the external LDAP server to be used. For example, ``meta``.
* **suffix**: Captures the domain name of internal OpenLDAP user, to refine the user search while attempting to authenticate the user. For example, ``"dc=omnia,dc=test"``.
* **rootdn**: Admin or root username of the internal OpenLDAP server set up by Omnia. For example, ``cn=admin,dc=omnia,dc=test``.
* **rootpw**: Admin password for the internal OpenLDAP server. For example, ``Dell1234``.

* **uri**: Captures the IP of the external LDAP server along with the port and the domain of the user in ``"ldap://<IP  of external LDAP server>:<Port number>/<suffix>"`` format. For example, ``"ldap://10.5.0.104:389/dc=omnia,dc=test"``.
* **suffixmassage**: ``suffixmassage`` allows you to dynamically move the LDAP client information from the existing internal OpenLDAP server to the external LDAP server that you want to configure as a proxy. This is provided in the ``suffixmassage <suffix1> <suffix2>`` format.

        * ``<suffix1>`` is the internal OpenLDAP server suffix (base DN).
        * ``<suffix2>`` is the external LDAP server suffix (base DN).

* **binddn**: Admin username and domain of the external LDAP server.
* **credentials**: Admin password for the external LDAP server.

* **TLSCACertificateFile**: Omnia, by default, creates the TLSA certificate in ``/etc/openldap/certs/ldapserver.crt``.
* **TLSCertificateFile**: Omnia, by default, creates the TLS certificate in ``/etc/openldap/certs/ldapserver.crt``.
* **TLSCertificateKeyFile**: Omnia, by default, creates the certificate key file in ``/etc/pki/tls/certs/ldapserver.key``.

.. note::
   * The values for ``suffix`` and ``rootdn`` parameters in the ``slapd.conf`` file must be the same as those provided in the ``input/security_config.yml`` file.

   * If you have your own set of TLS certificates and keys that you want to utilize instead of the default ones created by Omnia, then you can provide the path to them in the ``input/security_config.yml`` file. During ``omnia.yml`` execution, the user provided certificates and key files are copied from the OIM to the ``auth_server`` (OpenLDAP). An example for the certificate and key entries in the ``input/security_config.yml`` file for the proxy OpenLDAP server is provided below: ::

           # Certificate Authority(CA) issued certificate file path
           tls_ca_certificate: "/root/certificates/omnia_ca_cert.crt"
           # OpenLDAP Certificate file path
           tls_certificate: "/root/certificates/omnia_cert.pem"
           # OpenLDAP Certificate key file path
           tls_certificate_key: "/root/certificates/omnia_cert_key.key"

    Use the same certificates and keys in the ``slapd.conf`` file, as shown below:

      RHEL: ::

         TLSCACertificateFile    /etc/pki/tls/certs/omnia_ca_cert.crt
         TLSCertificateFile      /etc/pki/tls/certs/omnia_cert.pem
         TLSCertificateKeyFile   /etc/pki/tls/certs/omnia_cert_key.key

   * Multiple external LDAP servers can also be configured on the proxy server. The OpenLDAP proxy server allows users from multiple external LDAP servers to authenticate onto the cluster. You can provide two sets of external LDAP server details as shown below: ::

            uri "ldap://10.5.0.104:389/dc=omnia1,dc=test"
            idassert-bind
             bindmethod=simple
             binddn="cn=admin,dc=omnia,dc=test"
             credentials="Dell1234"
             flags=override
             mode=none

            uri "ldap://10.5.0.105:389/dc=omnia2,dc=test"
            idassert-bind
             bindmethod=simple
             binddn="cn=admin,dc=omnia,dc=test"
             credentials="Dell12345"
             flags=override
             mode=none

3. Once the new configurations are present in the ``slapd.conf`` file, execute the following OpenLDAP server "slaptest" command to apply the configurations: ::

    slaptest -f /usr/local/openldap/etc/openldap/slapd.conf -F /usr/local/openldap/etc/openldap/slapd.d


4. Change the schema ownership to LDAP and set the necessary file permissions (770). Execute the following commands to do so: ::

    chown -R ldap:ldap /usr/local/openldap/etc/openldap/slapd.d/
    chown root:ldap /usr/local/openldap/etc/openldap/slapd.d/
    chmod -R 754 /usr/local/openldap/etc/openldap/slapd.d/
    chmod 770 /usr/local/openldap/etc/openldap/slapd.d/

5. Restart the internal OpenLDAP server to seal in the configurations. Execute the following command to restart the server: ::

    systemctl restart slapd-ltb.service


Once these configurations are applied on the internal OpenLDAP server, it sets up the external LDAP server as an authentication server. The internal OpenLDAP server doesn't store any kind of user data and no users can be created/modified from here.