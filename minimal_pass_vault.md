# Set up a minimal pass vault with split-gpg
(this assumes you have already set up a PGP-vault in step 2.)


1. Install required tools in template
```sudo apt install qubes-gpg-split pass vim git```

2. create a new qube, vault based on the debian 13 minimal template
```qvm-create --template=debian-11-minimal --label=black vault```

Also set their netvm to none
```qvm-prefs vault netvm none```

3. Configure vault to use split-gpg
```echo "export QUBES_GPG_DOMAIN=gpg-vault" >> ~/.profile```

4. Initialize password store on vault domain

`mkdir -p ~/.password-store/.extensions

`cd ~/.password-store/.extensions

copy https://raw.githubusercontent.com/kulinacs/pass-qubes/master/qubes.bash

`chmod +x ~/.password-store/.extensions/qubes.bash

`echo "export PASSWORD_STORE_ENABLE_EXTENSIONS=true" >> ~/.profile

`pass qubes init user@vault.local

`pass git init


5. Add qrexec policy to remove prompts (qubes 4.0)
in dom0 edit /etc/qubes-rpc/policy/qubes.Gpg and put the following at the top of the file
vault gpg-vault allow


Fix for qube unable to start uxterm.

`sudo apt install locales-all` in template


### Script

```
#!/usr/bin/env bash
# pass qubes - Password Store Extension (https://www.passwordstore.org/)
#
# Copyright (C) 2017 Nicklaus McClendon <nicklaus@kulinacs.com>
# Copyright (C) 2012 - 2017 Jason A. Donenfeld <Jason@zx2c4.com>. All Rights Reserved.
# This file is licensed under the GPLv2+. Please see COPYING for more information.


GPG="qubes-gpg-client-wrapper"

PROGRAM="${0##*/}"
COMMAND="$1"

case "$1" in
	init) shift;			cmd_init "$@" ;;
	help|--help) shift;		cmd_usage "$@" ;;
	version|--version) shift;	cmd_version "$@" ;;
	show|ls|list) shift;		cmd_show "$@" ;;
	find|search) shift;		cmd_find "$@" ;;
	grep) shift;			cmd_grep "$@" ;;
	insert|add) shift;		cmd_insert "$@" ;;
	edit) shift;			cmd_edit "$@" ;;
	generate) shift;		cmd_generate "$@" ;;
	delete|rm|remove) shift;	cmd_delete "$@" ;;
	rename|mv) shift;		cmd_copy_move "move" "$@" ;;
	copy|cp) shift;			cmd_copy_move "copy" "$@" ;;
	git) shift;			cmd_git "$@" ;;
	*)				cmd_extension_or_show "$@" ;;
esac
exit 0
```
