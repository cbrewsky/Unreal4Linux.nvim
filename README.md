# Unreal4Linux.nvim
Unreal Engine support for Neovim on Linux. A Linux port of [zadirion/Unreal.nvim](https://github.com/zadirion/Unreal.nvim).
![image](https://raw.githubusercontent.com/zadirion/Unreal.nvim/main/image.png)

**Requirements**

- Linux, with Unreal Engine built from source (or an installed build) for Linux. The plugin calls `Engine/Build/BatchFiles/Linux/Build.sh`, which uses the engine's bundled clang toolchain and dotnet runtime
- `clangd` installed and on your `PATH` (e.g. from your distro's package manager or Mason)
- written against Unreal Engine 5.x. Other versions are untested
- (optional) If you don't already have your own configuration, I recommend you use my neovim configuration specialized for development in Unreal Engine https://github.com/zadirion/UnrealHero.nvim

**Installation**

Install with lazy.nvim:
```lua
  {
    'cbrewsky/Unreal4Linux.nvim',
    dependencies = { 'tpope/vim-dispatch' },
  }
```
Or with packer:
```lua
  use {'cbrewsky/Unreal4Linux.nvim',
    requires =
    {
        {"tpope/vim-dispatch"}
    }
  }
```
After installing, open one of your Unreal project's source files, and run `UnrealGenWithEngine`. This will go through all the engine source files and will generate a compatible clang compile-command for each, so that the lsp can properly parse them.
It will take a long time to go through all of them, but you only need to run this command once, for your engine.
After running it for the first time, it will open a configuration file in a new buffer. In this buffer set the value of the `"EngineDir"` key to the path to Unreal Engine on your system. For example,

```jsonc
// UnrealNvim.json
{
  "version": "0.0.2",
  "EngineDir": "/home/you/UnrealEngine/",
  "Targets": [
    // ...
  ]
}
```

After doing that and saving the file, run `:UnrealGenWithEngine` again.

From here onwards, you can use `:UnrealGen` to generate the compile commands for just your project files. Feel free to do so every time you feel like the lsp is not picking up your symbols, such as when you added new source code files to your project or if you updated to latest changelist/commig in your version control. 
`:UnrealGen` will always ask you which target to generate compile_commands.json for. Just input the number corresponding to the desired configuration, and it will generate the json right next to the uproject

This should cause your LSP to start recognizing the Unreal types, including the ones from .generated.h files.

**Commands**
- `:UnrealGenWithEngine` generates the compile_commands.json and the compiler rsp files for the engine source code, so your LSP can properly parse the source code
- `:UnrealGen` generates the compile_commands.json and the compiler rps files for your project, so your LSP can properly parse the source code
- `:UnrealBuild` builds the project with unreal
- `:UnrealRun` runs the project. It does not build it even if the source is out of date
- `:UnrealCD` sets the current directory to the root folder of the unreal project (the one with the .uproject in it). I personally use this so Telescope only searches in the project directory, making it faster, especially for live_grep

**Known Limitations**
- you can't run with a debugger attached at the moment. If you need one, launch the editor/game under `gdb`/`lldb` (or a DAP client such as nvim-dap) yourself.
- this fork targets Linux only (`PlatformName` defaults to `Linux`). For Windows, use the upstream [Unreal.nvim](https://github.com/zadirion/Unreal.nvim)
- you can only abort a build using `:AbortDispatch` and it will only work for the actual unreal build step, it won't work for the RSP generation build step

**Troubleshooting**
- if you notice that some of the symbols in your project are not recognize/found, it is possible clangd's index cache is broken somehow. You can find clang's cache in the .cache directory that will sit next to your .uproject. It is full of .idx files, each corresponding to a source code file in your project. Close nvim, delete the .cache directory, reopen vim, navigate to one of your project's files. It should trigger clangd to rebuild the index.
- Unreal4Linux.nvim's log is written to `~/.local/share/nvim/unrealnvim.log` (`stdpath("data")`), but you need to enable logging first in nvim by setting `vim.g.unrealnvim_debug = true`. clangd's LSP log can be found at `~/.local/state/nvim/lsp.log` (`:LspLog`). If you are unsure whether clangd 'sees' some of your code, looking at this log helps
- precompiled headers are stripped from the generated clang response files (`*.clang.rsp`), since clangd can't load PCHs built by Unreal's bundled clang version
