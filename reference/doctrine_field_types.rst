Doctrine Field Types Reference
==============================

In addition to the `Doctrine mapping types`_, Symfony provides a set of
custom Doctrine types in the ``Symfony\Bridge\Doctrine\Types`` namespace.

UID Types
---------

These types allow storing :doc:`UIDs </components/uid>` as Doctrine fields.

``uuid``
~~~~~~~~

Stores a :doc:`UUID </components/uid>` as a native GUID type if available, or
as a 16-byte binary otherwise.

**Class:** :class:`Symfony\\Bridge\\Doctrine\\Types\\UuidType`

Example usage:

.. code-block:: php-attributes

    // src/Entity/Product.php
    namespace App\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Symfony\Bridge\Doctrine\Types\UuidType;
    use Symfony\Component\Uid\Uuid;

    #[ORM\Entity]
    class Product
    {
        #[ORM\Column(type: UuidType::NAME)]
        private Uuid $sku;

        // ...
    }

See :ref:`Storing UUIDs in Databases <uid-uuid-doctrine>` in the UID component
documentation for more details, including how to use UUIDs as primary keys.

``ulid``
~~~~~~~~

Stores a :ref:`ULID <ulid>` as a native GUID type if available, or as a
16-byte binary otherwise.

**Class:** :class:`Symfony\\Bridge\\Doctrine\\Types\\UlidType`

Example usage:

.. code-block:: php-attributes

    // src/Entity/Product.php
    namespace App\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Symfony\Bridge\Doctrine\Types\UlidType;
    use Symfony\Component\Uid\Ulid;

    #[ORM\Entity]
    class Product
    {
        #[ORM\Column(type: UlidType::NAME)]
        private Ulid $identifier;

        // ...
    }

See :ref:`Storing ULIDs in Databases <uid-ulid-doctrine>` in the UID component
documentation for more details, including how to use ULIDs as primary keys.

DatePoint Types
---------------

These types allow storing :class:`Symfony\\Component\\Clock\\DatePoint` objects
from the :doc:`Clock component </components/clock>`. They convert to/from
``DatePoint`` objects automatically.

====================  ==========================  =========================================================
Type                  Extends Doctrine type       Class
====================  ==========================  =========================================================
``date_point``        ``datetime_immutable``      :class:`Symfony\\Bridge\\Doctrine\\Types\\DatePointType`
``day_point``         ``date_immutable``          :class:`Symfony\\Bridge\\Doctrine\\Types\\DayPointType`
``time_point``        ``time_immutable``          :class:`Symfony\\Bridge\\Doctrine\\Types\\TimePointType`
====================  ==========================  =========================================================

Example usage:

.. code-block:: php-attributes

    // src/Entity/Product.php
    namespace App\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Symfony\Component\Clock\DatePoint;

    #[ORM\Entity]
    class Product
    {
        // Symfony autodetects the 'date_point' type when type-hinting with DatePoint
        #[ORM\Column]
        private DatePoint $createdAt;

        // you can also set the type explicitly
        #[ORM\Column(type: 'date_point')]
        private DatePoint $updatedAt;

        #[ORM\Column(type: 'day_point')]
        public DatePoint $releaseDate;

        #[ORM\Column(type: 'time_point')]
        public DatePoint $openingTime;

        // ...
    }

Choosing Between Similar Types
------------------------------

``datetime_immutable`` vs ``date_point``
    Use ``date_point`` when you want to work with
    :class:`Symfony\\Component\\Clock\\DatePoint` objects, which makes your code
    easier to test with the :doc:`Clock component </components/clock>`. Use
    ``datetime_immutable`` if you don't need the Clock component features.

``uuid`` vs ``ulid``
    Both are 128-bit identifiers. UUIDs are more widely used and recognized.
    ULIDs are lexicographically sortable by creation time, which avoids index
    fragmentation in databases. See the :doc:`UID component docs </components/uid>`
    for a detailed comparison.

.. _`Doctrine mapping types`: https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/basic-mapping.html#reference-mapping-types
