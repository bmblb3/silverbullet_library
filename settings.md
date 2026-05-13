#meta

# Shortcut keys and Action buttons
```space-lua
command.update {
  name = "Navigate: Back in History",
  key = "Ctrl-Shift--"
}

command.update {
  name = "Navigate: Forward in History",
  key = "Ctrl-Shift-+"
}
```

```space-lua
actionButton.define {
  icon = "inbox",
  description = "Go to inbox",
  priority=2.9,
  run = function()
    local all_inbox = query[[
      from page = index.tag "page"
      where ( page.name.match("^Inbox") or table.includes(page.tags, "inbox") )
      order by page.created
      select page.name
    ]]

    if all_inbox[1] ~= nil then
      editor.navigate(all_inbox[1])
    end
  end
}
```

```space-lua
local function set_read_only(enabled)
  editor.setUiOption("forcedROMode", enabled)
  editor.rebuildEditorState()
end

local function parse_json(value)
  return js.window.JSON.parse(value)
end

local function format_report(report)
  local lines = {}

  for key, value in pairs(report or {}) do
    table.insert(lines, ("%s: %s"):format(key, tostring(value)))
  end

  return table.concat(lines, "\n")
end

local function show_sync_success(response)
  local report = format_report(response.report)

  editor.flashNotification(response.message, "info", {
    actions = {
      {
        name = "Show more",
        run = function()
          editor.flashNotification(report)
        end,
      },
    },
  })
end

local function navigate_to_task(task)
  editor.navigate(("%s@%s"):format(task.page, task.character))
end

local function show_invalid_markdown_tasks(tasks)
  for _, task in pairs(tasks or {}) do
    editor.flashNotification(tostring(task.description), "warning", {
      timeout = 0,
      actions = {
        {
          name = "Go",
          run = function()
            navigate_to_task(task)
          end,
        },
      },
    })
  end
end

local function show_sync_failure(response)
  if response.error_kind ~= "invalid_markdown" then
    editor.flashNotification(response.message, "warning")
    return
  end

  show_invalid_markdown_tasks(response.invalid_markdown_tasks)
end

local function sync_taskwarrior()
  local process = shell.run("bridge", {"sync", "--space", "/space"})

  if process.code == 0 then
    show_sync_success(parse_json(process.stdout))
    return
  end

  show_sync_failure(parse_json(process.stderr))
end

local function handle_error(error_message)
  editor.flashNotification(tostring(error_message), "error")
end

actionButton.define({
  description = "Sync with taskwarrior",
  icon = "send",
  priority=2.8,

  run = function()
    set_read_only(true)

    xpcall(sync_taskwarrior, handle_error)

    set_read_only(false)
  end,
})
```
