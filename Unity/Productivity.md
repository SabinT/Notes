# Project Setup

Common first time stuff on an empty repo:

```
git init

curl -o .gitignore https://github.com/github/gitignore/raw/main/Unity.gitignore
curl -o .gitattributes https://gist.github.com/nemotoo/b8a1c3a0f1225bb9231979f389fd4f3f/raw/dc3e8cab80fc62d1c60db70c761b1ffa636aa796/.gitattributes

git add .gitignore
git add .gitattributes
git commit -m "Initialize with gitignore/git LFS attributes"

# Optional: add useful packages
# openupm add com.newtonsoft.json
# openupm install com.madsbangh.easybuttons
# openupm add jillejr.newtonsoft.json-for-unity
# openupm add com.dbrizov.naughtyattributes
```

# Better Inspectors

Lots of useful attributes that save you the trouble of writing custom inspectors:
https://github.com/dbrizov/NaughtyAttributes

```
openupm add com.dbrizov.naughtyattributes
```

OR, add to manifest.json:

```
"com.dbrizov.naughtyattributes": "https://github.com/dbrizov/NaughtyAttributes.git#upm"
```
