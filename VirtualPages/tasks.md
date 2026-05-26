#meta

```space-lua
tasks = {}

function tasks.tracked()
  return query[[
      from t = index.tag "task"
      where t.name.match("https://taskking.732685803.xyz")
      order by t.page
  ]]
end

function tasks.untracked()
  return query[[
      from t = index.tag "task"
      where not t.name.match("https://taskking.732685803.xyz")
      order by t.page
  ]]
end

function tasks.heading(title, count, color_when_present, color_when_empty)
  local color = color_when_empty

  if count > 0 then
    color = color_when_present
  end

  return ("%s <span style=\"color: %s\">#%d</span>"):format(title, color, count)
end

function tasks.row(task)
  if task.page ~= nil then
    return ("- [[%s]] %s"):format(tostring(task.page), tostring(task.name))
  end

  return ("- %s"):format(tostring(task.name))
end

virtualPage.define {
  pattern = "tasks:",
  run = function(_)
    local tracked = tasks.tracked()
    local untracked = tasks.untracked()
    local result = {
      tasks.heading("# Tracked in taskwarrior", #tracked, "green", "red")
    }

    for _, task in ipairs(tracked) do
      table.insert(result, tasks.row(task))
    end

    table.insert(result, tasks.heading("# Not Tracked in taskwarrior", #untracked, "red", "green"))

    for _, task in ipairs(untracked) do
      table.insert(result, tasks.row(task))
    end

    return table.concat(result, "\n")
  end
}
```
