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
local navigateToInboxHead = function()
  local inboxPages = query[[
    from page = index.tag "page"
    where ( page.name.match("^Inbox") or table.includes(page.tags, "inbox") )
    order by page.created
    select page.name
  ]]

  if inboxPages[1] ~= nil then
    editor.navigate(inboxPages[1])
  end
end

command.define {
  name = "Inbox: Process",
  key = "Ctrl-Alt-i",
  run = navigateToInboxHead
}

actionButton.define {
  icon = "inbox",
  description = "Process inbox",
  priority = 2.9,
  command = "Inbox: Process",
}
```