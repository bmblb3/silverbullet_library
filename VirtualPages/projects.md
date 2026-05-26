#meta

```space-lua
projects = {}

function projects.active()
  return query[[
      from page = index.tag "page"
      where (
        page.name.match("^Projects/")
        and not page.name.match("^Projects/Archived")
      )
      order by page.name
  ]]
end

function projects.archived()
  return query[[
      from page = index.tag "page"
      where (
        page.name.match("^Projects/Archived")
      )
      order by page.name
  ]]
end

virtualPage.define {
  pattern = "projects:",
  run = function(_)
    local result = {"# Active projects\n"}

    for _, project in ipairs(projects.active()) do
        table.insert(result, templates.fullPageItem(project))
    end

    table.insert(result, "\n# Archived projects\n")

    for _, project in ipairs(projects.archived()) do
        table.insert(result, templates.fullPageItem(project))
    end

    return table.concat(result)
  end
}
```
