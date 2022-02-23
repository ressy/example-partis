# partis example usage

My from-scratch [partis] setup via conda, plus some notes on basic usage as I
worked through things.

## Setup

Setting up conda environment with dependencies:

    conda env update --file environment.yml
    conda activate example-partis

(Pro tip: use [mamba](https://mamba.readthedocs.io/en/latest/index.html) as a
drop-in replacement for conda for an enormous speed-up of dependency
resolution.)

Setting up partis itself:

    # Lots of additional software is supplied via git submodules, so we'll make
    # sure those are all available
    git submodule update --init --recursive
    # I had some trouble with the ham dependency as described here:
    # https://github.com/psathyrella/ham/issues/16
    # manually copying these files below is a workaround that doesn't requiring
    # modifying anything
    mkdir -p partis/packages/ham/_build
    cp partis/packages/ham/{src/*.cc,_build}
    ( cd partis && ./bin/build.sh with-simulation )
    # Dependencies are installed in $CONDA_PREFIX but partis itself lives right
    # here, so we'll put it on the PATH
    export PATH="$PATH:$PWD/partis/bin"

Compiling all the different components via the build script can take a while.

The PATH change could be [incorporated into the conda environment](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html),
too, instead of being manual.  Also, the partis repo is huge (> 1 GB with all
submodules included) so I'm keeping a single copy separate from the conda
environments (though in this example it's contained right here as a
subdirectory).

## Usage

Running all tests (paired/non-paired, slow/non-slow) with unlimited print width
so we can see everything, and from a subshell inside the partis repository
since the test script expects that:

    $ ( cd partis && ./test/test.py --run-all --print-width 0)
    run ./test/test.py --print-width 0
    cache-parameters-simu            .../example-partis/partis/bin/partis cache-parameters --dont-write-git-info --infname test/ref-results/test/simu.yaml --parameter-dir test/new-results/test/parameters/simu --sw-cachefname test/new-results/test/parameters/simu/sw-cache.yaml --is-simu --random-seed 1 --n-procs 10
    annotate-new-simu                .../example-partis/partis/bin/partis annotate --dont-write-git-info --infname test/ref-results/test/simu.yaml --parameter-dir test/new-results/test/parameters/simu --sw-cachefname test/new-results/test/parameters/simu/sw-cache.yaml --plot-annotation-performance --is-simu --plotdir test/new-results/annotate-new-simu-annotation-performance --only-csv-plots --random-seed 1 --n-procs 10 --outfname test/new-results/annotate-new-simu.yaml
    ...

It complains a little for me about changes in runtimes and counts of output
files, but everything seems to run OK in the end.  I wasn't totally clear from
the docs how to supply paired heavy and light chain sequences, but the test
script includes an example that scrolls by as it runs:

    run ./test/test.py --print-width 0 --paired
    cache-parameters-simu            .../example-partis/partis/bin/partis cache-parameters --dont-write-git-info --paired-loci --paired-indir test/paired/ref-results/test/simu --parameter-dir test/paired/new-results/test/parameters/simu --is-simu --random-seed 1 --n-procs 10
    partition-new-simu               .../example-partis/partis/bin/partis partition --dont-write-git-info --paired-loci --paired-indir test/paired/ref-results/test/simu --parameter-dir test/paired/new-results/test/parameters/simu --plot-annotation-performance --is-simu --plotdir test/paired/new-results/partition-new-simu-annotation-performance --only-csv-plots --no-partition-plots --random-seed 1 --n-procs 10 --paired-outdir test/paired/new-results/partition-new-simu
        chose seed uids 4353653356901731430-igh 4353653356901731430-igk with loci igh igk from cluster with size 6 (was asked for size in [5, 10])
    seed-partition-new-simu          .../example-partis/partis/bin/partis partition --dont-write-git-info --paired-loci --paired-indir test/paired/ref-results/test/simu --parameter-dir test/paired/new-results/test/parameters/simu --is-simu --seed-unique-id 4353653356901731430-igh:4353653356901731430-igk --seed-loci igh:igk --random-seed 1 --n-procs 10 --paired-outdir test/paired/new-results/seed-partition-new-simu
    get-selection-metrics-new-simu   .../example-partis/partis/bin/partis get-selection-metrics --dont-write-git-info --paired-loci --random-seed 1 --n-procs 10 --paired-outdir test/paired/new-results/partition-new-simu --chosen-ab-fname test/paired/new-results/get-selection-metrics-new-simu-chosen-abs.csv
    cache-parameters-data            .../example-partis/partis/bin/partis cache-parameters --dont-write-git-info --paired-loci --paired-indir test/paired-data --parameter-dir test/paired/new-results/test/parameters/data --n-max-queries 50 --random-seed 1 --n-procs 10
    simulate                         .../example-partis/partis/bin/partis simulate --dont-write-git-info --paired-loci --parameter-dir test/paired/new-results/test/parameters/data --n-sim-events 10 --n-trees 10 --n-leaf-distribution geometric --n-leaves 5 --min-observations-per-gene 5 --random-seed 1 --n-procs 10 --paired-outdir test/paired/new-results/test/simu --indel-frequency 0.2

Some relevant options:

      --paired-loci         Set this if input contains sequences from more than
                            one locus (igh+igk+igl all together). Input can be
                            specified either with --infname (in which case it
                            will be automatically split apart by loci), or with
                            --paired-indir (whose files must conform to the
                            same conventions). It will then run the specified
                            action on each of the single locus input files, and
                            (if specified) merge the resulting partitions.
                            (default: False)
      --infname INFNAME     input sequence file in .fa, .fq, .csv, or partis
                            output .yaml (if .csv, specify id string and
                            sequence headers with --name-column and
                            --seq-column) (default: None)
      --paired-indir PAIRED_INDIR
                            Directory with input files for use with
                            --paired-loci.  Must conform to file naming
                            conventions from bin/split-loci.py (really
                            paircluster.paired_dir_fnames()), i.e. the files
                            generated when --infname and --paired-loci are set.
                            (default: None)
      --input-metafnames INPUT_METAFNAMES
                            colon-separated list of yaml/json files, each of
                            which has meta information for the sequences in
                            --infname (and --queries-to-include-fname, although
                            that file can also include its own input meta
                            info), keyed by sequence id. If running multiple
                            steps (e.g.  cache-parameters and partition), this
                            must be set for all steps. See
                            https://github.com/psathyrella/partis/blob/master/docs/subcommands.md#input-meta-info
                            for an example. (default: None)

As hinted at by the help text for paired-indir, the input-metafnames option is
the way to supply pairing information when giving paired input in a FASTA via
`--infname` (the alternative being the already-split-and-prepared version
supplied via `--paired-indir`, where the metadata is supplied by YAML in that
directory).  In the tests, cache-parameters is explicitly run as a separate
step from those that follow with the already-prepared input directory
test/paired/ref-results/test/simu.

The docs example for `--input-metafnames` gives more general metadata examples,
but for paired sequences specifically, meta.yaml in the simu directory above
shows the way:

    {
      "3652967728590454646-igh":
        { "paired-uids": ["3652967728590454646-igk"], "locus": "igh" },
      "1161960272708420805-igk":
        { "paired-uids": ["1161960272708420805-igh"], "locus": "igk" },
      "1161960272708420805-igh":
        { "paired-uids": ["1161960272708420805-igk"], "locus": "igh" },
    # ...
      "3652967728590454646-igk":
        { "paired-uids": ["3652967728590454646-igh"], "locus": "igk" },
    # ...
    }

This way each sequence in a pair references the other sequence.  For example,
`3652967728590454646-igh` has a single item in a list of `paired-uids`
pointing to `3652967728590454646-igk`, and vice versa, and each locus is given.
Those sequence IDs match what's in the all-seqs.fa file alongside the YAML.

So if we just had the all-seqs.fa and the meta.yaml, we can create the full
input directory from those.  The partis script has a few hardcoded instances of
`./bin/...` so we need to be inside the partis directory for this to work.

    $ cd partis
    $ partis cache-parameters --paired-loci \
        --infname test/paired/ref-results/test/simu/all-seqs.fa \
        --input-metafnames test/paired/ref-results/test/simu/meta.yaml \
        --parameter-dir ../paired-example-params \
        --paired-outdir ../paired-example-input
    run ./bin/extract-pairing-info.py test/paired/ref-results/test/simu/all-seqs.fa ../paired-example-input/meta.yaml
      extract_pairing_info(): read 126 sequences with 63 droplet ids
        droplet id separators (set automatically): -  indices: [0]
           e.g. uid '7842229920653554932-igl' --> droplet id '7842229920653554932' contig id 'igl'
        contigs per
          droplet     count   fraction
            2           63     1.000
    run ./bin/split-loci.py test/paired/ref-results/test/simu/all-seqs.fa --outdir ../paired-example-input --input-metafname ../paired-example-input/meta.yaml
      --input-metafnames: added meta info for 126 sequences from ../paired-example-input/meta.yaml: paired-uids
        read pairing info for 126 seqs from input meta file
      running vsearch on 126 sequences:
    ...etc...
    $ cd ..

Now there's a paired input directory (paired-example-input) and a parameters
directory (paired-example-params):

    $ ls paired-example-*
    paired-example-input:
    igh.fa  igh+igk  igh+igl  igk.fa  igl.fa  meta.yaml
    
    paired-example-params:
    igh  igk  igl

We can run a partition command with that already-prepared input directory:

    partis partition --paired-loci --paired-indir paired-example-input --parameter-dir paired-example-params --paired-outdir paired-example-outdir

By the end the file `paired-example-outdir/partition-igh.yaml` doesn't just
have partition info for heavy chain (igh) sequences, but is also updated for
any info from light chain (igk/igl) sequences too. (See
[here](https://github.com/psathyrella/partis/blob/master/docs/paired-loci.md#output-directory)).

So what's the file layout?  See the [output formats
documentation](https://github.com/psathyrella/partis/blob/master/docs/output-formats.md).

There are command-line options for the number of partitions to include
(`--n-partitions-to-write`) and for which partitions the event details should
be written (`--write-additional-cluster-annotations`).  I think the description
of the annotation list (the events section) and the partition list (the
partition section) implies they are consistently ordered, such that the first
list of dictionaries of rearrangement events corresponds to the first
partition.  But, note the disclaimer about not assuming the first partition
listed is the most likely one.  (Open question: if the default number of
partitions is 10, why do I only see one here?)

In this case I have one event list (one dictionary per recombination event)
corresponding to the one partition given.  To pick one example event, the heavy
chain and light chain sequence IDs (amongst other attributes) in the first
dictionary are:

    "unique_ids":
              [
                "0062966375468546659-igh",
                "0828116068712122954-igh",
                "5280291432076549965-igh",
                "0779828650680679142-igh",
              ],
    "paired-uids":
              [
                ["0062966375468546659-igk"],
                ["0828116068712122954-igk"],
                ["5280291432076549965-igk"],
                ["0779828650680679142-igk"],
              ],

This matches the first nested list within the one partition given lower in the file:

      "partitions":
        [
          {
            "n_procs": 1,
            "logprob": 0.0,
            "n_clusters": 12,
            "partition":
              [
                [
                  "0062966375468546659-igh",
                  "0828116068712122954-igh",
                  "5280291432076549965-igh",
                  "0779828650680679142-igh",
                ],

There are 12 total nested lists in that one partition with 12 total event
dictionaries to match.

What if we hadn't used paired information and just partitioned by heavy chain?
We can let it do the parameter setup again automatically instead of running
that in a separate command:

    $ partis partition --infname paired-example-input/igh.fa --outfname partition.yaml --write-full-yaml-output
      note: --parameter-dir not set, so using default: _output/paired-example-input_igh
      parameter dir does not exist, so caching a new set of parameters before running action 'partition': _output/paired-example-input_igh
    caching parameters
    ...

(I've been reformatting the more limited JSON/YAML output with nice whitespace,
but we can get a more human-readable output from partis automatically with the
`--write-full-yaml-output` argument).

Now we get only the `-igh` IDs mentioned in the partition file, and the
partitioning isn't quite the same.  For one thing we do get the default 10
items (why now, but not before?) but still just the annotation details for the
one best-looking partition.  If we were to give `--n-partitions-to-write 1` we
would just get the one list of nested sequence ID lists and the one set of
annotations.  But in any case the best partitioning here has 11 clusters
instead of the 12 we have from the paired analysis.

If I go back to the paired one and try to get more candidate partitions, I
still only get one in the output.  The log probability is also given as 0
(...100% probability?)?  Is this a consequence of some of the per-locus
detail being lost when merging information for the separate per-locus
partitionings together?

## Output Handling

Partis has a wonderfully information-rich command-line interface to the details
in these output files.

Summarizing the heavy-chain-only version:

    $ partis view-output --parameter-dir _output --outfname partition.yaml | less -SR
    partitions:
                   logprob   delta   index  clusters  n_procs 
                  -inf                  0      20        6        -  -        4027451380298081898-igh:1482346277797483476-ig...
                  -inf        nan       1      19        4        -  -        4027451380298081898-igh:1482346277797483476-ig...
                  -inf        nan       2      18        4        -  -        4027451380298081898-igh:1482346277797483476-ig...
                  -inf        nan       3      17        4        -  -        4027451380298081898-igh:1482346277797483476-ig...
                  -inf        nan       4      16        4        -  -        4027451380298081898-igh:1482346277797483476-ig...
                  -inf        nan       5      15        3        -  -        4027451380298081898-igh:1482346277797483476-ig...
                  -inf        nan       6      14        2        -  -        4027451380298081898-igh:1482346277797483476-ig...
                  -inf        nan       7      13        2        -  -        4027451380298081898-igh:1482346277797483476-ig...
                  -inf        nan       8      12        2        -  -        4027451380298081898-igh:1482346277797483476-ig...
          best*   -3252.29    inf       9      11        1        -  -        4027451380298081898-igh:1482346277797
    annotations:
    ...

So the best partition is the last one there, with the highest probability.
That's what the one set of annotations in the file refer to.  Lower down in the
output the sequence content with boundaries of V/D/J/nontemplated nucleotides
are shown, invariant codons highlighted, etc.

For the paired version:

    $ partis view-output --parameter-dir paired-example-params --outfname paired-example-outdir/partition-igh.yaml | less -SR

It shows the one partition identified, with the same sort of sequence
annotations given as for the heavy-chain-only version.

The parse-output script can convert a partition info file to other formats,
like FASTA:

    $ parse-output.py partition.yaml partition.fa
      found 11 clusters in best partition
        taking all 11 clusters
      writing 63 sequences to partition.fa

I should be able to get AIRR output with that, too, but it crashes for me when
I try:

    $ parse-output.py --airr-output partition.yaml partition.tsv
      found 11 clusters in best partition
        taking all 11 clusters
      writing 11 sequences to partition.tsv
       writing airr annotations to partition.tsv
    Traceback (most recent call last):
      File "./partis/bin/parse-output.py", line 179, in <module>
        utils.write_airr_output(args.outfile, annotation_list, cpath=cpath, extra_columns=args.extra_columns, skip_columns=args.skip_columns)
      File "/data/home/jesse/dev/examples/example-partis/partis/python/utils.py", line 1649, in write_airr_output
        aline = get_airr_line(line, iseq, partition=None if cpath is None else cpath.partitions[cpath.i_best], extra_columns=extra_columns, skip_columns=skip_columns, debug=debug)
      File "/data/home/jesse/dev/examples/example-partis/partis/python/utils.py", line 1566, in get_airr_line
        aline[akey] = get_droplet_id(pline['unique_ids'][iseq])
    TypeError: get_droplet_id() takes at least 3 arguments (1 given)

In any case the existing JSON/YAML files should be easy enough to parse as needed.

Remaining questions:

 * Why the one-partition behavior and logprob of 0 with `--paired-loci`?
 * Why does `--airr-output` crash?

[partis]: https://github.com/psathyrella/partis
