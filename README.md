Discord Music Bot
This Discord bot allows users to play music from YouTube and Spotify in a voice channel. It can join a voice channel, play songs, add songs to a queue, skip songs, and handle basic music control commands like pause, resume, and stop.

Features
Join and leave voice channels
Play music from YouTube and Spotify
Add songs to a queue
Skip, pause, resume, and stop music playback
Handle both YouTube URLs and Spotify tracks/playlists
Libraries Used
1. discord.py
discord: A Python wrapper for the Discord API, enabling interaction with Discord servers.
discord.ext.commands: An extension module to help organize commands for the bot.
2. yt-dlp
A YouTube video downloader library that allows you to download and extract information from YouTube videos.
3. subprocess
A standard Python module used to execute system commands, used here to verify the presence of FFmpeg.
4. collections.deque
A module that provides a double-ended queue, ideal for implementing a music queue.
5. spotipy
A lightweight Python library for the Spotify Web API. It allows your program to interact with Spotify’s API to fetch tracks and playlists.
6. re
Python’s regular expression library used for pattern matching in strings.
Installation Instructions
Step 1: Install Python Libraries
You can install the required Python libraries using pip. Open your terminal or command prompt and run the following commands:

bash
Copiar código
pip install discord.py yt-dlp spotipy
Step 2: Install FFmpeg
FFmpeg is a powerful multimedia framework used to handle audio and video files. Follow these steps to install FFmpeg:

For Windows:
Download FFmpeg:

Go to the FFmpeg download page.
Select the appropriate version for Windows and download the zip file.
Extract FFmpeg:

Extract the contents of the zip file to a folder, e.g., C:\ffmpeg.
Add FFmpeg to PATH:

Open Control Panel.
Go to System and Security > System.
Click on Advanced system settings.
Click on Environment Variables.
Under System variables, find the PATH variable, select it, and click Edit.
Click New and add the path to the FFmpeg bin directory (e.g., C:\ffmpeg\bin).
Click OK to close all dialog boxes.
Verify Installation:

Open a new command prompt and type ffmpeg -version. If installed correctly, FFmpeg version information will be displayed.
For macOS:
Install Homebrew (if not already installed):

Open the Terminal app and paste the following command:
bash
Copiar código
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
Install FFmpeg:

Run the following command in the Terminal:
bash
Copiar código
brew install ffmpeg
For Linux:
Install FFmpeg:

Open your terminal and run the following command based on your distribution:
bash
Copiar código
sudo apt update
sudo apt install ffmpeg
Verify Installation:

Type ffmpeg -version in the terminal. You should see FFmpeg version information if the installation was successful.
Bot Commands
!join: Join the voice channel of the user.
!leave: Leave the current voice channel.
!play <URL or search query>: Play a song from YouTube or Spotify.
!pause: Pause the current song.
!resume: Resume the paused song.
!stop: Stop the current song.
!skip: Skip to the next song in the queue.
!swap <YouTube URL or Spotify URL or song number>: Swap the current song with a specific song.
Running the Bot
Replace 'YOUR_BOT_TOKEN' and 'YOUR_SPOTIPY_CLIENT_ID' and 'YOUR_SPOTIPY_CLIENT_SECRET' with your actual bot token and Spotify API credentials in the code. Then, run the bot with:

python
Copiar código
bot.run('YOUR_BOT_TOKEN')
Example Code
python
Copiar código
import discord
from discord.ext import commands
import yt_dlp as youtube_dl
import subprocess
from collections import deque
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials
import re

# Set up Discord intents
intents = discord.Intents.default()
intents.message_content = True

# Create bot instance
bot = commands.Bot(command_prefix="!", intents=intents)

# Create a queue to store music tracks
music_queue = deque()

# Spotify API credentials
SPOTIPY_CLIENT_ID = 'YOUR_SPOTIPY_CLIENT_ID'
SPOTIPY_CLIENT_SECRET = 'YOUR_SPOTIPY_CLIENT_SECRET'
sp = spotipy.Spotify(auth_manager=SpotifyClientCredentials(client_id=SPOTIPY_CLIENT_ID, client_secret=SPOTIPY_CLIENT_SECRET))

# Event when the bot is ready
@bot.event
async def on_ready():
    print(f'Bot {bot.user} is online!')

# Command to join a voice channel
@bot.command(name='join')
async def join(ctx):
    if ctx.author.voice:
        channel = ctx.author.voice.channel
        await channel.connect()
    else:
        await ctx.send("You are not in a voice channel!")

# Command to leave a voice channel
@bot.command(name='leave')
async def leave(ctx):
    if ctx.voice_client:
        await ctx.voice_client.disconnect()
    else:
        await ctx.send("I'm not in a voice channel!")

# Command to play music
@bot.command(name='play')
async def play(ctx, *, query):
    if ctx.voice_client:
        try:
            # Check if the query is a YouTube URL
            if re.match(r'https?://(www\.)?(youtube\.com|youtu\.be)/.+', query):
                music_queue.append(query)
                await ctx.send("YouTube URL added to the queue.")
            # Check if the query is a Spotify playlist URL
            elif re.match(r'https?://(open\.)?spotify\.com/playlist/.*', query):
                playlist_id = query.split('/')[-1].split('?')[0]
                results = sp.playlist_tracks(playlist_id)
                ctx.bot.last_playlist = results  # Store the last loaded playlist
                for item in results['items']:
                    track = item['track']
                    track_name = track['name']
                    artist_name = track['artists'][0]['name']
                    music_queue.append(f"ytsearch:{track_name} {artist_name}")
                await ctx.send(f"Added {len(results['items'])} tracks from the playlist to the queue.")
            # Check if the query is a Spotify track URL
            elif re.match(r'https?://(open\.)?spotify\.com/track/.*', query):
                track_id = query.split('/')[-1].split('?')[0]
                track = sp.track(track_id)
                track_name = track['name']
                artist_name = track['artists'][0]['name']
                music_queue.append(f"ytsearch:{track_name} {artist_name}")
                await ctx.send(f"Added the track {track_name} to the queue.")
            else:
                # Search by name
                results = sp.search(q=query, limit=1)
                if not results['tracks']['items']:
                    await ctx.send("No tracks found with that name!")
                    return

                track = results['tracks']['items'][0]
                track_name = track['name']
                artist_name = track['artists'][0]['name']
                music_queue.append(f"ytsearch:{track_name} {artist_name}")
                await ctx.send(f"Added the track {track_name} to the queue.")

            if not ctx.voice_client.is_playing():
                await play_next(ctx)
        except Exception as e:
            await ctx.send(f"An error occurred while trying to play the music: {str(e)}")
            print(f"Error: {str(e)}")
    else:
        await ctx.send("I'm not in a voice channel!")

