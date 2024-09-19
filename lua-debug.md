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
