[![Build Status](https://travis-ci.com/matrix-profile-foundation/go-matrixprofile.svg?branch=master)](https://travis-ci.com/matrix-profile-foundation/go-matrixprofile)
[![Windows Build Status](https://ci.appveyor.com/api/projects/status/tp7cqme05eytqw94?svg=true)](https://ci.appveyor.com/api/projects/status/tp7cqme05eytqw94?svg=true)
[![codecov](https://codecov.io/gh/matrix-profile-foundation/go-matrixprofile/branch/master/graph/badge.svg)](https://codecov.io/gh/matrix-profile-foundation/go-matrixprofile)
[![Go Report Card](https://goreportcard.com/badge/github.com/matrix-profile-foundation/go-matrixprofile)](https://goreportcard.com/report/github.com/matrix-profile-foundation/go-matrixprofile)
[![GoDoc](https://godoc.org/github.com/matrix-profile-foundation/go-matrixprofile?status.svg)](https://godoc.org/github.com/matrix-profile-foundation/go-matrixprofile)
[![License](https://img.shields.io/badge/License-Apache2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

# go-matrixprofile

Golang library for computing a matrix profiles and matrix profile indexes. Features also include time series discords, time series segmentation, and motif discovery after computing the matrix profile. Visit [The UCR Matrix Profile Page](https://www.cs.ucr.edu/~eamonn/MatrixProfile.html) for more details into matrix profiles.

## Contents
- [Installation](#installation)
- [Quick start](#quick-start)
- [Case Studies](#case-studies)
  * [Matrix Profile](#matrix-profile)
  * [Multi-Dimensional Matrix Profile](#multi-dimensional-matrix-profile)
- [Benchmarks](#benchmarks)
- [Contributing](#contributing)
- [Testing](#testing)
- [Other Libraries](#other-libraries)
- [Contact](#contact)
- [License](#license)
- [Citations](#citations)

## Installation
```sh
$ go get github.com/matrix-profile-foundation/go-matrixprofile
```

## Quick start
```go
// example_mp.go
package main

import (
	"fmt"

	mp "github.com/matrix-profile-foundation/go-matrixprofile"
)

func main() {
	sig := []float64{0, 0.99, 1, 0, 0, 0.98, 1, 0, 0, 0.96, 1, 0}

	p, err := mp.New(sig, nil, 4)
	if err != nil {
		panic(err)
	}

	if err = p.Compute(mp.NewComputeOpts()); err != nil {
		panic(err)
	}

	fmt.Printf("Signal:         %.3f\n", sig)
	fmt.Printf("Matrix Profile: %.3f\n", p.MP)
	fmt.Printf("Profile Index:  %5d\n", p.Idx)
}
```
```sh
$ go run example_mp.go
Signal:         [0.000 0.990 1.000 0.000 0.000 0.980 1.000 0.000 0.000 0.960 1.000 0.000]
Matrix Profile: [0.014 0.014 0.029 0.029 0.014 0.014 0.029 0.029 0.029]
Profile Index:  [    4     5     6     7     0     1     2     3     4]
```

## Case studies
### Matrix Profile
Going through a completely synthetic scenario, we'll cover what features to look for in a matrix profile, and what the additional Discords, TopKMotifs, and Segment tell us. We'll first be generating a fake signal that is composed of sine waves, noise, and sawtooth waves. We then run STOMP on the signal to calculte the matrix profile and matrix profile indexes.

![mpsin](https://github.com/matrix-profile-foundation/go-matrixprofile/blob/master/mp_sine.png)
subsequence length: 32

* signal: This shows our raw data. Theres several oddities and patterns that can be seen here. 
* matrix profile: generated by running STOMP on this signal which generates both the matrix profile and the matrix profile index. In the matrix profile we see several spikes which indicate that these may be time series discords or anomalies in the time series.
* corrected arc curve: This shows the segmentation of the time series. The two lowest dips around index 420 and 760 indicate potential state changes in the time series. At 420 we see the sinusoidal wave move into a more pulsed pattern. At 760 we see the pulsed pattern move into a sawtooth pattern.
* discords: The discords graph shows the top 3 potential discords of the defined subsequence length, m, based on the 3 highest peaks in the matrix profile. This is mostly composed of noise.
* motifs: These represent the top 6 motifs found from the time series. The first being the initial sine wave pattern. The second is during the pulsed sequence on a fall of the pulse to the noise. The third is during the pulsed sequence on the rise from the noise to the pulse. The fourth and fifth are the sawtooth patterns.

The code to generate the graph can be found in [this example](https://https://github.com/matrix-profile-foundation/go-matrixprofile/blob/master/example_caseStudy_test.go#L104).

### Multi-Dimensional Matrix Profile
Based on [4] we can extend the matrix profile algorithm to multi-dimensional scenario.

![mpkdim](https://github.com/matrix-profile-foundation/go-matrixprofile/blob/master/mp_kdim.png)
subsequence length: 25

* signal 0-2: the 3 time series dimensions
* matrix profile 0-2: the k-dimensional matrix profile representing choose k from d time series. matrix profile 1 minima represent motifs that span at that time across 2 time series of the 3 available. matrix profile 2 minima represents the motifs that span at that time across 3 time series.

The plots can be generated by running
```sh
$ make example
go test ./... -run=Example
ok  	github.com/matrix-profile-foundation/go-matrixprofile	0.256s
ok  	github.com/matrix-profile-foundation/go-matrixprofile/av	(cached) [no tests to run]
ok  	github.com/matrix-profile-foundation/go-matrixprofile/siggen	(cached) [no tests to run]
ok  	github.com/matrix-profile-foundation/go-matrixprofile/util	(cached) [no tests to run]
```
A png file will be saved in the top level directory of the repository as `mp_sine.png` and `mp_kdim.png`

## Benchmarks
```sh
BenchmarkMStomp-4                     	      39	  29853485 ns/op	 7336245 B/op	  227071 allocs/op
BenchmarkZNormalize-4                 	 7112282	       185 ns/op	     256 B/op	       1 allocs/op
BenchmarkMovmeanstd-4                 	   89810	     13628 ns/op	   32768 B/op	       4 allocs/op
BenchmarkCrossCorrelate-4             	   15190	     75262 ns/op	   24584 B/op	       3 allocs/op
BenchmarkMass-4                       	   15421	     78660 ns/op	   24842 B/op	       4 allocs/op
BenchmarkDistanceProfile-4            	   15190	     79092 ns/op	   24842 B/op	       4 allocs/op
BenchmarkCalculateDistanceProfile-4   	  220363	      4625 ns/op	       0 B/op	       0 allocs/op
BenchmarkStmp/m32_pts1k-4             	      15	  77806814 ns/op	24209736 B/op	    3892 allocs/op
BenchmarkStmp/m128_pts1k-4            	      16	  70673766 ns/op	22496294 B/op	    3508 allocs/op
BenchmarkStamp/m32_p2_pts1k-4         	      25	  46207243 ns/op	24284148 B/op	    3909 allocs/op
BenchmarkStomp/m128_p1_pts__1024-4    	     152	   7740858 ns/op	  196805 B/op	      28 allocs/op
BenchmarkStomp/m128_p2_pts__4096-4    	      13	  81826774 ns/op	 1116937 B/op	      39 allocs/op
BenchmarkStomp/m128_p2_pts_16384-4    	       1	1342203283 ns/op	 4776832 B/op	      45 allocs/op
BenchmarkStomp/m128_p4_pts_16384-4    	       1	1269550826 ns/op	 7153728 B/op	      67 allocs/op
BenchmarkStomp/m1024_p2_pts_16384-4   	       1	1235325258 ns/op	 4776832 B/op	      45 allocs/op
BenchmarkMpx/m128_p1_pts__1024-4      	     564	   2310017 ns/op	   84591 B/op	      26 allocs/op
BenchmarkMpx/m128_p2_pts__4096-4      	      63	  19927988 ns/op	  400206 B/op	      32 allocs/op
BenchmarkMpx/m128_p2_pts_16384-4      	       4	 294076163 ns/op	 1708912 B/op	      33 allocs/op
BenchmarkMpx/m128_p4_pts_16384-4      	       4	 327366290 ns/op	 2237776 B/op	      45 allocs/op
BenchmarkMpx/m1024_p2_pts_16384-4     	       4	 330582811 ns/op	 1737584 B/op	      33 allocs/op
BenchmarkUpdate-4                     	      80	  13849082 ns/op	  795065 B/op	      18 allocs/op
```

Ran on a 2018 MacBookAir on Jan 6, 2020
```sh
    Processor: 1.6 GHz Intel Core i5
       Memory: 8GB 2133 MHz LPDDR3
           OS: macOS Mojave v10.14.6
 Logical CPUs: 4
Physical CPUs: 2
```
```sh
$ make bench
```

## Contributing
* Fork the repository
* Create a new branch (feature_\* or bug_\*)for the new feature or bug fix
* Run tests
* Commit your changes
* Push code and open a new pull request

## Testing
Run all tests including benchmarks
```sh
$ make all
```
Just run benchmarks
```sh
$ make bench
```
Just run tests
```sh
$ make test
```

## Other libraries
* R: [github.com/franzbischoff/tsmp](https://github.com/franzbischoff/tsmp)
* Python: [github.com/target/matrixprofile-ts](https://github.com/target/matrixprofile-ts)

## Contact
* Austin Ouyang (aouyang1@gmail.com)

## License
The Apache 2.0 License. See [LICENSE](https://github.com/matrix-profile-foundation/go-matrixprofile/blob/master/LICENSE) for more details.

Copyright (c) 2020 Matrix Profile Foundation and contributors.

## Citations
[1] Chin-Chia Michael Yeh, Yan Zhu, Liudmila Ulanova, Nurjahan Begum, Yifei Ding, Hoang Anh Dau, Diego Furtado Silva, Abdullah Mueen, Eamonn Keogh (2016). [Matrix Profile I: All Pairs Similarity Joins for Time Series: A Unifying View that Includes Motifs, Discords and Shapelets](https://www.cs.ucr.edu/~eamonn/PID4481997_extend_Matrix%20Profile_I.pdf). IEEE ICDM 2016.

[2] Yan Zhu, Zachary Zimmerman, Nader Shakibay Senobari, Chin-Chia Michael Yeh, Gareth Funning, Abdullah Mueen, Philip Berisk and Eamonn Keogh (2016). [Matrix Profile II: Exploiting a Novel Algorithm and GPUs to break the one Hundred Million Barrier for Time Series Motifs and Joins](https://www.cs.ucr.edu/~eamonn/STOMP_GPU_final_submission_camera_ready.pdf). IEEE ICDM 2016.

[3] Hoang Anh Dau and Eamonn Keogh (2017). [Matrix Profile V: A Generic Technique to Incorporate Domain Knowledge into Motif Discovery](https://www.cs.ucr.edu/~eamonn/guided-motif-KDD17-new-format-10-pages-v005.pdf). KDD 2017.

[4] Chin-Chia Michael Yeh, Nickolas Kavantzas, Eamonn Keogh (2017).[Matrix Profile VI: Meaningful Multidimensional Motif Discovery](https://www.cs.ucr.edu/%7Eeamonn/Motif_Discovery_ICDM.pdf). ICDM 2017.

[5] Shaghayegh Gharghabi, Yifei Ding, Chin-Chia Michael Yeh, Kaveh Kamgar, Liudmila Ulanova, Eamonn Keogh (2017). [Matrix Profile VIII: Domain Agnostic Online Semantic Segmentation at Superhuman Performance Levels](https://www.cs.ucr.edu/%7Eeamonn/Segmentation_ICDM.pdf). ICDM 2017.
