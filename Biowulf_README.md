# MetaErg

Dong, X., & Strous, M. (2019). An Integrated Pipeline for Annotation and Visualization of Metagenomic Contigs. *Frontiers in Genetics*, *10*, 999. https://doi.org/10.3389/fgene.2019.00999

https://github.com/xiaoli-dong/metaerg

- Put modulefiles dir below in your [`MODULEPATH`](https://lmod.readthedocs.io/en/latest/020_advanced.html) .  (i.e. can also copy module file to your own folder named /yourdir/MetaErg/1.2.0.lua , then do `module use /yourdir/` ).  Then you can load the module and run:

```bash
module use /data/BCBB_microbiome_db/modulefiles ## only needed for module path
module load MetaErg/1.2.0
metaerg.pl --dbdir /data/BCBB_microbiome_db/metaerg_db/db contig.fasta
```



### Modulefile MetaErg/1.2.0.lua:

```lua
local app = myModuleName()
local version = myModuleVersion()
local base = pathJoin("/data/BCBB_microbiome_db/software",app)


help([[
MetaErg https://github.com/xiaoli-dong/metaerg
This module help: /data/BCBB_microbiome_db/software/MetaErg/1.2.0/Biowulf_README.md
      ]])
whatis("MetaErg")
whatis("Version: " .. version)
whatis("URL: https://github.com/xiaoli-dong/metaerg")

always_load("diamond/2.0.5")
always_load("blast/2.11.0+")
always_load("hmmer/3.3.1")
always_load("prodigal/2.6.3")
always_load("java/12.0.1") -- needed for minced
always_load("glpk/4.65") -- needed for MinPath
always_load("perl/5.24.3")


prepend_path("PATH", pathJoin(base,version,"bin"))
prepend_path("PATH", pathJoin(base,version,"src/MinPath-master"))
prepend_path("MANPATH", pathJoin(base,version,"share", "man"))
prepend_path("MANPATH", pathJoin(base,version,"man"))
prepend_path("PERL5LIB", pathJoin(base,version,"lib", "perl5"))
setenv("MinPath", pathJoin(base,version,"src/MinPath-master"))

if (mode() == "load") then
   LmodMessage("[+] Loading ", app, " ", version)
elseif (mode() == "unload") then
   LmodMessage("[-] Unloading ", app, " ", version)
end
```



- I use Biowulf modules where I can, and otherwise follow MetaErg [Dockerfile](https://github.com/xiaoli-dong/metaerg/blob/master/Dockerfile)

- I download MetaErg repo at  https://github.com/xiaoli-dong/metaerg/commit/e679b7e131f974e53fcbc9cd9032917c9f05becf latest commit to */data/BCBB_microbiome_db/MetaErg/1.2.0* .  **Update:** see [my fork](https://github.com/pooranis/metaerg)

  - Call this directory ***$DIR*** in these notes
  - make version ~~1.0.4 as this is the version in the Dockerfile~~ 1.2.0 is version when you do `metaerg.pl --help`

- Install perl modules - we set PERL5LIB in modulefile ^

  - Download metaerg repo and search for individual bioperl modules because [installing all bioperl fails ](https://github.com/bioperl/Bio-Procedural/issues/3)

  - The following modules are sufficient

    - Bio::Annotation::Collection
    - DBI 
    - Archive::Extract
    - DBD::SQLite
    - File::Copy::Recursive
    - Bio::DB::EUtilities
    - LWP::Protocol::https

  - Install libs like so:

```bash
cpanm -l $DIR Bio::Annotation::Collection
```

  - Also have to install SWISS .  According to Dockerfile

```bash
git clone https://git.code.sf.net/p/swissknife/git swissknife-git
cd swissknife-git
perl Makefile.PL PREFIX=$DIR
make
make test
make install
```

- Download [ARAGORN](http://www.ansikte.se/ARAGORN/Downloads/) to \$DIR/src and compile according to instructions.  I use aragorn1.2.41.c .  Move binary to $DIR/bin
- Download from [minCED releases 0.4.2](https://github.com/ctSkennerton/minced/releases/tag/0.4.2) minced script and minced.jar to $DIR/bin. Needs Java 2.12 and up to run as in modulefile
- Download ~~[MinPath repo link exact commit](https://github.com/mgtools/MinPath/tree/8fcbf8823f0b3d0c9279f8be407c8caa0e55ed47)~~ [my fork of MinPath](https://gitlab.com/pooranis/minpath) to \$DIR/src .  
- Download [Phobius 1.01](https://phobius.sbc.su.se/) which it seems MetaErg is [now using](https://github.com/xiaoli-dong/metaerg/commit/406f9e4a41a54643fedadf958cc550a7ac311846) instead of signalp and tmhmm.  Contrary to the phobius documentation, MetaErg calls it as `phobius.pl` instead of `phobius` So I linked both from \$DIR/src/phobius to the \$DIR/bin directory.
  - (??? Maybe not needed any more) To Do: Ask biowulf to uncouple signalP from trinotate, until then download signalP 5.0b Linux x86_64 and TMHMM 2.0c  from [DTU software](https://services.healthtech.dtu.dk/software.php) to \$DIR/src and link in \$DIR/bin and \$DIR/lib.  Change shebangs of perl scripts in tmhmm to `#!/usr/bin/env perl`(???)
- I fix small database path errors in [my fork of metaerg](https://github.com/pooranis/metaerg)
- Check tools are installed with [`check_tools.pl`](https://github.com/pooranis/metaerg/blob/master/bin/check_tools.pl)

### Databases

- Downloaded 2/12/2021 to */data/BCBB_microbiome_db/metaerg_db*

```bash
wget http://ebg.ucalgary.ca/metaerg/db.tar.gz
```

