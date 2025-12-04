---
name: Library/bmblb3/VirtualPages/inbox
tags: meta/library
---
```space-lua
virtualPage.define {
  pattern = "inbox:",
  run = function(_)
    local result = {"# INBOX"}

    table.insert(result, "## Notes in Inbox/")
    local inbox_md_files = query[[
      from index.tag "page"
      where name.match("^Inbox")
    ]]
    for md_file in inbox_md_files do
      local line = {}
      table.insert(line, "- [[")
      table.insert(line, md_file.ref)
      table.insert(line, "]]")
      table.insert(result, table.concat(line))
    end
    table.insert(result, "\n")

    table.insert(result, "## Tagged as #inbox")
    local inbox_tags = query[[from index.tag "inbox"]]
    for tag in inbox_tags do
      local line = {}
      table.insert(line, "- [")
      table.insert(line, tag.name)
      table.insert(line, "](")
      table.insert(line, tag.ref)
      table.insert(line, ") ")
      table.insert(line, tag.page)
      table.insert(result, table.concat(line))
    end
    table.insert(result, "\n")

    return table.concat(result, "\n")
  end
}

command.define {
  name = 'Inbox: All',
  key = 'Ctrl-Alt-i',
  run = function()
    editor.navigate("inbox:")
  end
}
```