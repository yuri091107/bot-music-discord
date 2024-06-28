# Discord Music Bot

This Discord bot allows users to play music from YouTube and Spotify in a voice channel. It can join a voice channel, play songs, add songs to a queue, skip songs, and handle basic music control commands like pause, resume, and stop.

## Features

- Join and leave voice channels
- Play music from YouTube and Spotify
- Add songs to a queue
- Skip, pause, resume, and stop music playback
- Handle both YouTube URLs and Spotify tracks/playlists

## Libraries Used

### 1. [discord.py](https://discordpy.readthedocs.io/en/stable/)
- **discord**: A Python wrapper for the Discord API, enabling interaction with Discord servers.
- **discord.ext.commands**: An extension module to help organize commands for the bot.

### 2. [yt-dlp](https://github.com/yt-dlp/yt-dlp)
- A YouTube video downloader library that allows you to download and extract information from YouTube videos.

### 3. [subprocess](https://docs.python.org/3/library/subprocess.html)
- A standard Python module used to execute system commands, used here to verify the presence of FFmpeg.

### 4. [collections.deque](https://docs.python.org/3/library/collections.html#collections.deque)
- A module that provides a double-ended queue, ideal for implementing a music queue.

### 5. [spotipy](https://spotipy.readthedocs.io/en/2.19.0/)
- A lightweight Python library for the Spotify Web API. It allows your program to interact with Spotify’s API to fetch tracks and playlists.

### 6. [re](https://docs.python.org/3/library/re.html)
- Python’s regular expression library used for pattern matching in strings.

## Installation Instructions

### Step 1: Install Python Libraries

You can install the required Python libraries using `pip`. Open your terminal or command prompt and run the following commands:

```bash
pip install discord.py yt-dlp spotipy











### Step 2: Install FFmpeg

FFmpeg is a powerful multimedia framework used to handle audio and video files. Follow these steps to install FFmpeg:

#### For Windows:

1. **Download FFmpeg**:
   - Go to the [FFmpeg download page](https://ffmpeg.org/download.html).
   - Select the appropriate version for Windows and download the zip file.

2. **Extract FFmpeg**:
   - Extract the contents of the zip file to a folder, e.g., `C:\ffmpeg`.

3. **Add FFmpeg to PATH**:
   - Open Control Panel.
   - Go to System and Security > System.
   - Click on Advanced system settings.
   - Click on Environment Variables.
   - Under System variables, find the PATH variable, select it, and click Edit.
   - Click New and add the path to the FFmpeg bin directory (e.g., `C:\ffmpeg\bin`).
   - Click OK to close all dialog boxes.

4. **Verify Installation**:
   - Open a new command prompt and type `ffmpeg -version`. If installed correctly, FFmpeg version information will be displayed.

#### For macOS:

1. **Install Homebrew** (if not already installed):
   - Open the Terminal app and paste the following command:
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **Install FFmpeg**:
   - Run the following command in the Terminal:
     ```bash
     brew install ffmpeg
     ```

#### For Linux:

1. **Install FFmpeg**:
   - Open your terminal and run the following command based on your distribution:
     ```bash
     sudo apt update
     sudo apt install ffmpeg
     ```

2. **Verify Installation**:
   - Type `ffmpeg -version` in the terminal. You should see FFmpeg version information if the installation was successful.

## Bot Commands

- `!join`: Join the voice channel of the user.
- `!leave`: Leave the current voice channel.
- `!play <URL or search query>`: Play a song from YouTube or Spotify.
- `!pause`: Pause the current song.
- `!resume`: Resume the paused song.
- `!stop`: Stop the current song.
- `!skip`: Skip to the next song in the queue.
- `!swap <YouTube URL or Spotify URL or song number>`: Swap the current song with a specific song.

## Running the Bot

Replace `'YOUR_BOT_TOKEN'` and `'YOUR_SPOTIPY_CLIENT_ID'` and `'YOUR_SPOTIPY_CLIENT_SECRET'` with your actual bot token and Spotify API credentials in the code. Then, run the bot with:

```python
bot.run('YOUR_BOT_TOKEN')


