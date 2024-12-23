<div align="center">
<img src="https://github.com/oysandvik94/curl.nvim/assets/25078429/65ad4dd4-cb7a-4ef9-a503-ff6693129efb" data-canonical-src="https://github.com/oysandvik94/curl.nvim/assets/25078429/65ad4dd4-cb7a-4ef9-a503-ff6693129efb" width="300" height="300" />
  
# curl.nvim
  
💪 Integrate curl and jq in Neovim. 💪

</div>

https://github.com/oysandvik94/curl.nvim/assets/25078429/9c25d289-c293-41c4-9d8d-40a0e8b013ed

curl.nvim allows you to run HTTP requests with curl from a scratchpad, and display the formatted output

- Introduces the ".curl" filetype, where pressing enter will execute a curl request under the cursor
- Quality of life formatting features, so that writing out curl commands is a _little_ less tedious
- Output is formatted using JQ
- Open a curl command buffer that is either persisted globally or per working directory
- Store and retrieve collections (named files) that are either persisted globally or per working directory
- It's just curl, so all the headers and auth flags you already know works

See [the features section](<README#✨ Features>) for more information.

The plugin aims to be 100% compatible with curl; if a curl command can execute in your shell,
you will be able to paste it in to the scratch buffer and run it.
Because of this, the plugin attempts to get the balance of being ergonomic and convenient, while
still using the knowledge of curl you already have.

## Installation and requirements

- [Curl](https://curl.se)
- [jq](https://jqlang.github.io/jq/),
- Linux/Mac (I dont have a windows machine to test, feel free to create a PR)

Installation example for [Lazy](https://github.com/folke/lazy.nvim):

```lua
{
  "oysandvik94/curl.nvim",
  cmd = { "CurlOpen" },
  dependencies = {
  "nvim-lua/plenary.nvim",
},
config = true,
}
```

To get started quickly, check out some commands under to get you going.
For more detailed documentation, see examples under [Features](<README#✨ Features>)!

```vim
" A buffer that is scoped to the current working directory
:CurlOpen

" A global buffer that will be the same for all Neovim instances
:CurlOpen global

" Create or open a new collection with the given scope
:CurlOpen collection global {any_name}
:CurlOpen collection scoped {any_name}

" Open a picker to select a collection
:CurlCollection global
:CurlCollection scoped
```

These commands will open the curl.nvim tab. In the left buffer, you can paste or write curl
commands, and by pressing Enter, the command will execute, and the output will be shown and
formatted in the rightmost buffer.

If you wish, you can select the text in the right buffer, and filter it using jq, i.e.
`ggVG! jq '{query goes here}'`

Below follows some example keymaps, (see the [API docs](<README#Lua api>) for possibilites)

```lua
local curl = require("curl")
curl.setup({})

vim.keymap.set("n", "<leader>cc", function()
    curl.open_curl_tab()
end, { desc = "Open a curl tab scoped to the current working directory" })

vim.keymap.set("n", "<leader>co", function()
    curl.open_global_tab()
end, { desc = "Open a curl tab with gloabl scope" })

-- These commands will prompt you for a name for your collection
vim.keymap.set("n", "<leader>csc", function()
      curl.create_scoped_collection()
end, { desc = "Create or open a collection with a name from user input" })

vim.keymap.set("n", "<leader>cgc", function()
      curl.create_global_collection()
end, { desc = "Create or open a global collection with a name from user input" })

vim.keymap.set("n", "<leader>fsc", function()
      curl.pick_scoped_collection()
end, { desc = "Choose a scoped collection and open it" })

vim.keymap.set("n", "<leader>fgc", function()
      curl.pick_global_collection()
end, { desc = "Choose a global collection and open it" })
```

To verify the installation run `:checkhealth curl`.

## Configuration

You can configure curl.nvim by running the `curl.setup()` function, passing a table as the argument.

Or if you use [Lazy](https://github.com/folke/lazy.nvim), just pass the table into `opts` as described [here](https://lazy.folke.io/spec#spec-setup).

<details>
<summary>Default Config</summary>

```lua
{
  -- Table of strings to specify default headers to be included in each request, i.e. "-i"
  default_flags = {},
  -- Specify an alternative curl binary that will be used to run curl commands
  -- String of either full path, or binary in path
  curl_binary = nil,
  -- Specify how to open curl
  -- use "tab" to open in separate tab
  -- use "split" to open in horizontal split
  -- use "vsplit" to open in vertical split
  open_with = "tab"
  mappings = {
    execute_curl = "<CR>"
  }
}
```

</details>

## ✨ Features

### 💪 .curl filetype

Opening any file with the ".curl" file extension will activate this plugins features.
You will get some syntax highlighting and the ability to execute curl commands from you buffer.
Since any ".curl" file will work, you can manage your own collection instead of using the builtin
system, and even check in files to your repository.

### 💪 Formatting

#### No quotes needed

JSON bodies do not have to be wrapped in quotes, making it easier to format JSON with JQ (va{:!jq)

<details>
<summary>See example</summary>

```bash
curl -X POST https://jsonplaceholder.typicode.com/posts
-H 'Content-Type: application/json'
-d
{
  "title": "now try this"
}
```

</details>

#### No trailing \\

You dont need a trailing \\, but it won't matter if they are there, making it easier to copy-paste requests

<details>
<summary>See example</summary>

```bash
curl -X POST https://jsonplaceholder.typicode.com/posts \
-H 'Content-Type: application/json' \
-d '{"title": "now try this"}'
```

</details>

#### Comment out lines

Headers and/or parts of the body can be commented out using '#', making ad-hoc experimenting with
requests easier

<details>
<summary>See example</summary>

```bash
curl -X POST https://jsonplaceholder.typicode.com/posts
-H 'Content-Type: application/json'
-d
{
  # "title": "remember me"
  "title": "now try this"
}
```

</details>

### 💪 Headers

Basic auth and bearer tokens work, and can be retrieved from environment variables

> [!CAUTION]  
> The command scratch buffer is stored in plaintext in your Neovim data directory, be careful when using literal secrets!

<details>
<summary>See example</summary>

```bash
curl -u "username:password" http://httpbin.org/basic-auth/username/password

curl -u "username:$PASSWORD_TEST" http://httpbin.org/basic-auth/username/mypassword

curl -X GET "https://httpbin.org/bearer" -H "accept: application/json" -H "Authorization: Bearer myrandomtoken"

curl -X GET "https://httpbin.org/bearer" -H "accept: application/json" -H "Authorization: Bearer $TOKEN_TEST"
```

</details>

### 💪 Collections

There are multiple ways to work with the scratch buffers, so you can tailor it to your own workflow.
By default, running ":CurlOpen" will open a command buffer that is tied to your current working directory.
If you treat directories as "projects", and always open neovim in the root of your project directories,
then this option might be useful for you.

Using ":CurlOpen global" you can open a global buffer that will always be the same across your
Neovim instances.

If you want more control, or would like to organize your curl commands in logical collections,
you can use the "collection" functionality to create named command buffers.

These are also scoped either globally or per working directory.

```vim
" Create or open a collection
:CurlOpen collection global mycoolcurls
:CurlOpen collection scoped mycoolcurls

" Choose a collection from a picker
:CurlCollection scoped
:CurlCollection global
```

```lua
require("curl").open_global_collection("mycoolcurls")
require("curl").open_scoped_collection("mycoolcurls")

-- This will prompt you for a name instead
require("curl").create_global_collection()

-- Choose a collection from a picker
require("curl").pick_scoped_collection()
```

The pickers are based on vim.ui.select, which means it will use whatever frontend you have configured.
This might be the neovim default picker, or telescope/fzf-lua if configured. See for example
[dressing.nvim](https://github.com/stevearc/dressing.nvim) or [this telescope extension](https://github.com/nvim-telescope/telescope-ui-select.nvim/tree/master) for ways of configuring this.

In the future, I (or someone else) might create a dedicated telescope/fzf picker to get features
like preview enabled.

## Lua api

This section describes all the methods in the exposed lua api

<details>
<summary>See lua api</summary>

```lua
local curl = require('curl')

-- These functions will open the curl tab with the given command buffer
-- If the curl tab is open, it will replace the existing command buffer with the selected on, and
-- go to the tab

-- Command buffer scoped to the current working directory
curl.open_curl_tab()
-- Globally scoped command buffer available in all Neovim instances
curl.open_global_tab()

-- Close the tab containing curl buffers
curl.close_curl_tab()

-- Executes the curl command under the cursor when the command buffer is open
-- Also executed by the "execute_curl" mapping, as seen in the configuration. Mapped to <CR> by default
curl.execute_curl()

--------------------

-- The below functions are related to collections

-- Will either open or create a collection with the given name as input
curl.open_scoped_collection({name})
curl.open_global_collection({name})

-- Same as get_global_colletion(), but does not take
-- input and promps the user for a name with vim.ui.input
curl.create_global_collection()
curl.create_scoped_collection()

-- Return a list of collections in a table
-- The name is the given name, not the filename (".curl" extension is omitted)
curl.get_global_collections()
curl.get_scoped_collections()

-- Opens a picker using vim.ui.select() that will open the
-- given collection when selected
curl.pick_global_collection()
curl.pick_scoped_collection()

-- Specify an alternative curl binary that will be used to run curl commands
-- String of either full path, or binary in path
curl.set_curl_binary("someothercurl")
```

</details>

## Troubleshooting

### DAP

If you are debugging using [nvim-dap](https://github.com/mfussenegger/nvim-dap), then you might
be in a workflow where you have a debugging session active, and your curl request triggers a
breakpoint. This will cause the curl buffer to change to the buffer containing the breakpoint.

This can be disorienting, but can be relieved by using the `:h switchbuf` option.

```lua
vim.opt.switchbuf = "usetab"
```

"usetab" will instead use an open window containing a buffer with the breakpoint, instead of
messing up your curl window.

See `:h switchbuf` for other values that might also fit your workflow.

## Future plans

Interesting features that might arrive soon:

- Format JSON under the cursor in the scratch window with a single keybind
- Be able to do simple jq queries in the output window. For example: while the cursor is
  on a key in the json, execute a keybind to filter the entire json for that key
- Enhance organization, by maybe folds, creating a picker for commands in the scratch,
  or multiple named scratches

## Similar plugins

- [kulala.nvim](https://github.com/mistweaverco/kulala.nvim) using HTTP file syntax instead.
  This is similar to Jetbrains HTTP client and vscode rest-client.
- [rest.nvim](https://github.com/rest-nvim/rest.nvim) using HTTP file syntax instead.
  This is similar to Jetbrains HTTP client and vscode rest-client.

## Contributing

Would you like to contribute? Noice, read [CONTRIBUTING.md](CONTRIBUTING.md)!
