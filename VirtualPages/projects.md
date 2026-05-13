#meta

```space-lua
projects = {}

function projects.unfinished()
  return query[[
      from page = index.tag "page"
      where (
        page.name.match("^Project")
        and not table.includes(page.tags, "finished")
      )
      order by page.name
  ]]
end

function projects.finished()
  return query[[
      from page = index.tag "page"
      where (
        page.name.match("^Project")
        and table.includes(page.tags, "finished")
      )
      order by page.name
  ]]
end

virtualPage.define {
  pattern = "projects:",
  run = function(_)
    local result = {"# Unfinished projects\n"}

    for _, project in ipairs(projects.unfinished()) do
        table.insert(result, templates.pageItem(project))
    end

    table.insert(result, "\n# Finished projects\n")

    for _, project in ipairs(projects.finished()) do
        table.insert(result, templates.pageItem(project))
    end

    return table.concat(result)
  end
}
```
