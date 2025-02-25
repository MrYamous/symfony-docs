Symfony Core Team
=================

The **Symfony Core** team is the group of developers that determine the
direction and evolution of the Symfony project. Their votes rule if the
features and patches proposed by the community are approved or rejected.

All the Symfony Core members are long-time contributors with solid technical
expertise and they have demonstrated a strong commitment to drive the project
forward.

This document states the rules that govern the Symfony core team. These rules
are effective upon publication of this document and all Symfony Core members
must adhere to said rules and protocol.

Core Team Member Role
---------------------

In addition to being a regular contributor, core team members are expected to:

* Review, approve, and merge pull requests;
* Help enforce, improve, and implement Symfony :doc:`processes and policies </contributing/index>`;
* Participate in the Symfony Core Team discussions (on Slack and GitHub).

Core Team Member Responsibilities
---------------------------------

Core Team members are unpaid volunteers and as such, they are not expected to
dedicate any specific amount of time on Symfony. They are expected to help the
project in any way they can, from reviewing pull requests, writing documentation
to participating in discussions and helping the community in general, but their
involvement is completely voluntary and can be as much or as little as they
want.

Core Organization
-----------------

Symfony Core members are divided into groups. Each member can only belong to one
group at a time. The privileges granted to a group are automatically granted to
all higher priority groups.

The Symfony Core groups, in descending order of priority, are as follows:

1. **Project Leader**

   * Elects members in any other group;
   * Merges pull requests in all Symfony repositories.

2. **Mergers Team**

   * Merge pull requests on the main Symfony repository.

In addition, there are other groups created to manage specific topics:

* **Security Team**: manages the whole security process (triaging reported vulnerabilities,
  fixing the reported issues, coordinating the release of security fixes, etc.);
* **Symfony UX Team**: manages the `UX repositories`_;
* **Documentation Team**: manages the whole `symfony-docs repository`_.

Active Core Members
~~~~~~~~~~~~~~~~~~~

* **Project Leader**:

  * **Fabien Potencier** (`fabpot`_).

* **Mergers Team** (``@symfony/mergers`` on GitHub):

  * **Nicolas Grekas** (`nicolas-grekas`_);
  * **Christophe Coevoet** (`stof`_);
  * **Christian Flothmann** (`xabbuh`_);
  * **Kévin Dunglas** (`dunglas`_);
  * **Javier Eguiluz** (`javiereguiluz`_);
  * **Grégoire Pineau** (`lyrixx`_);
  * **Ryan Weaver** (`weaverryan`_);
  * **Robin Chalas** (`chalasr`_);
  * **Yonel Ceruto** (`yceruto`_);
  * **Tobias Nyholm** (`Nyholm`_);
  * **Wouter De Jong** (`wouterj`_);
  * **Alexander M. Turek** (`derrabus`_);
  * **Jérémy Derussé** (`jderusse`_);
  * **Oskar Stark** (`OskarStark`_);
  * **Mathieu Santostefano** (`welcomattic`_);
  * **Kevin Bond** (`kbond`_);
  * **Jérôme Tamarelle** (`gromnan`_);
  * **Berislav Balogović** (`hypemc`_);
  * **Mathias Arlaud** (`mtarld`_);
  * **Florent Morselli** (`spomky`_);
  * **Alexandre Daubois** (`alexandre-daubois`_).

* **Security Team** (``@symfony/security`` on GitHub):

  * **Fabien Potencier** (`fabpot`_);
  * **Jérémy Derussé** (`jderusse`_).

* **Symfony UX Team** (``@symfony/ux`` on GitHub):

  * **Ryan Weaver** (`weaverryan`_);
  * **Kevin Bond** (`kbond`_);
  * **Simon André** (`smnandre`_);
  * **Hugo Alliaume** (`kocal`_);
  * **Matheo Daninos** (`webmamba`_).

* **Documentation Team** (``@symfony/team-symfony-docs`` on GitHub):

  * **Fabien Potencier** (`fabpot`_);
  * **Ryan Weaver** (`weaverryan`_);
  * **Christian Flothmann** (`xabbuh`_);
  * **Wouter De Jong** (`wouterj`_);
  * **Javier Eguiluz** (`javiereguiluz`_).
  * **Oskar Stark** (`OskarStark`_).

Former Core Members
~~~~~~~~~~~~~~~~~~~

They are no longer part of the core team, but we are very grateful for all their
Symfony contributions:

* **Bernhard Schussek** (`webmozart`_);
* **Abdellatif AitBoudad** (`aitboudad`_);
* **Romain Neutron** (`romainneutron`_);
* **Jordi Boggiano** (`Seldaek`_);
* **Lukas Kahwe Smith** (`lsmith77`_);
* **Jules Pietri** (`HeahDude`_);
* **Jakub Zalas** (`jakzal`_);
* **Samuel Rozé** (`sroze`_);
* **Tobias Schultze** (`Tobion`_);
* **Maxime Steinhausser** (`ogizanagi`_);
* **Titouan Galopin** (`tgalopin`_);
* **Michael Cullum** (`michaelcullum`_);
* **Thomas Calvet** (`fancyweb`_).

