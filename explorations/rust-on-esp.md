# Running Rust on esp32

[Main Page](../README.md)

Table of Contents
1. [Sources](./rust-on-esp.md#sources)
1. [Seting Up](./rust-on-esp.md#setting-up)
1. [Issues](./rust-on-esp.md#issues)
2. [Configuring rust-analyzer for neovim](./rust-on-esp.md#configuring-rust-analyzer-for-neovim)
3. [Configuring rust-analyzer for vscode](./rust-on-esp.md#configuring-rust-analyzer-for-vscode)

## Sources

-  The esp rust [book](https://esp-rs.github.io/book/introduction.html)
   - gives a great overview of the platform and supported features.
- esp32 demo programs [here](https://github.com/ivmarkov/rust-esp32-std-demo/tree/main)
   - This example shows how to run some demo programs on esp32 using the library wrapping
    around the esp-idf functions. It looks like most of the important parts of the esp-idf
    API are exposed by the rust library


## Setting Up
I have created an example repo containing a cut down version of rbpf. It was
possible to flash it onto an esp32 and without issues. You can find it [here](https://github.com/SzymonKubica/rbpf-on-esp-idf)

Prerequisites:
- See [here](https://github.com/esp-rs/esp-idf-template#prerequisites) for the
  list of required things to use the cargo template for esp-idf.
- First, you need to install esp-idf as explained [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html#step-1-install-prerequisites)
- Second, follow the instructions [here](https://esp-rs.github.io/book/installation/riscv-and-xtensa.html) to install espup to setup the dev environment for the Xtensa targets.

Once you have everythign correctly installed, you should be able to clone the above
repo, build it and flash onto the esp32. To flash and monitor the output of the system, use the following command:
```
cargo espflash flash --monitor
```
It should correctly figure out which port it is supposed to target.
Once the code is loaded you can attach to it using
```
espflash monitor
```


## Issues

- [FIXED] (see configuration instructions below) rust-analyzer in neovim uses
  the target toolchain as opposed to the default system one, because of this,
  the rust LSP fails to start, as esp toolchain is a custom toolchain which
  doesn't support components. A possible workaround is to symlink the rust
  analyzer of the target toolchain to the host toolchain.
  ```
  ln -sf ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/bin/rust-analyzer ~/.rustup/toolchains/esp/bin/rust-analyzer

  ```
- [FIXED] I tried flashing the board using the rust tools provided and it seems a bit
  unreliable. Although the idf.py tool provided by esp-idf flashes just fine, the
  espflash which rust provides fails to interrupt the device and says
  'device or resource busy'.

- Apparently the vscode extension [works](https://rust-analyzer.github.io/manual.html#toolchain) if configured properly
- Someone managed to get it to work partially on Neovim [here](https://github.com/rust-lang/rust-analyzer/issues/15828)


## Configuring rust-analyzer for vscode

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

## Configuring rust-analyzer for neovim

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

The above steps were tested on my machine and so far the go-to-definition and
documentation on hower functionalities work fine. There are some limitations
with the expansion proc macros (see
[here](https://www.reddit.com/r/rust/comments/13d2tls/rustanalyzer_and_toolchain_for_esp/)).
I'm not sure if that is going to be a blocker moving forward.



