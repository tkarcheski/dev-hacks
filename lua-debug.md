To format the output of your `dump` function so that any decimals are limited to four decimal points, you can modify the function to check if the value is a number and then format it accordingly. Here's how you can adjust your function for this:

```lua
function dump(o)
    if type(o) == 'table' then
        local s = '{ '
        for k, v in pairs(o) do
            if type(k) ~= 'number' then k = '"' .. k .. '"' end
            s = s .. '[' .. k .. '] = ' .. dump(v) .. ','
        end
        return s .. '} '
    else
        if type(o) == 'number' then
            return string.format("%.4f", o)
        else
            return tostring(o)
        end
    end
end
```

In this updated version of your function, there's an additional check within the `else` block to see if the object `o` is a number. If it is, it uses `string.format("%.4f", o)` to format the number to four decimal points. If `o` is not a number, it simply converts it to a string as before. 

This function will now handle tables, numbers (with decimal formatting), and other types correctly. Keep in mind that this function will round numbers to four decimal places, which might not be suitable for all applications, especially if precision is critical.

In Lua, you can append two tables into one by using a loop to add the elements from one table to the other. Here's a simple function to accomplish this task:

```lua
function appendTables(table1, table2)
    for _, value in ipairs(table2) do
        table.insert(table1, value)
    end
    return table1
end
```

You can use this function to append two tables like this:

```lua
local table1 = {1, 2, 3}
local table2 = {4, 5, 6}

local combinedTable = appendTables(table1, table2)

-- Now, combinedTable will contain elements from both table1 and table2
-- combinedTable = {1, 2, 3, 4, 5, 6}
```

Keep in mind that this method modifies `table1` directly and returns it as the combined table. If you want to preserve the original tables, you can create a new table to store the result without modifying the original tables. Here's a variant of the function that does that:

```lua
function combineTables(table1, table2)
    local result = {}
    for _, value in ipairs(table1) do
        table.insert(result, value)
    end
    for _, value in ipairs(table2) do
        table.insert(result, value)
    end
    return result
end
```

Using this version, you can achieve the same result while keeping the original tables intact:

```lua
local table1 = {1, 2, 3}
local table2 = {4, 5, 6}

local combinedTable = combineTables(table1, table2)

-- Now, table1 and table2 remain unchanged, and combinedTable = {1, 2, 3, 4, 5, 6}
```

In Lua, when you compare a number to a string, Lua tries to convert the string to a number before performing the comparison. This means that the comparison can work, but it can also lead to unexpected results if the string doesn't represent a valid number.

Here's an example of comparing an integer with a string in Lua:

```lua
local a = 5
local b = "5"

if a == tonumber(b) then
  print("a is equal to b")
else
  print("a is not equal to b")
end
```

Here I'm explicitly using `tonumber(b)` to convert the string `b` to a number before performing the comparison, but you could also directly compare them:

```lua
if a == b then
  print("a is equal to b")
else
  print("a is not equal to b")
end
```

Lua will try to convert the string to a number before performing the comparison. If `b` was something that doesn't represent a number, like `"five"`, then the comparison would result in `false`.
