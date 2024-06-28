Discord Music Bot
This Discord bot lets you play music from YouTube and Spotify in your voice channel! It can join your channel, play songs, queue them up, skip them, and handle basic controls like pause, resume, and stop.

Features
Join and leave voice channels
Play music from YouTube and Spotify
Add songs to a queue
Skip, pause, resume, and stop music playback
Handle both YouTube URLs and Spotify tracks/playlists
Libraries Used
discord.py: A Python wrapper for the Discord API (interaction with Discord servers)
yt-dlp: Downloads and extracts information from YouTube videos
subprocess: Executes system commands (used to verify FFmpeg)
collections.deque: A double-ended queue, perfect for music queues
spotipy: Interacts with Spotify's Web API to fetch tracks and playlists
re: Python's regular expression library (used for pattern matching)
Installation
1. Install Python Libraries

Open your terminal and run:

Bash
pip install discord.py yt-dlp spotipy
Use o código com cuidado.
content_copy
2. Install FFmpeg

FFmpeg is a multimedia framework needed for audio/video handling. Follow the steps for your OS:

Windows:

Download FFmpeg from the official website (find the appropriate version).
Extract the downloaded zip file to a folder (e.g., C:\ffmpeg).
Add FFmpeg to your system PATH:
Open Control Panel > System and Security > System.
Click "Advanced system settings" and then "Environment Variables."
Under "System variables," find "Path," select it, and click "Edit."
Click "New" and add the path to your FFmpeg bin directory (e.g., C:\ffmpeg\bin).
Click "OK" to close all dialogs.
Verify installation: Open a new command prompt and type ffmpeg -version. You should see FFmpeg version information.
macOS:

Install Homebrew (if not already installed):
Bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
Use o código com cuidado.
content_copy
Install FFmpeg:
Bash
brew install ffmpeg
Use o código com cuidado.
content_copy
Verify installation: Type ffmpeg -version in Terminal. You should see FFmpeg version information.
Linux:

Open your terminal and run the following command based on your distribution:

Bash
sudo apt update  # For Debian/Ubuntu
sudo apt install ffmpeg
Use o código com cuidado.
content_copy
Verify installation: Type ffmpeg -version in the terminal. You should see FFmpeg version information.

Bot Commands
!join: Joins your voice channel.
!leave: Leaves the current voice channel.
!play <URL or search query>: Plays a song from YouTube or Spotify.
!pause: Pauses the current song.
!resume: Resumes the paused song.
!stop: Stops the current song.
!skip: Skips to the next song in the queue.
!swap <YouTube URL or Spotify URL or song number>: Swaps the current song with a specific song.
Running the Bot
1. Update the Code

Replace the following placeholders in the code with your actual credentials:

'YOUR_BOT_TOKEN': Your Discord bot token
'YOUR_SPOTIPY_CLIENT_ID': Your Spotify API client ID
'YOUR_SPOTIPY_CLIENT_SECRET': Your Spotify API client secret
2. Run the Bot

Open your terminal, navigate to the directory containing the bot code, and run:

Python
bot.run('YOUR_BOT_TOKEN')
Use o código com cuidado.
content_copy
This README provides a clear and well-formatted guide for your Discord music bot on GitHub. Feel free to customize it further with screenshots or additional notes!
