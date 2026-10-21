# Installing and updating

Hermes runs as `athena` on Athenaeum and loads plugins from `~athena/.hermes/plugins`. Kayla works from her own
account and pushes to GitHub; athena installs from GitHub.

## As athena: `sync-plugin`

```
sync-plugin melete                         one plugin
sync-plugin holonomic thymos melete --restart
                                           several, one restart at the end (order does not matter)
sync-plugin entelechy                      a group (groups are in the list file)
sync-plugin oikos --restart                all of them
sync-plugin thymos --branch claude/x       a branch instead of main
sync-plugin --new hermes-<name> --restart  add a new plugin: clone it, add it to the list, install and enable it
```

The plugins it knows are in `~/.config/sync-plugin/plugins`. Each install moves the copy it replaces to
`~/.hermes/plugin-backups/` (the last 3 of each are kept). A plugin's settings are in Hermes Desktop under
**Settings → Plugins**, or `hermes config set plugins.entries.<name>.settings.<key> <value>`.

## As Kayla: `sync-repos`

Keeps your own SSH clones in `~/Github Repos` current, and with `--install` runs athena's sync-plugin:

```
sync-repos oikos --install --restart
```

## Checking

```
hermes plugins list
hermes melete            her quiet moments, and what is in them
hermes soma look         her body, now
hermes pronoia           what she is holding
```

## Mistakes worth avoiding

- Run Hermes commands as athena (`sudo -iu athena`), not as yourself: your own account has its own Hermes config.
- A `.json` settings file with a missing comma falls back to all defaults; check with `python3 -m json.tool FILE`.
- Keys and tokens stay out of chat: paste them straight into the command or file that needs them.
