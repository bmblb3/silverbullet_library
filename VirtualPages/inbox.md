#meta

```space-lua
function inbox_pages()
  return query[[
      from page = index.tag "page"
      where (
        page.name.match("^Inbox") or
        table.includes(page.tags, "inbox")
      )
      order by page.created
  ]]
end

function inbox_page.row(page)
  return ("- [[%s]]"):format(page.name)
end

virtualPage.define {
  pattern = "inbox:",
  run = function(_)
    local result = {"# Inbox\n"}

    for _, page in ipairs(inbox_pages()) do
        table.insert(result, inbox_page.row(page))
    end
    return table.concat(result)
  end
}
```
