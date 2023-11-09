# Colors

## Bash Colors
[Script to display all terminal colors](https://askubuntu.com/a/1044802)
```sh
msgcat --color=test
```



## Putty Colors
We need to use `putty-256color` instead of `xterm-256color` or else the `Home` key is not working.

[Putty shows prompt with no color, but Linux SSH can](https://superuser.com/a/1502895)
In Putty, change Settings -> Connection > Data > Terminal-type string to: `putty-256color`.
["Emulate" 256 colors in PuTTY terminal](https://superuser.com/a/436928)
1. Configure Putty

In Settings > Windows > Colours there is a check box for "Allow terminal to use xterm 256-colour mode".
2. Let the app know

You'll probably have to change Settings -> Connection > Data > Terminal-type string to: `xterm-256color`

if your server has a terminfo entry for `putty-256color`, typically in `/usr/share/terminfo/p/putty-256color`, you can set `Putty`'s Terminal-Type to `putty-256color` instead.

The main thing here is to make the server use an available `Terminfo` entry that most closely matches the way `Putty` is configured.

### 24bit
* [Getting 24-bit color working in terminals](https://pisquare.osisoft.com/s/Blog-Detail/a8r1I000000GvXBQA0/console-things-getting-24bit-color-working-in-terminals)




## LSCOLORS Schemes
```sh
for theme in $(vivid themes); do
  echo "Theme: $theme";
  LS_COLORS=$(vivid generate $theme);
  ls;
  echo;
done
vivid generate one-light
LSCOLORS=$(vivid generate one-light)
```



