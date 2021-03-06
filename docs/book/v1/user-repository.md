# User Repository

An authentication adapter can pull user information from a variety
of repositories:

- an [htpasswd](https://httpd.apache.org/docs/current/programs/htpasswd.html) file
- a database
- a cache

mezzio-authentication provides an interface,
`Mezzio\Authentication\UserRepositoryInterface`, to access this user
storage:

```php
namespace Mezzio\Authentication;

interface UserRepositoryInterface
{
    /**
     * Authenticate the credential (username) using a password
     * or using only the credential string (e.g. token based credential)
     * It returns the authenticated user or null.
     *
     * @param string $credential can be also a token
     */
    public function authenticate(string $credential, string $password = null) : ?UserInterface;
}
```

It contains only the `authenticate()` function, to authenticate the user's
credential. If authenticated, the result will be a `UserInterface` instance;
otherwise, a `null` value is returned.

## Configure the User Repository

In order to use a user repository adapter, we need to configure it. For instance,
to consume an `htpasswd` file, we need to configure the path to the file.
Such configuration is provided in the `authentication` hierarchy provided to
your [PSR-11](http://www.php-fig.org/psr/psr-11/) container. We demonstrate
examples of such configuration below.

Using [Mezzio](https://docs.mezzio.dev/mezzio/), this
configuration can be stored in a file under the `/config/autoload/` folder.  We
suggest to use a `.local.php` suffix &mdash; e.g.
`/config/autoload/auth.local.php` &mdash; as local configuration is not stored
in the version control system.

You can also provide this configuration using a [ConfigProvider.php](https://github.com/mezzio/mezzio-authentication/blob/master/src/ConfigProvider.php)
class. [Read this blog post](https://getlaminas.org/blog/2017-04-20-config-aggregator.html)
for more information on config providers.

## htpasswd Configuration

When using the htpasswd user repository implementation, you need only configure
the path to the `htpasswd` file and a `realm`. The `htpasswd` file must use bcrypt hash algorithm:

```php
return [
    'authentication' => [
        'realm' => 'insert realm value',
        'htpasswd' => 'insert the path to htpasswd file',
    ],
];
```

## PDO Configuration

When using the PDO user repository adapter, you will need to provide PDO
connection parameters, as well as information on the table, field names, and a
SQL statement for retrieving user roles:

```php
return [
    'authentication' => [
        'pdo' => [
            'dsn' => '',
            'username' => '',
            'password' => '',
            'table' => 'user table name',
            'field' => [
                'identity' => 'identity field name',
                'password' => 'password field name',
            ],
            'sql_get_roles'   => 'SQL to retrieve roles with :identity parameter',
            'sql_get_details' => 'SQL to retrieve user details by :identity',
        ],
    ],
];
```

The required parameters are `dsn`, `table`, and `field`.

The `dsn` value is the DSN connection string to be used to connect to the database.
For instance, using a SQLite database, a typical value is `sqlite:/path/to/file`.

The `username` and `password` parameters are optional parameters used to connect
to the database. Depending on the database, these parameters may not be required;
e.g. [SQLite](https://sqlite.org/) does not require them.

The `table` value is the name of the table containing the user credentials.

The `field` parameter contains the field name of the `identity` of the user and
the user `password.` The `identity` of the user can be a username, an email, etc.

The `sql_get_roles` setting is an optional parameter that contains the SQL query
for retrieving the user roles. The identity value must be specified using the
placeholder `:identity`. For instance, if a role is stored in a user table, a
typical query might look like the following:

```sql
SELECT role FROM user WHERE username = :identity
```

The `sql_get_details` parameter is similar to `sql_get_roles`: it specifies the
SQL query for retrieving the user's additional details, if any.

For instance, if a user has an email field this can be returned as additional
detail using the following query:

```sql
SELECT email FROM user WHERE username = :identity
```

### PDO Service Name

> Since 1.1.0

As an alternative, you can provide a service name instead of PDO connection
parameters. In such a case, you can substitute the key `service` for the `dsn`
configuration key:

```php
return [
    'authentication' => [
        'pdo' => [
            'service' => \PDO::class,                 // "service" instead of "dsn"
            'table' => 'user table name',
            'field' => [
                'identity' => 'identity field name',
                'password' => 'password field name',
            ],
            'sql_get_roles'   => 'SQL to retrieve roles with :identity parameter',
            'sql_get_details' => 'SQL to retrieve user details by :identity',
        ],
    ],
];
```

The parameters `table` and `field` still remain required, and one or the other
of `dsn` or `service` **MUST** be present.

When specifying the `service` key, the value **MUST** evaluate to an existing
service that resolves to a `PDO` instance.
