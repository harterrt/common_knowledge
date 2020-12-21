# Introduction

Docere is a workflow and set of tools for publishing data analyses.

Docere sounds like "dose air"

# Installation

Docere is hosted on PyPI, so is installable via `pip`:

    pip install docere

Once installed, see usage details via:

    docere --help
    docere render --help

# Design Principles

* Analysts should have total control of their report presentation
* Analysts should be able to get their work reviewed
* Tools should be simple and do one thing well
* Changes to reports should be tracked,
  but reproducibility is the responsibility of the analyst (not the tool)

# High Level Workflow

## Generate a report

Docere starts with a `report` representing an analysis or a unit of knowledge.
An analyst can generate their report using any tools they like.
The only requirement is that their analysis result in a static HTML document,
or has a URL.

## Submit report to a knowledge-repo

All reports are stored in a central git repository, called the `knowledge repository`.
To submit a new report,
open a pull request against the knowledge repository.

You can add your report in either of two ways.

If you need to **use Docere to host HTML**, you should
create a directory containing a *metadata file* named `report.json` or `report.toml` file.
([TOML] is an INI-like configuration format, which is more flexible than JSON.)
The rendered HTML should go in the same directory.

If you want to link to **a report that's hosted somewhere else**, like Google Docs,
then add a .toml or .json file with any name to the external reports directory
configured for your repository. It's probably named `external`.

At a minimum, your metadata file should include the following fields:

* `title`: The title of the report
* `publish_date`: YYYY-MM-DD format
* `author`: The author's name, or `authors`: an array of author names

Fields that may be optional are:

* `link`: a URL for an external report
* `abstract`: an abstract to be rendered in the TOC


The following fields are the set of tags associated with a report. They're designed to be high-level in order to allow for discovery of similar knowledge. Find the set of tags [here](https://docs.google.com/spreadsheets/d/1RAz-0zyVSC-gM8nbxVfdCs0x8Bpz5ydWIR3ZZ6MAr9M/edit?usp=sharing) that can be used for the __product__, __area__ and __artifact__ attributes. The __project__ attribute is a free form tag that allows for work to be grouped under larger, longer term projects. For example, there could be multiple artifacts produced as the result of a major release, such as our recent 81 release named Shirley.

Multiple tags should be represented in list format (see example).

* `product`: the product that the work in the report is associated with
* `area`: a more descriptive focus area of the product
* `artifact`: the type of knowledge artifact the report is
* `project`: if the report falls under an umbrella project, specify here. For now this is a free-form field.

An identical example in each format:

<table>
<thead><tr><th>

`report.toml`

</th><th>

`report.json`

</th></tr></thead>
<tbody><tr>
<td>

```toml
# TOML can have comments.
title = "My Cool Report"
publish_date = "2020-01-30"
author = "Mo Zilla"
# TOML supports multiline strings:
abstract = """Lorem ipsum dolor sit amet, consectetur adipiscing elit,
sed do eiusmod tempor incididunt ut labore et dolore magna aliqua."""
product = "desktop"
area = ["bookmarks", "accounts"]
artifact = "experiment"
project = "project name"
```

</td>
<td>

```json
{
  "title": "My Cool Report",
  "publish_date": "2020-01-30",
  "author": "Mo Zilla",
  "abstract": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
  "product": "desktop",
  "area": ["bookmarks", "accounts"],
  "artifact": "experiment",
  "project": "project name"
}
```

</td>
</tr></tbody></table>

If desired, this is an opportunity to get review for your analysis.

[TOML]: https://toml.io/en/

## Render content

You should configure CI to trigger docere when PRs are merged to master.
Docere will then:

* Copy the knowledge-repo to a new directory named `output`
* Gather metadata for all known reports
* Pass the metadata to the `metadata generators` to create the necessary `metadata pages`

`Metadata pages` are auto-generated documents produced to make reports more discoverable.
For example, docere will generate a homepage that lists all reports in anti-chronological order.
Other `metadata pages` could include: RSS feeds, topic pages, or reports by a specific contributor.

I intend most `metadata generators` be implemented as plugins to this system.
For now, I'm including some very simple `metadata generators` by default.

## Upload content

Docere does not handle uploading the rendered site to a server.
We recommend configuring this through your CI provider.
We've included an example `.travis.yml` in this repository.
You can view the rendered documentation
[here](http://docere-test.s3-website-us-east-1.amazonaws.com/).

# Advantages and Weaknesses

## HTML is difficult to review in GitHub

A docere `knowledge repository` stores raw HTML files.
This gives the analyst complete control over the format of the report,
but comes with some notable disadvantages.

HTML diffs are often cluttered with boilerplate.
Even worse, GitHub doesn't allow you to review the rendered HTML page in your browser.
It would be much nicer if we could store markdown documents in the `knowledge repository`
and render these to HTML when generating the static site.
In fact, this is what AirBnB's [knowledge-repo] does.

We decided against storing markdown because it takes control away from the analyst.
Presenting data in a meaningful and compelling format is a difficult task.
Different reports need different formats.
It is **not this tool's job to be opinionated**.

## `reports` aren't inherently reproducible

This tool does not save any of the code used to generate a report.
Instead, **the analyst is responsible for making their results reproducible**.
This can be done by linking to a commit in a GitHub repository
or by including the code itself in the `report's` directory in the `knowledge repository`.

## Requires interacting with Git

Using Git to store `reports` makes it easy to get review and track changes.
However, some users will not be comfortable interacting with Git.
For now, these users are out of luck.

We may eventually explore a simpler front end,
but this is not current on our roadmap.
Docere is build to be composable,
so it should be easy to roll your own interface if you so desire!


# Appendix

## FAQ

### Another static site generator?

_Why_ are you building _another_ static site generator, Ryan?
Why!

I just couldn't find any other static site generator
that let's the analyst have total control over the report.
Docere, on the other hand, just aggregates reports.
It **doesn't render them**.

For example, checkout
[this prototype using pelican](https://github.com/harterrt/dpel).
We're storing HTML files,
but everything is squeezed into the default pelican theme.
I decided it would take more work to build a minimal template for pelican
than to just start over.

### Why use `report.json` config files?

Many static site generators prefer using front-matter to store metadata.
In my experience adding front-matter to a report is an unnecessary pain.

If you already have a working toolchain for creating reports,
modifying your templates to include front-matter is frustrating.
It's much easier to compose these toolchains if you use an external config file.

For example, you could create a simple bash script that
starts a branch in the knowledge-repo,
copies your report to the knowledge-repo,
and copies a boilerplate `report.json` config to the right directory.
Now imagine if that config needed to reside inside the HTML file.


[knowledge-repo]: https://github.com/airbnb/knowledge-repo
