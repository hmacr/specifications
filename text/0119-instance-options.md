- Title: Instance options
- Start Date: 2022-03-01

# Instance options

## 1. Summary

The instance options let the users configure Meilisearch when launching the search engine using
- environment variables
- command-line options

An example when setting an environment variable to change the folder path where the Meilisearch data are stored:

```bash
export MEILI_DB_PATH=./meilifiles
./meilisearch
```

Same behavior using the command-line option:

```bash
./meilisearch --db-path ./meilifiles
```

## 2. Motivation

When Meilisearch is launched, the default configuration may not meet the specific needs of users. Meilisearch exposes configurable options to allow users to fine-tune the behavior of the search engine.

## 3. Functional Specification

The users can configure Meilisearch when launching the search engine using
- **environment variables**. Ex: `--db-path`
- **command-line options or CLI options**. Ex: `MEILI_DB_PATH`

There are 2 categories of CLI (command-line interface) options:
- the ones that expect a value. Ex: `--db-path "./my_path"`.
- the ones that don't expect any value, called also "flags". Ex: `--no-analytics`. Their implicit values are booleans.

### 3.1. Some specific behaviors

#### 3.1.1. Priority between CLI options and environment variables

Command-line options take precedence over environment variables. If the same configuration option is specified both as a command-line option and as an environment variable, Meilisearch will use the command-line option and its respective value.

#### 3.1.2. Flags: accepted values for the corresponding environment variable

The options that do not expect any value when using the command-line option accepts the following value when using the corresponding environment variable: `n`, `no`, `f`, `false`, `off`, and `0` as `false`. An absent environment variable will also be considered as `false`. Everything else is considered `true`.

Example with the option to make SSL mandatory:
- `export MEILI_SSL_REQUIRE_AUTH=yes` means that mandatory SSL is enabled.
- `export MEILI_SSL_REQUIRE_AUTH=off` means that mandatory SSL is disabled.
- No variable set means that mandatory SSL is disabled.

### 3.2. Error behavior

1. Some configuration options must specify a value to be valid. Using such a command-line option or an environment variable without specifying a value will throw an error and interrupt the launch process.

Example:

❌ Wrong

```bash
./meilisearch --db-path

error: The argument '--db-path <DB_PATH>' requires a value but none was supplied
```

✅ Correct

```bash
./meilisearch --db-path ./meilifiles
```

2. Some command-line options take an implicit boolean as a value. In this case, the users should not set any value when using the option.

❌ Wrong

```bash
./meilisearch --ssl-require-auth yes

error: Found argument 'yes' which wasn't expected, or isn't valid in this context
```

✅ Correct

```bash
./meilisearch --ssl-require-auth
```

The expected behavior of each flag is described in the list above.

### 3.3. Exhaustive list of options

