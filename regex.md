# Regular Expressions
MariaDB Server 10.0 replaced the old regular expression library to the modern Perl Compatible Regular Expressions (PCRE). This complements `RLIKE()` with a few additions:
* `REGEXP_REPLACE(subject, pattern, replace)` - Replaces all occurrences of a pattern.
* `REGEXP_INSTR(subject, pattern)` - Position of the first appearance of a regex .
* `REGEXP_SUBSTR(subject,pattern)` - Returns the matching part of a string.

## Try it
1. `SELECT REGEXP_INSTR('BJÖRN','N')` - Returns the position of the first occurrence of the regex pattern in the string subject, else 0 if pattern not found. 
2. `SELECT REGEXP_SUBSTR('See https://mariadb.org/en/foundation/ for details', 'https?://[^/]*') AS site;` - Returns the part of the string subject that matches the regular expression pattern, else an empty string
3. ` SELECT REGEXP_REPLACE('<html><head><title>title</title><body>body</body></html>','<.+?>',' ') AS strip_html;` - Strip HTML tags with a non greedy match 
4. `SELECT REGEXP_REPLACE('James Bond','^(.*) (.*)$','\\2, \\1') AS reorder_name;` - Reorder a name using back-references to the sub-expressions