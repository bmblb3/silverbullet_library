#meta

```space-lua
virtualPage.define {
  pattern = "projects:",
  run = function(_)
    -- Ongoing projects
    local result = {"# Ongoing projects\n"}

    local ongoing_projects = query[[
        from index.tag "page"
        where
            table.includes(tags, "projects")
            and table.includes(tags, "ongoing")
        order by lastModified desc
    ]]

    for _, project in ipairs(ongoing_projects) do
        table.insert(result, templates.pageItem(project))
    end

    -- All projects
    table.insert(result, "\n# All projects\n")

    local projects = query[[
        from index.tag "page"
        where table.includes(tags, "projects")
        order by lastModified desc
    ]]

    for _, project in ipairs(projects) do
        table.insert(result, templates.pageItem(project))
    end

    return table.concat(result)
  end
}
```