Core Membership Application
~~~~~~~~~~~~~~~~~~~~~~~~~~~

About once a year, the core team discusses the opportunity to invite new members.

Core Membership Revocation
~~~~~~~~~~~~~~~~~~~~~~~~~~

A Symfony Core membership can be revoked for any of the following reasons:

* Refusal to follow the rules and policies stated in this document;
* Lack of activity for the past six months;
* Willful negligence or intent to harm the Symfony project;
* Upon decision of the **Project Leader**.

Code Development Rules
----------------------

Symfony project development is based on pull requests proposed by any member
of the Symfony community. Pull request acceptance or rejection is decided based
on the votes cast by the Symfony Core members.

Pull Request Voting Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``-1`` votes must always be justified by technical and objective reasons;

* ``+1`` votes do not require justification, unless there is at least one
  ``-1`` vote;

* Core members can change their votes as many times as they desire
  during the course of a pull request discussion;
* Core members are not allowed to vote on their own pull requests.

Pull Request Merging Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~

A pull request **can be merged** if:

* It is a :ref:`minor change <core-team_minor-changes>`;
* Enough time was given for peer reviews;
* It is a bug fix and at least two **Mergers Team** members voted ``+1``
  (only one if the submitter is part of the Mergers team) and no Core
  member voted ``-1`` (via GitHub reviews or as comments).
* It is a new feature and at least two **Mergers Team** members voted
  ``+1`` (if the submitter is part of the Mergers team, two *other* members)
  and no Core member voted ``-1`` (via GitHub reviews or as comments).

Pull Request Merging Process
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All code must be committed to the repository through pull requests, except for
:ref:`minor change <core-team_minor-changes>` which can be committed directly
to the repository.

**Mergers** must always use the command-line ``gh`` tool provided by the
**Project Leader** to merge pull requests.

When merging a pull request, the tool asks for a category that should be chosen
following these rules:

* **Feature**: For new features and deprecations; Pull requests must be merged
  in the development branch.
* **Bug**: Only for bug fixes; We are very conservative when it comes to
  merging older, but still maintained, branches. Read the :doc:`maintenance`
  document for more information.
* **Minor**: For everything that does not change the code or when they don't
  need to be listed in the CHANGELOG files: typos, Markdown files, test files,
  new or missing translations, etc.
* **Security**: It's the category used for security fixes and should never be
  used except by the security team.

Getting the right category is important as it is used by automated tools to
generate the CHANGELOG files when releasing new versions.

Release Policy
~~~~~~~~~~~~~~

The **Project Leader** is also the release manager for every Symfony version.

Symfony Core Rules and Protocol Amendments
------------------------------------------

The rules described in this document may be amended at any time at the
discretion of the **Project Leader**.

.. _core-team_minor-changes:

.. note::

    Minor changes comprise typos, DocBlock fixes, code standards
    violations, and minor CSS, JavaScript and HTML modifications.

.. _`symfony-docs repository`: https://github.com/symfony/symfony-docs
.. _`UX repositories`: https://github.com/symfony/ux
.. _`fabpot`: https://github.com/fabpot/
.. _`webmozart`: https://github.com/webmozart/
.. _`Tobion`: https://github.com/Tobion/
.. _`nicolas-grekas`: https://github.com/nicolas-grekas/
.. _`stof`: https://github.com/stof/
.. _`dunglas`: https://github.com/dunglas/
.. _`jakzal`: https://github.com/jakzal/
.. _`Seldaek`: https://github.com/Seldaek/
.. _`weaverryan`: https://github.com/weaverryan/
.. _`aitboudad`: https://github.com/aitboudad/
.. _`xabbuh`: https://github.com/xabbuh/
.. _`javiereguiluz`: https://github.com/javiereguiluz/
.. _`lyrixx`: https://github.com/lyrixx/
.. _`chalasr`: https://github.com/chalasr/
.. _`ogizanagi`: https://github.com/ogizanagi/
.. _`Nyholm`: https://github.com/Nyholm
.. _`sroze`: https://github.com/sroze
.. _`yceruto`: https://github.com/yceruto
.. _`michaelcullum`: https://github.com/michaelcullum
.. _`wouterj`: https://github.com/wouterj
.. _`HeahDude`: https://github.com/HeahDude
.. _`OskarStark`: https://github.com/OskarStark
.. _`romainneutron`: https://github.com/romainneutron
.. _`lsmith77`: https://github.com/lsmith77/
.. _`derrabus`: https://github.com/derrabus/
.. _`jderusse`: https://github.com/jderusse/
.. _`tgalopin`: https://github.com/tgalopin/
.. _`fancyweb`: https://github.com/fancyweb/
.. _`welcomattic`: https://github.com/welcomattic/
.. _`kbond`: https://github.com/kbond/
.. _`gromnan`: https://github.com/gromnan/
.. _`smnandre`: https://github.com/smnandre/
.. _`kocal`: https://github.com/kocal/
.. _`webmamba`: https://github.com/webmamba/
.. _`hypemc`: https://github.com/hypemc/
.. _`mtarld`: https://github.com/mtarld/
.. _`spomky`: https://github.com/spomky/
.. _`alexandre-daubois`: https://github.com/alexandre-daubois/
