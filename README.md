# NAO Top 10 Dataset


This document describes a dataset of human NetHack gameplay curated from the
publically available and permissively licensed recordings available at
http://nethack.alt.org.

The original data consists of a very large collection of human gameplay in the
form of raw TTY terminal command recordings, which is a complex format to
ingest. We release a processed version of a subset of the dataset, with the
following transformations:

We have used python-based terminal emulator pyte
(https://github.com/selectel/pyte) to decode the serialized terminal commands
into fixed-size tensors of terminal characters, colors, backgrounds, and cursor
locations.
We remove any sessions that use non-default user interface settings, including
terminal sizes larger than the standard 24x80, interface configurations such
as Curses, and non-ASCII character remappings. This brings the data format
closer to the observations that would be seen by an agent using the NetHack
Learning Environment (NLE: https://github.com/facebookresearch/nle).
We have curated the dataset (which consists of over 7
million gameplay sessions, taking hundreds of terabytes of storage space once
decoded) by filtering the dataset to include only the gameplay from the top
10 NetHack players as ranked by Z-score (https://nethackwiki.com/wiki/Z-score),
a measure of player ability. We
annotate each gameplay session with the following metadata where it has been
possible to parse from the observations:


| Metadata                | Description                                       |
|-------------------------|---------------------------------------------------|
| ``alignment``           | Character alignment                               |
| ``length``              | Number of timesteps in the session                |
| ``max_dungeon_number``  | Maximum dungeon number reached                    |
| ``max_exp_level``       | Maximum experience level reached                  |
| ``max_exp_points``      | Maximum experience points achieved                |
| ``max_level_number``    | Maximum dungeon level reached                     |
| ``min_dungeon_number``  | Minimum dungeon number reached                    |
| ``min_exp_level``       | Minimum experience level reached                  |
| ``min_exp_points``      | Minimum experience points achieved                |
| ``min_level_number``    | Minimum dungeon level reached                     |
| ``rank``                | First character rank achieved                     |
| ``role``                | Character role (class)                            |
| ``stamp``               | Unix timestamp when the session was started       |
| ``username``            | Username of the player that recorded the session  |

The metadata is available as a pickle archived dictionary that maps session IDs
to metadata dictionaries with the keys described above.

The full curated subset contains 16478 gameplay sessions. Note that a
gameplay session starts when the user logs in, and ends when the user logs out.
A complete episode can therefore consist of multiple consecutive sessions.
The dataset in total amounts to approximately 184M timesteps, and is
approximately 12GB in size.

## Data

The entire dataset can be downloaded as a single file here:
[nao_top10.tar](https://storage.googleapis.com/dm_nethack/nao_top10.tar)
\[11.8 GB\]

We also provide separate access to the metadata and recorded gameplay of
individual users to allow for downloading smaller subsets of the dataset (note
that all of the following files are included in `nao_top10.tar` above):

| File | Size |
|------|------|
|[metadata.pkl](https://storage.googleapis.com/dm_nethack/nao_top10/metadata.pkl) | 10 KB |
|[78291.tar](https://storage.googleapis.com/dm_nethack/nao_top10/78291.tar)| 10.1 MB|
|[Fek.tar](https://storage.googleapis.com/dm_nethack/nao_top10/Fek.tar)| 1.1 GB|
|[Luxidream.tar](https://storage.googleapis.com/dm_nethack/nao_top10/Luxidream.tar)| 8.2 MB |
|[Stroller.tar](https://storage.googleapis.com/dm_nethack/nao_top10/Stroller.tar)| 3.8 GB |
|[Tariru.tar](https://storage.googleapis.com/dm_nethack/nao_top10/Tariru.tar)| 1.5 GB |
|[YumYum.tar](https://storage.googleapis.com/dm_nethack/nao_top10/YumYum.tar)| 1.8 GB |
|[oh6.tar](https://storage.googleapis.com/dm_nethack/nao_top10/oh6.tar)| 1.7 MB |
|[rschaff.tar](https://storage.googleapis.com/dm_nethack/nao_top10/rschaff.tar)| 29.3 MB |
|[stenno.tar](https://storage.googleapis.com/dm_nethack/nao_top10/stenno.tar)| 1.3 MB |
|[stth.tar](https://storage.googleapis.com/dm_nethack/nao_top10/stth.tar)| 3.6 GB |

All of the data originates from http://nethack.alt.org and is licensed under
a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).


## Usage

The data is organized as ``[username]/[session_id].npz`` where session_id is a
base64 string, and the file is in ``.npz`` format, which can be loaded using
``numpy``:

    session = np.load(filename)                 # Dict of observations
    tty_chars = session['tty_chars']            # Shape = (T, 24, 80)
    tty_colors = session['tty_colors']          # Shape = (T, 24, 80)
    tty_background = session['tty_background']  # Shape = (T, 24, 80)
    tty_cursor = session['tty_cursor']          # Shape = (T, 2,)

The tty_chars, tty_colors, and tty_cursor have similar semantics to the
corresponding observations from the NLE environment. tty_background corresponds
to the background color used to represent the specials observation in NLE when
rendered in the browser, although since specials is a bitfield that can have
multiple bits enabled, the background colors do not map directly to specials.

We also include a short rendering utility that uses PyGame to generate images of
arbitrary states, for ease of visualizing the data. The code will generate
images of states from the dataset, as well as observations from the NLE
environment. The renderer can also take arbitrary foreground and background
color arrays, which we use in the [paper](https://openreview.net/forum?id=sKc6fgce1zs)
to generate saliency visualizations and to highlight differences between states.
The renderer can be used to visualize observations from the dataset as follows:

    image_renderer = ImageRenderer()
    session = np.load(filename)
    # Get the fifth observation of the session as a dict.
    # We choose the fifth frame here because the first few are just setup.
    frame_num = 5
    observation = dict(
        tty_chars=session['tty_chars'][frame_num],
        tty_colors=session['tty_colors'][frame_num],
        tty_background=session['tty_background'][frame_num],
        tty_cursor=session['tty_cursor'][frame_num])
    img = image_renderer.render(observation)

## Installation

Install `cmake`:

    sudo apt install cmake

Install `nle`, `numpy`, and `pygame`:

    pip install -r ./requirements.txt

## Citation

If you found this dataset useful for academic work, then please cite the
corresponding paper: [Learning About Progress From Experts](https://openreview.net/forum?id=sKc6fgce1zs).

    @inproceedings{brucelearning,
      title={{Learning About Progress From Experts}},
      author={Bruce, Jake and Anand, Ankit and Mazoure, Bogdan and Fergus, Rob},
      booktitle={International Conference on Learning Representations},
      year={2023}
    }
