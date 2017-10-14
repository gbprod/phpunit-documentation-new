

.. _incomplete-and-skipped-tests:

===============================
7. Incomplete and Skipped Tests
===============================

.. _incomplete-and-skipped-tests.incomplete-tests:

Incomplete Tests
################

When you are working on a new test case class, you might want to begin
by writing empty test methods such as:
::

    public function testSomething()
    {
    }
to keep track of the tests that you have to write. The
problem with empty test methods is that they are interpreted as a
success by the PHPUnit framework. This misinterpretation leads to the
test reports being useless -- you cannot see whether a test is actually
successful or just not yet implemented. Calling
``$this->fail()`` in the unimplemented test method
does not help either, since then the test will be interpreted as a
failure. This would be just as wrong as interpreting an unimplemented
test as a success.

If we think of a successful test as a green light and a test failure
as a red light, we need an additional yellow light to mark a test
as being incomplete or not yet implemented.
``PHPUnit_Framework_IncompleteTest`` is a marker
interface for marking an exception that is raised by a test method as
the result of the test being incomplete or currently not implemented.
``PHPUnit_Framework_IncompleteTestError`` is the
standard implementation of this interface.

:ref:`incomplete-and-skipped-tests.incomplete-tests.examples.SampleTest.php`
shows a test case class, ``SampleTest``, that contains one test
method, ``testSomething()``. By calling the convenience
method ``markTestIncomplete()`` (which automatically
raises an ``PHPUnit_Framework_IncompleteTestError``
exception) in the test method, we mark the test as being incomplete.

**Example 7.1 Marking a test as incomplete**

.. code-block:: php
    :name: incomplete-and-skipped-tests.incomplete-tests.examples.SampleTest.php

    <?php
    use PHPUnit\Framework\TestCase;

    class SampleTest extends TestCase
    {
        public function testSomething()
        {
            // Optional: Test anything here, if you want.
            $this->assertTrue(true, 'This should already work.');

            // Stop here and mark this test as incomplete.
            $this->markTestIncomplete(
              'This test has not been implemented yet.'
            );
        }
    }
    ?>
.. code-block:: bash
    :name: incomplete-and-skipped-tests.incomplete-tests.examples.SampleTest.php-bash

An incomplete test is denoted by an ``I`` in the output
of the PHPUnit command-line test runner, as shown in the following
example:

::

    $  phpunit --verbose SampleTest
    PHPUnit 6.4.0 by Sebastian Bergmann and contributors.

    I

    Time: 0 seconds, Memory: 3.95Mb

    There was 1 incomplete test:

    1) SampleTest::testSomething
    This test has not been implemented yet.

    /home/sb/SampleTest.php:12
    OK, but incomplete or skipped tests!
    Tests: 1, Assertions: 1, Incomplete: 1.

:ref:`incomplete-and-skipped-tests.incomplete-tests.tables.api`
shows the API for marking tests as incomplete.

.. _incomplete-and-skipped-tests.incomplete-tests.tables.api:

API for Incomplete Tests
========================

Method
Meaning

``void markTestIncomplete()``
Marks the current test as incomplete.

``void markTestIncomplete(string $message)``
Marks the current test as incomplete using ``$message`` as an explanatory message.

.. _incomplete-and-skipped-tests.skipping-tests:

Skipping Tests
##############

Not all tests can be run in every environment. Consider, for instance,
a database abstraction layer that has several drivers for the different
database systems it supports. The tests for the MySQL driver can of
course only be run if a MySQL server is available.

:ref:`incomplete-and-skipped-tests.skipping-tests.examples.DatabaseTest.php`
shows a test case class, ``DatabaseTest``, that contains one test
method, ``testConnection()``. In the test case class'
``setUp()`` template method we check whether the MySQLi
extension is available and use the ``markTestSkipped()``
method to skip the test if it is not.

**Example 7.2 Skipping a test**

.. code-block:: php
    :name: incomplete-and-skipped-tests.skipping-tests.examples.DatabaseTest.php

    <?php
    use PHPUnit\Framework\TestCase;

    class DatabaseTest extends TestCase
    {
        protected function setUp()
        {
            if (!extension_loaded('mysqli')) {
                $this->markTestSkipped(
                  'The MySQLi extension is not available.'
                );
            }
        }

        public function testConnection()
        {
            // ...
        }
    }
    ?>
.. code-block:: bash
    :name: incomplete-and-skipped-tests.skipping-tests.examples.DatabaseTest.php-bash

A test that has been skipped is denoted by an ``S`` in
the output of the PHPUnit command-line test runner, as shown in the
following example:

::

    $  phpunit --verbose DatabaseTest
    PHPUnit 6.4.0 by Sebastian Bergmann and contributors.

    S

    Time: 0 seconds, Memory: 3.95Mb

    There was 1 skipped test:

    1) DatabaseTest::testConnection
    The MySQLi extension is not available.

    /home/sb/DatabaseTest.php:9
    OK, but incomplete or skipped tests!
    Tests: 1, Assertions: 0, Skipped: 1.

:ref:`incomplete-and-skipped-tests.skipped-tests.tables.api`
shows the API for skipping tests.

.. _incomplete-and-skipped-tests.skipped-tests.tables.api:

API for Skipping Tests
======================

Method
Meaning

``void markTestSkipped()``
Marks the current test as skipped.

``void markTestSkipped(string $message)``
Marks the current test as skipped using ``$message`` as an explanatory message.

.. _incomplete-and-skipped-tests.skipping-tests-using-requires:

Skipping Tests using @requires
##############################

In addition to the above methods it is also possible to use the
``@requires`` annotation to express common preconditions for a test case.

.. _incomplete-and-skipped-tests.requires.tables.api:

Possible @requires usages
=========================

Type
Possible Values
Examples
Another example

``PHP``
Any PHP version identifier
@requires PHP 5.3.3
@requires PHP 7.1-dev

``PHPUnit``
Any PHPUnit version identifier
@requires PHPUnit 3.6.3
@requires PHPUnit 4.6

``OS``
A regexp matching `PHP_OS <http://php.net/manual/en/reserved.constants.php#constant.php-os>`_
@requires OS Linux
@requires OS WIN32|WINNT

``function``
Any valid parameter to `function_exists <http://php.net/function_exists>`_
@requires function imap_open
@requires function ReflectionMethod::setAccessible

``extension``
Any extension name along with an optional version identifier
@requires extension mysqli
@requires extension redis 2.2.0

**Example 7.3 Skipping test cases using @requires**

.. code-block:: php
    :name: incomplete-and-skipped-tests.skipping-tests.examples.DatabaseClassSkippingTest.php

    <?php
    use PHPUnit\Framework\TestCase;

    /**
     * @requires extension mysqli
     */
    class DatabaseTest extends TestCase
    {
        /**
         * @requires PHP 5.3
         */
        public function testConnection()
        {
            // Test requires the mysqli extension and PHP >= 5.3
        }

        // ... All other tests require the mysqli extension
    }
    ?>
.. code-block:: bash
    :name: incomplete-and-skipped-tests.skipping-tests.examples.DatabaseClassSkippingTest.php-bash

If you are using syntax that doesn't compile with a certain PHP Version look into the xml
configuration for version dependent includes in :ref:`appendixes.configuration.testsuites`


