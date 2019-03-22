The following instructions explain how to set up your development environment for Lichess on Windows.

GNU/Linux instructions: https://github.com/ornicar/lila/wiki/Lichess-Development-Onboarding

The smoothest way to run lila on Windows is by [enabling the Windows Subsystem for Linux](https://docs.microsoft.com/de-de/windows/wsl/install-win10), installing an operating system, and following the GNU/Linux instructions as linked above.

If you're running Ubuntu using WSL, then you'll run into problems when installing some of the required packages because `apt-key` is bugged in WSL. But there's a workaround - all `apt-key` commands you come across have to be rewritten in the way as described in this issue comment: https://github.com/Microsoft/WSL/issues/3286#issuecomment-395980867
