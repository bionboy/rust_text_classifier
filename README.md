# Rust Text Classifier

A Reddit bot powered by a text classifier for determining if text is about the
[videogame Rust](https://rust.facepunch.com) or the
[programming language Rust](https://www.rust-lang.org)

## Overview

A high-level overview of the actions of the bot is simply to listen to the
stream of new textposts on the
[r/rust subreddit](https://www.reddit.com/r/rust/). When it sees a new post
then it runs a text classifier that's been trained to discern text about the
videogame vs programming language. If it believes that the title+body is about
the videogame (based on a configurable prediction threshold, see analysis for
more information), then it simply leaves a comment listing this conclusion
along with several popular Rust game subreddits that might fit the post.

## Installation

_Note: This has only been tested on Linux and the following assumes a Linux
environment_

### Required Configuration

This repo is essentially assumed to be installed under
`/opt/rust_text_classifier` with only one required change being

- A config file called `config.json` (`sample_config.json` acts as a template)

If desired a different `posts_corpus` can be used and several files will be
automatically generated

- `posts.db` simply keeps track of classifications on posts
- `text_classifier.pkl` which is a pickled form of the classifier to avoid having to retrain each time the program is launched

The bot expects:
 - A config file at `$XDG_CONFIG_HOME/rust_text_analyzer/config.json`
   - `sample_config.json` is an example config with all the required fields
 - A posts corpus at `$XDG_DATA_HOME/rust_text_analyzer/posts_corpus`
   - A sample corpus is provided in the repo (I personally use a larger one, but am not sure where it would make sense to host)

### Dependencies

This project uses [`poetry`](https://github.com/python-poetry/poetry) for
handling dependencies and virtual environments. With poetry installed getting
all the dependencies setup and then running the bot is as simple as running the
following from the project dir

```bash
poetry install --no-dev  # Only need to do this once
poetry run ./bot  # Uses the virtual environment created above
```

Alternatively you can use your system's package manager, or you can manually
use pip to install the dependencies (I don't think it supports reading from
`pyproject.toml` yet, but I could be wrong.

## Analysis

_Note: The classifier always uses an equal number of posts from each category,
so even though there are more posts about the game available it will only
select enough to match the posts about the lang_

There is an `analysis` script for some basic (read as hacky) analysis. This
test simply trains a classifier off 80% of the posts found in `posts_corpus`
and then tests the accuracy using the remaining 20% of the posts. This test is
run 100 times with the values for each category being averaged together and
reported. This is repeated using `50%`, `60%`, and `70%` as the threshold.

Running these tests locally with a corpus of 222 Rust Lang posts and ~500 Rust
Game posts to choose from gets the values reported below which indicated that
even the most liberal threshold (`50%`) would inaccurately flag less than 2% of
Rust Lang posts as being about the Game, while managing to detect over 97% of
Rust Game posts accurately.

| Category | Threshold | Correct | Incorrect | Ignored |
| :---: | :---: | :---: | :---: | :---: |
| Lang | 50% | 98.29% | 1.71% | 0.00% |
| Game | 50% | 97.52% | 2.48% | 0.00% |
| Lang | 60% | 96.12% | 1.07% | 2.81% |
| Game | 60% | 92.41% | 1.53% | 6.06% |
| Lang | 70% | 90.80% | 0.74% | 8.46% |
| Game | 70% | 84.14% | 1.03% | 14.83% |

## Notes on Posts Corpus

The posts corpus is generated by running a simple script that fetches all new
text posts from `r/rust` along with a number of Rust Game subreddits every 5
minutes. From there the posts from `r/rust` are manually classified into
`r_rust_correct` and `r_rust_incorrect`. This is done to best match the
information that the bot will attempt to classify although using older data
would likely also work well.

The classifier also prefers posts from `r_rust_incorrect` before all other Rust
Game posts since it best matches the data it's attempt to classify, so it will
use posts from there before any other Rust Game posts (The loaded posts are
always shuffled though to guarantee better randomness for testing accuracy).

## License

All contents in this repo excluding the content within the `posts_corpus`
directory is licensed under either of

 - Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 - MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
