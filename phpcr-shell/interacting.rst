Interacting
===========

The PHPCRSH is designed to be a hybrid of a filesystem and an RDBMS shell. You can
both navigate the content hierarchy and execute queries.

.. note::

    PHPCRSH supports "aliases". In this chapter we will use aliases rather than the full
    version of the commands, for example "ls" instead of "node:list", "rm" instead of "node:remove" etc.
    See the chapter on :ref:`phpcrsh_configuration_aliases` for more information.

This chapter aims to highlight some but not all of the features of the shell. For a full
list of features use the ``list`` command.

The current path 
----------------

You can navigate the content hierarchy using the :ref:`phpcr_shell_command_shellpathchange` (or `cd` for short). The
`pwd` command is the alias for :ref:`phpcr_shell_command_shellpathshow` and displays the current working path:

.. code-block:: bash

    PHPCRSH > pwd
    /
    PHPCRSH > cd cms
    PHPCRSH > pwd
    /cms

Listing node contents
---------------------

You can list the contents of a node with the :ref:`phpcr_shell_command_nodelist` command (or `ls`):

.. code-block:: bash

    PHPCRSH > ls
    +-----------------+-----------------+-----------------+
    | cms/            | nt:unstructured |                 |
    | jcr:primaryType | NAME            | nt:unstructured |
    +-----------------+-----------------+-----------------+

Which also accepts a target:

.. code-block:: bash

    PHPCRSH > ls cms
    +--------------------+-----------------+-----------------------------------+
    | pages/             | nt:unstructured |                                   |
    | posts/             | nt:unstructured |                                   |
    | routes/            | nt:unstructured |                                   |
    | jcr:primaryType    | NAME            | nt:unstructured                   |
    | jcr:mixinTypes     | NAME            | [0] phpcr:managed                 |
    | phpcr:class        | STRING          | Acme\BasicCmsBundle\Document\Site |
    | phpcr:classparents | STRING          |                                   |
    +--------------------+-----------------+-----------------------------------+

And a depth:

.. code-block:: bash

    PHPCRSH > ls -L2
    +--------------------------------------------------------------------------+-----------------+-----------------------------------+
    | cms/                                                                     | nt:unstructured |                                   |
    |   pages/                                                                 | nt:unstructured |                                   |
    |   | main/                                                                | nt:unstructured |                                   |
    |   | jcr:primaryType                                                      | NAME            | nt:unstructured                   |
    |   posts/                                                                 | nt:unstructured |                                   |
    |   | Consequatur quisquam recusandae asperiores accusamus nihil repellat. | nt:unstructured |                                   |
    |   | Velit soluta explicabo eligendi occaecati debitis et saepe eum.      | nt:unstructured |                                   |
    |   | jcr:primaryType                                                      | NAME            | nt:unstructured                   |
    |   routes/                                                                | nt:unstructured |                                   |
    |     page/                                                                | nt:unstructured |                                   |
    |     post/                                                                | nt:unstructured |                                   |
    |     jcr:primaryType                                                      | NAME            | nt:unstructured                   |
    |   jcr:primaryType                                                        | NAME            | nt:unstructured                   |
    |   jcr:mixinTypes                                                         | NAME            | [0] phpcr:managed                 |
    |   phpcr:class                                                            | STRING          | Acme\BasicCmsBundle\Document\Site |
    |   phpcr:classparents                                                     | STRING          |                                   |
    | jcr:primaryType                                                          | NAME            | nt:unstructured                   |
    +--------------------------------------------------------------------------+-----------------+-----------------------------------+

In addition to listing the actual node content, you can also show the
node properties and children which are defined in the schema with the ``-t`` option
(**t** for template). The second of the following two examples illustrates this option:

.. code-block:: bash

    PHPCRSH> ls
    +--------------------+-------------------------+------------------------------------------------+
    | home               | slinpTest:article       | Home                                           |
    | jcr:primaryType    | NAME                    | slinpTest:article                              |
    | title              | STRING                  | Slinp Web Content Framework                    |
    +--------------------+-------------------------+------------------------------------------------+
    PHPCRSH> ls -T
    +--------------------+-------------------------+------------------------------------------------+
    | home               | slinpTest:article       | Home                                           |
    | @*                 | nt:base                 |                                                |
    | jcr:primaryType    | NAME                    | slinpTest:article                              |
    | title              | STRING                  | Slinp Web Content Framework                    |
    | @tags              | STRING                  |                                                |
    +--------------------+-------------------------+------------------------------------------------+

In the above examples you see first the "current" contents of the node, in the second we use the
``-t`` option to list "template" items, i.e. items which are defined in the node schema but which
are as yet unrealized. Template items are indicated with the ``@`` symbol. The ``*`` indicates zero or
many.

Editing nodes
-------------

You can edit nodes simply by using your system's default editor (as defined by the ``$EDITOR`` environment
variable).


.. code-block:: bash

    PHPCRSH> node:edit cms

The above will open an editor, e.g. VIM, with a YAML file similar to the following:

.. code-block:: yaml

    'jcr:primaryType':
        type: Name
        value: 'slinpTest:article'
    title:
        type: String
        value: Home
    tags:
        type: String
        value: [automobiles, trains, planes]

You can edit the node properties, then save and quit the editor, the node will then be
updated in the session.

Saving and refreshing the session
---------------------------------

Changes made to nodes in the session are not persisted immediately (with the exception
of :ref:`phpcr_shell_command_nodecopy` which is a workspace command).

To persist changes to the repository you must call :ref:`phpcr_shell_command_sessionsave` (or ``save``).

You can also refresh (or reset) the session by calling :ref:`phpcr_shell_command_sessionrefresh` (or ``refresh``).

Queries
-------

PHPCRSH supports the JCR-SQL2 query language:

.. code-block:: bash

    PHPCRSH > SELECT title FROM slinpTest:article
    +--------------------------------------------+-----------------------------+
    | Path                                       | slinpTest:article.title     |
    +--------------------------------------------+-----------------------------+
    | /slinp/web/root                            | Slinp Web Content Framework |
    | /slinp/web/root/home                       | Home                        |
    | /slinp/web/root/articles/Faster-than-light | Faster than light           |
    +--------------------------------------------+-----------------------------+
    3 rows in set (0.01 sec)
    PHPCRSH > SELECT title FROM slinpTest:article WHERE title="Home"
    +----------------------+-------------------------+
    | Path                 | slinpTest:article.title |
    +----------------------+-------------------------+
    | /slinp/web/root/home | Home                    |
    +----------------------+-------------------------+
    1 rows in set (0.04 sec)

For more information on JCR-SQL2 refer to the articles on the 
`official PHPCR website <http://phpcr.github.io/documentation/>`_.

In addition to SELECT PHPCR Shell supports non-standard UPDATE and DELETE queries:

.. code-block:: bash

    PHPCRSH > DELETE FROM [slinpTest:article] WHERE title="Home"
    1 row(s) affected in 0.01s

.. code-block:: bash

    PHPCRSH > UPDATE [slinpTest:article] SET title="Away" WHERE title="Home"
    1 row(s) affected in 0.01s
