# CHTKC software documentation

CHTKC is a k-mer counting software written in C language. It is based on a lock-free hash table which uses chaining to resolve collisions.


## Installation

The latest release of CHTKC source code can be downloaded from [github](https://github.com/wjnjlcn/chtkc/releases).

To compile CHTKC, please ensure `zlib` (to support gzip-compressed inputs) and `cmake` (version 3.0.2 or higher) are installed on the target system.

The source code can be compiled using:

```shell
mkdir build
cd build
cmake ..
make
```

The `build` directory will contain two versions of CHTKC binary files:

* `chtkc`: normal version.
* `chtkco`: optimized version.


!!! Note
    Under small RAM usage, `chtkco` has better performance, because it can allocate more nodes than `chtkc`. However, `chtkco` can only use up to 4G (4294967295) nodes, when the available memory is big enough, `chtkc` may instead allocate more nodes than `chtkco`. For example, when counting 28-mers, if the RAM usage is over 150 GB, please consider using `chtkc`.

## Usage

`chtkc` and `chtkco` share the same usage, take `chtkc` for example, the basic usage is as follows:

```shell
chtkc <CMD> [OPTION...] ARGS...
```

Currently, `chtkc` supports three subcommands, `<CMD>` can be `count`, `histo` or `dump`.

### Couting k-mers

`chtkc count` is the command to do k-mer counting, the usage is as follows:

```shell
chtkc count [OPTION...] FILE...
```

* `-k, --kmer-len` - the length of k-mer.
* `-m, --mem` - The memory size to be used during the k-mer counting process. In bytes, can be specified using `M/G` for convenient.
* `-o, --out` - The path of output file.
* `-t, --threads` - The number of threads to be used.
* `--fa` - Specify the input files are in `FASTA` format, cannot be used with `--fq`.
* `--fq` - Specify the input files are in `FASTQ` format, cannot be used with `--fa`.
* `--gz` - Specify the input are `gzip` compressed files.
* `--count-max` - The maximum k-mer frequency value when counting, frequencies larger than this value will be recorded as this value. `Default: 255`
* `--filter-max` - K-mers with frequency larger than this value will not be exported into the final result.
* `--filter-min` - K-mers with frequency smaller than this value will not be exported into the final result. `Default: 2`
* `--log` - The path of the log file.
* `--bs` - The size of read buffers, increasing the value (over 5000000) can improve the performance when counting `gzip` files, in bytes.
* `--rt` - The number of reading threads, this can be considered to be used when counting for multiple `gzip` input files.
* `-?, --help` - Print the usage of `chtkc`.  


### Generate histogram for k-mer counting results

`chtkc histo` produces a file containing the number of k-mers for each frequency value, the usage is as follows:

```shell
chtkc histo [OPTION...] RESULT
```

* `-o, --out` The path of the output file.

### Output ASCII format of k-mer counting results

`chtkc dump` outputs the ASCII format of the k-mer counting results, the usage is as follows:

```shell
chtkc dump [OPTION...] RESULT
```

* `-o, --out` The path of the output file.

### Examples

An example dataset can be downloaded [here](https://github.com/wjnjlcn/chtkc/raw/master/examples/test.fq), and some usage examples are as follows:

```shell
chtkc count -k 28 -m 500M --fq -o result test.fq
chtkc count -k 55 -m 50G -t 16 --fq -o result --filter-min=1 test.fq --log=test.log
chtkc count -k 28 -m 500M --fq --gz -o result test.fq.gz
chtkc histo -o histo.txt result
```

## Output

`chtkc count` outputs a binary file as result, containing each k-mer and its frequency.

The output file contains a 32 bytes header recording basic counting parameters, the k-mer and frequency information comes after the header. All the `<k-mer, frequency>` pair are concatenated with each other.

The occupation space of a `<k-mer, frequency>` pair is determined by the length of k-mer and the maximum count value (specified by `--count-max` option). `A`, `C`, `G` and `T` are encoded as `00`, `01`, `10` and `11`. Either the k-mer sequence and its frequency is encoded in **Little-endian** format. For example:

* If counting for 21-mers (`--count-max=255`),  `CTCGAACGCCGATTAATGGCC: 35` is stored as `10100101 11000011 01100011 00011001 11011000 00000001 00100011` (The last byte stores the frequency value).

* If counting for 41-mers (`--count-max=255`), `CTCGAACGCCGATTAATGGTTTGCAGGTGCCCATGTTTGAA: 35` is stored as `11100000 11101111 01010100 10101110 11100100 10101111 11000011 01100011 00011001 11011000 00000001 00100011` (The last byte stores the frequency value).

* If counting for 41-mers (`--count-max=400`), `CTCGAACGCCGATTAATGGTTTGCAGGTGCCCATGTTTGAA: 335` is stored as `11100000 11101111 01010100 10101110 11100100 10101111 11000011 01100011 00011001 11011000 00000001 01001111 00000001` (The last two byte store the frequency value).
