---
title: "Tidy Processes in Python"
date: 2019-11-07T20:12:00-05:00
tags: [Python]
---

The [subprocess module](https://docs.python.org/3/library/subprocess.html#subprocess.run) from the Python standard library makes it easy to start processes from within your Python script.

But, it gets more complex if:

1. You need to run the subprocess concurrently with your script
2. The subprocess creates its own child processes

In my case, I had a [Flask](https://flask.palletsprojects.com/en/1.1.x/) server that needed to be running while my experiment script ran since my script made API calls to the server.

{{<mermaid>}}
graph LR;
  A[experiment.py] -->|request| B[api_server.py]
  B -->|response| A
{{</mermaid>}}

Dealing with concurrency (1) is not too hard as its documented clearly in the `subprocess` docs.
Just use `subprocess.Popen`:


```python
# experiment.py
import subprocess

proc = subprocess.Popen(['python', 'flask_app.py'])
result = run_experiment() # sends API requests to flask app
print(result)
```

Running `python experiment.py` once and the experiment runs successfully! :smile:

**BUT**, running `python experiment.py` again reveals the ugly truth:

```sh
python flask_app.py
 * Serving Flask app "flask_app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
Traceback (most recent call last):
  File "flask_app.py", line 12, in <module>
    app.run(debug=True)
  File "/Users/pedro/Library/Caches/pypoetry/virtualenvs/tidy-process-py3.7/lib/python3.7/site-packages/flask/app.py", line 990, in run
    run_simple(host, port, self, **options)
  File "/Users/pedro/Library/Caches/pypoetry/virtualenvs/tidy-process-py3.7/lib/python3.7/site-packages/werkzeug/serving.py", line 988, in run_simple
    s.bind(server_address)
OSError: [Errno 48] Address already in use
```

The Flask server from the first run is still running, causing new runs to crash when a new Flask server tries to take the same port (by default port `5000` for Flask).
`proc.kill` doesn't save us since that will kill the `python flask_app.py` subprocess, but the Flask server actually runs in a child process *of that subprocess*, leaving us with a [zombie process](https://en.wikipedia.org/wiki/Zombie_process) :zombie:.

## Solution

The solution is to recursively kill all the children processes :skull::[^1]
[^1]: Yikes, OS programming is violent...

```python
# tidy_process.py
from contextlib import contextmanager
import subprocess

import psutil


@contextmanager
def tidy_process(*args, **kwargs):
    proc = subprocess.Popen(*args, **kwargs)
    try:
        yield proc
    finally:
        for child in psutil.Process(proc.pid).children(recursive=True):
            child.kill()
        proc.kill()
```

Note the use of `@contextmanager` in conjunction with `try`/`finally` blocks that guarantee that the process (and its descendant processes) are killed even if an exception occurs in the calling code.

If you aren't familiar with [context managers](https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager), I highly recommend [Raymond Hettinger's introduction to them](https://youtu.be/OSGv2VnC0go?t=2365).

## Usage

```python
# experiment.py
with open('out.log', 'w') as out, open('err.log', 'w') as err:
    command = ['python', 'flask_app.py']
    with tidy_process(command, stdout=out, stderr=err) as proc:
        run_simulation()
        0 / 0 # still tidy, even when exceptions occur
```

Here, I log `stdout`/`stderr` of the subprocess to files.
Not necessary, but convenient for debugging :bug:.
