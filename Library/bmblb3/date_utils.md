---
name: Library/bmblb3/date_utils
tags: meta/library
---
```space-lua
function date.current_PI()
  today = os.time()
  seconds_per_week = 60*60*24*7
  for offset_weeks = 0, 13 do
    offset_seconds = offset_weeks*seconds_per_week
    test_time = today - offset_seconds
    YY = os.date("%y", test_time)
    WW = os.date("%W", test_time)
    if string.endsWith(WW, '6') then
      return YY..WW
    end
  end
  return test_day
end
```