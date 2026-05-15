#meta

# Shortcut keys and Action buttons
```space-lua
command.update {
  name = "Navigate: Back in History",
  key = "Ctrl-Shift--"
}

command.update {
  name = "Navigate: Forward in History",
  key = "Ctrl-Shift-+"
}
```

```space-lua
actionButton.define {
  icon = "inbox",
  description = "Go to inbox",
  priority=2.9,
  run = function()
    local all_inbox = query[[
      from page = index.tag "page"
      where ( page.name.match("^Inbox") or table.includes(page.tags, "inbox") )
      order by page.created
      select page.name
    ]]

    if all_inbox[1] ~= nil then
      editor.navigate(all_inbox[1])
    end
  end
}
```