# Function to play the next track in the queue
async def play_next(ctx):
    if len(music_queue) > 0:
        query = music_queue.popleft()
        ydl_opts = {
            'format': 'bestaudio/best',
            'noplaylist': True,
            'default_search': 'ytsearch',
            'quiet': True
        }
        with youtube_dl.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(query, download=False)
            url2 = info['entries'][0]['url'] if 'entries' in info else info['url']
        
        ffmpeg_path = "C:\\Program Files\\ffmpeg\\bin\\ffmpeg.exe"  # Specify the full path to ffmpeg

        # Log the URL and ffmpeg path for debugging
        print(f"URL: {url2}")
        print(f"FFmpeg Path: {ffmpeg_path}")

        # Ensure ffmpeg is working by running a test command
        try:
            subprocess.check_output([ffmpeg_path, "-version"])
            print("FFmpeg is working correctly.")
        except subprocess.CalledProcessError as e:
            print(f"Error with FFmpeg: {e.output}")
            await ctx.send("Error verifying FFmpeg.")
            return

        # Create FFmpeg audio source with proper parameters
        source = discord.FFmpegPCMAudio(url2, executable=ffmpeg_path, before_options="-reconnect 1 -reconnect_streamed 1 -reconnect_delay_max 5", options="-vn")
        ctx.voice_client.play(source, after=lambda e: bot.loop.create_task(play_next(ctx)))
    else:
        await ctx.send("The music queue is empty! Leaving the voice channel.")
        await ctx.voice_client.disconnect()

# Command to pause the current track
@bot.command(name='pause')
async def pause(ctx):
    if ctx.voice_client and ctx.voice_client.is_playing():
        ctx.voice_client.pause()
    else:
        await ctx.send("Nothing is playing right now!")

# Command to resume the paused track
@bot.command(name='resume')
async def resume(ctx):
    if ctx.voice_client and ctx.voice_client.is_paused():
        ctx.voice_client.resume()
    else:
        await ctx.send("The music is not paused!")

# Command to stop the current track
@bot.command(name='stop')
async def stop(ctx):
    if ctx.voice_client and ctx.voice_client.is_playing():
        ctx.voice_client.stop()
    else:
        await ctx.send("Nothing is playing right now!")

# Command to skip the current track
@bot.command(name='skip')
async def skip(ctx):
    if ctx.voice_client and ctx.voice_client.is_playing():
        ctx.voice_client.stop()
        await play_next(ctx)
    else:
        await ctx.send("Nothing is playing right now!")

# Command to swap the current track
@bot.command(name='swap')
async def swap(ctx, query: str):
    try:
        # Check if the query is a number
        if query.isdigit():
            track_number = int(query)
            # Ensure a playlist was previously loaded
            if 'last_playlist' in ctx.bot.__dict__:
                results = ctx.bot.last_playlist
                if 1 <= track_number <= len(results['items']):
                    chosen_track = results['items'][track_number - 1]['track']
                    track_name = chosen_track['name']
                    artist_name = chosen_track['artists'][0]['name']
                    music_queue.appendleft(f"ytsearch:{track_name} {artist_name}")

                    if ctx.voice_client.is_playing():
                        ctx.voice_client.stop()

                    await play_next(ctx)
                    await ctx.send(f"Now playing: {track_name} - {artist_name}")
                else:
                    await ctx.send("Invalid track number!")
            else:
                await ctx.send("No playlist loaded. Please add a playlist first.")
        # Check if the query is a YouTube URL
        elif re.match(r'https?://(www\.)?(youtube\.com|youtu\.be)/.+', query):
            music_queue.appendleft(query)
            if ctx.voice_client.is_playing():
                ctx.voice_client.stop()
            await play_next(ctx)
            await ctx.send("YouTube URL added to the queue and now playing.")
        # Check if the query is a Spotify playlist URL
        elif re.match(r'https?://(open\.)?spotify\.com/playlist/.*', query):
            playlist_id = query.split('/')[-1].split('?')[0]
            results = sp.playlist_tracks(playlist_id)
            ctx.bot.last_playlist = results  # Store the last loaded playlist
            await ctx.send(f"Playlist loaded with {len(results['items'])} tracks. Use !swap <number> to switch to a specific track.")
        else:
            await ctx.send("Please provide a valid Spotify playlist URL or track number.")
    except Exception as e:
        await ctx.send(f"An error occurred while processing your request: {str(e)}")
        print(f"Error: {str(e)}")

# Run the bot with your token
bot.run('YOUR_BOT_TOKEN')
Replace YOUR_BOT_TOKEN, YOUR_SPOTIPY_CLIENT_ID, and YOUR_SPOTIPY_CLIENT_SECRET with your actual bot token and Spotify API credentials.
