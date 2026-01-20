# Error Handling

## Errors in Tasks

All uncaught exceptions in a task are treated as fatal exceptions and the task is marked as
failed. The error description and error code are stored in the `error` attribute of the
`pulpcore.app.models.Task` object and returned to the user.

!!! warning "Security: Tracebacks Are Not Returned"
    To prevent information disclosure, **exception tracebacks are logged server-side but are
    NOT stored in the task or returned to API users**. This prevents sensitive information
    (URLs, credentials, file paths) from being exposed through the API.

## Exception Types

### Standard Python Exceptions

For programming errors and standard error scenarios, use [built-in Python exceptions](https://docs.python.org/3/library/exceptions.html)
(e.g., `ValueError`, `TypeError`, `KeyError`). These are appropriate for logic errors and invalid inputs.

### PulpException - Base Class for Domain Errors

For known Pulp-specific error scenarios (timeouts, authentication failures, validation errors),
use exceptions that inherit from `pulpcore.exceptions.PulpException`. Each PulpException:

- Has a unique **error code** (e.g., `PLP0005`)
- Has an associated **HTTP status code** (defaults to 500)
- Implements a user-safe `__str__()` method that describes the error without sensitive data

When a `PulpException` is raised in a task, only the error code and description are returned
to the user. Tracebacks are logged but never exposed through the API.

## Available Pulp Exceptions

Listed below are all available PulpExceptions, sorted by error code.

### InternalErrorException (PLP0000)

Signals an unexpected internal error. This is raised automatically by the task system when
an uncaught exception occurs that is not a `PulpException`.

**Usage in code:**
```python
# In pulpcore/tasking/tasks.py:87
safe_exc = InternalErrorException()
task.set_failed(safe_exc)
```

**User sees:** `"An internal error occurred."`

**When used:** Automatically raised by the task system for unexpected errors (equivalent of HTTP 500).

---

### MissingPlugin (PLP0002)

Raised when a requested plugin is not installed.

**Usage in code:**
```python
# In pulpcore/app/apps.py:57
raise MissingPlugin(plugin_app_label)
```

**User sees:** `"Plugin with Django app label <name> is not installed."`

---

### DigestValidationError (PLP0003)

Raised when a file fails digest/checksum validation during download or artifact validation.

**Usage in code:**
```python
# In pulpcore/download/base.py:238
raise DigestValidationError(actual_digest, expected_digest, url=self.url)
```

**User sees:** `"A file located at the url {url} failed validation due to checksum. Expected '{expected}', Actual '{actual}'"`

---

### SizeValidationError (PLP0004)

Raised when a file fails size validation during download or artifact validation.

**Usage in code:**
```python
# In pulpcore/download/base.py:253
raise SizeValidationError(actual_size, expected_size, url=self.url)
```

**User sees:** `"A file located at the url {url} failed validation due to size. Expected '{expected}', Actual '{actual}'"`

---

### TimeoutException (PLP0005)

Raised when a download or network request times out.

**Usage in code:**
```python
# In pulpcore/download/http.py:265
raise TimeoutException(self.url)
```

**User sees:** `"Request timed out for {url}. Increasing the total_timeout value on the remote might help."`

---

### ResourceImmutableError (PLP0006)

Raised when attempting to modify an immutable resource (e.g., a published repository version).

**Usage in code:**
```python
# In pulpcore/app/models/repository.py:1073
raise ResourceImmutableError(self)
```

**User sees:** `"Cannot update immutable resource {model_pk} of type {model_type}"`

---

### DomainProtectedError (PLP0007)

Raised when attempting to delete a domain that still contains repositories with content.

**Usage in code:**
```python
# In pulpcore/app/models/domain.py:72
raise DomainProtectedError()
```

**User sees:** `"You cannot delete a domain that still contains repositories with content."`

---

### DnsDomainNameException (PLP0008)

Raised when DNS resolution fails for a URL during download operations.

**Usage in code:**
```python
# In pulpcore/download/http.py:306
raise DnsDomainNameException(self.url)
```

**User sees:** `"URL lookup failed."`

---

### UrlSchemeNotSupportedError (PLP0009)

Raised when an unsupported URL scheme is provided (e.g., `ftp://` when only `http://` and
`https://` are supported).

**Usage in code:**
```python
# In pulpcore/download/factory.py:180
raise UrlSchemeNotSupportedError(url)
```

**User sees:** `"URL: {url} not supported."`

---

### ProxyAuthenticationError (PLP0010)

Raised when proxy authentication fails (HTTP 407 response from proxy server).

**Usage in code:**
```python
# In pulpcore/download/http.py:276
raise ProxyAuthenticationError(self.proxy)
```

**User sees:** `"Proxy authentication failed for {proxy_url}. Please check your proxy credentials."`

---

### RepositoryVersionDeleteError (PLP0011)

Raised when attempting to delete a repository version when it's the only version remaining.

**Usage in code:**
```python
# In pulpcore/app/tasks/repository.py:47
raise RepositoryVersionDeleteError()
```

**User sees:** `"Cannot delete repository version. Repositories must have at least one repository version."`
