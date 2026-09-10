# PHP-FPM Configuration

## PHP Configuration Value Resolution

This container applies a systematic approach to determine the final value of any PHP directive. The resolution process is hierarchical aN/Aensures that container-level overrides always take precedence.

### 1. Resolution Hierarchy

The final value of a PHP-FPM directive is determined in the following order:

1. **Container-level override**
   - If the directive is explicitly set via an environment variable or container parameter, this value **always takes priority**.
   - This allows runtime customization without modifying `.conf` files.

2. **Fixed default value**
   - If no container override is provided, a built-in **fixed default** is used.

## PHP-FPM Configuration Environment

This table summarizes how PHP-FPM configuration directives are mapped to environment variables in our containerized setup. It also shows the default values depending on the context and the final value actually used in production or development environments.

### Global directives

| Directive                             | Environment Variable                      | Default (prd)     | Default (dev)     | Documentation                                                                              |
|---------------------------------------|-------------------------------------------|-------------------|-------------------|--------------------------------------------------------------------------------------------|
| `error_log`                           | `PHP_FPM_ERROR_LOG`                       | `/proc/self/fd/2` | `/proc/self/fd/2` | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `log_level`                           | `PHP_FPM_LOG_LEVEL`                       | `notice`          | `notice`          | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `log_limit`                           | `PHP_FPM_LOG_LIMIT`                       | `8192`            | `8192`            | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `process_control_timeout`             | `PHP_FPM_PROCESS_CONTROL_TIMEOUT`         | `15s`             | `15s`             | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |

#### Draining requests on shutdown

`process_control_timeout` is how long the php-fpm master waits for its children to finish the request they are serving before killing them. The php-fpm default is `0`, which means it does not wait at all — a graceful stop terminates workers mid-request, and clients see a 502.

Three timeouts have to line up, from shortest to longest:

| | value | set where |
|---|---|---|
| `process_control_timeout` | `15s` | this variable |
| supervisor `stopwaitsecs` | `20s` | `config/supervisor.d/php-fpm.ini.tmpl` |
| the platform's grace period | **must exceed 15s** | see below |

If the platform's grace period is shorter, the container is killed before the master has finished draining and the exit code is `137` instead of `0`:

| in flight | grace period | result |
|---|---|---|
| a 3s request | 10s (plain `docker stop`) | request completes, container stops in ~1.5s, exit `0` |
| a 15s request | 30s (OpenShift/Kubernetes default) | request completes, container stops in ~13s, exit `0` |
| a 15s request | 10s (plain `docker stop`) | container killed at 10s, exit `137` |

Kubernetes and OpenShift default `terminationGracePeriodSeconds` to 30 and need no change. For compose, set `stop_grace_period: 30s`; for a bare `docker stop`, pass `-t 30`. Lower this variable instead if you would rather cap the wait — the container then always stops cleanly, at the cost of cutting requests that run longer.

### Pool directives

