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
```
