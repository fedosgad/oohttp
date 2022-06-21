## How to apply patches

When patch changes files in http2 code, h2_bundle must be regenerated. This is done in a 'with-http2'
branch. To avoid merging http2/ into main, a special branch 'without-http2' is used as a transit
branch with only non-merge commit being "Remove http2/ folder". Only 'without-http2' can be merged
into main.

Patching steps as follows:

```bash
set -ex

# Pull upstream changes from x/net/http2
git checkout main
git remote add x-net https://github.com/golang/net || git fetch golang x-net	# Add/update remote
git branch -D x-net-upstream x-net-http2-upstream || true			# Remove old branches
git fetch x-net									# Load upstream changes
git checkout -b x-net-upstream x-net/internal-branch.go1.17-vendor		# New branch from selected upstream version
git subtree split -P http2/ -b x-net-http2-upstream				# Extract http2/ directory to a separate branch

# Create working branches
git checkout main
git checkout -b with-http2
git subtree add -P http2 x-net-http2-upstream	# http2/ now contains files from x/net/http2
git checkout -b without-http2
rm -rf http2/				# Only run this once
git commit -a -m "Remove http2/ folder"	# Only run this once
git checkout with-http2

# apply patch
# perform all checks from oohttp/README.md
bundle -o=h2_bundle.go -prefix=http2 -tags='!nethttpomithttp2' github.com/ooni/oohttp/http2	# bundle new version of http2
# commit changes

# Merging changes into 'main'
git checkout without-http2
git merge with-http2
git rm -r http2/
git commit --no-edit
git checkout main
git merge without-http2 -m "Patch description here"
```
