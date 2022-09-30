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
rm package-lock.json
git diff --exit-code update.sh
status=$?
[ $status -eq 0 ] && echo "update.sh up to date" || (echo "Adding update commit for update.sh" && git add update.sh && git commit -m "updated update.sh")
git restore .
git pull
rm package-lock.json
python -c "from pathlib import Path; import json; pkgj = Path('package.json').read_text(); pkgus = json.loads(pkgj); pkgus['scripts']['make-linux'] = 'yarn electron-packager . webcord --platform=linux --overwrite'; pkgw = json.dumps(pkgus, indent=2); Path('package.json').write_text(pkgw)"
yarn 
yarn add electron-packager
npm install
npm ci
npm update
npm run make-linux
```

### Freetube
This `update.sh` script is inside the git cloned repo without any other stuff

```bash
#!/bin/bash
git diff --exit-code update.sh
status=$?
[ $status -eq 0 ] && echo "update.sh up to date" || (echo "Adding update commit for update.sh" && git add update.sh && git commit -m "updated update.sh")
git restore .
git pull
python -c "from pathlib import Path; import json; pkgj = Path('package.json').read_text(); pkgus = json.loads(pkgj); pkgus['scripts']['make-linux'] = 'yarn electron-packager . freetube --platform=linux --overwrite'; pkgw = json.dumps(pkgus, indent=2); Path('package.json').write_text(pkgw)"
yarn clean
yarn 
yarn add electron-packager
npm run rebuild:electron
npm run pack:main
npm run pack:renderer
npm run pack:workers
npm run make-linux
```
