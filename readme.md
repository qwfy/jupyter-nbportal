# Jupyter Notebook Portal


## What's this?

Jupyter Notebook Portal accepts input from a RabbitMQ queue, creates a new cell for that input and executes it. This allows one to send text from other places, such as text editors, to the notebook.


## How does it work?

It does so by connecting to a RabbitMQ server when the notebook is loaded in the browser, and a Javascript file handles the messages received from the server.

The choice of RabbitMQ isn't optimal, since it's sort of heavy for such a simple thing, but it's readily available, thus saves the effort of building a websocket server.


## How to use it?

First, install this plugin:

    git clone https://github.com/qwfy/jupyter-nbportal.git

    # if using conda, you can append `--sys-prefix` to both of the following commands
    jupyter nbextension install ./jupyter-nbportal
    jupyter nbextension enable jupyter-nbportal/main

See [Jupyter Notebook's Installing and enabling extensions](https://jupyter-notebook.readthedocs.io/en/latest/extending/frontend_extensions.html#installing-and-enabling-extensions) for more switches.

Then, install RabbitMQ and an extension, depends on your Linux distribution, these steps may vary, and root privilege maybe needed.

    # install RabbitMQ itself, if you are using Arch Linux:
    sudo pacman -S rabbitmq

    # of course, start it, if you are using Arch Linux:
    sudo systemctl start rabbitmq.service

    # optionally, you may want to enable it:
    sudo systemctl enable rabbitmq.service

    # finally, enable Web STOMP:
    sudo rabbitmq-plugins enable rabbitmq_web_stomp

Finally, send text to a RabbitMQ queue:

- If you are using Neovim, there is a plugin you can use, [vim-senter](https://github.com/qwfy/vim-senter)
- If you are using this for other purposes, you can just send a message to the queue, (see Internals).


## Internals

When the notebook is loaded in the browser, this plugin will try to connect to a RabbitMQ server located at `localhost` (currently changeable only by editing code), and subscribe to a queue, whose name is derived from current notebook's name, (see Queue Name Deriving), and act according to the message received, (see Message Format).


### Queue Name Deriving

Substitute the `/` in the relative file name with `;`, you get the queue name.

For example, if the current directory is `/home/jack/notebooks`, and the notebook is `/home/jack/notebooks/a/b/c.ipynb`, the queue name would be `a;b;c.ipynb`.

Note, this may cause name clashes.


### Message Format

Message is a JSON string, and has at least the key `command`, telling the plugin what to do. (note, if you plan to extend this, you might want to consider other message format, like Protocol Buffer 2 or alike, since it has statically defined definitions, so you won't get a mismatch between the sender and receiver).

Currently this plugin has only one command, (hence the choice of JSON), it is:

    { "command": "insert_code_at_bottom_and_execute"
    , "data": <code to be executed, data type is string>
    }