| Directive                                  | Environment Variable                               | Default (prd)     | Default (dev)     | Documentation                                                                              |
|--------------------------------------------|----------------------------------------------------|-------------------|-------------------|--------------------------------------------------------------------------------------------|
| `access.log`                               | `PHP_FPM_ACCESS_LOG`                               | `/proc/self/fd/2` | `/proc/self/fd/2` | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `access.format`                            | `PHP_FPM_ACCESS_FORMAT`                            | `"%R - %u %t \"%m %r%Q%q\" %s %f %{milli}d %{kilo}M %C%%"` | `"%R - %u %t \"%m %r%Q%q\" %s %f %{milli}d %{kilo}M %C%%"` | [Link](https://www.php.net/manual/en/install.fpm.configuration.php) |
| `clear_env`                                | `PHP_FPM_CLEAR_ENV`                                | `no`              | `no`              | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `listen.mode`                              | `PHP_FPM_LISTEN_MODE`                              | `0777`            | `0777`            | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `listen.allowed_clients`                   | `PHP_FPM_LISTEN_ALLOWED_CLIENTS`                   | `127.0.0.1`       | `127.0.0.1`       | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `pm`                                       | `PHP_FPM_PM`                                       | `ondemand`        | `ondemand`        | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `pm.max_children`                          | `PHP_FPM_PM_MAX_CHILDREN`                          | `40`              | `40`              | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `pm.process_idle_timeout`                  | `PHP_FPM_PM_PROCESS_IDLE_TIMEOUT`                  | `10s`             | `10s`             | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `pm.max_requests`                          | `PHP_FPM_PM_MAX_REQUESTS`                          | `0`               | `0`               | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `catch_workers_output`                     | `PHP_FPM_CATCH_WORKERS_OUTPUT`                     | `yes`             | `yes`             | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `decorate_workers_output`                  | `PHP_FPM_DECORATE_WORKERS_OUTPUT`                  | `no`              | `no`              | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `request_terminate_timeout`                | `PHP_FPM_REQUEST_TERMINATE_TIMEOUT`                | `75s`             | `75s`             | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `request_terminate_timeout_track_finished` | `PHP_FPM_REQUEST_TERMINATE_TIMEOUT_TRACK_FINISHED` | `no`              | `no`              | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `request_slowlog_timeout`                  | `PHP_FPM_REQUEST_SLOWLOG_TIMEOUT`                  | `0`               | `0`               | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `request_slowlog_trace_depth`              | `PHP_FPM_REQUEST_SLOWLOG_TRACE_DEPTH`              | `20`              | `20`              | [Link](https://www.php.net/manual/en/install.fpm.configuration.php)                        |
| `slowlog`                                  | `PHP_FPM_SLOWLOG`                                  | `/app/var/log/php-fpm-slow.log` | `/app/var/log/php-fpm-slow.log` | [Link](https://www.php.net/manual/en/install.fpm.configuration.php) |

#### Bounding a stuck request

Two timers are involved and they do not cover the same thing.

`max_execution_time` is PHP's own, and it counts only the time PHP spends running. On Unix a
blocking system call is not counted, so a query waiting on a database, a call to a dead upstream or
a read from a slow mount is not bounded by it at all:

| setup | result |
|---|---|
| `max_execution_time=5`, `request_terminate_timeout=0`, script in `sleep(20)` | HTTP 200 after **20.04s** |
| `max_execution_time=5`, `request_terminate_timeout=5`, same script | HTTP 502 after **5.33s** |

The worker is not released when the caller gives up either. A script that writes nothing never
notices the disconnection: with the timeout at `0`, php-fpm still reported the process as active 25s
after the client had been interrupted at 3s.

`request_terminate_timeout` is php-fpm's, and it is the only one that covers those cases. It
defaults to `75s` here, just above nginx's `fastcgi_read_timeout` (65s) and apache's `Timeout` (60s):
a request still running then has already failed for its caller, so raising the default costs nothing
observable and returns the worker to the pool. Raise it together with the web server's own timeout
if your application genuinely needs longer.

`request_terminate_timeout_track_finished` stays `no`, so work done after `fastcgi_finish_request()`
— a Symfony `kernel.terminate`, for instance — is not counted against the timeout.

### Deprecated variables

These variables are still accepted for backward compatibility but will be removed in a future version. Replace them with their documented successors.

| Deprecated Variable | Replacement | Notes |
|---------------------|-------------|-------|
| `PHP_FPM_REQUEST_MAX_MEMORY_IN_MEGABYTES` | `PHP_MEMORY_LIMIT` | Value is converted automatically (`16` → `16M`). A warning is emitted at startup. |
| `PHP_FPM_MAX_CHILDREN` | `PHP_FPM_PM_MAX_CHILDREN` | Direct alias. A warning is emitted at startup. |
| `CONTAINER_HEAP_PERCENT` | `PHP_MEMORY_LIMIT` | Accepted and ignored: nothing reads it and no warning is emitted, so a value set here has never had any effect. Set `PHP_MEMORY_LIMIT` instead. |