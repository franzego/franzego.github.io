---
title: "Hello, World: Building a Go-Powered Blog from Scratch"
date: 2026-05-28
draft: false
tags: ["golang", "infrastructure", "hugo", "ci-cd"]
---

Trying to set up a personal blog is not quite straightforward. There was no one stop guide for setting up one using github pages. 
After solving the issue, i decided to document the steps i followed for my future self and anyone who ends up in the same position as I was.

## Hugo
Written in Go, Hugo treats content like source code. It takes Markdown, runs it through Go's html/template engine, and spits out a static site in milliseconds.

If it means anything, I did this on Ubuntu (wsl). It is probably similar to macOS. If you are on windows, just use wsl please - it saves a lot of headaches.

### The Installation Trap
If you are on Ubuntu, your first instinct is likely sudo apt install hugo. It is a trap. The standard repositories often carry outdated binaries, frequently missing the "extended" version required to compile SCSS for modern themes.

Instead of relying on the package manager, I pulled the standalone binary directly from the source. It is cleaner and gives you absolute control over the runtime. Go about it like this: 

```bash
# Pull the archive
wget https://github.com/gohugoio/hugo/releases/download/v0.162.1/hugo_0.162.1_linux-amd64.tar.gz (the specific version may have changed at the time of your usage).

# Extract
tar -zxvf hugo_0.162.1_linux-amd64.tar.gz

# Move the binary to your path
sudo mv hugo /usr/local/bin/

# Clean up the remnants
rm hugo_0.162.1_linux-amd64.tar.gz README.md LICENSE

# Check version
hugo version

```
Hopefully the commands above went without any hitch. On to the next

### Scaffolding the System
With the Go binary in my path, initializing the architecture is trivial. 

```bash
hugo new site your-blog
cd your-blog
git init

```
For the frontend, I wanted something minimalist - a layout that respects the reader's time and focuses entirely on the typography and the code blocks. I went with PaperMod, adding it as a Git submodule so it can be updated without polluting the repository's commit history.

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
echo "theme = 'PaperMod'" >> hugo.toml

```
After this, you run this command:
```bash
hugo new contents/posts/hello-world.md 

```
You create a repostiory on github: username.github.io. Do not include a gitignore or readme.md file. After that run your remote add origin command. then git branch -M main. Don't push yet. Relax.


### Automating the Deployment (and Fixing the Pipeline)
The goal was a pure "docs-as-code" pipeline: I push a Markdown file to the main branch, and a CI/CD runner handles the build and deployment to GitHub Pages.

We create a workflows file for this repo. The code for it is in the .github/workflows/hugo.yml file in this repo. 
After that we add, commmit and push. 


I provisioned an Ubuntu runner in .github/workflows/hugo.yml. However, the modern CI environment is a moving target. GitHub is currently deprecating Node 20 actions, which caused the pipeline to throw a loud warning. I silenced it by forcing the runner to use Node 24 and bumping the Hugo action to v3 

### A Syntax Lesson in the Front Matter
The pipeline ran, and immediately crashed with a fatal exit code 1:
```text
unmarshal failed: toml: expected character =
```
t is always the smallest details that bring a system down. Hugo relies heavily on "front matter"—the metadata at the top of every Markdown file. I had inadvertently mixed YAML syntax (which uses colons, like draft: false) inside a block that Hugo was trying to parse as TOML.
```yaml
---
title: "Hello World"
date: 2026-05-28
draft: false
---
```
Once the parser had the correct characters, the pipeline turned green. The site was live and active.

Setting this up feels much like writing a good Go program: strict, explicit, and highly performant. The infrastructure is now invisible, leaving me with nothing to do but write.