# Welcome to MkDocs

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

## first

**s**

$s$ 

s

d

s

d

sd

s

d

s

## second

fs

dfd

f

d

fd

f

d

f

df

d

f

d

s

d

s

[first](#first)

a

a



a

a

a

```cpp
#include <iostream>
#include <cstdio>
#include <cmath>
#include <vector>

using namespace std;

const int m = 3; // jieshu
const int a = 1, b = 2; // [a, b]
const double h = 0.1; // h

const int n = (b - a) / h;

struct vec {
	double v[m];
}w[n + 1], k[5];
double t[n + 1];

vec operator +(const vec u, const vec v) {
	vec res;
	for(int i = 0; i < m; ++i) {
		res.v[i] = u.v[i] + v.v[i]; 
	}
	return res;
}
vec operator -(const vec u, const vec v) {
	vec res;
	for(int i = 0; i < m; ++i) {
		res.v[i] = u.v[i] - v.v[i]; 
	}
	return res;
}
vec operator *(double w, const vec u) {
	vec res;
	for(int i = 0; i < m; ++i) {
		res.v[i] = w * u.v[i]; 
	}
	return res;
}
vec operator /(const vec u, double w) {
	vec res;
	for(int i = 0; i < m; ++i) {
		res.v[i] = u.v[i] / w; 
	}
	return res;
}


vec f(double t, vec u) { // f
	vec res;
	res.v[0] = u.v[1];
	res.v[1] = u.v[2];
	res.v[2] = 5 * log(t) + 9 + 4 / t / t / t * u.v[0] - 3 / t / t * u.v[1] + 1 / t * u.v[2];
	return res;
}
double y(double t) { // y
	return -t * t + t * cos(log(t)) + t * sin(log(t)) + t * t * t * log(t);
}


int main() {

	w[0] = {0, 1, 3}; // initial value

	t[0] = a;
	for(int i = 1; i <= n; ++i) {
		t[i] = t[i - 1] + h;
	}
	for(int i = 1; i <= 3; ++i) {
		k[1] = h * f(t[i - 1], w[i - 1]);
		k[2] = h * f(t[i - 1] + h / 2, w[i - 1] + k[1] / 2);
		k[3] = h * f(t[i - 1] + h / 2, w[i - 1] + k[2] / 2);
		k[4] = h * f(t[i - 1] + h, w[i - 1] + k[3]);
		w[i] = w[i - 1] + (k[1] + 2 * k[2] + 2 * k[3] + k[4]) / 6;
	}
	for(int i = 4; i <= n; ++i) {
		w[i] = w[i - 1] + h * (55 * f(t[i - 1], w[i - 1]) - 59 * f(t[i - 2], w[i - 2]) + 37 * f(t[i - 3], w[i - 3]) - 9 * f(t[i - 4], w[i - 4])) / 24;
		w[i] = w[i - 1] + h * (9 * f(t[i], w[i]) + 19 * f(t[i - 1], w[i - 1]) - 5 * f(t[i - 2], w[i - 2]) + f(t[i - 3], w[i - 3])) / 24;
	}
	for(int i = 0; i <= n; ++i) {
		printf("%.1lf: %.8lf %.8lf\n", t[i], w[i].v[0], y(t[i]));
	}
	return 0;
}	
```

