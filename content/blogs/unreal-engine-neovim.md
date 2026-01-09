---
title: Unreal Engine and Neovim Workflow
slug: neovim-and-unreal-engine-workflow
description: A brief guide covering the essentials of HTML.
longDescription: Neovim for Unreal Engine C++ development. Lightweight, portable setup with code completion, debugging, and real-time build integration.
cardImage: "https://zaggonaut.dev/michael-dam-unsplash.webp"
tags: ["unral-engine", "neovim", "c++"]
readTime: 9
featured: true
timestamp: 2025-10-20T02:39:03+00:00
---

## Premise
**This guide is intended for Linux and macOS users.
I haven’t tested this workflow on Windows, so its compatibility is not guaranteed.**

---

## Why am I writing this tutorial?
I work on a large Unreal Engine project and have been developing in Rider for a while. It worked fine, but last year I decided to learn Vim and installed its plugin in Rider. 
The experience was hard and incredible at the same time. Now using Vim felt both fun and challenging!

However, I soon ran into two issues:

Rider was becoming increasingly heavy and sluggish for my Unreal Engine workflow.

I work across multiple PCs, programming languages, and game engines, and I wanted a single setup that works everywhere.

That’s when Neovim entered the stage.

---

## Goal
With this tutorial, I want to help you achieve a functional setup and workflow with Neovim and Unreal Engine, the simplest way possible. 
So even if you are a beginner (like me) using Neovim, this is also for you.

---

## Prerequisites
Before setting up Neovim, we need to prepare some other things.

