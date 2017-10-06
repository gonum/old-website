---
title: "Introduction to Statistics With Gonum"
date: 2017-10-04T11:17:11+02:00
Categories: ["go", "gonum", "stat"]
---

_The following was cross-posted (with minor changes) from [sbinet.github.io/posts/2017-10-04-intro-to-stats-with-gonum](https://sbinet.github.io/posts/2017-10-04-intro-to-stats-with-gonum/)_

This is the first of a series of short posts providing an introduction and code examples for using the Gonum packages.
This first post focuses on computing basic statistics using the [stat](https://godoc.org/gonum.org/v1/gonum/stat) package.

This first post is based off the content of this blog post:

 https://mubaris.com/2017-09-09/introduction-to-statistics-using-numpy

but using [Go](https://golang.org) and [Gonum](https://gonum.org) instead of Python and `numpy`.

[Gonum](https://gonum.org) is _"a set of packages designed to make writing numerical and scientific algorithms productive, performant and scalable."_
Please refer to this [post](/post/introtogonum) for more details on Gonum.

# Gonum and statistics

Gonum provides many statistical functions.
Let's use it to calculate the mean, median, standard deviation and variance of a small dataset.

[embedmd]:# (../../static/code/intro-to-stats-with-gonum/stats.go go)
```go
// file: stats.go

package main

import (
	"fmt"
	"math"
	"sort"

	"gonum.org/v1/gonum/stat"
)

func main() {
	xs := []float64{
		32.32, 56.98, 21.52, 44.32,
		55.63, 13.75, 43.47, 43.34,
		12.34,
	}

	fmt.Printf("data: %v\n", xs)

	// computes the weighted mean of the dataset.
	// we don't have any weights (ie: all weights are 1)
	// so we just pass a nil slice.
	mean := stat.Mean(xs, nil)
	variance := stat.Variance(xs, nil)
	stddev := math.Sqrt(variance)

	// stat.Quantile needs the input slice to be sorted.
	sort.Float64s(xs)
	fmt.Printf("data: %v (sorted)\n", xs)

	// computes the median of the dataset.
	// here as well, we pass a nil slice as weights.
	median := stat.Quantile(0.5, stat.Empirical, xs, nil)

	fmt.Printf("mean=     %v\n", mean)
	fmt.Printf("median=   %v\n", median)
	fmt.Printf("variance= %v\n", variance)
	fmt.Printf("std-dev=  %v\n", stddev)
}
```

The program above performs some rather basic statistical operations on our dataset:

```sh
$> go run stats.go
data: [32.32 56.98 21.52 44.32 55.63 13.75 43.47 43.34 12.34]
data: [12.34 13.75 21.52 32.32 43.34 43.47 44.32 55.63 56.98] (sorted)
mean=     35.96333333333334
median=   43.34
variance= 285.306875
std-dev=  16.891029423927957
```

The astute reader will no doubt notice that the variance value displayed here
differs from the one obtained with `numpy.var`:

```python
>>> xs=[32.32, 56.98, 21.52, 44.32, 55.63, 13.75, 43.47, 43.34, 12.34]
>>> xs.sort()
>>> np.mean(xs)
35.963333333333338
>>> np.median(xs)
43.340000000000003
>>> np.var(xs)
253.60611111111109
>>> np.std(xs)
15.925015262507948
```

This is because `numpy.var` uses `len(xs)` as the divisor while `gonum/stats`
uses the unbiased sample variance (_ie:_ the divisor is `len(xs)-1`):

```python
>>> np.var(xs, ddof=1)
285.30687499999999
>>> np.std(x, ddof=1)
16.891029423927957
```

If one wants the uncorrected estimator, [stat.Moment](https://godoc.org/gonum.org/v1/gonum/stat#Moment) can be used instead.

With this quite blunt tool, we can analyse some real data from real life.
We will use a dataset pertaining to the salary of European developers, all 1147 of them :).
We have this dataset in a file named `salary.txt`.

[embedmd]:# (../../static/code/intro-to-stats-with-gonum/stats-salary.go go)
```go
// file: stats-salary.go

package main

import (
	"bufio"
	"fmt"
	"log"
	"math"
	"os"
	"sort"

	"gonum.org/v1/gonum/stat"
)

func main() {
	f, err := os.Open("salary.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()

	var xs []float64
	scan := bufio.NewScanner(f)
	for scan.Scan() {
		var v float64
		txt := scan.Text()
		_, err = fmt.Sscanf(txt, "%f", &v)
		if err != nil {
			log.Fatalf(
				"could not convert to float64 %q: %v",
				txt, err,
			)
		}
		xs = append(xs, v)
	}

	// make sure scanning the file and extracting values
	// went fine, without any error.
	if err = scan.Err(); err != nil {
		log.Fatalf("error scanning file: %v", err)
	}

	fmt.Printf("data sample size: %v\n", len(xs))

	mean := stat.Mean(xs, nil)
	variance := stat.Variance(xs, nil)
	stddev := math.Sqrt(variance)

	sort.Float64s(xs)
	median := stat.Quantile(0.5, stat.Empirical, xs, nil)

	fmt.Printf("mean=     %v\n", mean)
	fmt.Printf("median=   %v\n", median)
	fmt.Printf("variance= %v\n", variance)
	fmt.Printf("std-dev=  %v\n", stddev)
}
```

And here is the output:

```
$> go run ./stats-salary.go
data sample size: 1147
mean=     55894.53879686138
median=   48000
variance= 3.0464263289031615e+09
std-dev=  55194.44110508921
```

The data file can be obtained from here: [salary.txt](/code/intro-to-stats-with-gonum/salary.txt)
together with a much more detailed one there: [salary.csv](/code/intro-to-stats-with-gonum/salary.csv).

*By Sebastien Binet*
