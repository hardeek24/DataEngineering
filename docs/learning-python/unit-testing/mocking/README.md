# Mocking and Patching

Mocking and patching are crucial techniques in software testing that allow the replacement of parts of a system with fake objects called **mocks**.
This technique simulates behavior and data, which enables testing specific parts of a system without relying on other parts.
As a result, this technique can help isolate and identify issues in the code.

**Mocking** creates a fake object that mimics the behavior of a real object. The purpose is to isolate the code under test from external dependencies, such as databases or web services.

**Patching**, on the other hand, involves replacing the implementation of a function or method with a mock object.

## Patching using `monkeypatch`

Pytest provides a `monkeypatch` fixture that helps safely modify attributes, dictionary items, environment variables,
or `sys.path` for importing in tests.

In below example, we define a mock response object with a JSON method that returns the expected response from API request.
It will use `monkeypatch` to replace `requests.get` with our mock response object
so that when `get_data` is called, it returns our mock response instead of making a real API request.

```python title="main.py"
import requests

def get_json(url):
    r = requests.get(url)
    return r # in case of valid API Url, we will return r.json()
```

```python title="test_main.py"
import requests
from main import get_json

def test_get_json(monkeypatch):

    def mock_get(*args, **kwargs):
        return {"mock_key": "mock_response"}

    monkeypatch.setattr(requests, "get", mock_get)

    result = get_json("https://fakeurl")
    assert result["mock_key"] == "mock_response"
```

## Using `mocker`

We need to use plugin `pytest-mock`, it's a pytest plugin that offers an easier-to-use API and integrates seamlessly with pytest fixtures.

```python title="main.py"
def divide(a, b):
    return a / b

def compute(a, b):
    result = divide(a, b)
    return result * 100
```

```python title="test_main.py"
import pytest
from main import compute

def test_compute(mocker):
    mocker.patch('main.divide', return_value=2)
    assert compute(10, 5) == 200
```

## MagicMock object

In addition to `mocker.patch()`, mocker also provides a `mocker.MagicMock` object that can be used to create mock objects with custom attributes.

Here are some of the methods provided by the `MagicMock` object:

- `assert_called_once()`: This method is used to assert that a mock object was called exactly once.
- `assert_called_with(arg1, arg2, ...)`: used to assert that a mock object was called with the specified arguments.
- `assert_not_called()`: used to assert that a mock object was never called.
- `assert_any_call(arg1, arg2, ...)`: used to assert that a mock object was called with the specified arguments at least once.

```python
import pytest

def test_my_function_1(mocker):
    json = mocker.MagicMock()
    json.loads.assert_not_called()
    json.loads('{"key": "value"}')
    json.loads.assert_called()
    json.loads.assert_called_once()
    json.loads.assert_called_with('{"key": "value"}')

def test_my_function_2(mocker):
    json = mocker.MagicMock()
    json.loads.assert_called_once()
```

## Try yourself - Mocking context managers

**Mocking context managers** is a common technique in testing for isolating the code being tested from its external dependencies.
Context managers are objects that define how a particular block of code should be managed.

- we will test `my_module.py` with its test cases `test_my_module.py`

```python title="my_module.py"
class FileManager:
    def __init__(self, filename):
        self.filename = filename

    def __enter__(self):
        self.file = open(self.filename, 'r')
        return self.file

    def __exit__(self, exc_type, exc_value, tb):
        self.file.close()

def count_lines_in_file(filename):
    with FileManager(filename) as file:
        return len(file.readlines())
```

```python title="test_my_module.py"
import io
import my_module
import pytest

def test_count_lines_in_file(mocker):
    mock_file_manager = mocker.MagicMock()
    mock_file = mocker.MagicMock(spec=io.IOBase)
    mock_file_manager.return_value.__enter__.return_value = mock_file

    mocker.patch('my_module.FileManager', mock_file_manager)

    # Set up the mock file to return 5 lines
    mock_file.readlines.return_value = ['line\n'] * 5

    assert my_module.count_lines_in_file('filename.txt') == 5
```

- run command

```bash
docker build -t pytest-mocking docs/learning-python/unit-testing/mocking/ && docker run pytest-mocking
```

## Result

```bash
============================= test session starts ==============================
platform linux -- Python 3.9.19, pytest-8.2.0, pluggy-1.5.0
rootdir: /test
plugins: mock-3.14.0
collected 1 item

test_my_module.py .                                                      [100%]

============================== 1 passed in 0.01s ===============================
```
