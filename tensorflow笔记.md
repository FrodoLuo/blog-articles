### VSCode 中对 TensorFlow v2 的 intellisence 的问题

Step by step solution:

Find your tensorflow_core installation (the python package). This is e.g. in ~/.local/lib/python3.6/site-packages, any other site-packages folder you might use (e.g. from virtualenv, pyenv,...)
Create a folder to use for IDE navigation, e.g. ~/.local/virtual-site-packages
Create a symlink in that folder called tensorflow to your tensorflow_core package (mind the name difference, this is intentional!)
Add the path created in the 2nd step to python.autoComplete.extraPaths in VSC (use the full path, i.e. replace your username)
Example for me using pyenv with Python 3.6.9:

```shell
mkdir ~/.local/virtual-site-packages
ln -s ~/.pyenv/versions/3.6.9/lib/python3.6/site-packages/tensorflow_core ~/.local/virtual-site-packages/tensorflow
```

Then use

```json
 "python.autoComplete.extraPaths": [
        "/home/<USER>/.local/virtual-site-packages"
    ],
```

> 摘自 https://github.com/tensorflow/tensorflow/issues/32982#issuecomment-545414061
