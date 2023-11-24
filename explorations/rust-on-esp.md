# Running Rust on esp32

[Main Page](../README.md)

This example shows how to run some demo programs on esp32 using the library wrapping
around the esp-idf functions
[here](https://github.com/ivmarkov/rust-esp32-std-demo/tree/main)

The esp rust [book](https://esp-rs.github.io/book/introduction.html) gives an overview of what is currently possible.

Issues

- rust-analyzer in neovim seems to use the target toolchain as opposed to the default
  system one, because of this, the rust LSP fails to start, as esp toolchain is
  a custom toolchain which doesn't support components. A possible workaround
  is to symlink the rust analyzer of the target toolchain to the host toolchain.
  ```
  ln -sf ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/bin/rust-analyzer ~/.rustup/toolchains/esp/bin/rust-analyzer

  ```
  - tried that, unfortunately it didn't work.
  - It seems like there are some issues with using the custom toolchain when it
    comes to using tools like rust-analyzer

- I tried flashing the board using the rust tools provided and it seems a bit
  unreliable. Although the idf.py tool provided by esp-idf flashes just fine, the
  espflash which rust provides fails to interrupt the device and says
  'device or resource busy'.

- The main limitation of running rust on esp32 is the limited support for the esp
  toolchain, the compilation works fine, however the code analysis is a bit lacking.

- Apparently the vscode extension [works](https://rust-analyzer.github.io/manual.html#toolchain) if configured properly
- Someone managed to get it to work partially on Neovim [here](https://github.com/rust-lang/rust-analyzer/issues/15828)


## How to cofigure rust-analyzer on vscode:

1. Download the following extensions:
   - rust
   - rust-analyzer
   - Rust Targets

2. Set the following configuration in the settings.json file
```
 "rust-analyzer.server.extraEnv": { "RUSTUP_TOOLCHAIN": "stable" },
  "rust-analyzer.checkOnSave.allTargets": false,
  "rust-targets.targets": [
    "system",
    "x86_64-pc-windows-gnu",
    "x86_64-apple-darwin",
    "wasm32-unknown-unknown",
    "x86_64-unknown-linux-gnu"
  ],
  "rust-analyzer.cargo.target": "x86_64-unknown-linux-gnu"
```

The above assumes that you are using the x86_64 linux toolchain for rust.
The above configuration instructs rust-analyzer to use the stable toolchain to
analyse the project. Some features such as proc macro expansion don't work, however
I was able to get jump to def and autocompletions to work in vscode.

## How to configure rust-analyzer on neovim

The below steps were tested on my machine and so far the go-to-definition and
documentation on hower functionalities work fine.

1. Use your package manager to get the simrat39/rust-tools.nvim plugin
2. Use the following configuration:

```
rust_tools.setup({
  server = {
    on_attach = function(_, bufnr)
      -- enable other keybindings
      on_attach()
      -- Hover actions
      vim.keymap.set('n', '<Leader>h', rust_tools.hover_actions.hover_actions, { buffer = bufnr })
      -- Code action groups
      vim.keymap.set('n', '<Leader>a', rust_tools.code_action_group.code_action_group, { buffer = bufnr })
    end,
    --Trying to set it so that the rust analyzer works on esp32 targets.
    imports = {
      granularity = {
        group = "module",
      },
      prefix = "self",
    },
    cargo = {
      buildScripts = {
        enable = true,
      },
      target = { "x86_64-unknown-linux-gnu" }, -- This line is important - we want to run rust analyzer with the default system toolchain.
    },
    procMacro = {
      enable = true
    },
  },
})

```
3. Figure out the location in your system where rustup stores all toolchains, for
   me it was under: `~/.local/share/rustup/toolchains`

4. Create the following symlink:
```
ln -sf ~/.local/share/rustup/toolchains/stable-x86_64-unknown-linux-gnu/bin/rust-analyzer ~/.local/share/rustup/toolchains/esp/bin/rust-analyzer                                                                                         ✔
```
Adapting your path appropriately so that the esp toolchain rust-analyzer points
to the one from the stable toolchain.

5. After that the lsp should work without any issues.



