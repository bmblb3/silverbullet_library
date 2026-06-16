#meta

```space-lua
index_page = {}

function index_page.count(count, color)
  if color == nil then
    return ("#%d"):format(count)
  end

  return ("<span style=\"color: %s\">#%d</span>"):format(color, count)
end

function index_page.unlinkedColor(count)
  if count > 0 then
    return "red"
  end

  return "green"
end

virtualPage.define {
  pattern = "index:",
  run = function(_)
    local inbox_count = #inbox_pages.get()
    local unlinked_docs_count = #unlinked_documents.get()

    local result = {
      "",
      ("# - [Inbox](inbox:) %s"):format(index_page.count(inbox_count)),
      ("# - [Documents that are not linked anywhere](udocs:) %s"):format(
        index_page.count(unlinked_count, index_page.unlinkedColor(unlinked_count))
      ),
      ""
    }

    return table.concat(result, "\n")
  end
}
```
