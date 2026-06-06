#meta

```space-lua
projects = {}

function projects.active()
  return query[[
      from page = index.tag "page"
      where (
        page.name.match("^Projects/")
        and page.active
      )
      order by page.name
  ]]
end

function projects.inactive()
  return query[[
      from page = index.tag "page"
      where (
        page.name.match("^Projects/")
        and not page.active
      )
      order by page.name
  ]]
end

function projects.openTaskCountsByPage()
  local counts = {}
  local tasks = query[[
      from t = index.tasks()
      where not t.done
  ]]

  for _, task in ipairs(tasks) do
    if task.page ~= nil then
      local page = tostring(task.page)
      counts[page] = (counts[page] or 0) + 1
    end
  end

  return counts
end

function projects.displayName(page)
  return page:sub(string.len("Projects/") + 1)
end

function projects.row(project, open_task_counts, color_when_open, color_when_empty)
  local page = tostring(project.name)
  local label = projects.displayName(page)
  local count = open_task_counts[page] or 0
  local color = color_when_empty

  if count > 0 then
    color = color_when_open
  end

  return ("- [[%s|%s]] <span style=\"color: %s\">#%d</span>"):format(page, label, color, count)
end

virtualPage.define {
  pattern = "projects:",
  run = function(_)
    local open_task_counts = projects.openTaskCountsByPage()
    local result = {"# Active projects"}

    for _, project in ipairs(projects.active()) do
        table.insert(result, projects.row(project, open_task_counts, "green", "red"))
    end

    table.insert(result, "# Inactive projects")

    for _, project in ipairs(projects.inactive()) do
        table.insert(result, projects.row(project, open_task_counts, "red", "green"))
    end

    return table.concat(result, "\n")
  end
}
```
