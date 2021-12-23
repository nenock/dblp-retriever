# dblp-retriever

Retrieve paper metadata from conference proceedings and journals indexed in DBLP.
Currently, retrieval of the following properties is supported:

* paper title
* authors with pids from DBLP
* heading (corresponds to journal issue or conference session name)
* page range (optional)
* paper length (optional)
* link to electronic edition of paper
* link to dblp entry of paper within the venue (paper_id)

The tool validates the page ranges and adds a log message to column `comment` in case possible inconsistencies are detected (optional).

[![DOI](https://zenodo.org/badge/166964646.svg)](https://zenodo.org/badge/latestdoi/166964646)

# Setup

Python 3 is required. The dependencies are specified in `requirements.txt`.
To install those dependencies execute:

    pip3 install -r requirements.txt

**Optional:** Setup virtual environment with [pyenv](https://github.com/pyenv/pyenv) 
and [virtualenv](https://github.com/pyenv/pyenv-virtualenv) before executing the above command:

    pyenv install 3.7.2
    pyenv virtualenv 3.7.2 dblp-retriever_3.7.2
    pyenv activate dblp-retriever_3.7.2
    
    pip3 install --upgrade pip

# Usage

Basic usage:

    python3 dblp-retriever.py -i <path_to_input_file> -o <path_to_output_dir>

Call without parameters to get information about possible parameters:

    python3 dblp-retriever.py
    
    usage: dblp-retriever.py [-h] -i INPUT_FILE -o OUTPUT_DIR [-d DELIMITER]
    dblp-retriever.py: error: the following arguments are required: -i/--input-file, -o/--output-dir


# Configuration

As input, the tool expects a CSV file with the following three columns: `venue`, `year` (optional), and `identifier`.
Column `venue` is a custom name for the conference or journal, `year` should be self-explanatory, and `identifier` is 
the DBLP identifier of a particular journal volume or conference proceeding.
If no year is provided, the year is attempted to be derived for each paper on the page of the venue separately. Use this
feature, if you have to deal with inter-annual journals or conferences.

The identifier can be extracted from the DBLP-URL as follows.
In this example, we extract `conf/sigsoft/fse2018` as the identifier of the ESEC/FSE 2018 proceedings:

[![dblp-identifier](doc/dblp-identifier.png "DBLP Identifier")](https://dblp.org/db/conf/sigsoft/fse2018.html)

An exemplary input file can be found [here](input/AI.csv):

| venue | year | identifier         | comment |
|-------|------|--------------------|---------|
| AAAI  |      | conf/aaai/aaai80   |         |
| ...   | ...  | ...                |         |
| IJCAI |      | conf/ijcai/ijcai69 |         |
| ...   | ...  | ...                |         |

To retrieve the paper metadata for the configured venues, you just need to run the following command:

    python3 dblp-retriever.py -i input/AI.csv -o output/

The tool logs the retrieval process:

    2021-12-23 20:12:53,845 dblp-retriever_logger INFO: Reading venues from input/AI.csv...
    2021-12-23 20:12:53,862 dblp-retriever_logger INFO: 67 venues have been imported.
    2021-12-23 20:12:54,141 dblp-retriever_logger INFO: Successfully retrieved TOC of venue: conf/aaai/aaai80
    2021-12-23 20:12:54,402 dblp-retriever_logger INFO: Successfully parsed TOC of venue: conf/aaai/aaai80 with a paper number of: 95
    2021-12-23 20:12:54,640 dblp-retriever_logger INFO: Successfully retrieved TOC of venue: conf/aaai/aaai82
    2021-12-23 20:12:54,922 dblp-retriever_logger INFO: Successfully parsed TOC of venue: conf/aaai/aaai82 with a paper number of: 104
    ...
    2021-12-23 20:14:28,696 dblp-retriever_logger INFO: Successfully retrieved TOC of venue: conf/ijcai/ijcai2020
    2021-12-23 20:14:31,147 dblp-retriever_logger INFO: Successfully parsed TOC of venue: conf/ijcai/ijcai2020 with a paper number of: 778
    2021-12-23 20:14:31,155 dblp-retriever_logger INFO: Exporting papers to output/AI.csv...
    2021-12-23 20:14:32,381 dblp-retriever_logger INFO: 23454 papers have been exported.

And writes the [retrieved data](output/AI.csv) to the configured output directory:

| venue | year | identifier             | heading                            | paper_id                                                             | title                                                                                       | authors                                                                                                       | electronic_edition                                   |
|-------|------|------------------------|------------------------------------|----------------------------------------------------------------------|---------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|------------------------------------------------------|
| AAAI  | 1980 | conf/aaai/aaai80       | Early Vision Processing            | https://dblp.org/db/conf/aaai/aaai80.html#conf/aaai/Witkin80         | A Statistical Technique for Recovering Surface Orientation from Texture in Natural Imagery. | 27/4344: Andrew P. Witkin                                                                                     | http://www.aaai.org/Library/AAAI/1980/aaai80-001.php |
| …     | …    | …                      | …                                  |                                                                      | …                                                                                           | …                                                                                                             | …                                                    |
| IJCAI | 2020 | conf/ijcai/ijcai2020   | Demos                              | https://dblp.org/db/conf/ijcai/ijcai2020.html#conf/ijcai/ZhangZCMM20 | AI-Powered Oracle Bone Inscriptions Recognition and Fragments Rejoining.                    | 82/4043: Chongsheng Zhang; 269/4546: Ruixing Zong; 99/8875: Shuang Cao; 269/4627: Yi Men; 269/4545: Bofeng Mo | https://doi.org/10.24963/ijcai.2020/779              |

If you want to retrieve paper metadata for several years of a venue, please carefully check the log for errors.
A common cause for errors is DBLP's changing way IDs of venues are built:

| venue | year | identifier           |
|-------|------|----------------------|
| ...   | ...  | ...                  |
| CHI   | 1991 | conf/chi/**chi1991** |
| CHI   | 1992 | conf/chi/**chi92**   |
| CHI   | 1993 | conf/chi/**chi1993** |
| CHI   | 1994 | conf/chi/**chi1994** |
| CHI   | 1995 | conf/chi/**chi95**   |
| ...   | ...  | ...                  |
