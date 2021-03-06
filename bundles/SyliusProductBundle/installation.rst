Installation
============

We assume you're familiar with `Composer <http://packagist.org>`_, a dependency manager for PHP.

Use following command to add the bundle to your `composer.json` and download package.
If you have `Composer installed globally <http://getcomposer.org/doc/00-intro.md#globally>`_.

.. code-block:: bash

    $ composer require sylius/product-bundle:0.1.*

Otherwise you have to download .phar file.

.. code-block:: bash

    $ curl -sS https://getcomposer.org/installer | php
    $ php composer.phar require sylius/product-bundle:0.1.*

Adding required bundles to the kernel
-------------------------------------

First, you need to enable the bundle inside the kernel.
If you're not using any other Sylius bundles, you will also need to add `SyliusResourceBundle` and its dependencies to kernel.
Don't worry, everything was automatically installed via Composer.

.. code-block:: php

    <?php

    // app/AppKernel.php

    public function registerBundles()
    {
        $bundles = array(
            new FOS\RestBundle\FOSRestBundle(),
            new JMS\SerializerBundle\JMSSerializerBundle($this),
            new Stof\DoctrineExtensionsBundle\StofDoctrineExtensionsBundle(),
            new Sylius\Bundle\ProductBundle\SyliusProductBundle(),
            new Sylius\Bundle\ResourceBundle\SyliusResourceBundle(),
            new WhiteOctober\PagerfantaBundle\WhiteOctoberPagerfantaBundle(),

            // Other bundles...
            new Doctrine\Bundle\DoctrineBundle\DoctrineBundle(),
        );
    }

.. note::

    Please register the bundle before *DoctrineBundle*. This is important as we use listeners which have to be processed first.

Creating your entities
----------------------

You have to create your own **Product** entity, living inside your application code.
We think that **keeping the app-specific bundle structure simple** is a good practice, so
let's assume you have your ``ShopBundle`` registered under ``Acme\ShopBundle`` namespace.

.. code-block:: php

    <?php

    // src/Acme/ShopBundle/Entity/Product.php
    namespace Acme\ShopBundle\Entity;

    use Sylius\Bundle\ProductBundle\Model\Product as BaseProduct;

    class Product extends BaseProduct
    {
    }

Now define the entity mapping inside ``Resources/config/doctrine/Product.orm.xml`` of your bundle.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>

    <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
                      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                      xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                                          http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

        <entity name="Acme\ShopBundle\Entity\Product" table="sylius_product">
            <id name="id" column="id" type="integer">
                <generator strategy="AUTO" />
            </id>
            <one-to-many field="properties" target-entity="Sylius\Bundle\ProductBundle\Model\ProductPropertyInterface" mapped-by="product">
                <cascade>
                    <cascade-all />
                </cascade>
            </one-to-many>
        </entity>

    </doctrine-mapping>

Container configuration
-----------------------

Put this configuration inside your ``app/config/config.yml``.

.. code-block:: yaml

    sylius_product:
        driver: doctrine/orm # Configure the doctrine orm driver used in documentation.
        classes:
            product:
                model: Acme\ShopBundle\Entity\Product # Your product entity.

And configure doctrine extensions which are used in assortment bundle:

.. code-block:: yaml

    stof_doctrine_extensions:
        orm:
            default:
                sluggable: true
                timestampable: true

Routing configuration
---------------------

Add folowing to your ``app/config/routing.yml``.

.. code-block:: yaml

    sylius_product:
        resource: @SyliusProductBundle/Resources/config/routing.yml

Updating database schema
------------------------

Remember to update your database schema.

Run the following command.

.. code-block:: bash

    $ php app/console doctrine:schema:update --force

.. warning::

    This should be done only in **dev** environment! We recommend using Doctrine migrations, to safely update your schema.
