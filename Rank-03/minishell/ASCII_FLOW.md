# minishell - ASCII Flow

## General Flow

```text
+-----------------------------+
|           main()            |
+--------------+--------------+
               |
               v
+-----------------------------+
| init_data + bump_shlvl      |
+--------------+--------------+
               |
               v
       +-------+--------+
       | isatty(STDIN)? |
       +---+---------+--+
           |         |
         yes         no
           |         |
           v         v
+----------------+  +-------------------+
|  inter_mode()  |  |  non_inter_mode() |
+--------+-------+  +---------+---------+
         |                    |
         +---------+----------+
                   |
                   v
        +----------------------+
        |   command loop       |
        +----------+-----------+
                   |
                   v
        +----------------------+
        | read input line(s)   |
        +----------+-----------+
                   |
                   v
      +------------+----------------+
      | valid and non-empty input ? |
      +------+---------------+------+
             |               |
            no              yes
             |               |
             |     +---------v--------+
             |     |    tokenize()    |
             |     +---------+--------+
             |               |
             |               v
             |     +---------+--------+
             |     |      parse()     |
             |     +---------+--------+
             |               |
             |               v
             |     +---------+------------------------------+
             |     | expand vars -> wildcard bonus ->       |
             |     | restore masked wildcards               |
             |     +---------+------------------------------+
             |               |
             |               v
             |     +---------+--------+
             |     | prepare_heredocs |
             |     +---------+--------+
             |               |
             |               v
             |     +---------+--------+
             |     |    execute()     |
             |     +---------+--------+
             |               |
             |               v
             |     +---------+--------+
             |     | free tokens/tree |
             |     +------------------+
             |               |
             +---------------+
                             |
                             v
                    (next iteration)

Final cleanup:
- free_env_list()
- rl_clear_history()
- close stdin/stdout backups
- return exit_status
```

## Internal Flow of execute()

```text
+------------------+
|   execute(d)     |
+--------+---------+
         |
         v
+--------+------------------------------+
| root->type ?                          |
+----+---------+---------+------+-------+
     |         |         |      |
    CMD     SUBSHELL    PIPE   ; / && ||
     |         |         |      |
     v         v         v      v
+----+---+  +--+-----+ +--+--+ +----------------+
|cmd type|  |subshell| |pipe | |logical/semicolon|
+----+---+  +--------+ +-----+ +----------------+
     |
     v
+----+-------------------------------------+
| parent builtin / fork builtin / external |
+------------------------------------------+
```

## Wildcard Bonus Focus

```text
input args
   |
   v
expand_all_node_args()
   |
   v
expand_wildcards_recursive()
   |
   +--> expand_wildcards_node()
   |        |
   |        +--> process_single_argument()
   |        +--> match_pattern()
   |
   v
restore_masked_wildcards_recursive()
   |
   v
execute()
```
