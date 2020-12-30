---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: '0.8'
    jupytext_version: 1.5.0
kernelspec:
  display_name: 'Python 3.8.6 64-bit (''codeforecon'': conda)'
  language: python
  name: codeforecon
---

# Reading and Writing Data

```{code-cell} ipython3
:tags: ["remove-cell"]

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Set seed for reproducibility
np.random.seed(10)
# Set max rows displayed for readability
pd.set_option('display.max_rows', 6)
# Plot settings
plot_style = {'xtick.labelsize': 20,
                  'ytick.labelsize': 20,
                  'font.size': 22,
                  'figure.autolayout': True,
                  'figure.figsize': (10, 5.5),
                  'axes.titlesize': 22,
                  'axes.labelsize': 20,
                  'lines.linewidth': 4,
                  'lines.markersize': 6,
                  'legend.fontsize': 16,
                  'mathtext.fontset': 'stix',
                  'font.family': 'STIXGeneral',
                  'legend.frameon': False}
plt.style.use(plot_style)
```

In this chapter, you'll learn about reading and writing data.

This chapter uses the **pandas** and **pandas-datareader** packages. If you're running this code, you may need to install these packages using, for example, `pip install packagename` on your computer's command line. (If you're not sure what a command line or terminal is, take a quick look at the basics of coding chapter.)

![From the pandas documentation](https://pandas.pydata.org/pandas-docs/stable/_images/02_io_readwrite1.svg)

There are a huge range of input and output formats available in **pandas**: Stata (.dta), Excel (.xls, .xlsx), csv (tab or comma or whatever, use the `sep=` keyword), big data formats (HDF5, parquet), JSON, SAS, SPSS, SQL, and more; there's a [full list](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html) of formats available in the documentation.

## Reading in data from a file

The syntax for reading in data is usually very similar and of the form `pd.read_` and then the format e.g. `df = pd.read_stata('path/to/statafile.dta')` for Stata. Here's a really typical example of reading in a csv called 'data.csv':

```python
import os
import pandas as pd

df = pd.read_csv(os.path.join('path', 'to', 'data.csv'))
```

This code assumes that python is being executed in a directory that has a subdirectory 'path', which has a subdirectory 'to', inside which is 'data.csv'. Why are we using `os.path.join(...)` instead of just passing `pd.read_csv('path/to/data.csv'))`? The answer is that paths are not all alike; they're different on Linux/Mac and Windows. By saying `os.path.join` we tell Python to figure out how to create a path from the folder names.

TODO: Change this to use pathlib. There are introductory videos to pathlib and its use available [here](https://calmcode.io/pathlib/do-not-hardcode.html).

### Reading data from lots of files

Quite often, you have a case where you need to read in data from many files at once. There are two tools that will help with this: glob and concatenate.

Imagine you have a directory full of files with names like 'Jan-2001-data.csv', 'Feb-2001-data.csv', and so on. Glob can help you grab the names of all of these files.

```python
import glob

list_of_files = glob.glob(os.path.join('directory', '*-data.csv'))
```

Here, the `*` character is a wildcard that can match to anything (including any number of characters). You can keep it more specific by, for example, searching for a single wildcard character with `?` or any single digit with `[0-9]`.

Okay, so you have a big list of file paths: now what!? Assuming that the files have the same structure (eg the same columns), we can use `pd.read_csv` in a list followed by `pd.concat` to collapse these down either by index or by column (typically it's by index):

```python
df = pd.concat([pd.read_csv(x) for x in list_of_files], axis=0)
```

## Reading data from the web

### Files from the internet

As you will have seen in some of the examples in this book, it's easy to read data from the internet once you have the url and file type. Here, for instance, is an example that reads in the 'storms' dataset:

```{code-cell} ipython3
pd.read_csv('https://vincentarelbundock.github.io/Rdatasets/csv/dplyr/storms.csv')
```

### APIs

Using an API (application programming interface) is another way to draw down information from the interweb. Their just a way for one tool, say Python, to speak to another tool, say a server, and usefully exchange information. The classic use case would be to post a request for data that fits a certain query via an API and to get a download of that data back in return. (You should always preferentially use an API over webscraping a site.)

Because they are designed to work with any tool, you don't actually need a programming language to interact with an API, it's just a *lot* easier if you do.

```{note}
An API key is needed in order to access some APIs. Sometimes all you need to do is register with site, in other cases you may have to pay for access.
```

 To see this, let's directly use an API to get some time series data. We will make the call out to the internet using the **requests** package.

An API has an 'endpoint', the base url, and then a URL that encodes the question. Let's see an example with the ONS API for which the endpoint is "https://api.ons.gov.uk/". The rest of the API has the form 'key/value', for example we'll ask for timeseries data 'timeseries' followed by 'JP9Z' for the vacancies in the UK services sector. We then ask for 'dataset' followed by 'UNEM' to specify which overarching dataset the series we want is in. The last part asks for the data with 'data'. Often you won't need to know all of these details, but it's useful to see a detailed example.

The data that are returned by APIs are typically in JSON format, which looks a lot like a nested Python dictionary and its entries can be accessed in the same way--this is what is happening when getting the series' title in the example below. JSON is not good for analysis, so we'll use **pandas** to put the data into shape.

```{code-cell} ipython3
import requests

url = 'https://api.ons.gov.uk/timeseries/JP9Z/dataset/UNEM/data'

# Get the data from the ONS API:
json_data = requests.get(url).json()

# Prep the data for a quick plot
title = json_data['description']['title']
df = (pd.DataFrame(pd.json_normalize(json_data['months']))
        .assign(date=lambda x: pd.to_datetime(x['date']),
                value=lambda x: pd.to_numeric(x['value']))
        .set_index('date'))

df['value'].plot(title=title, ylim=(0, df['value'].max()*1.2), lw=3.);
```

We've talked about *reading* APIs. You can also create your own to serve up data, models, whatever you like! This is an advanced topic and we won't cover it; but if you do need to, the simplest way is to use [Fast API](https://fastapi.tiangolo.com/). You can find some short video tutorials for Fast API [here](https://calmcode.io/fastapi/hello-world.html).

#### An easier way to interact with (some) APIs

Although it didn't take much code to get the ONS data, it would be even better if it was just a single line, wouldn't it? Fortunately there are some packages out there that make this easy, but it does depend on the API (and APIs come and go over time).

By far the most comprehensive library for accessing extra APIs is [**pandas-datareader**](https://pandas-datareader.readthedocs.io/en/latest/), which provides convenient access to:

- FRED
- Quandl
- World Bank
- OECD
- Eurostat

and more.

Let's see an example using FRED (the Federal Reserve Bank of St. Louis' economic data library). This time, let's look at the UK unemployment rate:

```{code-cell} ipython3
import pandas_datareader.data as web

df_u = web.DataReader('LRHUTTTTGBM156S', 'fred')

df_u.plot(title='UK unemployment (percent)',
          legend=False,
          ylim=(2, 6),
          lw=3.);
```

## Writing data to file

The syntax for writing to a file is also very consistent, taking the form `df.to_*` where `*` might be `csv`, `stata`, or a number of output formats (you can even output to your computer's clipboard!).

In general, you *do* need to specify the file extension though, i.e. when saving data you should specify `df.to_csv('dataout.csv')` rather than `df.to_csv('dataout')`. As with reading in, there are plenty of options for how to output data, and you can find more on outputs in the **pandas** [documentation](https://pandas.pydata.org/docs/user_guide/io.html).

### Formats

You may wonder what file format to use in your work. Note that data formats differ in whether they are text based (csv, JSON) or binary (encoded and compressed, like Python's pickle format). The former tend to be more easy to use with a range of tools, while the latter are usually faster to read/write and take up less space on disk.

There's a lot to be said for plain old csv because it's interoperable between so many different software tools. I highly recommend it for your final results, if they will be shared. Its only trouble is that it's not very efficiency with space, and it's not 'intelligent' about column datatypes. If you want a format for intermediate data within a project, I tend to recommend parquet, which scales well to big data (it is very efficient with disk space) and has excellent read and write speeds. Feather is interoperable between Python and R and may also be a good choice. Depending on the structure of your data, you may find JSON a good option too.

If you're interested in how effective the different data formats are, there blog posts that address this question [here](https://towardsdatascience.com/the-best-format-to-save-pandas-data-414dca023e0d) and [here](https://ursalabs.org/blog/2019-10-columnar-perf/).

It's best *not* to use formats associated with proprietary software, especially if the standard may change over time (Stata files change with the version of Stata used!!) or if opening the data in that tool might change it (hello Excel). It's also good practice *not* to use a data storage format that cannot easily be opened by other tools. For this reason, I don't generally recommend Python's pickle format or R's RDA format (though of course it's fine if your data is completely internal to your project and you're only using one language).

## Reading and writing text, tex, md, and other text-editor friendly file formats

It's frequently the case that you'll want to write an individual table, chunk of text, or other content that can be opened with a text editor to file. **pandas** has some convenience functions for this (there was a short example in the data analysis quickstart). Let's say we had a table:

```{code-cell} ipython3
df = pd.read_csv('https://vincentarelbundock.github.io/Rdatasets/csv/dplyr/storms.csv')
table = (df.groupby(['month'])
           .agg({'wind': 'mean',
                 'pressure': 'mean'}))
table
```

Our options for export of the `table` variable (which has datatype `pandas.core.frame.DataFrame`) are varied. For instance, we could use

```python
table.to_latex(caption='A Table', label='tab:descriptive')
```

to create data suitable for putting in a .tex file, `table.to_string()` to get plain text, and `table.to_markdown()` for md content. There's even a `table.to_html()`!

Each of these export options accepts a filepath to write to, eg one can write `table.to_latex(os.path.join('path', 'to', 'file.md'))`.

```{note}
`.to_markdown()` has a dependency on another package, [**tabulate**](https://github.com/astanin/python-tabulate), which is for pretty-printing tables in Python and on the command line. You can install it using `pip install tabulate`.
```

If you have a string, bit of tex, or chunk of markdown that isn't coming directly from **pandas**, you can use base Python to write it to a file. Let's say we wanted to take some text,

```python
text = 'The greatest improvement in the productive powers of labour, and the greater part of the skill, dexterity, and judgment with which it is anywhere directed, or applied, seem to have been the effects of the division of labour.'
```

and write it to file. The command would be

```python
open('file.txt', 'w').write(text)
```

## Review

If you know how to read in data and text from file(s), the internet, and APIs, and write out to file too, then you've mastered the content of this chapter!
