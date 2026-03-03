Asset Preloading and Resource Hints with HTTP/2 and WebLink
===========================================================

Symfony provides native support (via the `WebLink`_ component)
for managing ``Link`` HTTP headers, which are the key to improve the application
performance when using HTTP/2 and preloading capabilities of modern web browsers.

``Link`` headers are used in `HTTP/2 Server Push`_ and W3C's `Resource Hints`_
to push resources (e.g. CSS and JavaScript files) to clients before they even
know that they need them. WebLink also enables other optimizations that work
with HTTP 1.x:

* Asking the browser to fetch or to render another web page in the background;
* Making early DNS lookups, TCP handshakes or TLS negotiations.

Something important to consider is that all these HTTP/2 features require a
secure HTTPS connection, even when working on your local machine. The main web
servers (Apache, nginx, Caddy, etc.) support this, but you can also use the
`Docker installer and runtime for Symfony`_ created by Kévin Dunglas, from the
Symfony community.

Installation
------------

In applications using :ref:`Symfony Flex <symfony-flex>`, run the following command
to install the WebLink feature before using it:

.. code-block:: terminal

    $ composer require symfony/web-link

Preloading Assets
-----------------

Imagine that your application includes a web page like this:

.. code-block:: html

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>My Application</title>
        <link rel="stylesheet" href="/app.css">
    </head>
    <body>
        <main role="main" class="container">
            <!-- ... -->
        </main>
    </body>
    </html>

In a traditional HTTP workflow, when this page is loaded, browsers make one
request for the HTML document and another for the linked CSS file. With
preloading, you can hint the browser to start fetching critical resources (like
stylesheets, fonts or scripts) earlier, before it discovers them in the HTML.

This is useful for resources that are not directly linked in the HTML but are
needed early (e.g. a font file referenced inside a CSS stylesheet).

To preload a resource, use the ``preload()`` Twig function provided by WebLink.
The `"as" attribute`_ is required, as browsers use it to prioritize resources
correctly and comply with the content security policy:

.. code-block:: html+twig

    <head>
        <!-- ... -->
        <link rel="preload" href="{{ preload('/fonts/myfont.woff2', {as: 'font'}) }}">
        <!-- you can add optionally add more attributes to the preload link -->
        <!-- <link rel="preload" href="{{ preload('/fonts/myfont.woff2', {as: 'font', type: 'font/woff2', crossorigin: 'anonymous'}) }}"> -->

        <link rel="stylesheet" href="/app.css">
    </head>

The ``preload()`` function adds a ``Link`` HTTP header to the response (e.g.
``Link: </fonts/myfont.woff2>; rel="preload"; as="font"``). This tells the
browser (or an HTTP/2 compatible server or CDN) to start fetching the resource
as early as possible. You can also combine it with the ``asset()`` function:

.. code-block:: html+twig

    <link rel="preload" href="{{ preload(asset('build/app.css'), {as: 'style'}) }}" as="style">
    <link rel="stylesheet" href="{{ asset('build/app.css') }}">

.. tip::

    When using the :doc:`AssetMapper component </frontend/asset_mapper>` (e.g.
    ``importmap('app')``), there's no need to add the ``<link rel="preload">``
    tag. The ``importmap()`` Twig function automatically adds the ``Link`` HTTP
    header for you when the WebLink component is available.

Additionally, according to `the Priority Hints specification`_, you can signal
the priority of the resource to download using the ``importance`` attribute:

.. code-block:: html+twig

    <head>
        <!-- ... -->
        <link rel="preload" href="{{ preload('/app.css', {as: 'style', importance: 'low'}) }}" as="style">
        <!-- ... -->
    </head>

How does it work?
~~~~~~~~~~~~~~~~~

The WebLink component manages the ``Link`` HTTP headers added to the response.
When using the ``preload()`` function, a header like this is added to the
response: ``Link </fonts/myfont.woff2>; rel="preload"; as="font"``

This header is interpreted in two ways depending on the infrastructure:

* **Browser preloading**: Modern browsers read ``Link`` headers and start
  fetching the resources as early as possible, before parsing the full HTML.
  This works with both HTTP/1.x and HTTP/2.
* **HTTP/2 Server Push**: According to `the Preload specification`_, some
  HTTP/2 servers detect this header and push the resource to the client
  in the same connection, before the browser even requests it. Popular proxy
  services and CDNs including `Cloudflare`_, `Fastly`_ and `Akamai`_ also
  support this.

If you want the browser to preload the resource via an early request but want
to prevent the server from pushing it, use the ``nopush`` option:

.. code-block:: html+twig

    <head>
        <!-- ... -->
        <link rel="preload" href="{{ preload('/app.css', {as: 'style', nopush: true}) }}" as="style">
        <!-- ... -->
    </head>

Resource Hints
--------------

