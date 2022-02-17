# partis example usage

My [partis] setup via conda and some basic usage.

Setting up conda environment with dependencies:

    conda env update --file environment.yml
    conda activate example-partis

Setting up partis itself:

    git submodule update --init --recursive
    # I had some trouble with the ham dependency as described here:
    # https://github.com/psathyrella/ham/issues/16
    # manually copying these files below is a workaround that doesn't requiring
    # modifying anything
    mkdir -p partis/packages/ham/_build
    cp partis/packages/ham/{src/*.cc,_build}
    ( cd partis && ./bin/build.sh )
    export PATH="$PATH:$PWD/partis/bin"

(The PATH change could be [incorporated into the conda environment](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html),
too, instead of being manual.)

[partis]: https://github.com/psathyrella/partis
