#meta

```space-lua
inbox_pages = {}

function inbox_pages.get()
  return query[[
      from page = index.tag "page"
      where (
        page.name.match("^Inbox/")
      )
      order by page.created
  ]]
end

virtualPage.define {
  pattern = "inbox:",
  run = function(_)
    local result = {"# Inbox"}

    for _, page in ipairs(inbox_pages.get()) do
        table.insert(result, ("- [[%s]]"):format(page.name))
    end
    return table.concat(result, "\n")
  end
}
```
