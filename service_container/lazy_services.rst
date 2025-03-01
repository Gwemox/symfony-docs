.. index::
   single: Dependency Injection; Lazy Services

Lazy Services
=============

.. seealso::

    Another way to inject services lazily is via a :doc:`service subscriber </service_container/service_subscribers_locators>`.

Why Lazy Services?
------------------

In some cases, you may want to inject a service that is a bit heavy to instantiate,
but is not always used inside your object. For example, imagine you have
a ``NewsletterManager`` and you inject a ``mailer`` service into it. Only
a few methods on your ``NewsletterManager`` actually use the ``mailer``,
but even when you don't need it, a ``mailer`` service is always instantiated
in order to construct your ``NewsletterManager``.

Configuring lazy services is one answer to this. With a lazy service, a
"proxy" of the ``mailer`` service is actually injected. It looks and acts
like the ``mailer``, except that the ``mailer`` isn't actually instantiated
until you interact with the proxy in some way.

.. caution::

    Lazy services do not support `final`_ classes.

    In PHP versions prior to 8.0 lazy services do not support parameters with
    default values for built-in PHP classes (e.g. ``PDO``).

Installation
------------

In order to use the lazy service instantiation, you will need to install the
``symfony/proxy-manager-bridge`` package:

.. code-block:: terminal

    $ composer require symfony/proxy-manager-bridge

Configuration
-------------

You can mark the service as ``lazy`` by manipulating its definition:

.. configuration-block::

    .. code-block:: yaml

        # config/services.yaml
        services:
            App\Twig\AppExtension:
                lazy: true

    .. code-block:: xml

        <!-- config/services.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <services>
                <service id="App\Twig\AppExtension" lazy="true"/>
            </services>
        </container>

    .. code-block:: php

        // config/services.php
        namespace Symfony\Component\DependencyInjection\Loader\Configurator;

        use App\Twig\AppExtension;

        return function(ContainerConfigurator $configurator) {
            $services = $configurator->services();

            $services->set(AppExtension::class)->lazy();
        };


Once you inject the service into another service, a virtual `proxy`_ with the
same signature of the class representing the service should be injected. The
same happens when calling ``Container::get()`` directly.

The actual class will be instantiated as soon as you try to interact with the
service (e.g. call one of its methods).

To check if your proxy works you can check the interface of the received object::

    dump(class_implements($service));
    // the output should include "ProxyManager\Proxy\LazyLoadingInterface"

.. note::

    If you don't install the `ProxyManager bridge`_ , the container will skip
    over the ``lazy`` flag and directly instantiate the service as it would
    normally do.

Additional Resources
--------------------

You can read more about how proxies are instantiated, generated and initialized
in the `documentation of ProxyManager`_.

.. _`ProxyManager bridge`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bridge/ProxyManager
.. _`proxy`: https://en.wikipedia.org/wiki/Proxy_pattern
.. _`documentation of ProxyManager`: https://github.com/Ocramius/ProxyManager/blob/master/docs/lazy-loading-value-holder.md
.. _`final`: https://www.php.net/manual/en/language.oop5.final.php
