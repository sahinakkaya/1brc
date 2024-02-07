# 1BRC: One Billion Row Challenge in Python

This is fork of https://github.com/ifnesi/1brc. See [Changes](#changes)
section to know what is changed.

Python implementation of Gunnar's 1 billion row challenge:
- https://www.morling.dev/blog/one-billion-row-challenge
- https://github.com/gunnarmorling/1brc

## Creating the measurements file with 1B rows

First install the Python requirements:
```
python3 -m pip install -r requirements.txt
```

The script `createMeasurements.py` will create the measurement file:
```
usage: createMeasurements.py [-h] [-o OUTPUT] [-r RECORDS]

Create measurement file

options:
  -h, --help            show this help message and exit
  -o OUTPUT, --output OUTPUT
                        Measurement file name (Default is measurements.txt)
  -r RECORDS, --records RECORDS
                        Number of records to create (Default is 1000000000)
```

Example:
```
% python3 createMeasurements.py
Creating measurement file 'measurements.txt' with 1,000,000,000 measurements...
 - Wrote 10,000,000 measurements in 8.92 seconds
 - Wrote 20,000,000 measurements in 17.82 seconds
 - Wrote 30,000,000 measurements in 26.73 seconds
 - Wrote 40,000,000 measurements in 35.54 seconds
 - Wrote 50,000,000 measurements in 44.36 seconds
 - Wrote 60,000,000 measurements in 53.07 seconds
 .
 .
 .
 - Wrote 980,000,000 measurements in 880.98 seconds
 - Wrote 990,000,000 measurements in 889.99 seconds
Created file 'measurements.txt' with 1,000,000,000 measurements in 898.92 seconds
```

Be patient as it can take more than 15 minutes to have the file generated.

Maybe as another challenge is to speed up the generation of the measurements file :slightly_smiling_face:

## Performance (on a MacBook Pro M1 32GB)
| Interpreter | Script | user | system | cpu | total |
| ----------- | ------ | ---- | ------ | --- | ----- |
| python3 | calculateAveragePolars.py | 77.84 | 3.64 | 703% | 11.585 |
| pypy3 | calculateAveragePypy.py | ~~139.15~~<br>135.25 | ~~3.02s~~<br>2.92 | ~~699%~~<br>735% | ~~20.323~~<br>18.782 |
| python3 | calculateAverageDuckDB.py | 186.78 | 4.21 | 806% | 23.673 |
| pypy3 | calculateAverage.py | ~~284.90~~<br>242.89 | ~~9.12~~<br>6.28 | ~~749%~~<br>780% | ~~39.236~~<br>31.926 |
| python3 | calculateAverage.py | ~~378.54~~<br>329.20 | ~~6.94~~<br>3.77 | ~~747%~~<br>793% | ~~51.544~~<br>41.941 |
| python3 | calculateAveragePypy.py | ~~573.77~~<br>510.93 | ~~2.70~~<br>1.88 | ~~787%~~<br>793% | ~~73.170~~<br>64.660 |

The script `calculateAveragePolars.py` was suggested by [Taufan](https://github.com/mtaufanr) on this [post](https://github.com/gunnarmorling/1brc/discussions/62#discussioncomment-8026402).

The script `calculateAveragePypy.py` was created by [donalm](https://github.com/donalm), a +2x improved version of the initial script (`calculateAverage.py`) when running in pypy3, even capable of beating the implementation using [DuckDB](https://duckdb.org/) `calculateAverageDuckDB.py`.

[Olivier Scalbert](https://github.com/oscalbert) has made a simple but incredible suggestion where performance increased by an average of 15% (table above has been updated), thank you :slightly_smiling_face:

His suggestions were to change from:
```
if measurement < result[location][0]:
    result[location][0] = measurement
if measurement > result[location][1]:
    result[location][1] = measurement
result[location][2] += measurement
result[location][3] += 1
```

to:
```
_result = result[location]
if measurement < _result[0]:
    _result[0] = measurement
if measurement > _result[1]:
    _result[1] = measurement
_result[2] += measurement
_result[3] += 1
```

Python can be surprising sometimes.

## Changes
In the original 1BRC you need to find min, mean, max for all stations. I thought it might be interesting challenge if we add one more statistics, standard deviation, and compare how it would affect the performance.

There were 4 implementations in the original repository.
```
- calculateAverage.py
- calculateAveragePolars.py
- calculateAverageDuckDB.py
- calculateAveragePypy.py
```

I added standard deviation calculations to first two implementations. 

In `calculateAverage.py`, input file is processed chunk by chunk. These chunks are then merged to get the final result. In the classical stddev formula, you use mean while processing your input list. But I didn't want to first calculate mean and then process the file again to calculate stddev, so I used [Welford's algorithm](https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance#Welford%27s_online_algorithm) which allowed me to calculate mean and stddev with a single iteration. Then I used the solution described [here](https://stackoverflow.com/questions/7753002/adding-combining-standard-deviations) to combine multiple stddev's into one.

For `calculateAveragePolars.py`, I didn't need to do much. I just used the function provided by `polars` library: `pl.std("measurement").alias("std_measurement")`.

And here are the results:

## Performance (on a MacBook Pro M2 Pro 16GB, input file: 100M rows)
| Interpreter |  Script | version | user | system | cpu | total |
| ----------- | ------ | ---- | ------ | --- | ----- | ---- |
| python3 | calculateAverage.py | original | 42.32 | 0.58 | 773% | 5.546 |
| python3 | calculateAverage.py | this-repo | ~~55.98~~<br>46.41 | ~~0.58~~<br>0.61 | ~~783%~~<br>767% | ~~7.219~~<br>6.126 |
| python3 | calculateAveragePolars.py | original | 11.11 | 0.55 | 707% | 1.648 |
| python3 | calculateAveragePolars.py | this-repo | 21.07 | 4.92 | 639% | 4.065 |


