.. index::
   single: Doctrine; ORM Configuration Reference
   single: Configuration Reference; Doctrine ORM

Référence de configuration
==========================

.. configuration-block::

    .. code-block:: yaml

        doctrine:
            dbal:
                default_connection:   default
                connections:
                    default:
                        dbname:               database
                        host:                 localhost
                        port:                 1234
                        user:                 utilisateur
                        password:             secret
                        driver:               pdo_mysql
                        driver_class:         MonEspaceDeNom\MonDriverBDD
                        options:
                            foo: bar
                        path:                 %kernel.data_dir%/data.sqlite
                        memory:               true
                        unix_socket:          /tmp/mysql.sock
                        wrapper_class:        MonEncapsulationDoctrineDbalConnection
                        charset:              UTF-8
                        logging:              %kernel.debug%
                        platform_service:     MaPlateformeDeServiceBDD
                    conn1:
                        # ...
                types:
                    custom: Sensio\HelloBundle\MonTypePersonnalise
            orm:
                auto_generate_proxy_classes:    true
                proxy_namespace:                Proxies
                proxy_dir:                      %kernel.cache_dir%/doctrine/orm/Proxies
                default_entity_manager:         default # Requis
                entity_managers:
                    # En définir au moins un
                    default:
                        # Le nom de la connexion DBAL (celle nommée default est utilisé lorsqu'aucune n'est définie)
                        connection:                     conn1
                        mappings: # Required
                            HelloBundle: ~
                        class_metadata_factory_name:    Doctrine\ORM\Mapping\ClassMetadataFactory
                        # Les systèmes de caches doivent être array, apc, xcache ou memcache
                        metadata_cache_driver:          array
                        query_cache_driver:             array
                        result_cache_driver:
                            type:           memcache
                            host:           localhost
                            port:           11211
                            instance_class: Memcache
                            class:          Doctrine\Common\Cache\MemcacheCache
                    em2:
                        # ...

    .. code-block:: xml

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

            <doctrine:config>
                <doctrine:dbal default-connection="default">
                    <doctrine:connection
                        name="default"
                        dbname="database"
                        host="localhost"
                        port="1234"
                        user="utilisateur"
                        password="secret"
                        driver="pdo_mysql"
                        driver-class="MonEspaceDeNom\MonDriverBDD"
                        path="%kernel.data_dir%/data.sqlite"
                        memory="true"
                        unix-socket="/tmp/mysql.sock"
                        wrapper-class="MonEncapsulationDoctrineDbalConnection"
                        charset="UTF-8"
                        logging="%kernel.debug%"
                        platform-service="MaPlateformeDeServiceBDD"
                    />
                    <doctrine:connection name="conn1" />
                    <doctrine:type name="custom" class="Sensio\HelloBundle\MonTypePersonnalise" />
                </doctrine:dbal>

                <doctrine:orm default-entity-manager="default" auto-generate-proxy-classes="true" proxy-namespace="Proxies" proxy-dir="%kernel.cache_dir%/doctrine/orm/Proxies">
                    <doctrine:entity-manager name="default" query-cache-driver="array" result-cache-driver="array" connection="conn1" class-metadata-factory-name="Doctrine\ORM\Mapping\ClassMetadataFactory">
                        <doctrine:metadata-cache-driver type="memcache" host="localhost" port="11211" instance-class="Memcache" class="Doctrine\Common\Cache\MemcacheCache" />
                        <doctrine:mapping name="HelloBundle" />
                    </doctrine:entity-manager>
                    <doctrine:entity-manager name="em2" connection="conn2" metadata-cache-driver="apc">
                        <doctrine:mapping
                            name="DoctrineExtensions"
                            type="xml"
                            dir="%kernel.dir%/../src/vendor/DoctrineExtensions/lib/DoctrineExtensions/Entity"
                            prefix="DoctrineExtensions\Entity"
                            alias="DExt"
                        />
                    </doctrine:entity-manager>
                </doctrine:orm>
            </doctrine:config>
        </container>
