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
    local result = {"# Inbox\n"}

    -- for _, page in ipairs(inbox_pages()) do
    --     table.insert(result, templates.fullPageItem(page))
    -- end
    return table.concat(result)
  end
}
```