`Resource Hints`_ are used by applications to help browsers when deciding which
resources should be downloaded, preprocessed or connected to first.

The WebLink component provides the following Twig functions to send those hints:

* ``dns_prefetch()``: "indicates an origin (e.g. ``https://foo.cloudfront.net``)
  that will be used to fetch required resources, and that the user agent should
  resolve as early as possible".
* ``preconnect()``: "indicates an origin (e.g. ``https://www.google-analytics.com``)
  that will be used to fetch required resources. Initiating an early connection,
  which includes the DNS lookup, TCP handshake, and optional TLS negotiation, allows
  the user agent to mask the high latency costs of establishing a connection".
* ``prefetch()``: "identifies a resource that might be required by the next
  navigation, and that the user agent *should* fetch, such that the user agent
  can deliver a faster response once the resource is requested in the future".
* ``prerender()``: " **deprecated** and superseded by the `Speculation Rules API`_,
  identifies a resource that might be required by the next
  navigation, and that the user agent *should* fetch and execute, such that the
  user agent can deliver a faster response once the resource is requested later".

The component also supports sending HTTP links not related to performance and
any link implementing the `PSR-13`_ standard. For instance, any
`link defined in the HTML specification`_:

.. code-block:: html+twig

    <head>
        <!-- ... -->
        <link rel="alternate" href="{{ link('/index.jsonld', 'alternate') }}">
        <link rel="preload" href="{{ preload('/app.css', {as: 'style', nopush: true}) }}" as="style">
        <!-- ... -->
    </head>

The previous snippet will result in this HTTP header being sent to the client:
``Link: </index.jsonld>; rel="alternate",</app.css>; rel="preload"; nopush``

You can also add links to the HTTP response directly from controllers and services::

    // src/Controller/BlogController.php
    namespace App\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\WebLink\GenericLinkProvider;
    use Symfony\Component\WebLink\Link;

    class BlogController extends AbstractController
    {
        public function index(Request $request): Response
        {
            // using the addLink() shortcut provided by AbstractController
            $this->addLink($request, (new Link('preload', '/app.css'))->withAttribute('as', 'style'));

            // alternative if you don't want to use the addLink() shortcut
            $linkProvider = $request->attributes->get('_links', new GenericLinkProvider());
            $request->attributes->set('_links', $linkProvider->withLink(
                (new Link('preload', '/app.css'))->withAttribute('as', 'style')
            ));

            return $this->render('...');
        }
    }

.. tip::

    The possible values of link relations (``'preload'``, ``'preconnect'``, etc.)
    are also defined as constants in the :class:`Symfony\\Component\\WebLink\\Link`
    class (e.g. ``Link::REL_PRELOAD``, ``Link::REL_PRECONNECT``, etc.).

Parsing Link Headers
--------------------

Some third-party APIs provide resources such as pagination URLs using the
``Link`` HTTP header. The WebLink component provides the
:class:`Symfony\\Component\\WebLink\\HttpHeaderParser` utility class to parse
those headers and transform them into :class:`Symfony\\Component\\WebLink\\Link`
instances::

    use Symfony\Component\WebLink\HttpHeaderParser;

    $parser = new HttpHeaderParser();
    // get the value of the Link header from the Request
    $linkHeader = '</foo.css>; rel="prerender",</bar.otf>; rel="dns-prefetch"; pr="0.7",</baz.js>; rel="preload"; as="script"';

    $links = $parser->parse($linkHeader)->getLinks();
    $links[0]->getRels();       // ['prerender']
    $links[1]->getAttributes(); // ['pr' => '0.7']
    $links[2]->getHref();       // '/baz.js'

.. versionadded:: 7.4

    The ``HttpHeaderParser`` class was introduced in Symfony 7.4.

.. _`WebLink`: https://github.com/symfony/web-link
.. _`HTTP/2 Server Push`: https://tools.ietf.org/html/rfc7540#section-8.2
.. _`Resource Hints`: https://www.w3.org/TR/resource-hints/
.. _`Docker installer and runtime for Symfony`: https://github.com/dunglas/symfony-docker
.. _`"as" attribute`: https://w3c.github.io/preload/#as-attribute
.. _`the Priority Hints specification`: https://wicg.github.io/priority-hints/
.. _`the Preload specification`: https://www.w3.org/TR/preload/#server-push-http-2
.. _`Cloudflare`: https://blog.cloudflare.com/announcing-support-for-http-2-server-push-2/
.. _`Fastly`: https://docs.fastly.com/en/guides/http2-server-push
.. _`Akamai`: https://http2.akamai.com/
.. _`link defined in the HTML specification`: https://html.spec.whatwg.org/dev/links.html#linkTypes
.. _`PSR-13`: https://www.php-fig.org/psr/psr-13/
.. _`Speculation Rules API`: https://developer.mozilla.org/docs/Web/API/Speculation_Rules_API
