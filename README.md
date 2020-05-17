# QMe

"Queue-me" is a jobs queue and dashboard generation tool that can be used
to specify executors (entities that run jobs) and actions for them. You can
use qme only on the command line, or if desired, via an interactive web dashboard.
The dashboard (and it's dependencies) are not required for using the base library.

[![PyPI version](https://badge.fury.io/py/qme.svg)](https://badge.fury.io/py/qme)

**under development**

## Documentation

This documentation will be moved to a docs folder, likely after a logo and branding
is developed, or @vsoch gets tired of writing things here!

## Install

(template in docs)

If you want to use a sqlite, mysql, or postgre database (and not the filesystem,
recommended) then do:

```bash
pip install qme[database]
pip install -e .[database]
```


If you want to install the web interface, you'll need flask and some extras.
These can be installed with "app":

```bash
pip install qme[app]
pip install -e .[app]
```

or just install all dependencies:

```bash
pip install qme[all]
pip install -e .[all]
```

## Configuration

Configuration is optional, and will (if desired) allow you to define a custom
database for your install. When you first install qme and run it, if you haven't
configured anything, a .qme file will be created in your home, and metadata
files stored here. For many settings, you can either set or update them via
the command line client with `qme config`, or set environment variables 
at runtime (or in your bash profile) for one off changes to default configurations.

## Environment

The following environment variables can be set to determine runtime behavior.

### QME_WORKERS
the number of multiprocessing workers to use (for executors that can use it). Set to be 2*2nproc + 1 if not set.

### QME_SHELL

the default shell for an interactive manager (defaults to ipython, then checks python, and bpython)

### QME_DATABASE

the database to use. For example, you can specify just `filesystem` or `sqlite`, or `postgres` or `mysql`.
For the last three, you can optionally specify `QME_DATABASE_STRING` to include a particular
set of credentials needed for access. This will be saved in your `QME_HOME` secrets.

### QME_DATABASE_STRING

If you have a custom string for a database or file, you can specify it with `QME_DATABASE_STRING`.
(todo add expfactory examples here)
See [database setup](#database-setup) for more details.

### QME_HOME

The "home" directory for QueueMe is by default placed in your $HOME in a directory called .qme.
Within that directory, you will see the following structure:

```bash
.qme/
  config.json (- configuration
  database/   (- database for filesystem, if applicable
```

If you want to change this location, then you'll need to (more permanently) export
`QME_HOME` in your bash profile, or perhaps in a container install.

### QME_SOCKET_UPDATE_SECONDS

If you are using the dashboard (which uses web sockets) this is the number of
seconds to update it. This basically will update the dashboard table
with the content of your Qme Database.

### Database Setup

When you first run a command, without any setup a filesystem database is used, and
the metadata and files are stored in your $HOME in a hidden `.qme` directory. 
The folder with database files would be at `$HOME/.qme/database`. This
is referred to as the "filesystem" database, and is appropriate for running in headless
environments where you don't have special privileges. However, if you 
have access to a more robust database (or want to use sqlite) you have several
database options to choose from. For any of these options, you will need
to install sqlachemy, which can be done with:

```bash
pip install qme[database]
```

While the filesystem database is suitable for use cases with few tasks or just
for testing, for anything else we recommend at least using an Sqlite database
that can better be queried.

**instructions will be written for customizing database, development being done with filesystem**

### Running Commands

These sections will be expanded as the library is developed.

#### Run
You can currently run a basic terminal command (one that is executed and has error,
output, and a return code) like the following:

```bash
$ qme run ls $PWD
DATABASE: filesystem
[shell-5b8a154e-5178-4b1c-811c-d493d4349f1f][returncode: 0]
```

The command is so quick that it gives you the result immediately. In the above,
we see the executor (shell) along with the unique id (the following uuid), 
and the full command (to list the expanded $PWD) and the return code 0.

#### Get 

Once we know a task id (`shell-5b8a154e-5178-4b1c-811c-d493d4349f1f` for the above)
we likely want to get a summary of it. We can do that with `qme get`, which 
expects a task id.

```bash
$ qme get shell-17f3d485-5820-4833-bc6c-1c8ed4ce31b7
DATABASE: filesystem
{
    "executor": "shell",
    "uid": "shell-17f3d485-5820-4833-bc6c-1c8ed4ce31b7",
    "data": {
        "pwd": "/home/vanessa/Desktop/Code/qme/tests",
        "user": "vanessa",
        "output": [
            "helpers.sh\n",
            "__init__.py\n",
            "__pycache__\n",
            "test_client.sh\n",
            "test_executor_shell.py\n",
            "test_filesystem.py\n"
        ],
        "error": [],
        "returncode": 0,
        "command": [
            "ls"
        ],
        "status": "complete",
        "pid": 15183
    }
}
```

Actually, we can get the last task run (the same as above) just with qme get.

```bash
$ qme get
```

It will retrieve the last updated entry in the database across executors.

#### List

For the command line, you can easily list tasks. For the filesystem database,
since we would need to read in several json files, the listing just shows the
task ids. If you do a general list with `qme ls`, it will show all task ids:

```bash
$ qme ls
DATABASE: filesystem
1  shell-7a2f5e23-27ef-49eb-a6a6-896ddc690117
2  shell-84f90411-a53e-4d40-92b0-706e5ddfa3b9
3  shell-5c61ce2b-988e-44fa-8678-a9704ca11b1a
4  shell-542e11ef-a1d1-47ce-90a7-40fc7829d29f
5  shell-0f85a77f-f442-46b1-88b1-7d55303b2119
6  shell-5a9e4413-b25d-4dca-826f-f9f8b27abf50
7  shell-2e65e814-388d-4617-bf89-ef307ba6fa40
8  shell-c07e9279-eee7-4778-9212-4ac617a6082e
9  shell-bcaae4db-2970-4674-9800-376c891c1454
10 shell-38307314-9825-435b-8fb6-6d37b3427a7b
11 shell-ad239488-2a27-47b9-8225-da227625e913
```

The above tasks are for the shell executor, which is the default if no special
executor is selected. You could explicitly state this like:

```bash
$ qme ls shell
DATABASE: filesystem
1  shell-7a2f5e23-27ef-49eb-a6a6-896ddc690117
2  shell-84f90411-a53e-4d40-92b0-706e5ddfa3b9
3  shell-5c61ce2b-988e-44fa-8678-a9704ca11b1a
4  shell-542e11ef-a1d1-47ce-90a7-40fc7829d29f
5  shell-0f85a77f-f442-46b1-88b1-7d55303b2119
6  shell-5a9e4413-b25d-4dca-826f-f9f8b27abf50
7  shell-2e65e814-388d-4617-bf89-ef307ba6fa40
8  shell-c07e9279-eee7-4778-9212-4ac617a6082e
9  shell-bcaae4db-2970-4674-9800-376c891c1454
10 shell-38307314-9825-435b-8fb6-6d37b3427a7b
11 shell-ad239488-2a27-47b9-8225-da227625e913
```


This library is heavily under development, not all code is in verison control,
and nowhere near ready for use!

### Clear

If you want to delete a task, just use clear with it's unique id:

```bash
$ qme clear shell-84f90411-a53e-4d40-92b0-706e5ddfa3b9
DATABASE: filesystem
This will delete task shell-84f90411-a53e-4d40-92b0-706e5ddfa3b9, are you sure? [n]|y: y
shell-84f90411-a53e-4d40-92b0-706e5ddfa3b9 has been removed.
```

You can also remove an entire executor:

```bash
$ qme clear shell
DATABASE: filesystem
This will delete all executor shell tasks, are you sure? [n]|y: n
```

or all tasks in the database:

```bash
$ qme clear
DATABASE: filesystem
This will delete all tasks, are you sure? [n]|y: n
```

Each time you'll be asked for a confirmation first, in case the command was 
run in error.

### Rerun

You can re-run any task, also based on it's taskid. A re-run will load the 
previous command, change to a different directory (if set) and then
re-run the command. The result will be stored under the  (updated) taskid.
Here is a quick example of showing an older task (run before some of the library
was developed) and then using re-run, and showing that the task is updated.
First, here is the original task:

```bash
$ qme get shell-5c61ce2b-988e-44fa-8678-a9704ca11b1a
DATABASE: filesystem
{
    "executor": "shell",
    "command": [
        "ls"
    ],
    "uid": "shell-5c61ce2b-988e-44fa-8678-a9704ca11b1a"
}
```

Now we re-run it:

```bash
$ qme rerun shell-5c61ce2b-988e-44fa-8678-a9704ca11b1a
DATABASE: filesystem
[shell-5c61ce2b-988e-44fa-8678-a9704ca11b1a][returncode: 0]
```

And finally, we see that the task is updated.

```bash
$ qme get shell-5c61ce2b-988e-44fa-8678-a9704ca11b1a
DATABASE: filesystem
{
    "executor": "shell",
    "command": [
        "ls"
    ],
    "uid": "shell-5c61ce2b-988e-44fa-8678-a9704ca11b1a",
    "data": {
        "pwd": "/home/vanessa/Desktop/Code/qme",
        "output": [
            "build\n",
            "CHANGELOG.md\n",
            "dist\n",
            "docs\n",
            "LICENSE\n",
            "MANIFEST.in\n",
            "paper\n",
            "qme\n",
            "qme.egg-info\n",
            "README.md\n",
            "setup.cfg\n",
            "setup.py\n",
            "tests\n"
        ],
        "error": [],
        "returncode": 0
    }
}
```

You can also rerun the last touched task without needing to specify the identifier.

```bash
$ qme rerun
```

## Executors

All executors should be derived from the [ExecutorBase](https://github.com/vsoch/qme/blob/master/qme/main/executor/base.py#L74) class that will ensure that each one exposes the needed functions. Each executor also has it's own view under [app/templates](qme/app/templates) that renders a page specific to it for the dashboard (under development).
You should reference the class to see the functions that are required and conditions for each.

### Metadata

#### Base

Each executor exposes the following metadata:

 - **pwd**: the present working directory where the command was run
 - **command**: the command that was run
 - **user**: the user that ran the command
 - **status**: the status of the operation. Since most basic commands save the first time upon completion, the status is usually complete, however this is subject to change. This must be one of "complete" "cancelled" or "running" or None.

The only metadata shown on the table (front) page of the dashboard is these common attributes.
For the filesystem database, since we'd need to read many separate files, we just show
the executor type and unique id. The user must click on any particular execution to see
the full details.

#### Shell

 - **output**: the output stream of running the command
 - **error**: the error stream of running the command
 - **returncode**: the returncode from running the command
 - **pid**: the pid of the child process.


## Dashboard

The dashboard can be started with `qme start`

```bash
$ qme start
DATABASE: filesystem
Server initialized for gevent.
QueueMe!
```

If you add `--debug` it will run in debug mode:

```bash
$ qme start --debug
DATABASE: filesystem
Server initialized for gevent.
QueueMe!
 * Restarting with stat
DATABASE: filesystem
Server initialized for gevent.
QueueMe!
 * Debugger is active!
 * Debugger PIN: 210-139-092
```

By default, it will deploy the dashboard to [localhost:5000](http://localhost:5000).
The prototype is shown below (hugely subject to change!)

![docs/assets/img/prototype.png](docs/assets/img/prototype.png)

You can customize the port with `--port`:

```bash
$ qme start --port 8000
```

When it starts, it will initialize the queue and database as it would do with
any other command, so if you need to set this variable (and haven't done
so in your global config) you should do that here:

```bash
$ qme start --config_dir /tmp/custom_home
```

The server can also be run by calling the start function directly, and providing
a queue:

```python
from qme.app.server import start
from qme.main import Queue

queue = Queue(config_dir="/tmp/custom_home")
start(debug=True, queue=queue, port=5000)
```

or you can use the Queue defaults (config directory in $HOME/.qme with your database
specified in your `$HOME/.qme/config.ini` if you execute the script
directly:

```bash
$ python qme/app/server.py
```

This would be equivalent to calling the start command with defaults.

The interface is **under development** - there is currently a main table
page with commands run, and each will reveal it's type that can be clicked
on to see an executor-specific page.

### Logging

If you want to look at server logs for the dashboard, they will be printed
by defualt to your Qme Home (`$HOME/.qme`) in a file called `dashboard.log`:

```bash
$ cat /home/vanessa/.qme/dashboard.log 
Starting Thread
2020-05-16 16:13:29,555 - qme.app.server - DEBUG - Client connected
2020-05-16 16:13:29,555 - qme.app.server - DEBUG - Starting Thread
2020-05-16 16:13:33,644 - qme.app.server - DEBUG - Client connected
```

### Environment

 * Free software: MPL 2.0 License

## TODO

 - each executor should have unique id that is used for logger, database, etc.
 - design models for filesystem or relational database
 - base should be able to use a user defined database for jobs (define on onset)
