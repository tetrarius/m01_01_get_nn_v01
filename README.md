*This project has been created as part of the 42 curriculum by <your_login>.*

## Description

**get_next_line** is a C function that reads one line at a time from a file descriptor. Each successive call returns the next line until the file descriptor reaches EOF or an error occurs. The returned string includes the terminating `\n` character, except when the file does not end with one.

The project introduces the concept of **static variables** in C — a local variable whose value persists between function calls.

## Instructions

### Compilation

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c
```

`BUFFER_SIZE` controls how many bytes are read from the file descriptor per `read()` call. It can be set to any positive integer. If the flag is omitted, the default value of **42** is used.

### Usage

```c
#include "get_next_line.h"

int main(void)
{
    int     fd;
    char    *line;

    fd = open("file.txt", O_RDONLY);
    while ((line = get_next_line(fd)) != NULL)
    {
        printf("%s", line);
        free(line);
    }
    close(fd);
    return (0);
}
```

## Algorithm

The implementation uses a **static buffer accumulator (stash)** approach:

1. **Read into stash** — `read()` fills a temporary buffer of `BUFFER_SIZE` bytes. The buffer is appended to a persistent `static char *stash` via `ft_strjoin`. Reading continues until a `\n` is found in the stash or EOF is reached.
2. **Extract line** — once a complete line (terminated by `\n` or EOF) exists in the stash, it is copied into a new string and returned.
3. **Update stash** — everything after the `\n` is kept in the stash for the next call. If nothing remains, the stash is freed and set to `NULL`.

**Why this approach:**
- Only as many bytes as needed are read per call (requirement from the subject).
- The stash survives between calls thanks to `static` storage, without needing global variables.
- Memory is fully freed: the old stash is freed on every join and on cleanup.

**BUFFER_SIZE impact:**
- Large values (e.g. 10 000 000) — fewer `read()` syscalls, but large stack/heap allocations per call.
- Small values (e.g. 1) — many `read()` syscalls, but minimal memory per call.
- The function is correct for any positive value.

## Resources

- `man 2 read` — POSIX read() specification
- `man 3 malloc` / `man 3 free` — heap memory management
- *The C Programming Language*, Kernighan & Ritchie — static variables (§4.6)
- 42 subject: `en.subject.pdf` (included in repository)

### AI usage

Claude Code (claude-sonnet-4-6) was used to:
- Review the implementation against the subject requirements and identify a memory management bug (`malloc` failure path in `read_to_stash` returning stale stash instead of freeing and returning NULL).
- Generate this README structure.

All algorithmic decisions, code structure, and implementation were developed independently before AI review.
