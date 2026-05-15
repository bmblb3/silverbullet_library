#meta

# Taskbridge action button
```space-lua
local function set_read_only(enabled)
  editor.setUiOption("forcedROMode", enabled)
  editor.rebuildEditorState()
end

local function present(value, fallback)
  if value ~= nil and tostring(value) ~= "" then
    return tostring(value)
  end

  return fallback
end

local function parse_json(value)
  if value == nil or tostring(value) == "" then
    return nil, "Bridge returned no JSON output"
  end

  local ok, decoded = pcall(function()
    return js.window.JSON.parse(value)
  end)

  if ok then
    return decoded, nil
  end

  return nil, tostring(decoded)
end

local function format_report(report)
  local lines = {}

  for key, value in pairs(report or {}) do
    table.insert(lines, ("%s: %s"):format(key, tostring(value)))
  end

  return table.concat(lines, "\n")
end

local function show_sync_success(response)
  local message = present(response and response.message, "Taskwarrior sync completed")
  local report = response and response.report
  local report_text = format_report(report)

  if report == nil or report_text == "" then
    editor.flashNotification(message, "info")
    return
  end

  editor.flashNotification(message, "info", {
    actions = {
      {
        name = "Show report",
        run = function()
          editor.flashNotification(report_text)
        end,
      },
    },
  })
end

local function find_source(issue, source_name)
  for _, source in pairs(issue.sources or {}) do
    if tostring(source.source) == source_name then
      return source
    end
  end

  return nil
end

local function navigate_to_silverbullet_source(source)
  if source.character ~= nil then
    editor.navigate(("%s@%s"):format(tostring(source.page), tostring(source.character)))
    return
  end

  editor.navigate(tostring(source.page))
end

local function open_taskking_source(source)
  js.window.open(tostring(source.taskking_url), "_blank")
end

local function add_issue_action(actions, name, run)
  table.insert(actions, {
    name = name,
    run = run,
  })
end

local function describe_silverbullet_source(source)
  if source == nil or source.page == nil then
    return nil
  end

  if source.line ~= nil then
    return ("%s:%s"):format(tostring(source.page), tostring(source.line))
  end

  if source.character ~= nil then
    return ("%s@%s"):format(tostring(source.page), tostring(source.character))
  end

  return tostring(source.page)
end

local function describe_taskwarrior_source(source)
  if source == nil or source.uuid == nil then
    return nil
  end

  return ("Taskwarrior %s"):format(tostring(source.uuid))
end

local function append_context(message, silverbullet_source, taskwarrior_source)
  local details = {}
  local silverbullet_detail = describe_silverbullet_source(silverbullet_source)
  local taskwarrior_detail = describe_taskwarrior_source(taskwarrior_source)

  if silverbullet_detail ~= nil then
    table.insert(details, silverbullet_detail)
  end

  if taskwarrior_detail ~= nil then
    table.insert(details, taskwarrior_detail)
  end

  if #details == 0 then
    return message
  end

  return ("%s\n%s"):format(message, table.concat(details, " | "))
end

local function show_issue(issue, fallback_message)
  local silverbullet_source = find_source(issue, "silverbullet")
  local taskwarrior_source = find_source(issue, "taskwarrior")
  local message = present(issue.message, fallback_message)
  local actions = {}

  if silverbullet_source ~= nil and silverbullet_source.page ~= nil then
    add_issue_action(actions, "Go to line", function()
      navigate_to_silverbullet_source(silverbullet_source)
    end)
  end

  if taskwarrior_source ~= nil and taskwarrior_source.taskking_url ~= nil then
    add_issue_action(actions, "Open Taskking", function()
      open_taskking_source(taskwarrior_source)
    end)
  end

  editor.flashNotification(append_context(message, silverbullet_source, taskwarrior_source), "warning", {
    timeout = 0,
    actions = actions,
  })
end

local function show_structured_failure(response)
  local fallback_message = present(response and response.message, "Taskwarrior sync failed")
  local shown_issue = false

  for _, issue in pairs(response.issues or {}) do
    shown_issue = true
    show_issue(issue, fallback_message)
  end

  if shown_issue then
    return
  end

  editor.flashNotification(fallback_message, "warning", {
    timeout = 0,
  })
end

local function show_unstructured_failure(process, parse_error)
  local detail = present(process.stderr, present(process.stdout, parse_error or "No bridge output"))

  editor.flashNotification(("Taskwarrior sync failed\n%s"):format(detail), "error", {
    timeout = 0,
  })
end

local function sync_taskwarrior()
  local process = shell.run("bridge", {"sync", "--space", "/space"})
  local output = process.stdout

  if process.code ~= 0 then
    output = process.stderr
  end

  local response, parse_error = parse_json(output)

  if response == nil then
    show_unstructured_failure(process, parse_error)
    return
  end

  if process.code == 0 then
    show_sync_success(response)
    return
  end

  show_structured_failure(response)
end

local function handle_error(error_message)
  editor.flashNotification(("Taskwarrior sync failed\n%s"):format(tostring(error_message)), "error", {
    timeout = 0,
  })
end

actionButton.define({
  description = "Sync with Taskwarrior",
  icon = "send",
  priority = 2.8,

  run = function()
    set_read_only(true)

    local ok, error_message = xpcall(sync_taskwarrior, function(error_message)
      return error_message
    end)

    set_read_only(false)

    if not ok then
      handle_error(error_message)
    end
  end,
})
```
