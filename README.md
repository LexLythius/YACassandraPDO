# Introduction

Pretty much very experimental PDO driver for Cassandra CQL.

This repository is a fork of: https://code.google.com/a/apache-extras.org/p/cassandra-pdo/
We cloned it on GitHub because the original project seems dead.

This version is developped for CQL3 target, we do not provide support for former versions of CQL.

We need some testers, so if you find BUGS or inconsistencies, feel free to report them.

# What is different from the Datastax version?
 - Added support for float numbers
 - Added support for collections (map, set, list)
 - Fixed a lot of bugs on integer convertions
 - Fixed parameters binding
 - Some other minor fixes

# Branches

- 'master' branch is the driver derived from the datastax version.
- 'retry' branch has been modified according to our needs. The only difference with the master version is that, in case of a connection or request failure we repeat the process untill we reach the maximum retry attempts or the request finishes in success.

# Dependencies

  - PHP and PDO
  - Thrift
  - Boost (shared_ptr)


# How to build?

First install thrift from http://thrift.apache.org/download/. Thrift should depend on
boost shared_ptr so while installing thrift you are installing rest of the dependencies
for pdo_cassandra (apart from PHP and PDO of course).

pdo_cassandra ./configure script takes the following options:

 - --with-pdo-cassandra[=FILE] where file is optional path to Cassandra Thrift definitions file. The file is
    bundled with the package so it is unlikely that alternative file is needed unless testing new versions.

 - --with-thrift-dir[=DIR] can be used to specify 'non-standard' thrift installation prefix.

 - --with-boost-dir[=DIR] can be used to specify 'non-standard' boost installation prefix.

# Create a debian package from the sources

- ln -s packaging/debian debian
- dpkg-buildpackage -rfakeroot -us -uc

# Running tests

After a successful build tests can be executed using an instance of Cassandra. Default config
for Cassandra should be fine.

Before make test:

    $ cp tests/config.inc-dist tests/config.inc
    $ $EDITOR tests/config.inc

This is to prevent accidental keyspaces dropping that are in use.

# DSN

The DSN format is as follows:

  - "cassandra:host=127.0.0.1;port=9160"

Multiple hosts can be specified as follow:

  - "cassandra:host=127.0.0.1;port=9160,host=localhost;port=9160"


# Prepared statements

CQL doesn't support prepared statements so PDO emulation layer is used for this. Notice that quoting of the
identifiers is a simple escaping method call on the data.

It is mandatory to specify the right types of parameters
$stmt->bindValue('1', PDP::PARAM_STR) will result in a failure with cassandra 1.2.2
$stmt->bindValue('1', PDP::PARAM_INT) will be ok

# Transactions

Transactions are not supported and calling PDO::beginTransaction will result in an exception.


# Contributing

Pull requests containing fixes and/or additional tests are highly appreciated.


# Handling large integers

This driver will try to convert integers to PHP data types unless PDO_CASSANDRA_ATTR_PRESERVE_VALUES
is used. If an overflow would happen, PHP_INT_MAX is returned and an error is raised.
Note that by default PDO will silently ignore the error, unless error mode is set to exceptions.


# Driver specific attributes for PDO::setAttribute

<table>
    <tr>
        <td>PDO::CASSANDRA_ATTR_NUM_RETRIES</td>
        <td>integer</td>
        <td>The amount of connection retries</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_RETRY_INTERVAL</td>
        <td>integer</td>
        <td>Sets how many times to keep retrying a host before marking it as down.</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_MAX_CONSECUTIVE_FAILURES</td>
        <td>integer</td>
        <td>Sets how many times to keep retrying a host before marking it as down.</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_LINGER</td>
        <td>integer</td>
        <td>How long does the socket linger after it's being closed</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_NO_DELAY</td>
        <td>boolean</td>
        <td>Whether to enable/disable Nagle algorithm</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_CONN_TIMEOUT</td>
        <td>integer</td>
        <td>Connection timeout (Deprecated, see PDO::CASSANDRA_TIMEOUT)</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_RECV_TIMEOUT</td>
        <td>integer</td>
        <td>Receive timeout (Deprecated, see PDO::CASSANDRA_TIMEOUT)</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_SEND_TIMEOUT</td>
        <td>integer</td>
        <td>Send timeout (Deprecated, see PDO::CASSANDRA_TIMEOUT) </td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_COMPRESSION</td>
        <td>boolean</td>
        <td>Whether to enable/disable compression</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_THRIFT_DEBUG</td>
        <td>boolean</td>
        <td>Converts thrift debug output into PHP warnings</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_PRESERVE_VALUES</td>
        <td>boolean</td>
        <td>Preserves values as they come from Cassandra</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_TIMEOUT</td>
        <td>int</td>
        <td>Global timeout for a request (includes connection, send, recv and retries) (in ms)</td>
    </tr>
</table>

## The constructor honours the following options

These options can be passed in the fourth argument for PDO constructor

<table>
    <tr>
        <td>PDO::CASSANDRA_TIMEOUT</td>
        <td>int</td>
        <td>Global timeout for a request (includes connection, send, recv and retries) (in ms)</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_THRIFT_DEBUG</td>
        <td>Converts thrift debug output into PHP warnings</td>
    </tr>
    <tr>
        <td>PDO::CASSANDRA_ATTR_PRESERVE_VALUES</td>
        <td>boolean</td>
        <td>Preserves values as they come from Cassandra</td>
    </tr>
</table>
