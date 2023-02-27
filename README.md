# Update Scripts
A collection of scripts I use to update software I use on linux, most of which download and build from source; these scripts do not download the sources for the first time. 

### Firefox
One of the more complicated ones (for obvious reasons); following the instructions [here](https://firefox-source-docs.mozilla.org/contributing/contribution_quickref.html#firefox-contributors-quick-reference) to get and ran `bootstrap.py` inside a python virtual enviroment for the build process created with `python -m venv venv`. I setup the top-level file structure to look like this
```plain
<Top-Level Directory>
├── bootstrap.py
├── firefox
├── firefox-<latest-version>.en-US.linux-x86_64.tar.bz2
├── firefox_build
├── icons
├── mozbuild
├── mozilla-unified
├── old_tars
├── update_firefox.sh
└── venv
```
* The only enviromental variable I had to set was `MOZBUILD_STATE_PATH=<Top-Level Directory>/mozbuild`
* This is what my `mozconfig` in the source directory looks like

```
# Automatically download and use compiled C++ components:
ac_add_options --enable-artifact-builds
mk_add_options AUTOCLOBBER=1

# build firefox out-of-source
mk_add_options MOZ_OBJDIR=@TOPSRCDIR@/../firefox_build
```

Finally here is the `update.sh` script
```bash
#!/usr/bin/env bash
cd <Top-Level Directory>
. venv/bin/activate
cd mozilla-unified
hg pull central
hg up central
hg update
./mach build
./mach package
FIREFOXFILE=""
cd ..
for f in ./firefox_build/dist/firefox*.tar.bz2; do
  cp "$f" .
  FIREFOXFILE="$f"
done
rm -rf firefox
tar -xvjf "$FIREFOXFILE"
mv -f "$FIREFOXFILE" "./old_tars/"
echo "=========================="
echo "Firefox is now updated!"
echo "=========================="
deactivate
```

### Webcord
This `update.sh` script is inside the git cloned repo without any other stuff

```bash
#!/bin/bash
# Check if the file has been modified
if [[ $(git status --porcelain update.sh) ]]; then
  # Add the file to the staging area
  git add update.sh
  
  # Commit the changes
  git commit --no-verify -m "Automatic commit of update.sh"
  echo "update.sh was updated and automatically committed"
else
  # Do nothing
  echo "update.sh has not been modified"
fi
rm package-lock.json
git restore .
git reset
git clean -fd
git reset
git restore .
git pull --rebase origin master
npm ci
npm install
npm install --save-dev electron-packager
npm run make
```

### Freetube
This `update.sh` script is inside the git cloned repo without any other stuff

```bash
#!/bin/bash
# Check if the file has been modified
if [[ $(git status --porcelain update.sh) ]]; then
  # Add the file to the staging area
  git add update.sh
  
  # Commit the changes
  git commit --no-verify -m "Automatic commit of update.sh"
  echo "update.sh was updated and automatically committed"
else
  # Do nothing
  echo "update.sh has not been modified"
fi
git restore .
git reset
git clean -fd
git reset
git restore .
python -c "from pathlib import Path; import json; pkgj = Path('package.json').read_text(); pkgus = json.loads(pkgj); pkgus['scripts']['make-linux'] = 'yarn electron-packager . freetube --platform=linux --overwrite'; pkgw = json.dumps(pkgus, indent=2); Path('package.json').write_text(pkgw)"
yarn clean
yarn
yarn add electron-packager
yarn run ci
yarn run rebuild:electron
yarn run pack
yarn run pack:web
yarn run make-linux
```
