.. index::
   single: Security

Security
========

Symfony2 comes with a built-in security layer. It secures your application by
providing authentication and authorization.

*Authentication* ensures that the user is who he claims to be. *Authorization*
refers to the process of deciding whether a user is allowed to perform an
action or not (authentication is always performed before authorization).

This document is a quick overview over these main concepts, but it barely scratches
the surface. If you want to get to know the real power of Symfony2's security 
layer, you should also read these more specific documents: 
:doc:`Users </book/security/users>`,
:doc:`Authentication </book/security/authentication>`, and
:doc:`Authorization </book/security/authorization>`.

.. index::
   pair: Security; Configuration

Configuration
-------------

For most common use cases, the Symfony2 security can be easily configured from
your main configuration file; here is a typical configuration:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            encoders:
                Symfony\Component\Security\Core\User\User: plaintext

            providers:
                main:
                    users:
                        foo: { password: testing, roles: ROLE_USER }

            firewalls:
                main:
                    pattern:    /.*
                    http_basic: true
                    logout:     true

            access_control:
                - { path: /.*, role: ROLE_USER }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <security:config>
            <security:encoder class="Symfony\Component\Security\Core\User\User" algorithm="plaintext" />

            <security:provider>
                <security:user name="foo" password="testing" roles="ROLE_USER" />
            </security:provider>

            <security:firewall pattern="/.*">
                <security:http-basic />
                <security:logout />
            </security:firewall>

            <security:access-control>
                <security:rule path="/.*" role="ROLE_USER" />
            </security:access-control>
        </security:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('security', array(
            'encoders' => array(
                'Symfony\Component\Security\Core\User\User' => 'plaintext',
            ),
            'providers' => array(
                'main' => array(
                    'users' => array(
                        'foo' => array('password' => 'testing', 'roles' => 'ROLE_USER'),
                )),
            ),
            'firewalls' => array(
                'main' => array('pattern' => '/.*', 'http_basic' => true, 'logout' => true),
            ),
            'access_control' => array(
                array('path' => '/.*', 'role' => 'ROLE_USER'),
            ),
        ));

Most of the time, it is more convenient to outsource all security related
configuration into an external file. If you use XML, the external file can use
the security namespace as the default one to make it more readable:

.. code-block:: xml

        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <provider>
                    <password-encoder hash="sha1" />
                    <user name="foo" password="0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33" roles="ROLE_USER" />
                </provider>

                <firewall pattern="/.*">
                    <http-basic />
                    <logout />
                </firewall>

                <access-control>
                    <rule path="/.*" role="ROLE_USER" />
                </access-control>
            </config>
        </srv:container>

.. note::

    All examples in the documentation assume that you are using an external
    file with the default security namespace as above.

As you can see, the configuration has four sections:

* *encoder*: An encoder is used for hashing passwords of users;

* *provider*: A provider knows how to create users;

* *firewall*: A firewall defines the authentication mechanisms for the whole
  application or for just a part of it;

* *access-control*: Access control rules secure parts of your application with
  roles.

To sum up the workflow, the firewall authenticates the client based on the
submitted credentials and the user created by the user provider. Finally, 
access control is used to protect specific resources.

Authentication
--------------

Symfony2 supports many different authentication mechanisms out of the box, and
more can be easily added if needed; main ones are:

* HTTP Basic;
* HTTP Digest;
* Form based authentication;
* X.509 certificates.

Here is how you can secure your application with HTTP basic authentication:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    http-basic: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <http-basic />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('http_basic' => true),
            ),
        ));

Several firewalls can also be defined if you need different authentication
mechanisms for different parts of the application:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                backend:
                    pattern: /admin/.*
                    http_basic: true
                public:
                    pattern:  /.*
                    security: false

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall pattern="/admin/.*">
                <http-basic />
            </firewall>

            <firewall pattern="/.*" security="false" />
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'backend' => array('pattern' => '/admin/.*', 'http-basic' => true),
                'public'  => array('pattern' => '/.*', 'security' => false),
            ),
        ));

.. tip::

    Using HTTP basic is the easiest, but read the :doc:`Authentication
    </book/security/authentication>` document to learn how to configure
    other authentication mechanisms, how to configure a stateless
    authentication, how you can impersonate another user, how you can enforce
    https, and much more.

Users
-----

During authentication, Symfony2 asks a user provider to create the user object
matching the client request (via credentials like a username and a password).
To get started fast, you can define an in-memory provider directly in your
configuration:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            providers:
                main:
                    users:
                        foo: { password: foo }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider>
                <user name="foo" password="foo" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'provider' => array(
                'main' => array('users' => array(
                    'foo' => array('password' => 'foo'),
                )),
            ),
        ));

The above configuration defines a 'foo' user with a 'foo' password. After
authentication, you can access the authenticated user via the security context
(the user is an instance of :class:`Symfony\\Component\\Security\\Core\\User\\User`)::

    $user = $container->get('security.context')->getToken()->getUser();

.. tip::

    Using the in-memory provider is a great way to easily secure your personal
    website backend, to create a prototype, or to provide fixtures for your
    tests. Read the :doc:`Users </book/security/users>` document to learn
    how to avoid the password to be in clear, how to use a Doctrine Entity as
    a user provider, how to define several providers, and much more.

Authorization
-------------

Authorization is optional but gives you a powerful way to restrict access to
your application resources based user roles:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            providers:
                main:
                    users:
                        foo: { password: foo, roles: ['ROLE_USER', 'ROLE_ADMIN'] }
            access_control:
                - { path: /.*, role: ROLE_USER }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider>
                <user name="foo" password="foo" roles="ROLE_USER,ROLE_ADMIN" />
            </provider>

            <access-control>
                <rule path="/.*" role="ROLE_USER" />
            </access-control>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'provider' => array(
                'main' => array('users' => array(
                    'foo' => array('password' => 'foo', 'roles' => array('ROLE_USER', 'ROLE_ADMIN')),
                )),
            ),

            'access_control' => array(
                array('path' => '/.*', 'role' => 'ROLE_USER'),
            ),
        ));

The above configuration defines a 'foo' user with the 'ROLE_USER' and
'ROLE_ADMIN' roles and it restricts access to the whole application to users
having the 'ROLE_USER' role.

.. tip::

    Read the :doc:`Authorization </book/security/authorization>` document to
    learn how to define a role hierarchy, how to customize your template based
    on roles, how to define access control rules based on request attributes,
    and much more.
