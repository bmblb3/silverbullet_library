#meta

```space-lua
virtualPage.define {
  pattern = "projects:",
  run = function(_)
    
    -- Tracked projects
    local result = {"# Tracked projects\n"}
    local ongoing_projects = query[[
        from page = index.tag "page"
        where (
          ( page.name.match("^Project") or table.includes(page.tags, "project") )
          and table.includes(page.tags, "track")
        )
        order by page.name
    ]]

    for _, project in ipairs(ongoing_projects) do
        table.insert(result, templates.pageItem(project))
    end

    -- Rest of the projects
    table.insert(result, "\n# Rest of the projects\n")

    local projects = query[[
        from page = index.tag "page"
        where (
          ( page.name.match("^Project") or table.includes(page.tags, "project") )
          and not table.includes(page.tags, "track")
        )
        order by page.name
    ]]

    for _, project in ipairs(projects) do
        table.insert(result, templates.pageItem(project))
    end

    return table.concat(result)
  end
}
```