# Hyprland Plugin Template
# WARNING: Due to changes in the way Hyprland distributes headers, this is broken right now.

The goal of this repository is to create a robust `Hyprland` plugin template, with
- A working, extensible `Makefile`
- [`hyprload`](https://github.com/Duckonaut/hyprload) support out of the box
- Environment set up guide
- Clangd flags set up for autocomplete and error checking

It is highly recommended to read the [Plugin development](https://wiki.hyprland.org/Plugins/Development/Getting-Started/)
section of the Hyprland Wiki first. Some stuff will be different in this template, but it gives
you a general idea about what's going on

## Support
If you have any issues setting this up, open an issue in this repository. I will try to help.

## Setup
This is a github template repository. To use it, use the green **Use this template** button
at the top of the repository file view.

### Setting up a development environment
#### Text editor and autocompletion
For a code editor, I recommend VS Code or Neovim, but anything that can use Clangd will work
If you use Clangd, `make clangd` will generate a simple `compile_flags.txt` file with the proper
include paths and flags, which will make Clangd recognize the includes etc.
Alternatively, use [bear](https://github.com/rizsotto/Bear) to generate `compile_commands.json`
which is a bit more granular.

> **Warning**: Compiling the plugin should still be done using GCC 12+. Clang does not properly
> build Hyprland, and is very fussy about the hook system. You will most likely encounter errors
> like `cannot cast from type 'void (CCompositor::*)(CWindow *, wlr_surface *)' to pointer type 'void *'`
> This won't happen when building with GCC and can be ignored.

#### HYPRLAND_HEADERS
The most important part of setting up plugin builds is the `HYPRLAND_HEADERS` variable.
Plugins can hook directly into Hyprland's C++ code, which is what makes them so powerful.
Because of that, they need to be able to *see* the Hyprland source. `HYPRLAND_HEADERS` ensures
that.

When building your own plugins for testing, you will need to manually define it using
`export HYPRLAND_HEADERS=(PATH_TO_HYPRLAND_SOURCE_ROOT)` before running `make` commands. You
can use a local path if you keep `Hyprland` source anyway, but I'd definitely recommend using
`hyprload`. If you use your local source different from the `hyprload` one, make sure to
run `make pluginenv` in the Hyprland folder.

#### Hyprload, and why it's useful for plugin development
If you use `hyprload`, handing `HYPRLOAD_HEADERS` becomes a bit easier and more reliable.
By design it keeps a copy of Hyprland source code up to date with the Hyprland version you're
running in `$HOME/.local/share/hyprload/hyprland`, and you can use that as your
`HYPRLAND_HEADERS` path.

When users install your plugin via `hyprload`, it will automatically define `HYPRLAND_HEADERS`
to that path to ensure maximum compatibility.

When developing plugins and frequently changing them, the `make install` command will
automatically place your plugin build in the directory `hyprload` automatically loads. You can
reload plugins when testing using the `hyprload reload` dispatcher (bind it in your
`hyprland.conf`, or execute via `hyprctl dispatch hyprload reload`)

#### Making it Your Own
To change your plugin name, version, and author (that's you!) there are 3 variables that need
changing (I would like to streamline it somehow, but it's manageable for now)
- `main.cpp`: The `PLUGIN_INIT` function returns a struct with the plugin name, description,
  author and version. Change those.
- `Makefile`: At the top of the file, the variable `PLUGIN_NAME` contains the name of the plugin
  `.so` that will be built. This should generally match your plugin name.
- `hyprload.toml`: The `[examplePlugin]` and `[examplePlugin.build]` should be changed to match
  the name of your plugin. `hyprload` will look at these dictionaries for info about the plugin.
  For more info, see [hyprload docs](https://github.com/Duckonaut/hyprload#format)

## Building and testing
After making sure you have defined `HYPRLAND_HEADERS` (you might need to do this *every time
you open a new terminal* if you don't put it in your `.bashrc` or `.zshrc` or whatever), the
steps to build are simple

### Manually
- `make`: This will build the `PLUGIN_NAME.so` file.
- `hyprctl plugin unload $PWD/PLUGIN_NAME.so`: If you have an old version loaded, unload it
- `hyprctl plugin load $PWD/PLUGIN_NAME.so`: Load the plugin

Do note that if you only load/unload from the same path, Hyprland can ignore your changes.

### Using `hyprload`
This works rather well in nested Hyprland sessions, since `hyprload` keeps sessions separate.
- `make install`: This will build and copy the plugin to the `hyprload` plugin directory.
- Reload `hyprload` for the changes to take effect

This doesn't have the issue of ignoring changes, because of how `hyprload` handles its loaded
plugins.

### Nested Hyprland
Developing a plugin may be tough. You might crash Hyprland a couple times. For this reason,
it's a good idea to develop them in a nested Hyprland session. If you run `Hyprland` from an
existing Hyprland session, it'll open in a window. If this window crashes, it's pretty much fine!
Refer to the [Hyprland wiki](http://wiki.hyprland.org/Plugins/Development/Getting-Started/#setting-up-a-development-environment)
for more info.

## ""Publishing""
If you haven't messed up your `hyprload.toml` manifest too badly, anyone should be able to use 
your plugin by just adding `'YOUR_GITHUB_NAME/YOUR_PLUGIN'` to their own `hyprload.toml` config!
