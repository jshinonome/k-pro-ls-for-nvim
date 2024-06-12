### setup k-pro language server for neovim

0. save vscode-k-pro key to the path `~/.config/k-pro/k-pro`

1. install k-pro-ls, download the file from [here](https://github.com/jshinonome/k-pro-ls-for-nvim/releases/tag/1.2.0)

```
npm --global i ./jshinonome-k-pro-ls-1.2.0.tgz
```

2. install [vim-q-syntax](https://github.com/jshinonome/vim-q-syntax)

3. use lazy plugin manager and add the following configuration to `~/.config/nvim/init.lua`

```
require('lazy').setup({
  'neovim/nvim-lspconfig',
  "hrsh7th/cmp-nvim-lsp",
  "hrsh7th/nvim-cmp",
  'hrsh7th/cmp-vsnip',
  'hrsh7th/vim-vsnip',
})


local cmp = require 'cmp'
cmp.setup({
  sources = cmp.config.sources({
    { name = 'nvim_lsp' },
    { name = 'vsnip' },
    { name = 'buffer' },
  }),
  window = {
    completion = cmp.config.window.bordered(),
  },
  mapping = cmp.mapping.preset.insert({
    ['<C-b>'] = cmp.mapping.scroll_docs(-4),
    ['<C-f>'] = cmp.mapping.scroll_docs(4),
    ['<C-Space>'] = cmp.mapping.complete(),
    ['<C-e>'] = cmp.mapping.abort(),
    ['<CR>'] = cmp.mapping.confirm({ select = true }),
  }),
})

vim.api.nvim_create_autocmd('FileType', {
  pattern = 'q,k',
  callback = function()
    vim.lsp.start({
      name = 'q language server',
      cmd = { 'k-pro-ls', '--stdio' },
      filetypes = { 'q', 'k' },
      root_dir = vim.fs.dirname(vim.fs.find({ 'src' }, { upward = true })[1]),
    })
  end,
})

local cfg = {
  extra_trigger_chars = { "[", ";" }
}
require 'lsp_signature'.setup(cfg)


vim.api.nvim_create_autocmd('LspAttach', {
  group = vim.api.nvim_create_augroup('UserLspConfig', {}),
  callback = function(ev)
    -- Enable completion triggered by <c-x><c-o>
    vim.bo[ev.buf].omnifunc = 'v:lua.vim.lsp.omnifunc'

    -- Buffer local mappings.
    -- See `:help vim.lsp.*` for documentation on any of the below functions
    local opts = { buffer = ev.buf }
    vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, opts)
    vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
    vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)
    vim.keymap.set('n', 'gi', vim.lsp.buf.implementation, opts)
    vim.keymap.set('n', '<C-k>', vim.lsp.buf.signature_help, opts)
    vim.keymap.set('n', '<space>wa', vim.lsp.buf.add_workspace_folder, opts)
    vim.keymap.set('n', '<space>wr', vim.lsp.buf.remove_workspace_folder, opts)
    vim.keymap.set('n', '<space>wl', function()
      print(vim.inspect(vim.lsp.buf.list_workspace_folders()))
    end, opts)
    vim.keymap.set('n', '<space>D', vim.lsp.buf.type_definition, opts)
    vim.keymap.set('n', '<space>rn', vim.lsp.buf.rename, opts)
    vim.keymap.set({ 'n', 'v' }, '<space>ca', vim.lsp.buf.code_action, opts)
    vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)
    vim.keymap.set('n', '<space>f', function()
      vim.lsp.buf.format { async = true }
    end, opts)
  end,
})

```
