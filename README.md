# myhugoblog
The static site generator data for pdebashis.github.io.

1. Checkout the repo.
2. Checkout the subrepors - git submodule update --init --recursive
3. Switch to main branch
    cd pdebashis.github.io/public
    git switch main
4. Change the http remote to SSH (to avoid auth issues while pushing changes)
    git remote set-url origin git@github.com:pdebashis/pdebashis.github.io.git
3. Run hugo -t hugo-theme-console inside pdebashis.github.io Folder.
4. The static site is generated inside public directory which is a submodule of pdebashis.github.io repository.
5. ???
6. Profit



