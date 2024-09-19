To perform a case-insensitive search with `grep` while ignoring binary files, you can combine the `-i` option for case insensitivity with the `-I` option. Here's the command:

```bash
grep -rIi "search_pattern" /path/to/directory
```

- `-r`: Recursively search through directories.
- `-I`: Ignore binary files.
- `-i`: Perform a case-insensitive search.
- `"search_pattern"`: The pattern you want to search for.
- `/path/to/directory`: The directory in which you want to perform the search.

This command will search for the specified pattern in all text files within the given directory, ignoring case and binary files.