- [Database path](#331-database-path)
- [Environment](#332-environment)
- [HTTP address & port binding](#333-http-address--port-binding)
- [Master key](#334-master-key)
- [Disable analytics](#335-disable-analytics)
- [Dump directory](#336-dump-directory)
- [Import dump](#337-import-dump)
- [Ignore missing dump](#338-ignore-missing-dump)
- [Ignore dump if DB exists](#339-ignore-dump-if-db-exists)
- [Log level](#3310-log-level)
- [Payload limit size](#3311-payload-limit-size)
- [Schedule snapshot creation](#3312-schedule-snapshot-creation)
- [Snapshot destination](#3313-snapshot-destination)
- [Import snapshot](#3314-import-snapshot)
- [Ignore missing snapshot](#3315-ignore-missing-snapshot)
- [Ignore snapshot if DB exists](#3316-ignore-snapshot-if-db-exists)
- [Max memory usage when indexing](#3317-max-memory-usage-when-indexing)
- [Max indexing threads](#3318-max-indexing-threads)
- [SSL authentication path](#3319-ssl-authentication-path)
- [SSL certificates path](#3320-ssl-certificates-path)
- [SSL key path](#3321-ssl-key-path)
- [SSL OCSP path](#3322-ssl-ocsp-path)
- [SSL require auth](#3323-ssl-require-auth)
- [SSL resumption](#3324-ssl-resumption)
- [SSL tickets](#3325-ssl-tickets)
- [Config file path](#3326-config-file-path)
- [Experimental enable Metrics](#3327-experimental-enable-metrics)
- [Experimental reduce indexing memory usage](#3328-experimental-reduce-indexing-memory-usage)

#### 3.3.1. Database path

**Environment variable**: `MEILI_DB_PATH`
**CLI option**: `--db-path`
**Default value**: `"data.ms/"`
**Expected value**: a filepath

Designates the location where database files will be created and retrieved.

#### 3.3.2. Environment

**Environment variable**: `MEILI_ENV`
**CLI option**: `--env`
**Default value**: `development`
**Expected value**: `production` or `development`

Configures the instance's environment. Value must be either `production` or `development`.

`production`:

- Setting a master key of at least 16 bytes is **mandatory**
- The search preview interface is disabled

`development`:

- Setting a master key is **optional**
- Setting a master key of at least 16 bytes is **optional**
- Search preview is enabled

#### 3.3.3. HTTP address & port binding

**Environment variable**: `MEILI_HTTP_ADDR`
**CLI option**: `--http-addr`
**Default value**: `"localhost:7700"`
**Expected value**: an HTTP address and port

Sets the HTTP address and port Meilisearch will use.

#### 3.3.4. Master key

**Environment variable**: `MEILI_MASTER_KEY`
**CLI option**: `--master-key`
**Default value**: `None`
**Expected value**: an alphanumeric string

Sets the instance's master key, automatically protecting all routes except [`GET /health`](/reference/api/health.md). This means you will need an API key to access endpoints such as `POST /search` and `GET /documents`.

You must supply an alphanumeric string when using this option.

If no master key is provided in a `development` environment, all routes will be unprotected and publicly accessible.

##### 3.3.4.1. Error and Warning messages

Providing a master key of at least 16 bytes is mandatory when `--env` is set to `production`; if none is given, Meilisearch will throw an error and refuse to launch.

```
Error: You must provide a master key to secure your instance in a production environment. It can be specified via the MEILI_MASTER_KEY environment variable or the --master-key launch option.

We generated a secure master key for you (you can safely use this token):

>> --master-key `:suggestedMasterKey` <<
```

Providing a master key of at least 16 bytes is mandatory when `--env` is set to `production`; if it is given but too short then, Meilisearch will throw an error and refuse to launch.

```
Error: The master key must be at least 16 bytes in a production environment. The provided key is only `:numBytes` bytes.

We generated a secure master key for you (you can safely use this token):

>> --master-key `:suggestedMasterKey` <<
```

Providing a master key of less than 16 bytes when `--env` is set to `development` displays a warning message.

```
Meilisearch started with a master key considered unsafe for use in a production environment.

A master key of at least 16 bytes will be required when switching to a production environment.

We generated a new secure master key for you (you can safely use this token):

>> --master-key `:suggestedMasterKey` <<

Restart Meilisearch with the argument above to use this new and secure master key.
```

Not providing a master key when `--env` is set to `development` displays a warning message.

```
No master key was found. The server will accept unidentified requests.

A master key of at least 16 bytes will be required when switching to a production environment.

If you need protection in a development environment, we generated a secure master key for you (you can safely use this token):

>> --master-key `:suggestedMasterKey` <<

Restart Meilisearch with the argument above to use this new and secure master key.
```


#### 3.3.5. Disable analytics

**Environment variable**: `MEILI_NO_ANALYTICS`
**CLI option**: `--no-analytics`
**Default**: Enabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Deactivates Meilisearch's built-in telemetry collect when enabled.

#### 3.3.6. Dump directory

**Environment variable**: `MEILI_DUMP_DIR`
**CLI option**: `--dump-dir`
**Default value**: `dumps/`
**Expected value**: a filepath pointing to a valid directory

Sets the directory where Meilisearch will create dump files. If the directory does not exist when a dump is generated it will be created.

#### 3.3.7. Import dump

**Environment variable**: `MEILI_IMPORT_DUMP`
**CLI option**: `--import-dump`
**Default value**: `none`
**Expected value**: a filepath pointing to a `.dump` file

Imports the dump file located at the specified path. Path must point to a `.dump` file.

Meilisearch will only launch once the dump data has been fully indexed.

More regarding dump behaviors in this [spec](https://github.com/meilisearch/specifications/blob/develop/text/0105-dumps-api.md).

#### 3.3.8. Ignore missing dump

**Environment variable**: `MEILI_IGNORE_MISSING_DUMP`
**CLI option**: `--ignore-missing-dump`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Prevents a Meilisearch instance from throwing an error when `--import-dump` does not point to a valid dump file.

This command will throw an error if `--import-dump` is not defined.

More information in this [section of the spec](https://github.com/meilisearch/specifications/blob/develop/text/0105-dumps-api.md#138---ignore-missing-dump).

#### 3.3.9. Ignore dump if DB exists

**Environment variable**: `MEILI_IGNORE_DUMP_IF_DB_EXISTS`
**CLI option**: `--ignore-dump-if-db-exists`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Prevents a Meilisearch instance with an existing database from throwing an error when using `--import-dump`. Instead, the dump will be ignored and Meilisearch will launch using the existing database.

This command will throw an error if `--import-dump` is not defined.

More information in this [section of the spec](https://github.com/meilisearch/specifications/blob/develop/text/0105-dumps-api.md#137---ignore-dump-if-db-exists).

#### 3.3.10. Log level

**Environment variable**: `MEILI_LOG_LEVEL`
**CLI option**: `--log-level`
**Default value**: `'INFO'`
**Expected value**: one of `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE`, or `OFF`

Defines how much detail should be present in Meilisearch's logs.

#### 3.3.11. Payload limit size

**Environment variable**: `MEILI_HTTP_PAYLOAD_SIZE_LIMIT`
**CLI option**: `--http-payload-size-limit`
**Default value**: `104857600` (~100MB)
**Expected value**: an integer (`104857600`) or a human readable size (`100Mb`)

Sets the maximum size of accepted payloads. Value must be given in bytes or explicitly stating a base unit. For example, the default value can be written as `107374182400`, `'107.7Gb'`, or `'107374 Mb'`.

#### 3.3.12. Schedule snapshot creation

**Environment variable**: `MEILI_SCHEDULE_SNAPSHOT`
**CLI option**: `--schedule-snapshot`
**Default when absent**: Disabled
**Optional value**: an integer

⚠️ This command-line option takes an **optional** integer value. Passing the flag with no value implies the default value of 86400.

- Omitting the flag, not defining `MEILI_SCHEDULE_SNAPSHOT`, or specifying `schedule_snapshot=false` in the configuration file disables the scheduled snapshots, and is the default behavior.
- Passing the flag without a value or specifying `schedule_snapshot=true` in the configuration file enables the scheduled snapshot with the default interval of 86400 seconds between each snapshot.
- Passing the flag with an integer value, defining the `MEILI_SCHEDULE_SNAPSHOT` to an integer value, or specifying `schedule_snapshot=x` with `x` an integer value in the configuration file enables the scheduled snapshots with an interval between each snapshot of the specified integer value, in seconds.

#### 3.3.13. Snapshot destination

**Environment variable**: `MEILI_SNAPSHOT_DIR`
**CLI option**: `--snapshot-dir`
**Default value**: `snapshots/`
**Expected value**: a filepath pointing to a valid directory

Sets the directory where Meilisearch will store snapshots. If the directory does not exist when a snapshot is generated it will be created.

#### 3.3.14. Import snapshot

**Environment variable**: `MEILI_IMPORT_SNAPSHOT`
**CLI option**: `--import-snapshot`
**Default value**: `None`
**Expected value**: a filepath pointing to a snapshot file

Launches Meilisearch after importing a previously-generated snapshot at the given filepath.

This command will throw an error if:

- A database already exists
- No valid snapshot can be found in the specified path

This behavior can be modified with the `--ignore-snapshot-if-db-exists` and `--ignore-missing-snapshot` options, respectively.

#### 3.3.15. Ignore missing snapshot

**Environment variable**: `MEILI_IGNORE_MISSING_SNAPSHOT`
**CLI option**: `--ignore-missing-snapshot`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Prevents a Meilisearch instance from throwing an error when `--import-snapshot` does not point to a valid snapshot file.

This command will throw an error if `--import-snapshot` is not defined.

#### 3.3.16. Ignore snapshot if DB exists

**Environment variable**: `MEILI_IGNORE_SNAPSHOT_IF_DB_EXISTS`
**CLI option**: `--ignore-snapshot-if-db-exists`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Prevents a Meilisearch instance with an existing database from throwing an error when using `--import-snapshot`. Instead, the snapshot will be ignored and Meilisearch will launch using the existing database.

This command will throw an error if `--import-snapshot` is not defined.

#### 3.3.17. Max memory usage when indexing

**Environment variable**: `MEILI_MAX_INDEXING_MEMORY`
**CLI option**: `--max-indexing-memory`
**Default value**: 2/3 of the available RAM of the machine
**Expected value**: an integer (`104857600`) or a human readable size (`100Mb`)

Set the maximum size of the RAM used by Meilisearch when indexing. By default, Meilisearch adapts its behavior to make the indexation use at maximum two-thirds of the available resources.

Value must be given in bytes or explicitly stating a base unit. For example, the default value can be written as `107374182400`, `'107.7Gb'`, or `'107374 Mb'`.

⚠️ WARNINGS
- We do not recommend setting the full RAM size of your machine. For example, when running Meilisearch on a machine with 4GB of RAM, do not set this options to `4Gb`
- This command-line option does not perfectly ensure the RAM usage but helps you manage multiple Meilisearch engines on the same machine (for example, using Kubernetes). The search engine cannot guarantee the exact usage of the RAM.
- If the number set is higher than the real available RAM in the machine, we cannot prevent Meilisearch from crashing.

#### 3.3.18. Max indexing threads

**Environment variable**: `MEILI_MAX_INDEXING_THREADS`
**CLI option**: `--max-indexing-threads`
**Default value**: half of the available threads of the machine
**Expected value**: an integer

Sets the maximum number of threads available to use during the indexation.

By default, in machines with multi-core processors, the indexer avoids using more than half of the available processing units. For example, if your machine has twelve cores, the indexer will try to use six of them at most. This ensures Meilisearch is always ready to perform searches, even while you are updating an index.
Users have to be aware that requiring the totality of the threads for the indexation can lead to a huge impact on the search experience.

Obviously, multi-threading is not possible in machines with only one processor core.

If the number set is higher than the real number of core available in the machine, Meilisearch will use the maximum number of available cores.

#### 3.3.19. SSL authentication path

**Environment variable**: `MEILI_SSL_AUTH_PATH`
**CLI option**: `--ssl-auth-path`
**Default value**: `None`
**Expected value**: a filepath

Enables client authentication in the specified path.

#### 3.3.20. SSL certificates path

**Environment variable**: `MEILI_SSL_CERT_PATH`
**CLI option**: `--ssl-cert-path`
**Default value**: `None`
**Expected value**: a filepath pointing to a valid SSL certificate

Sets the server's SSL certificates.

Value must be a path to PEM-formatted certificates. The first certificate should certify the KEYFILE supplied by `--ssl-key-path`. The last certificate should be a root CA.

#### 3.3.21. SSL key path

**Environment variable**: `MEILI_SSL_KEY_PATH`
**CLI option**: `--ssl-key-path`
**Default value**: `None`
**Expected value**: a filepath pointing to a valid SSL keyfile

Sets the server's SSL keyfiles.

Value must be a path to an RSA private key or PKCS8-encoded private key, both in PEM format.

#### 3.3.22. SSL OCSP path

**Environment variable**: `MEILI_SSL_OCSP_PATH`
**CLI option**: `--ssl-ocsp-path`
**Default value**: `None`
**Expected value**: a filepath pointing to a valid OCSP certificate

Sets the server's OCSP file. *Optional*

Reads DER-encoded OCSP response from OCSPFILE and staple to certificate.

#### 3.3.23. SSL require auth

**Environment variable**: `MEILI_SSL_REQUIRE_AUTH`
**CLI option**: `--ssl-require-auth`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Makes SSL authentication mandatory.

Sends a fatal alert if the client does not complete client authentication.

#### 3.3.24. SSL resumption

**Environment variable**: `MEILI_SSL_RESUMPTION`
**CLI option**: `--ssl-resumption`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Activates SSL session resumption.

#### 3.3.25. SSL tickets

**Environment variable**: `MEILI_SSL_TICKETS`
**CLI option**: `--ssl-tickets`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Activates SSL tickets.

#### 3.3.26. Config file path

**Environment variable**: `MEILI_CONFIG_FILE_PATH`
**CLI option**: `--config-file-path`
**Default**: `./config.toml`

Define the config file to load at Meilisearch launch.

See [Configuration File](0185-configuration-file.md) specification details.

#### 3.3.27. Experimental enable Metrics

**Environment variable**: `MEILI_EXPERIMENTAL_ENABLE_METRICS`
**CLI option**: `--experimental-enable-metrics`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Activate the `/metrics` endpoint to collect Meilisearch metrics for monitoring purposes. See [0174-metrics-api.md](0174-metrics-api.md).

#### 3.3.28. Experimental Reduce Indexing Memory Usage

**Environment variable**: `MEILI_EXPERIMENTAL_REDUCE_INDEXING_MEMORY_USAGE`
**CLI option**: `--experimental-reduce-indexing-memory-usage`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Enables the `MDB_WRITEMAP` option of LMDB, making the internal key-value store use much less RAM than usual.

## 4. Technical Aspects

N/A

## 5. Future Possibilities

- Redo the command-line to create a more interactive CLI
- Autocomplete for the options when using the Meilisearch CLI
