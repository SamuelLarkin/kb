# Configuration Files

* [Difference Between `.bashrc`, `.bash-profile`, and `.profile`](https://www.baeldung.com/linux/bashrc-vs-bash-profile-vs-profile)
* [What are the functional differences between .profile .bash_profile and .bashrc](https://serverfault.com/questions/261802/what-are-the-functional-differences-between-profile-bash-profile-and-bashrc)


`.bash_profile` and `.bashrc` are specific to `bash`, whereas `.profile` is read by many shells in the absence of their own shell-specific config files.
(`.profile` was used by the original Bourne shell.) `.bash_profile` or `.profile` is read by login shells, along with `.bashrc`; subshells read only `.bashrc`.
(Between job control and modern windowing systems, `.bashrc` by itself doesn't get used much. If you use `screen` or `tmux`, screens/windows usually run subshells instead of login shells.)

The idea behind this was that one-time setup was done by `.profile` (or shell-specific version thereof), and per-shell stuff by `.bashrc`.
For example, you generally only want to load environment variables once per session instead of getting them whacked any time you launch a subshell within a session, whereas you always want your aliases (which aren't propagated automatically like environment variables are).

Other notable shell config files:

`/etc/bash_profile` (fallback `/etc/profile`) is read before the user's .profile for system-wide configuration, and likewise `/etc/bashrc` in subshells (no fallback for this one).
Many systems including Ubuntu also use an `/etc/profile.d` directory containing shell scriptlets, which are `.` (`source`)-ed from `/etc/profile`; the fragments here are per-shell, with `*.sh` applying to all Bourne/POSIX compatible shells and other extensions applying to that particular shell.
