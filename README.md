# WIN2PhaseNet

## Summary 
Program to make various data for PhaseNet (Zhu and Beroza, 2019) from WIN waveform file and pick list.

## Output format
### 1. **train** mode
* npz waveform files: [OUTDIR]/[yymmddhhmmss from win_name]_[station].npz

| Key | Description |
| --- | --- |
| `data` | - event waveform data of one event / one station <br> - dataShape: **(9000, 3)** # means 90 seconds (100Hz) / 3 compornent <br> - data starts **30 seconds** before of 'itp'|
| `itp` | file path of Pick list |
| `its` | directory path of output npz waveform files |

* npz waveform list: ./npz.csv

### 2. **test** mode
* npz waveform files: [OUTDIR]/[yymmddhhmmss from win_name]_[station].npz

| Key | Description |
| --- | --- |
| `data` | - event waveform data of one event / one station <br> - dataShape: **(3000, 3)** # means 30 seconds (100Hz) / 3 compornent <br> - data starts **1 seconds** before of 'itp'|
| `itp` | file path of Pick list |
| `its` | directory path of output npz waveform files |

* npz waveform list: ./npz.csv

### 3. **cont** mode
* npz waveform files: [OUTDIR]/[yymmddhhmmss from win_name]_[station].npz

| Key | Description |
| --- | --- |
| `data` | - continuous waveform data of one station <br> - dataShape: **(3000, 3)** # means 30 seconds (100 Hz) / 3 compornent |

* npz waveform list: ./npz.csv

## Requirements
### 1. Input files
#### 1. Common for all modes
* This program
    * Copy to the local environment by **git clone**
    ```
    $ git clone https://github.com/RintarohSuzuki/WIN2PhaseNet.git
    ```

* Channel table
    * format:
        * txt format
            * For the detailed information, see https://wwweic.eri.u-tokyo.ac.jp/WIN/man.ja/win.html (only in Japanese)

    * Put the files at `./etc/stn.tbl`
        * You can change this path by `--chtbl` option

#### 2. Only for **train** and **test** mode
* Event WIN waveform files
    * format:
        * 'WIN' format
            * For the detailed information, see https://wwweic.eri.u-tokyo.ac.jp/WIN/man.en/winformat.html
        * **File name should be "Tyymmddhhmmss.dat"**
            * e.g.'T170716192301.dat'
            * This is used for output file name
        * **Only 100 Hz data is acceptable**

    * Make directry named `./data` and put the files there
        * You can change this path by `--indir` option

* Pick list
    * format:
        * csv format

        | Column | Description |
        | --- | --- |
        | `win_name` | the file name of a WIN waveform file |
        | `station` | station code |
        | `itp` | the data point of **P phase** from the start of each WIN waveform file |
        | `its` | the data point of **S phase** from the start of each WIN waveform file |

        * Sample: picks.csv
    * Put the file at `.`
    * **Only data from the station with BOTH P phase and S phase is processed**

#### 3. Only for **cont** mode
* Continuous WIN waveform files
    * format:
        * 'WIN' format
            * For the detailed information, see https://wwweic.eri.u-tokyo.ac.jp/WIN/man.en/winformat.html
        * **File name should be "Tyymmddhhmmss.dat"**
            * e.g.'T170716192301.dat'
            * This is used for output file name
        * **Only 30 seconds and 100 Hz data is acceptable**

    * Make directry named `./data` and put the files there
        * You can change this path by `--indir` option

### 2. Environments
* docker
* docker image tar file<br>
Download the docker image tar file ('win2npz-image.tar') from here:<br>
https://drive.google.com/file/d/13iTKSCtTa3yQEu6uKPZCCk6xVCTpAM0h/view?usp=sharing<br>
Put 'win2npz-image.tar' to `images/` directory.

* docker image tar file<br>
Download the docker image tar file ('phasenet-image.tar') from here:<br>
https://drive.google.com/file/d/1A6JjboZAIoboI-Y8enc-FxnHPiQuofQZ/view?usp=sharing<br>
Put 'phasenet-image.tar' to `WIN2PhaseNet/images/` directory.

## How to use
### 1. Pre-setting
* Set following option according to the situation

| Option | Description |
| --- | --- |
| `--mode {train,test,cont}` | specify the mode (see 'Output format' for the detailed infomation) |
| `--list LIST` | file path of Pick list (Required only for **train** and **test** mode) |
| `[--outdir OUTDIR]` | directory path of output npz waveform files (default: `./out`) |
| `[--indir INDIR]` | directory path of Win waveform files (default: `./data`) |
| `[--chtbl CHTBL]` | directory path of channel table file (default: `./etc/stn.tbl`) |

### 2. Execute WIN2PhaseNet
```
# load docker image and run the container
$ docker-run.bash

# run WIN2PhaseNet
(container)$ python3 src/win2npz.py --mode {train,test,cont} --list LIST [--indir INDIR] [--outdir OUTDIR]
# e.g. 
# (container)$ python3 src/win2npz.py --mode train --list picks.csv
# (container)$ python3 src/win2npz.py --mode test --list picks.csv
# (container)$ python3 src/win2npz.py --mode cont
```

### 3. Execute PhaseNet prediction
This program **NOT** contains PhaseNet but show how to use PhaseNet briefly<br>
Npz data made by **'cont' mode** of WIN2PhaseNet is required in advance

#### Download PhaseNet and model
```
$ git clone -b release https://github.com/AI4EPS/PhaseNet.git
$ git checkout master model
```

#### Execute prediction
```
# load docker image and run the container
$ cd WIN2PhaseNet
$ docker-run.bash phasenet

# run PhaseNet
(container)$ python run.py --mode=pred --ckdir=model/190703-214543 --data_dir=<OUTDIR path of WIN2PhaseNet> --data_list=<npz.csv path of WIN2PhaseNet output> --output_dir=output --save_result
# e.g. 
# (container)$ python run.py --mode=pred --ckdir=model/190703-214543 --data_dir=../WIN2PhaseNet/out --data_list=../WIN2PhaseNet/npz.csv --output_dir=output --save_result
```

## Acknowledgements
A part of this program was created by Uchida, N and Matsuzawa, T.

## References
* 斎藤 正徳 (1978): 漸化式ディジタルフィルターの自動設計, 物理探鉱, 31, 240-263. (In Japanese)
* Takagi, R., Uchida, N., Nakayama, T., Azuma, R., Ishigami, A., Okada, T., Nakamura, T., & Shiomi, K. (2019), Estimation of the orientations of the S-net cabled ocean-bottom sensors. Seismological Research Letters, 90(6), 2175–2187. https://doi.org/10.1785/0220190093
* Zhu, W., & Beroza, G. C. (2019), PhaseNet: A deep-neural-network-based seismic arrival-time picking method. Geophysical Journal International, 216(1), 261–273. https://doi.org/10.1093/gji/ggy423