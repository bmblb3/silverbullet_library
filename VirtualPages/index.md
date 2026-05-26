#meta

```space-lua
index_page = {}

function index_page.count(count, color)
  if color == nil then
    return ("#%d"):format(count)
  end

  return ("<span style=\"color: %s\">#%d</span>"):format(color, count)
end

function index_page.projectCounts()
  local open_task_counts = projects.openTaskCountsByPage()
  local green_count = 0
  local red_count = 0

  for _, project in ipairs(projects.active()) do
    local page = tostring(project.name)

    if (open_task_counts[page] or 0) > 0 then
      green_count = green_count + 1
    else
      red_count = red_count + 1
    end
  end

  for _, project in ipairs(projects.archived()) do
    local page = tostring(project.name)

    if (open_task_counts[page] or 0) > 0 then
      red_count = red_count + 1
    end
  end

  return green_count, red_count
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
    local project_green_count, project_red_count = index_page.projectCounts()
    local tracked_count = #tasks.tracked()
    local untracked_count = #tasks.untracked()
    local unlinked_count = #unlinked_documents.get()

    local result = {
      "# Index",
      "",
      ("- [Inbox](inbox:) %s"):format(index_page.count(inbox_count)),
      ("- [Projects](/projects:) %s %s"):format(
        index_page.count(project_green_count, "green"),
        index_page.count(project_red_count, "red")
      ),
      ("- [Tasks](/tasks:) %s %s"):format(
        index_page.count(tracked_count, "green"),
        index_page.count(untracked_count, "red")
      ),
      ("- [Documents that are not linked anywhere](udocs:) %s"):format(
        index_page.count(unlinked_count, index_page.unlinkedColor(unlinked_count))
      )
    }

    return table.concat(result, "\n")
  end
}
```