### Install Python
[https://www.python.org/](https://www.python.org/)

### Create a Python Virtual Environment
After you have installed Python, launch this command in your choosen directory.
`python -m venv /path/to/new/virtual/environment`

> 💡_Tip: Set the activate cmd as an alias in your terminal configuration, so that you can call it from everywhere in your system._
> E.g.: 
> `alias pythonenv="source /home/thomas/Documents/python-venv/bin/activate"`

[Documentation Link
](https://docs.python.org/3/library/venv.html#creating-virtual-environments)

### Install ue4cli
This will be very important for our workflow.
It is a command-line tool which provides a simplified interface to various functionality of the build system for Unreal Engine.

- [ue4cli](https://docs.adamrehn.com/ue4cli/overview/introduction-to-ue4cli)

After you install it, be sure to set the engine path using 
`ue4 setroot <your_unreal_engine_path>`

---

## Installing Neovim
There are a lot of ways to configure Neovim; you could spend hours digging through different setups, dotfiles, and YouTube tutorials, threads. But let's keep it simple.

My personal choice is [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim).

This is a good starting point for your configuration. It provides a clean, modern Neovim setup right out of the box, and it is very well-documented.

Next steps:
- Head over to the Kickstart repo.
- Follow the installation steps, take your time with it.
- Once it’s installed, open Neovim for the first time.

Done? Awesome. You now have a working Neovim setup.

---

## Unreal Engine Additional Configuration

### Add Clangd

In our Neovim LSP configuration, we set up clangd with some extra flags:

```diff
      local servers = {
+        clangd = { 'clangd', '--background-index', '--clang-tidy' },
        -- gopls = {},
        -- pyright = {},
        -- rust_analyzer = {},
        -- ... etc. See `:help lspconfig-all` for a list of all the pre-configured LSPs
        --
        -- Some languages (like typescript) have entire language plugins that can be useful:
        --    https://github.com/pmizio/typescript-tools.nvim
        --
        -- But for many setups, the LSP (`ts_ls`) will work just fine
        -- ts_ls = {},
        --

        lua_ls = {
          -- cmd = { ... },
          -- filetypes = { ... },
          -- capabilities = {},
          settings = {
            Lua = {
              completion = {
                callSnippet = 'Replace',
              },
              -- You can toggle below to ignore Lua_LS's noisy `missing-fields` warnings
              -- diagnostics = { disable = { 'missing-fields' } },
            },
          },
        },
      }

```
1. `clangd`
This is the C/C++ language server.

2. `--background-index`
This tells clangd to index your entire project in the background.

3. `--clang-tidy`
This enables clang-tidy integration, which is a tool for checking your code for code style issues, best-practice violations, and useful suggestions.

---

### Add a Keymap to Search Unreal’s Content Folder

Unreal Engine stores all game assets in the Content/ folder, which is often ignored due to .gitignore rules. 
This keymap lets you quickly search files directly in Content/, including hidden or ignored files.

```diff
      -- See `: help telescope.builtin`
      local builtin = require 'telescope.builtin'
      vim.keymap.set('n', '<leader>sh', builtin.help_tags, { desc = '[S]earch [H]elp' })
      vim.keymap.set('n', '<leader>sk', builtin.keymaps, { desc = '[S]earch [K]eymaps' })
      vim.keymap.set('n', '<leader>sf', builtin.find_files, { desc = '[S]earch [F]iles' })

+      -- Unreal Engine Specific Keymap --
+      ----------------------------------
+      vim.keymap.set('n', '<leader>su', function()
+        require('telescope.builtin').find_files {
+          cwd = 'Content',
+          hidden = true,
+          no_ignore = true,
+          prompt_title = 'Unreal Assets',
+        }
+      end, { desc = '[S]earch [U]nreal Content' })
+      ----------------------------------

      vim.keymap.set('n', '<leader>ss', builtin.builtin, { desc = '[S]earch [S]elect Telescope' })
      vim.keymap.set('n', '<leader>sw', builtin.grep_string, { desc = '[S]earch current [W]ord' })
      vim.keymap.set('n', '<leader>sg', builtin.live_grep, { desc = '[S]earch by [G]rep' })
      vim.keymap.set('n', '<leader>sd', builtin.diagnostics, { desc = '[S]earch [D]iagnostics' })
      vim.keymap.set('n', '<leader>sr', builtin.resume, { desc = '[S]earch [R]esume' })
      vim.keymap.set('n', '<leader>s.', builtin.oldfiles, { desc = '[S]earch Recent Files ("." for repeat)' })
      vim.keymap.set('n', '<leader><leader>', builtin.buffers, { desc = '[ ] Find existing buffers' })
```

---

### Add debug and custom plugins

Here, we enable the plugins we need for our configuration.
Feel free to enable other plugins if you want.

```diff
  -- NOTE: Next step on your Neovim journey: Add/Configure additional plugins for Kickstart
  --
  --  Here are some example plugins that I've included in the Kickstart repository.
  --  Uncomment any of the lines below to enable them (you will need to restart nvim).
  --
+  require 'kickstart.plugins.debug',
  -- require 'kickstart.plugins.indent_line',
  -- require 'kickstart.plugins.lint',
  -- require 'kickstart.plugins.autopairs',
  -- require 'kickstart.plugins.neo-tree',
  -- require 'kickstart.plugins.gitsigns', -- adds gitsigns recommend keymaps

  -- NOTE: The import below can automatically add your own plugins, configuration, etc from `lua/custom/plugins/*.lua`
  --    This is the easiest way to modularize your config.
  --
  --  Uncomment the following line and add your plugins to `lua/custom/plugins/*.lua` to get going.
+  { import = 'custom.plugins' },
```

---

### Add UnrealUtils Module

I made some utility functions to avoid hardcoded string paths. These will be used for our custom command to build our project directly inside Neovim.

Create the file `/.config/nvim/lua/custom/unreal-utils.lua`

```lua
local UnrealUtils = {}

function UnrealUtils.find_uproject_files()
  local cwd = vim.fn.getcwd()
  return vim.fn.globpath(cwd, '*.uproject', false, true)
end

function UnrealUtils.get_default_engine_path()
  return os.getenv 'UNREAL_ENGINE_PATH'
end

return UnrealUtils
```

`find_uproject_files()`
- This function searches through the current working directory (which will be your game project directory) to find a *.uproject file.

`get_default_engine_path()`
- This allows each machine to have its Unreal Engine path configured individually via an environment variable.

Now open your terminal configuration and add the environment variable.

Linux:
`export UNREAL_ENGINE_PATH=".../UnrealEngine/Engine/Binaries/Linux/UnrealEditor-Linux-DebugGame"`

MacOS:
`export UNREAL_ENGINE_PATH=".../UnrealEngine/Engine/Binaries/Mac/UnrealEditor-Mac-DebugGame.app/Contents/MacOS/UnrealEditor-Mac-DebugGame"`

---

### Setup the debug plugin
`/.config/nvim/lua/kickstart/plugins/debug.lua`

### Adding codelldb debugger

```diff
    require('mason-nvim-dap').setup {
      -- Makes a best effort to set up the various debuggers with
      -- reasonable debug configurations
      automatic_installation = true,

      -- You can provide additional configuration to the handlers,
      -- see mason-nvim-dap README for more information
      handlers = {},

      -- You'll need to check that you have the required things installed
      -- online, please don't ask me how to install them :)
      ensure_installed = {
        -- Update this to ensure that you have the debuggers for the langs you want
        'delve',
+        'codelldb',
      },
    }

```

Save, close and reopen Neovim, this ensures that your updated Lua config is loaded.

Run `:Mason`, then open the Mason UI and check that codelldb appears in the installed tools list.

### Dap C++ Configuration

Put it inside the `config` function:
```lua
config = function()
    local dap = require 'dap'
    local dapui = require 'dapui'
    -- Add the DAP configurations below 👇
```

```lua

    dap.adapters.codelldb = {
      type = 'server',
      port = '${port}',
      executable = {
        -- You can also use `vim.fn.stdpath("data") .. "/mason/bin/codelldb"` if installed via Mason
        command = vim.fn.stdpath 'data' .. '/mason/packages/codelldb/extension/adapter/codelldb',
        args = { '--port', '${port}' },
      },
    }


    local unreal_utils = require 'custom.unreal-utils'
    local uprojects = unreal_utils.find_uproject_files()

    dap.configurations.cpp = {
  {
    name = 'Dynamic Launch',
    type = 'codelldb',
    request = 'launch',
    cwd = '${workspaceFolder}',
    stopOnEntry = false,

    program = function()
      if vim.tbl_isempty(uprojects) then
        -- Fallback: ask for an executable path if no .uproject found
        return vim.fn.input('Path to executable: ', vim.fn.getcwd() .. '/', 'file')
      end
      return unreal_utils.get_default_engine_path()
    end,

    args = function()
      if vim.tbl_isempty(uprojects) then
        return {}
      end
      -- Launch unreal with the first found .uproject, and the provided args
      return { uprojects[1], '-log', '-debug' }
    end,
  },

```

### What it does:
Now the functions from UnrealUtils come in handy.
When you start a debug session with DAP, it first checks if an uproject file exists in the current directory: 
- If no .uproject is found, it prompts you to manually enter the path to a compiled C++ executable.
- If a .uproject is present, it automatically uses the default Unreal Engine executable (as returned by get_default_engine_path()) and passes the first .uproject file along with -log and -debug arguments to launch Unreal in debug mode.

---

### Unreal Engine Build Command for Neovim
`.config/nvim/lua/custom/plugins/init.lua`

This module creates a user command `:UEBuild` that:
- Opens a floating window showing the output of `ue4 build DebugGame`
- Highlights errors and warnings in real time
- Automatically closes the window on success or when pressing <Esc>
- Optionally continues debugging via DAP if build succeeds

```lua
-- You can add your own plugins here or in other files in this directory!
--  I promise not to create any merge conflicts in this directory :)
--
-- See the kickstart.nvim README for more information

local ns = vim.api.nvim_create_namespace 'BuildHighlights'

local function highlight_errors(buf)
  local lines = vim.api.nvim_buf_get_lines(buf, 0, -1, false)
  vim.api.nvim_buf_clear_namespace(buf, ns, 0, -1)

  for i, line in ipairs(lines) do
    if line:lower():find 'error:' then
      vim.api.nvim_buf_add_highlight(buf, ns, 'ErrorMsg', i - 1, 0, -1)
    elseif line:lower():find 'warning:' then
      vim.api.nvim_buf_add_highlight(buf, ns, 'WarningMsg', i - 1, 0, -1)
    end
  end
end

return {
  vim.api.nvim_create_user_command('UEBuild', function()
    local buf = vim.api.nvim_create_buf(false, true)
    vim.bo[buf].bufhidden = 'wipe'
    vim.bo[buf].filetype = 'log'

    local width = math.floor(vim.o.columns * 0.5)
    local height = math.floor(vim.o.lines * 0.5)
    local row = math.floor((vim.o.lines - height) / 2)
    local col = math.floor((vim.o.columns - width) / 2)

    local cwd = vim.fn.getcwd()
    local project_name = cwd:match '([^/]+)$'

    local win = vim.api.nvim_open_win(buf, true, {
      title = '*** Building ' .. project_name .. ' ***',
      title_pos = 'center',
      relative = 'editor',
      width = width,
      height = height,
      row = row,
      col = col,
      border = 'rounded',
      style = 'minimal',
    })

    local cmd = 'ue4 build DebugGame'
    vim.api.nvim_buf_set_lines(buf, 0, -1, false, { 'Running: ' .. cmd, '' })

    local function append(data)
      if not data then
        return
      end
      vim.api.nvim_buf_set_lines(buf, -1, -1, false, data)
      vim.api.nvim_win_set_cursor(win, { vim.api.nvim_buf_line_count(buf), 0 })
      highlight_errors(buf)
    end

    local job_finished = false

    vim.fn.jobstart({ 'bash', '-c', cmd }, {
      stdout_buffered = false,
      stderr_buffered = false,
      on_stdout = function(_, data)
        append(data)
      end,
      on_stderr = function(_, data)
        append(data)
      end,
      on_exit = function(_, code)
        append { '', '--- Process exited with code ' .. code .. ' ---' }
        job_finished = true
        if code == 0 then
          vim.notify('✅ Build succeeded: ', vim.log.levels.INFO)
          -- Close the window after 500ms
          vim.defer_fn(function()
            if vim.api.nvim_win_is_valid(win) then
              vim.api.nvim_win_close(win, true)
            end
            require('dap').continue()
          end, 500)
        else
          vim.notify('❌ Build failed (' .. code .. ')', vim.log.levels.ERROR)
        end
      end,
    })

    vim.keymap.set('n', '<Esc>', function()
      if job_finished and vim.api.nvim_win_is_valid(win) then
        vim.api.nvim_win_close(win, true)
      end
    end, { buffer = buf, nowait = true })
  end, {}),
}
```

>_Remember, you must be in a Python virtual environment to make this work._

---

## Last step, generate compile_commands.json

### Why do we need it?
`compile_commands.json` provides clangd with the full compilation context for your project, enabling code completion, navigation, and correct error reporting.

When we run the command `ue4 gen -vscode`, it already generates the JSON file for us, but it's not enough, and to work correctly we have to adjust it.

### Why should we modify the existing one?
We have to modify the existing `compile_commands.json` file because the one generated by Unreal or VSCode contains incomplete or incompatible compilation commands for clangd. By adjusting it and adding some compiler flags, standard, and ignoring unnecessary warnings, we ensure that clangd can correctly parse all files, understand macros and templates, provide accurate code completion, and highlight only real errors and warnings.

We can see below that by using the default generated JSON file, clangd shows several wrong errors.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gabb73rs4ja8j0n8uahm.png)

This is after we launch our script.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j526ir1h252eprxzvkxc.png)

We can see that clangd stops showing weird false errors, and the IntelliSense is working. 
Noice!

![noice_gif](https://media1.tenor.com/m/H6sjheSkU1wAAAAd/noice-nice.gif)

---

Now, if you didn't already do it, launch the command 
`ue4 gen -vscode` to generate the default one.

Then create the file: `generate_compile_commands.sh`, and launch it specifying the arguments.

```bash
#!/bin/bash

# Usage: ./generate_compile_commands.sh <folder_path> <project_name>

set -e

if [ $# -ne 2 ]; then
  echo "Usage: $0 <folder_path> <project_name>"
  exit 1
fi

folder="$1"
project="$2"

input="$folder/.vscode/compileCommands_${project}.json"
output="$folder/compile_commands.json"

if [ ! -d "$folder/.vscode" ]; then
  echo "Error: Folder '$folder/.vscode' not found."
  exit 1
fi

if [ ! -f "$input" ]; then
  echo "Error: File '$input' not found."
  exit 1
fi

cat "$input" | python3 -c '
import json, sys

j = json.load(sys.stdin)
for o in j:
    file = o["file"]
    arg = o["arguments"][1]
    o["arguments"] = [f"clang++ -std=c++20 -ferror-limit=0 -fdiagnostics-color=always "
                  "-Wall -Wextra -Wshadow-all "
                  "-Wno-unused-parameter -Wno-unused-private-field -Wno-missing-field-initializers "
                  "-Wno-inconsistent-missing-override -Wno-undefined-var-template -Wno-unused-variable "
                  "-Wno-sign-compare -Wno-expansion-to-defined -Wno-undef -Wno-deprecated-declarations -Wno-invalid-offsetof "
                  "-fno-ms-compatibility -fno-delayed-template-parsing -fmacro-backtrace-limit=0 "
                  f"{file} {arg}"]
print(json.dumps(j, indent=2))
' > "$output"

echo "Generated $output successfully."
```
## Conclusion

Awesome! 🎉🎉🎉

After running the script, you can finally open your project with `nvim .` and build it using `:UEBuild`, giving you a fully integrated and efficient workflow for developing Unreal Engine projects directly from Neovim.

---

## Additional plugins I use:
- neo-tree (you have to enable it in kickstart.nvim)
- catppuccin
- autosession
- barbar
- autopairs (you have to enable it in kickstart.nvim)
- gitsigns (you have to enable it in kickstart.nvim)

---

## Other useful resources:
[https://dev.to/taku25/unreal-engine-coding-with-neovim-plugins-revolutionize-your-ue-development-workflow-1mke](https://dev.to/taku25/unreal-engine-coding-with-neovim-plugins-revolutionize-your-ue-development-workflow-1mke)

[https://rodneylab.com/unreal-engine-with-neovim/](https://rodneylab.com/unreal-engine-with-neovim/)

[https://github.com/zadirion/Unreal.nvim](https://github.com/zadirion/Unreal.nvim)

[https://neunerdhausen.de/posts/unreal-engine-5-with-vim/](https://neunerdhausen.de/posts/unreal-engine-5-with-vim/)
