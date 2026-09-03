This is needed when you cannot remove a mailbox from an Outlook profile, because it was the **first** mailbox added to that profile.

- close Outlook if it is running
- search for the correct profile in `HKEY_CURRENT_USER\Software\Microsoft\Office\<version>\Outlook\Profiles`
- within the profile, search for `001F6641`, you should find 3 keys containing a value with that name. Backup the 3 keys, then delete them
- restart Outlook and try to delete the mailbox
- if it still cannot be deleted, try to repeat the procedure searching for `001F3001`

