## brew link

```
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
Could not symlink bin/pod
Target /usr/local/bin/pod
already exists. You may want to remove it:
  rm '/usr/local/bin/pod'

To force the link and overwrite all conflicting files:
  brew link --overwrite cocoapods

To list all files that would be deleted:
  brew link --overwrite --dry-run cocoapods

Possible conflicting files are:
/usr/local/bin/pod
/usr/local/bin/xcodeproj
```

